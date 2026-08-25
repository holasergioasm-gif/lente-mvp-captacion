# MVP de captación y filtro — Grupo Lente (Star Deportes / Otero Deportivo)

Objetivo del MVP: reemplazar la cola de CVs y el examen de 3 horas por un flujo web de 10 minutos que (a) diga la verdad del puesto antes de que la persona invierta tiempo, (b) capture disponibilidad real en vez de declarada, (c) simule el trabajo en video de toma única, y (d) entregue al dueño una lista ordenada con evidencia, no una decisión.

Principio rector: **el sistema corta por reglas duras y ordena por score. Decide un humano.** Todo se loguea para validar predictividad a 90 días y 6 meses.

---

## 1. Flujo (presupuesto: 10 minutos)

| Min | Paso | Qué pasa | Qué se registra |
|---|---|---|---|
| 0:00 | Landing | Realistic Job Preview: horario cortado (9–13 / 16–20), sucursal y horario rotativos, sueldo real, capacitación, "95% entra sin experiencia". Checkbox: "Leí esto y quiero seguir". | Timestamp, dispositivo, fuente (QR local / IG / web). Abandono en este paso = autoselección. |
| 0:30 | Consentimiento | Texto corto: se graba video, se transcribe, se guarda 90 días, se comparte solo con el equipo de selección. Menores de 18 no continúan. | Aceptación con timestamp. |
| 1:00 | Formulario duro | Ver sección 2. | Todos los campos + tiempo de llenado. |
| 3:00 | CV (opcional) | Subida de PDF/imagen. Si no tiene, sigue igual: el formulario es el CV. | Extracción automática de campos (sección 3). |
| 3:15 | Corte por reglas | Si cae en rojo: mensaje honesto e inmediato ("Por ahora no coincidimos en los turnos. Guardamos tus datos por si se abre uno compatible"). Si pasa: sigue. | Motivo de corte. |
| 3:30 | Prueba de cámara | 15 segundos: "Decí tu nombre y desde dónde nos escribís". Sirve para chequear audio/video y como calentamiento. | No se puntúa. |
| 4:00 | Video | 6 preguntas × (10 s lectura + 30 s grabación). Toma única, sin regrabar, sin pausar. Contador visible. | 6 clips, timestamps, si hubo intentos de salir de la pantalla. |
| 8:00 | Bloque escrito (opcional) | 3 dilemas A/B con "¿por qué?" en una línea. Ver sección 5. | Respuestas. **No se usan para cortar** hasta validar. |
| 9:30 | Cierre | "Recibido. Te escribimos por WhatsApp antes del [fecha concreta, no 'pronto']". | Fin de flujo. |
| post | Backend | Transcripción (whisper), extracción, rúbrica, flags de contradicción, dashboard ordenado. | Score por dimensión, flags, ranking. |

---

## 2. Formulario duro (2 minutos)

Reemplaza las preguntas 1, 2, 3, 4 del banco original. La 3 desaparece.

1. Nombre y apellido
2. Fecha de nacimiento (calcula edad; <18 no continúa)
3. Barrio / localidad (calcula tiempo de viaje a las sucursales)
4. WhatsApp
5. ¿Trabajás o estudiás? → Trabajo / Estudio / Ambas / Ninguna
   - Si estudia: qué, en qué año, **qué días y en qué horario cursa**
   - Si trabaja: dónde, **qué días y horario**
   - Si ninguna: ¿qué venís haciendo?
6. **Grilla de disponibilidad**: días (lun–dom) × franjas (9–13, 16–20). Marca casilleros.
7. **Turnos reales**: "Estos son tres turnos que tenemos abiertos. Marcá cuáles aceptarías." (tres combinaciones reales, incluyendo sábado y una sucursal lejana). Marcar los tres = bandera amarilla.
8. ¿Cuánto hace que buscás trabajo? (contrasta con fecha del CV)

Nota: la pregunta 6 y la 7 se diseñaron para contradecirse con el video V2 (martes típico). Ahí está la señal.

---

## 3. Qué hace y qué no hace la IA con el CV

**Hace (extracción):** edad, localidad, estudios y estado, experiencia previa en atención al público (sí/no, dónde), fecha del CV o última actualización, contacto.

**Hace (consistencia):** compara cada campo extraído contra el formulario. Diferencias = flag, no corte.

**No hace:** puntuar "actitud", "potencial" ni "motivación" desde el CV. No hay información para eso en el CV de alguien sin experiencia, y cualquier score sería ruido con aspecto de dato.

**Reglas duras de corte (las únicas automáticas):**
- Menor de 18.
- Tiempo de viaje > 60 minutos a todas las sucursales (umbral a calibrar con el dueño).
- Grilla de disponibilidad incompatible con todos los turnos abiertos.
- Contradicción flagrante: declaró disponibilidad en una franja donde dijo que cursa/trabaja.

Cada corte guarda el motivo. El dueño tiene que poder ver "cayó por X".

---

## 4. Video: 6 preguntas, toma única

Formato en pantalla: pregunta en texto grande, 10 s de lectura con cuenta regresiva, luego 30 s de grabación con cuenta regresiva. No hay botón de "repetir". Antes de arrancar se avisa: "Se graba una sola vez, como en el local".

Cada slot tiene 2–3 variantes que rotan al azar para que las respuestas no circulen entre candidatos.

### V1 — Reloj de arena (fluidez con desconocidos)
- "Tenés 30 segundos. Contame algo tuyo que no esté en ningún CV."
- Variante: "Tenés 30 segundos y no te voy a interrumpir. Contame qué te gusta hacer."
- Mide: tolerancia al silencio, calidez, fluidez sin guion. Es la única prueba que simula el core del laburo.

### V2 — Martes típico (disponibilidad real)
- "Contame cómo es un martes tuyo, desde que te levantás hasta que te acostás."
- Variante: "Contame qué hiciste el jueves pasado, hora por hora, más o menos."
- Mide: consistencia con la grilla y con turnos elegidos. Aquí se detectan las cursadas de 14 a 18 que no aparecieron en el formulario.

### V3 — Autocrítica (pregunta 25 original)
- "¿Qué te criticaría alguien que trabajó, estudió o compartió actividades con vos?"
- Variante (pregunta 6 original): "¿Qué aspecto de tu forma de ser sentís que tenés que mejorar?"
- Mide: capacidad de autocrítica concreta. Según el dueño, acá cae el 60%.

### V4 — 19:50 (pregunta 27 original)
- "Son las 19:50 y el local cierra a las 20. Entra un cliente que quiere ver varios pares antes de decidir. ¿Qué hacés y cómo lo encarás?"
- Mide: orientación al cliente vs. al horario, y cómo lo razona.

### V5 — Cliente molesto (pregunta 17 original)
- "Un cliente está molesto y te habla mal por un problema que vos no generaste. ¿Cómo actuás?"
- Variante (pregunta 18): "Un cliente quiere un producto, pero vos creés sinceramente que otro más barato le conviene. ¿Qué hacés?"
- Mide: manejo de conflicto y honestidad comercial.

### V6 — La verdad del puesto (reemplaza pregunta 21)
- "Viste el horario cortado y la sucursal rotativa. ¿Qué es lo que más te preocupa de eso? Sé sincero, no hay respuesta correcta."
- Variante: "¿Qué tendría que pasar para que te fueras de este trabajo?"
- Mide: honestidad sobre las condiciones que hoy expulsan al 80% en el primer año. La respuesta "nada, todo bien" es bandera amarilla, no verde.

---

## 5. Bloque escrito opcional (2 minutos, no corta)

Se mantienen tres dilemas del banco original como texto, con "¿por qué?" en una línea:
- Pregunta 13 (10 + aguinaldo vs. 12 sin aguinaldo)
- Pregunta 26 (esfuerzo vs. resultado)
- Pregunta 23 (mejor de un equipo malo vs. parte de un equipo excelente)

Se registran pero **no entran en el corte ni en el ranking** hasta que el cruce con desempeño a 90 días diga si discriminan algo. Si en 6 meses no predicen, se eliminan.

---

## 6. Rúbrica de puntuación (0–1–2 por dimensión, sobre transcripción)

| Dimensión | 0 | 1 | 2 | Fuente |
|---|---|---|---|---|
| Fluidez con desconocidos | Silencio, monosílabos, lee algo | Habla pero rígido o genérico | Natural, concreto, se nota la persona | V1, prueba de cámara |
| Consistencia de disponibilidad | Contradicción clara entre grilla, turnos y martes típico | Diferencias menores | Coincide todo, y el martes tiene detalle creíble | Formulario + V2 |
| Autocrítica | "Nada" / "soy muy perfeccionista" | Algo real pero vago | Concreto, con ejemplo | V3 |
| Criterio en situaciones | Rígido ("cerramos y listo") o servil sin criterio | Razonable | Prioriza al cliente, explica cómo, considera al equipo | V4, V5 |
| Honestidad sobre condiciones | "No me preocupa nada" | Nombra algo genérico | Nombra algo concreto y cómo lo manejaría | V6 |

**Corte para pasar a entrevista presencial:** ningún 0 en fluidez ni en consistencia; total ≥ 6/10.

**Cómo puntúa el modelo:** recibe la transcripción de cada clip + los datos del formulario + la rúbrica, y devuelve por dimensión: puntaje, cita textual que lo justifica, y flags de contradicción. El dashboard muestra la cita, no solo el número. Si el dueño no puede ver la frase que justificó un 0, el score no sirve.

**Calibración obligatoria:** los primeros 50 candidatos los puntúan a ciegas una persona del equipo y el modelo. Se mide acuerdo por dimensión. Se ajusta rúbrica o prompt hasta llegar a acuerdo aceptable antes de usar el ranking.

---

## 7. Preguntas del banco original que se descartan y por qué

| # | Motivo |
|---|---|
| 3 | Genera el problema que se quiere resolver. Reemplazada por grilla + turnos + martes típico. |
| 5, 7 | Respuesta coacheable, sin capacidad de discriminar. |
| 8, 9 | Requieren narrativa larga; en 30 s salen frases hechas. |
| 10, 11, 22 | Expectativas: útiles para la charla presencial, no para filtrar. |
| 12, 20 | Dilemas con respuesta socialmente deseable evidente. |
| 14 | Es la mejor pregunta STAR, pero no cabe en 30 s. Guardarla para la presencial. |
| 15, 16, 19 | Hipotéticos de buen comportamiento; todos responden lo correcto. |
| 24 | Ambigua, difícil de puntuar. |

---

## 8. Métricas del MVP (lo que hace que esto sea un instrumento y no un juguete)

**Embudo por paso:** landing → consentimiento → formulario → cámara → video completo → cierre. Abandono por paso y por dispositivo. Sin esto no se puede saber si el video filtra ganas o filtra comodidad tecnológica.

**Costo:** minutos humanos por contratación vs. proceso actual (dueño 7 horas sentado + 3 horas de examen por tanda).

**Predictividad (a 90 días y 6 meses):** para cada contratado, cruzar score por dimensión, flags y bloque escrito contra ticket promedio y permanencia. Es el mismo análisis pendiente para el examen actual (H2). El MVP tiene que generar esos datos desde el día uno.

**Control:** si es posible, correr un brazo en paralelo con audio por WhatsApp (mismas 6 preguntas) y comparar completados y calidad.

---

## 9. Notas técnicas mínimas

- Grabación: MediaRecorder en navegador; probar en Safari iOS y Chrome Android antes de lanzar, son el 90% de los candidatos. Clips de 30 s, 480p, audio prioritario.
- Transcripción: whisper (large-v3-turbo funciona bien en español rioplatense según lo ya probado).
- Retención de datos: 90 días, borrado automático, consentimiento explícito. No análisis facial ni biométrico.
- Anti-fraude real: toma única + 10 s de lectura + detectar cambio de pestaña durante grabación. No hace falta más.

---

## 10. Motor de cruces: cómo detecta el sistema inconsistencias

Principio: cada dato importante se captura en al menos tres fuentes, en formatos distintos y en momentos distintos. El sistema compara y marca; no interpreta intención. Un flag rojo pasa a revisión humana, no a rechazo automático (salvo las reglas duras de la sección 3).

### 10.1 Fuentes por nivel

| Nivel | Fuente | Cómo entra | Qué verifica | Costo para el candidato |
|---|---|---|---|---|
| 1 — interno | Formulario cerrado (grilla, turnos, estudia/trabaja) | Paso 3 del flujo | Declaración estructurada | 0 |
| 1 — interno | CV | Subida opcional | Fecha del CV, experiencia, estudios, domicilio | 0 |
| 1 — interno | Video V2 (martes típico / jueves pasado) | Toma única | Rutina real, sin posibilidad de ajustar a la grilla | 0 |
| 1 — interno | Bloque escrito | Paso 8 | Consistencia con V6 (qué le preocupa del puesto) | 0 |
| 1 — interno | Metadatos | Se registran solos | Día/hora del llenado, dispositivo, tiempo por paso, cambios de pestaña en video, tiempo de respuesta al WhatsApp | 0 |
| 2 — documental | DNI + selfie | Al inicio, con consentimiento | Edad, identidad | 1 min |
| 2 — documental | Historia laboral ANSES (PDF de Mi ANSES) | Pedido al pasar el corte de video, antes de la presencial | Empleos registrados, fechas, **duración en cada uno** | 2–5 min |
| 2 — documental | Constancia de alumno regular / captura SIU Guaraní con horarios | Ídem, solo si declaró estudiar | Cursada real, días y horarios | 2–5 min |
| 2 — relacional | Referencia laboral: WhatsApp de un ex responsable | Ídem, solo si declaró experiencia | Un audio de 1 minuto del referente | 0 (lo hace el equipo) |
| 3 — externo | Link opcional a LinkedIn/Instagram | Campo voluntario | Solo si trabajo/estudio declarado coincide | 0 |

No hay scraping de redes sin consentimiento. Ver riesgos en 10.4.

### 10.2 Matriz de cruces

| Dato | Fuente A | Fuente B | Fuente C | Regla de contradicción | Flag |
|---|---|---|---|---|---|
| Disponibilidad | Grilla | Turnos elegidos | V2 martes típico | Franja marcada libre en grilla pero en V2 está ocupada (cursa, trabaja, cuida a alguien) | **Rojo** |
| Disponibilidad | Turnos elegidos | Grilla | — | Marcó los tres turnos incluyendo sábado y sucursal lejana, pero grilla tiene huecos | Amarillo |
| Estudia | Formulario (días/horarios) | CV | Constancia / SIU | Horarios de cursada distintos entre fuentes, o constancia no coincide con carrera declarada | Rojo si choca con turno; amarillo si no |
| Trabaja actualmente | Formulario | V2 | Historia laboral ANSES | Dijo "no trabajo" y ANSES muestra alta vigente; o V2 menciona un laburo que no declaró | **Rojo** |
| Experiencia previa | CV | V1/V5 (menciones espontáneas) | ANSES + referencia | Empleo del CV no aparece en ANSES **y** el referente no responde o no lo recuerda | Amarillo (puede ser informal) |
| Duración en empleos anteriores | ANSES | CV | — | Tres o más empleos registrados de menos de 6 meses en los últimos 3 años | **Amarillo fuerte** — es el dato más cercano al problema de retención |
| Hace cuánto busca | Formulario | Fecha del CV | Metadatos | CV con más de 6 meses y declara "recién empiezo"; o al revés | Amarillo |
| Edad | Formulario | DNI | CV | Cualquier diferencia | **Rojo** |
| Domicilio / distancia | Formulario | DNI | CV | Domicilios distintos: usar el más lejano para calcular viaje | Amarillo |
| Día y hora del llenado | Metadatos | Grilla / V2 | — | Llenó el formulario un martes 15:00 y en V2 dijo que los martes trabaja de 14 a 18 | Amarillo débil (puede llenarlo en un recreo) |
| Preocupación por el puesto | V6 | Bloque escrito | — | En V6 "no me preocupa nada" y en el escrito elige seguridad/aguinaldo/previsibilidad en todo | Amarillo: deseabilidad social, no mentira |
| Integridad del video | Metadatos | — | — | Cambió de pestaña o minimizó durante la grabación; audio con más de una voz | Rojo para revisión manual |

Severidad: **Rojo** = revisión humana obligatoria antes de citar; dos rojos = no se cita salvo que el revisor lo justifique por escrito. **Amarillo** = se muestra en el dashboard junto a la cita textual; suma o resta en el ranking según calibración. Ninguna combinación de amarillos corta sola.

### 10.3 Secuencia: qué se pide y cuándo

Pedir documentos al inicio mata el embudo. El orden correcto:

1. Formulario + CV + video: sin documentos, salvo DNI + selfie (30 segundos, y ya filtra al que no es quien dice).
2. Corte por rúbrica (≥ 6/10, sin ceros).
3. **Solo a los que pasan**: mensaje por WhatsApp pidiendo historia laboral ANSES (todos), constancia de cursada (si estudia) y contacto de referencia (si tiene experiencia). Plazo: 48 horas. No mandarlo en 48 horas es un dato: se registra como abandono en etapa documental.
4. Cruce nivel 2, actualización de flags, entrevista presencial con los flags a la vista del entrevistador.

Lo que se le dice al candidato al pedir los documentos: "así armamos un turno que te cierre de verdad". Enmarcado como servicio, no como examen. Reduce la mentira más que cualquier verificación.

### 10.4 Por qué no scrapear redes sociales

- **Técnico**: Meta bloquea scraping; cuentas de candidatos jóvenes suelen ser privadas; hacerlo desde la misma infraestructura o dominio que paga pauta pone en riesgo la cuenta publicitaria.
- **Legal**: la Ley 25.326 exceptúa datos de acceso público irrestricto, pero un perfil de redes expone religión, política, orientación, salud, embarazo. Usarlo para decidir contrataciones se choca con la Ley 23.592 y el art. 17 de la LCT, y la empresa no puede explicar un rechazo diciendo "vimos tu Instagram". Consultar abogado laboralista antes, no después.
- **Señal**: un perfil no predice ventas ni permanencia. Lo que sí hace es activar sesgos estéticos y de clase del evaluador. Es ruido con forma de dato, con riesgo real.

Alternativa aceptable: campo opcional de link a LinkedIn/Instagram, cruce limitado a "coincide trabajo o estudio declarado". Nada más.

### 10.5 Qué mide el motor de cruces sobre sí mismo

- Tasa de flags por tipo y por etapa. Si un cruce nunca dispara, sobra. Si dispara en el 80%, está mal calibrado o la pregunta está mal hecha.
- Concordancia entre flag y resultado de la presencial: cuántos rojos el entrevistador confirmó como mentira vs. cuántos eran error de captura.
- A 6 meses: ¿los contratados con amarillos de duración en ANSES rotaron más? Es el test de la hipótesis de que el historial predice retención.
