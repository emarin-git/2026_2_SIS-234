# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/Lucas-RodriguezAncieta/IoTPractica3
**Informe:** hay **dos** informes: `Informe .md` (raíz, con un espacio en el nombre) y `docs/informe.md` (el que enlaza el README). Se evaluaron ambos (ver §2).
**Integrantes (según el historial):** Lucas Rodríguez Ancieta · Josué Camacho · Pablo Raya — *ningún informe incluye los nombres de los integrantes*
**Caso de uso:** puerta para mascotas — sensor infrarrojo digital + servomotor SG90
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **2** / 4 |
| Desarrollo e implementación (código y documentación) | **2** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **1** / 4 |
| **Total** | **7 / 16** |
| **Nota del componente** = (7 / 16) × 100 | **43.75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 17:44 (UTC-4), commit `0bfba9e` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Pablo Raya: 10 (`Informe .md`, `test/Pruebas`) · Josué Camacho: 6 (Lambda, Alexa, AWS, `docs/informe.md`) · Lucas Rodríguez: 3 (firmware inicial, `.gitignore`, cambio de pines) |

- Se evaluó el commit **`0bfba9e`**. Casi todo el repositorio se generó el 10-Nov; el firmware llegó el 9-Nov en un solo "Primer commit".
- **Dos informes que se contradicen.**
  - `Informe .md` (raíz) lo creó Pablo Raya a las 14:02 del 10-Nov, con el mensaje de commit "Update print statement from 'Hello' to 'Goodbye'". Declara que **Lambda, DynamoDB y Alexa están "Pendientes"**, que las 33 pruebas están "Pendiente de validación" y que todos los resultados cuantitativos están "Por medir". Su sección 8 (Anexos) está truncada: solo tiene un fragmento de frase.
  - `docs/informe.md` lo escribió Josué Camacho a las 12:38 y es el que enlaza el README. Declara la integración con Alexa, Lambda y DynamoDB como probada, con capturas.
  - El lector no puede saber cuál es la versión válida. Se tomó `docs/informe.md` como informe principal porque refleja el código entregado, y `Informe .md` como evidencia de una entrega desordenada.
- **El README sigue diciendo que el proyecto está pendiente:** "Lambda y Alexa siguen pendientes", "No se han ejecutado estas verificaciones".
- La matriz de pruebas está en `test/Pruebas`, un archivo **sin extensión** que ningún informe enlaza. Sus evidencias de video están en Google Drive, fuera del repositorio.
- **El cambio de pines se hizo en el código para que coincida con el diagrama.** El commit `efe4e81` ("fix diagramas con codigo", 17:20) intercambió los pines del sensor y del servo en `main.cpp` (27↔18), 25 minutos antes del cierre y sin una prueba posterior documentada. El propio `docs/informe.md` dice "debe comprobarse que las constantes de pines del firmware entregado coincidan con el diagrama final".
- No se encontraron certificados, claves privadas ni credenciales WiFi en el historial. Las políticas exportadas incluyen el número de cuenta de AWS (`243809381986`), que conviene anonimizar.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El caso de uso es razonable, y **se analiza con honestidad el riesgo físico**: el sensor no garantiza que el paso esté libre, una orden pendiente puede cerrar la puerta al reconectar y el prototipo no debe usarse con animales. La bandera `ENABLE_DOOR_MOVEMENT` es una decisión de seguridad correcta.
- Hay un diagrama de arquitectura, un diagrama de clases del firmware y **tres diagramas de secuencia** (vinculación, consulta del sensor y comando, con las ramas de error y de "sin vínculo").
- **Especificación del Shadow aceptable:** atributos, ubicación, tipo, valores válidos, uso de `desired`, `reported` y `delta`, y una aclaración correcta de que `servoAngle` es el ángulo ordenado, no uno medido.

**Observaciones**
- **Los requerimientos no funcionales no tienen valores medibles.** La tabla dice "Comprobación funcional" o "Pendientes de cuantificación". La exactitud, la latencia y la frecuencia no tienen un valor declarado que se pueda verificar.
- **Falta el diagrama de clases de la Lambda.** Se reemplazó por un *flowchart* de intents, con la explicación de que el diagrama anterior tenía "clases hipotéticas".
- **El diagrama de circuito es una imagen "proporcionada por el equipo"** (`11-diagrama-conexiones.png`). Los pines del diagrama no coincidían con el código ni con `Informe .md` hasta el cambio de última hora.
- **El mecanismo de vinculación ata la skill a un único Thing.** El código de 4 dígitos se compara con una única variable `PAIRING_CODE`, que corresponde a una única `THING_NAME` (`puerta-mascotas-01`). Un segundo usuario con otro objeto no puede vincularse. Esto **no cumple** el requisito de la consigna (§3: "La skill no debe estar atada a un único Thing").
- La frecuencia de reporte (lectura cada 200 ms, mínimo de 1 s entre reportes, respaldo de 30 s) se describe, pero no se justifica frente al consumo de mensajes de la capa gratuita.

### 3.2 Desarrollo e implementación — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El firmware separa `SensorIR`, `ControlServo` y `ShadowManager`, con las credenciales en archivos ignorados y plantillas sin secretos.
- `ShadowManager` resuelve problemas reales y los documenta. El caso más destacado es el diagnóstico del **defecto del `RETAIN`**: la sobrecarga de `publish()` con un `char*` activaba el bit de retención, y AWS cierra la conexión en los temas reservados. También valida JSON, tipo y versión, y confirma los reportes con `clientToken`.
- La política IAM exportada de la Lambda es de mínimo privilegio (las 3 acciones de DynamoDB y las 3 de IoT, sobre recursos concretos).

**Observaciones**
- **La Lambda no cubre los RF de vinculación tal como los pide la consigna.** `VincularThingIntentHandler` y `CambiarThingIntentHandler` solo aceptan el código de `PAIRING_CODE` y lo traducen a `THING_NAME`. La política IAM también restringe el acceso a `thing/puerta-mascotas-01`. **La skill solo funciona con un Thing fijo.**
- **Lambda poco modular** (743 líneas):
  - Cada handler vuelve a crear los clientes `boto3` y repite la lectura de DynamoDB y del Shadow.
  - Quedan restos de la plantilla de Alexa: `HelloWorldIntentHandler` e `IntentReflectorHandler`.
  - No hay una clase de repositorio ni de acceso al Shadow.
- **Documentación del código escrita como bitácora de diagnóstico,** no como una descripción del sistema. Hay párrafos enteros con "según los registros aportados por el usuario", "el usuario confirmó", "no se inspeccionó el archivo local" y "antes de entregar, corresponde confirmar…". Es texto de un asistente que no se editó.
- La configuración de AWS está en `aws/README.md` (496 líneas), pero el informe no incluye una tabla de **permisos asignados frente a mínimos**. La política IoT desplegada se describe solo como "versión 3 activa".

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay capturas reales del simulador (consulta del sensor con y sin detección), del Shadow (abierto, cerrado, sensor detectado), de DynamoDB, de la Lambda y de la política IoT.
- Se documenta una prueba de **orden pendiente con WiFi cortado**: se publica `closed` sin conexión y se aplica al reconectar (versiones 176 → 177 del Shadow).
- `test/Pruebas` organiza una matriz por requisito, con el resultado esperado, el obtenido, el estado y la evidencia.

**Observaciones (bajan el nivel)**
- **Ningún requisito no funcional se midió.** La estabilidad de 10 min (PNF-08) y los ciclos de apertura y cierre (PNF-09) figuran como "No ejecutada". No hay mediciones de latencia, exactitud ni frecuencia: `docs/informe.md` dice "No se dispone de una campaña reproducible de latencia, exactitud, disponibilidad…".
- **RF8 y RF9 incompletos:** "usuario sin vínculo" (PF-22) y "cambio a un segundo Thing" (PF-24) figuran como **No ejecutada**. No hay prueba de un Thing inexistente ni de dos usuarios.
- **Muchas pruebas "aprobadas" se apoyan solo en la palabra del equipo:** "el equipo informó", "el integrante responsable confirmó visualmente", "observación del equipo; captura…". El movimiento físico del servo no tiene foto ni video en el repositorio.
- No hay pruebas de las frases de la skill una por una (frase → intent → respuesta).
- `Informe .md`, el otro informe entregado, declara las 33 pruebas como "Pendiente de validación".

### 3.4 Resultados, conclusiones y recomendaciones — **1 / 4 (Necesita mejorar)**

**Observaciones**
- **No hay resultados cuantificables.** La tabla de `docs/informe.md` es cualitativa ("alternó entre detección y no detección", "movimiento confirmado por el integrante"), y la de `Informe .md` dice "Por medir" en los 9 indicadores.
- **Las conclusiones son generales** ("permite aplicar conceptos fundamentales de IoT…") y no se apoyan en datos medidos. Una conclusión, "la colaboración quedó respaldada por código, modelo de interacción…", no es una conclusión técnica.
- **Las recomendaciones incluyen tareas pendientes del propio grupo,** no recomendaciones técnicas: "Comprobar antes de entregar que el firmware coincida con el diagrama actualizado", "Incorporar una fotografía propia del montaje final", "Completar la integración de AWS Lambda".
- **Los anexos están incompletos:** la sección 8 de `Informe .md` está truncada, y en `docs/informe.md` el diagrama de conexiones aparece marcado como "(incorporar al repositorio)". No hay fotos del montaje.

---

## 4. Recomendaciones para próximas entregas

1. **Entregar un solo informe, coherente con el código.** Dos versiones contradictorias (una dice "pendiente" y otra "probado"), más un README que también dice "pendiente", hacen imposible saber qué se logró.
2. **Cumplir el requisito de varios usuarios.** La vinculación debe resolver cualquier Thing existente (por su nombre, un número de serie o un código asociado a cada Thing), no un único código fijo en una variable de entorno.
3. **Medir los requisitos no funcionales:** al menos 10 min de estabilidad con log, 10 o más comandos cronometrados (desde la voz hasta el servo), intervalos de reporte y una tasa de detección del sensor con N ensayos.
4. **Editar los textos generados con asistentes.** Frases como "según el usuario", "el usuario confirmó" o "antes de entregar, corresponde confirmar…" no deben llegar al informe final.
5. **No cambiar el código en el último minuto para que coincida con la documentación** sin volver a probarlo. Si el cableado cambió, hay que probarlo y registrarlo.
6. Limpiar la Lambda (quitar los handlers de la plantilla, crear los clientes una vez y separar el repositorio de vinculaciones y el acceso al Shadow en clases), y agregar fotos y video del montaje al repositorio.
7. Poner nombres de archivo claros (`Informe .md` tiene un espacio; `test/Pruebas` no tiene extensión) y mensajes de commit que describan el cambio.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Vinculación:** cómo se resuelve el Thing de un usuario, qué pasaría si un segundo usuario quisiera vincular `puerta-mascotas-02`, y qué habría que cambiar en el código y en la política IAM para permitirlo.
- **Defecto del `RETAIN`:** por qué la llamada `publish(topic, char*, n)` activaba la retención y por qué AWS cierra la conexión.
- **Sincronización:** qué hace el firmware al reconectar (`get`, `get/accepted`, versiones), cómo evita aplicar una orden antigua y qué riesgo tiene `ENABLE_DOOR_MOVEMENT = true`.
- **`clientToken`:** cómo se usa para confirmar un reporte propio y qué pasa si no llega `update/accepted` en 15 s.
- **Circuito:** qué pines usa realmente el montaje, por qué se cambiaron en el código a las 17:20 y si se probó después. Alimentación del SG90 desde VIN con un solo USB.
- **Informe:** por qué hay dos informes contradictorios y cuál es el válido.
- **Aporte individual:** Josué Camacho hizo la Lambda, Alexa, AWS y `docs/informe.md`; Pablo Raya, `Informe .md` y la matriz de pruebas; Lucas Rodríguez, el firmware inicial (en un solo commit) y el cambio de pines. Cada integrante debe poder explicar el firmware, la Lambda y la configuración de AWS.
