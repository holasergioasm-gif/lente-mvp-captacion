# HANDOFF · MVP de captación y filtro · Grupo Lente

Este repo contiene `index.html`: prototipo navegable completo, un solo archivo, sin dependencias ni build. Todo lo que dice **SIM:** en el código está simulado en el cliente. Tu trabajo es reemplazar cada SIM por su implementación real sin cambiar el flujo ni la lógica de cruce, que ya está escrita y probada en `evaluarReglas()`, `diffCV()` y `armarPreguntas()`.

Spec funcional completa: `mvp-captacion-grupo-lente.md` (secciones 1–10). Flujo: `flujo-mvp-grupo-lente.mermaid`.

## Principio innegociable

El sistema **corta solo por reglas duras** (edad, distancia, disponibilidad incompatible, contradicción flagrante) y **ordena por score**. Decide un humano. Todo flag y todo puntaje tiene que mostrar la cita textual que lo justifica. Si el dueño no puede ver por qué alguien tiene un 0, el número no vale.

## Stack sugerido

- Next.js (App Router) desplegado en Vercel. El cliente ya tiene cuenta.
- Postgres (Neon/Supabase) para candidatos, config y logs. Storage S3-compatible para clips y CVs con borrado automático a 90 días.
- Transcripción: faster-whisper `large-v3-turbo` con `language="es"` (ya validado por el cliente en español rioplatense) o un servicio equivalente.
- LLM para extracción de CV y puntuación con rúbrica. Salida JSON estricta, temperatura 0.
- WhatsApp: proveedor con API oficial de Meta (OTP + mensajes de estado). Nada de números personales.

## Qué está simulado y qué hay que construir

| En el prototipo | Implementación real |
|---|---|
| `config` en memoria, editable desde el panel | Tabla `config` con versión y `published_at`. La landing lee la versión publicada. Historial de cambios. |
| OTP: cualquier 4 dígitos | OTP real por WhatsApp. DNI + WhatsApp verificado = identidad. Un candidato = un DNI. |
| `config.bloqueados` array | Tabla `bloqueos(dni, hasta, motivo, levantado_por)`. Job cada hora: candidatos con `inicio` > 24 h y sin `fin` → bloqueo `bloqueoMeses`. Antes de bloquear, si hubo error técnico registrado (cámara denegada, upload fallido, desconexión), mandar link de reanudación por WhatsApp; bloquear solo si no completa en 24 h desde ese link. |
| `BARRIOS` → minutos fijos | Domicilio real (calle + localidad) → Distance Matrix u OSRM contra cada sucursal. Guardar minutos a la sucursal más cercana y a cada turno. |
| `CVS` de ejemplo | Endpoint `POST /api/cv/extract`: PDF o imagen → LLM → esquema de abajo. OCR previo si es foto. El candidato **no ve** la extracción. |
| Grabación en `MediaRecorder`, blobs en memoria | Subida por clip apenas termina (no esperar al final: si se cae en la pregunta 5, tenés 4 clips). Reintentos. `POST /api/clip` con `candidate_id`, `question_id`, `duration`, `tab_switches`. |
| Rúbrica "pendiente" | Pipeline asíncrono: clip → audio → transcripción → `POST /api/score` con rúbrica + formulario + extracción de CV → JSON de puntaje por dimensión con cita. Ver prompts abajo. |
| Mapa de consistencia a mano en `EJEMPLOS` | Se genera del `diff` + clasificación por repregunta (confirma con detalle / actualiza y explica / evade o vago / contradice). |
| Botones "Citar" y "Descartar" con toast | Envían WhatsApp con plantilla aprobada; el de citar pide además historia laboral ANSES, constancia de cursada si estudia, y contacto de referencia si tiene experiencia. Registran `decision`, `motivo`, `decidido_por`. |
| Variantes de pregunta | Cada slot fijo tiene 2–3 variantes; elegir al azar por candidato y registrar cuál recibió. |

## Esquema de extracción de CV (salida del LLM)

```json
{
  "fecha_cv": "YYYY-MM",            
  "domicilio": "localidad o barrio",
  "estudia": { "que": "carrera, institución, año", "dias": ["mar","jue"], "horario": "18-22" } ,
  "trabaja": { "donde": "", "rol": "", "desde": "YYYY-MM", "hasta": null },
  "empleos": [ { "donde": "", "rol": "", "desde": "YYYY-MM", "hasta": "YYYY-MM", "atencion_publico": true } ],
  "confianza": { "fecha_cv": 0.6, "estudia": 0.9, "trabaja": 0.8 }
}
```
`fecha_cv`: la fecha más reciente que aparezca en el documento (metadata del PDF, "actualizado", último empleo). Si no hay, `null` y no generar la repregunta de fecha. Campos con confianza < 0.5 no generan repreguntas de contradicción, solo de confirmación.

## Prompt de puntuación (estructura)

Sistema: sos evaluador de una rúbrica de selección para vendedores de local. Recibís (1) la rúbrica con anclas 0/1/2 por dimensión (sección 6 de la spec), (2) el formulario del candidato, (3) la extracción del CV, (4) la lista de preguntas con su transcripción, marcando cuáles son repreguntas dinámicas y qué discrepancia las originó. Devolvés solo JSON:

```json
{
  "dimensiones": { "fluidez": {"puntaje": 2, "cita": "…", "pregunta": "V1"}, "consistencia": {...}, "autocritica": {...}, "criterio": {...}, "honestidad": {...} },
  "repreguntas": [ { "id": "R-est", "clasificacion": "actualiza_y_explica|confirma_con_detalle|evade_o_vago|contradice", "cita": "…", "detalles_verificables": ["nombre del encargado", "horario"] } ],
  "flags": [ { "tipo": "rojo|amarillo", "k": "…", "txt": "…", "cita": "…" } ]
}
```
Reglas del prompt: puntuar solo con lo que está en la transcripción; sin inferencias sobre tono, acento, apariencia ni estado emocional; "evade_o_vago" = no aporta ningún detalle verificable (nombres, lugares, horarios, personas); "contradice" = afirma algo incompatible con formulario o CV, citar ambos lados.

**Calibración antes de usar el ranking:** los primeros 50 candidatos los puntúa también una persona a ciegas. Guardar ambos puntajes. Medir acuerdo por dimensión (kappa ponderado). No mostrar ranking al dueño hasta tener acuerdo aceptable.

## Datos que hay que loguear desde el día uno

Sin esto el MVP es un juguete. Cada evento con timestamp y `candidate_id`:
- Embudo: `landing_view`, `landing_cta`, `identidad_ok`, `inicio`, `form_ok`, `cv_ok`, `reglas_rojo`, `camara_ok`, `camara_denegada`, `clip_ok` (por pregunta), `escrito_ok`, `fin`, `abandono` (paso y motivo si hay error técnico).
- Por candidato: dispositivo, navegador, fuente (QR local / IG / web), tiempo por paso, `tab_switches` por pregunta, variante de cada pregunta.
- Post-contratación (lo carga el dueño): `contratado`, ticket promedio a 90 días, `activo_6m`. Es el cruce que valida o tira toda la rúbrica.

## Privacidad y legal (Argentina)

- Consentimiento explícito antes de grabar, con texto visible: qué se graba, quién lo ve, cuánto se guarda. Registrar aceptación.
- Borrado automático de clips, CV y transcripciones a los 90 días. Datos mínimos (DNI, decisión, fecha) pueden quedar para el bloqueo y las métricas.
- Menores de 18: no continúan.
- **No** análisis facial, de emociones ni de voz. Solo transcripción. No scraping de redes.
- El bloqueo de 6 meses se levanta a mano desde el panel y queda registrado quién lo levantó.

## Criterios de aceptación

1. Un candidato completa el flujo en el celular (Safari iOS y Chrome Android) en menos de 10 minutos con datos móviles.
2. Si se le cae la cámara en la pregunta 3, recibe un link por WhatsApp y retoma en la pregunta 3, no desde cero.
3. El panel muestra, para cada candidato, el mapa CV · formulario · video con cita textual por celda y las repreguntas que recibió con su discrepancia de origen.
4. Un cambio en "Condiciones" del panel aparece en la landing sin deploy.
5. Ingresar un DNI bloqueado muestra la fecha desde la que puede volver, y el panel permite levantarlo.
6. Ningún candidato queda descartado automáticamente por algo que no sea una regla dura con su motivo registrado.

## Cómo probar el prototipo

Abrir `index.html`. DNI `11111111` está bloqueado. En el paso de CV, el selector "modo demo" elige qué diría el CV; probar los tres casos y ver cómo cambian las repreguntas del video. El link "Panel" arriba a la derecha muestra lo que ve el dueño, con tres candidatos de ejemplo ya analizados y los reales que se hayan cargado en la sesión.
