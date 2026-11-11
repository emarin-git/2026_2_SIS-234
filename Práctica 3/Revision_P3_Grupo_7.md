# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/JesusxAriel/Practica3_Alexa_Aws_ESP32
**Informe:** `docs/informe_final.md`
**Integrantes (según el informe):** Leonardo Guillermo Köller · Ariel Jesús Mamani · Luis Felipe Velasco
**Caso de uso:** puerta inteligente — lector RFID RC522 + servomotor SG90 como cerrojo
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **1** / 4 |
| Desarrollo e implementación (código y documentación) | **2** / 4 |
| Pruebas y validaciones | **1** / 4 |
| Resultados, conclusiones y recomendaciones | **1** / 4 |
| **Total** | **5 / 16** |
| **Nota del componente** = (5 / 16) × 100 | **31.25** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 17:11 (UTC-4), commit `114d077` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor | Ariel Mamani: 3 (firmware, evidencias, informe) · Leonardo Koller: 1 (README, guías, modelo de Alexa, informe) · Luis Velasco: 1 (Lambda, pruebas, plantillas de AWS) |

- Se evaluó el commit **`114d077`**.
- **El repositorio completo se subió en 5 commits el mismo día de la entrega** (10-Nov, entre las 15:02 y las 17:11). No hay historial del desarrollo: cada integrante subió su parte en uno o dos commits. Este historial no permite verificar el aporte individual que pide §5.4.
- **El informe y la documentación fueron generados por un agente de IA (Codex) y se entregaron sin revisar.** El repositorio incluye `AGENTS.md` y `docs/PROMPT_CODEX.md`, con instrucciones para el agente. El informe habla en nombre del agente:
  - "el registro de Codex no acredita upload ni pruebas de hardware".
  - "No se cambiaron fuentes funcionales… para redactar este informe".
  - "Esta carpeta no contiene `.git`: no se crearon commits".
  - "el archivo `Práctica_3.md` citado por ese borrador no está disponible entre las fuentes revisadas. Por ello, no se declara cumplimiento normativo completo". Es decir, **el informe se redactó sin consultar la consigna.**
- Se subieron archivos compilados de Python (`lambda/__pycache__/*.pyc`, `tests/__pycache__/*.pyc`).
- No se encontraron certificados, claves privadas ni credenciales WiFi en el historial. Solo se publica `firmware/include/config.example.h`.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **1 / 4 (Necesita mejorar)**

**Fortalezas**
- El contrato del Shadow (§9.2) está bien especificado: propiedades, valores, semántica y la aclaración de que el `delta` llega como `state.cerrojo`.
- El diseño de la vinculación con un catálogo `CODE#<código>` → `thingName` y `USER#<userId>` → `thingName` en una sola tabla permite varios usuarios y varios Things.

**Observaciones (bajan el nivel)**
- **No hay requerimientos no funcionales.** El informe no declara valores de estabilidad, exactitud, tiempo de respuesta, frecuencia de reporte ni reconexión, que la consigna (§3.2) exige "declarar y verificar con valores medibles".
- **Los requerimientos funcionales no se definen.** Se presentan como "referencias heredadas" de un borrador anterior. El informe declara que **RF4 (consulta del sensor) "no está respaldado por el hardware actual"**, porque `EstadoPuertaIntent` consulta un campo `puerta` que el firmware ya no produce.
- **Faltan diagramas esenciales:**
  - No hay **diagramas de clases**, ni del firmware ni de la Lambda.
  - Solo hay **un** diagrama de secuencia, el del comando. Faltan la vinculación de un Thing y la consulta del sensor, que §4.2 pide de forma explícita.
  - El "circuito" es un *flowchart* de Mermaid, y el propio informe dice que "el cableado real debe demostrarse con fotografías".
- **El caso de uso no se justifica.** El documento usa una estructura de "Objetivos", "Materiales" e "Introducción", y no las secciones de §4.2. No se explica por qué se eligió un RC522 como sensor ni cómo responde a un problema real.
- **El RFID controla el actuador localmente** (la tarjeta desbloquea y el llavero bloquea). La consigna no lo prohíbe, pero esa ampliación se agregó al final y el propio informe dice que está "pendiente de evidencia de prueba física".
- No hay una justificación de la frecuencia de reporte (respaldo de 5 s) ni del consumo de mensajes.

### 3.2 Desarrollo e implementación — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El firmware separa `DoorHardware` (RC522 y servo, con una máquina de comando no bloqueante) de `CloudShadow` (WiFi, NTP, TLS, MQTT, GET al conectar, descarte de `delta` por versión y reconciliación de la orden RFID pendiente con una actualización condicionada por versión).
- La Lambda se organiza en clases (`LinkRepository`, `ShadowGateway`, `SkillController`), con validación del código (6 dígitos, sin 0 inicial), `DescribeThing` antes de vincular y 8 pruebas unitarias con dobles.
- Se exportan plantillas de la política IoT, la política de la Lambda, el esquema de DynamoDB y el Shadow de ejemplo.

**Observaciones**
- **La configuración de AWS no está documentada como se hizo.** El informe dice "No se inspeccionaron estos recursos durante la documentación" y que la presencia de las plantillas "no demuestra que las políticas reales coincidan". No hay una tabla de permisos asignados con su justificación ni capturas del Thing, la política, la tabla o el rol (E03, E04 y E06 figuran "por anexar").
- La Lambda conserva `EstadoPuertaIntent`, que lee un campo heredado y puede responder con un dato histórico que el hardware actual no produce. El propio informe advierte esta inconsistencia, pero no se corrigió.
- **El informe no explica el código.** Describe archivos y parámetros, pero no presenta fragmentos clave ni enlaces a líneas concretas.
- Se subieron archivos `.pyc` al repositorio.

### 3.3 Pruebas y validaciones — **1 / 4 (Necesita mejorar)**

**Fortalezas**
- Hay una tabla de pruebas de integración (IA-01 a IA-10) con la respuesta observada en el simulador, y algunas capturas (bienvenida, consulta del contador RFID, bloqueo).
- Se documenta con honestidad la confusión del modelo de voz ("abre el cerrojo" se clasificó como `BloquearIntent`) y cuáles frases funcionaron.

**Observaciones (bajan el nivel)**
- **Ningún requisito no funcional se midió.** El informe lo dice: "No se dispone de una campaña de mediciones reproducibles de latencia, exactitud angular, estabilidad continua o tiempo de reconexión". También registra invocaciones de la Lambda "próximas a ocho segundos", sin analizarlas.
- **La mayoría de las pruebas físicas son "declaradas por el equipo" sin evidencia.** El movimiento del servo, la recuperación de la conectividad y la publicación en el Shadow figuran en una tabla de "Ensayos declarados y anexos faltantes". No hay video ni captura del movimiento.
- **No hay pruebas de desvincular ni de cambiar la vinculación en el simulador,** ni de dos usuarios con Things distintos (RF9 y RF8, más allá de las pruebas unitarias con dobles).
- **La versión entregada del firmware (con el control RFID local) no se probó en el hardware:** "no se recibió una salida nueva del monitor que confirme la ejecución física con RFID local".
- El "plan de validación pendiente" (§12.5) reemplaza a las pruebas que debían ejecutarse antes de la entrega.

### 3.4 Resultados, conclusiones y recomendaciones — **1 / 4 (Necesita mejorar)**

**Observaciones**
- **No hay resultados cuantificables.** Los únicos números son el uso de RAM y flash del compilador y las 8 pruebas unitarias aprobadas.
- **Las conclusiones describen el estado del repositorio**, no resultados técnicos ("La incorporación de capturas y datos administrativos es una tarea de presentación").
- **No hay una sección de recomendaciones.** Hay un párrafo de "trabajo futuro" al final de las limitaciones, sin desarrollar extensiones hacia un uso real.
- **Los anexos están incompletos y el propio informe lo dice:** "Datos administrativos pendientes… Faltan institución/carrera, docente, grupo formal y atribución de contribuciones verificables". La tabla de anexos marca E03, E04, E06, E08 y E09 como no incorporados.

---

## 4. Recomendaciones para próximas entregas

1. **Leer la consigna y estructurar el informe según §4.2.** El informe se generó sin tener a la vista el enunciado. Por eso faltan los RNF, los diagramas de clases, dos diagramas de secuencia y la sección de recomendaciones.
2. **No entregar texto generado por una herramienta sin revisarlo.** Frases como "el registro de Codex no acredita…" o "no se crearon commits", o la sección "Datos administrativos pendientes", muestran que nadie del grupo leyó el documento antes de entregarlo. Cada integrante debe poder defender lo que firma.
3. **Trabajar con Git durante todo el proyecto,** no subir todo el último día. El historial de commits es la evidencia del aporte individual.
4. **Declarar y medir los RNF:** estabilidad (≥ 10 min con log), tiempo de respuesta (≥ 10 comandos cronometrados), frecuencia de reporte, reconexión con una orden pendiente y exactitud del lector (lecturas correctas sobre N presentaciones).
5. **Grabar la evidencia física:** un video del servo respondiendo a Alexa, capturas del Shadow antes y después, y de la tabla DynamoDB al vincular y desvincular.
6. Corregir `EstadoPuertaIntent` (eliminarlo o hacer que consulte un dato que el hardware produzca) y quitar los `.pyc` del repositorio.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

Como el informe fue generado por una herramienta, la defensa es clave para mostrar que el grupo domina el sistema. Se recomienda preparar:

- **Firmware:** cómo funciona `requestLockState()`, qué es la "orden RFID pendiente" y cómo se reconcilia con `desired` y `reported` mediante una actualización condicionada por versión, y qué pasa si el ESP32 se reinicia con una orden pendiente.
- **RC522:** el bus SPI y sus pines, cómo se detecta la retirada de la tarjeta (`PICC_WakeupA`, 750 ms sin respuesta) y por qué comparar UID no es autenticación segura.
- **Shadow:** por qué el `delta` llega como `state.cerrojo`, qué hace el ESP32 ante `/get/rejected` con 404 y por qué se descartan versiones ya vistas.
- **Lambda y vinculación:** el diseño `CODE#` / `USER#` en una sola tabla, cómo se valida el código y qué permisos IAM necesita la Lambda.
- **Alexa:** por qué "abre el cerrojo" se clasificaba como `BloquearIntent` y cómo se corrigió el modelo.
- **Tiempo de respuesta:** por qué algunas invocaciones de la Lambda tardaban cerca de 8 s y qué implica eso frente al límite de Alexa.
- **Aporte individual:** cada integrante tiene uno a tres commits del último día. Cada uno debe poder explicar el firmware, la Lambda y la configuración de AWS, y no solo los archivos que subió.
