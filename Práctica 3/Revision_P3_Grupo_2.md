# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/JoseCorralesAgreda/Practica-3-AWS
**Informe:** `docs/informe.md`
**Integrantes (según el historial):** Jose Franz Corrales Agreda · RashLop (Rasshid López Mejía) · Alejandro Bustamante — *el informe no tiene portada con los nombres de los integrantes*
**Caso de uso:** ambientador inteligente — PIR HC-SR501 + servomotor SG90 que presiona el pulsador
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **2** / 4 |
| **Total** | **10 / 16** |
| **Nota del componente** = (10 / 16) × 100 | **62.5** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 17:21 (UTC-4), merge `05f47e1` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Jose Corrales: 10 (firmware, informe) · RashLop: 3 (backend, infra, pruebas) · Alejandro Bustamante: 2 (intents, evidencias) |

- Se evaluó el commit **`05f47e1`**.
- No se encontraron certificados, claves privadas ni credenciales WiFi en el historial. Solo se publica `firmware/include/secrets.example.h`.
- El **número de cuenta de AWS** (`679937333206`) aparece en el README (ARN de la Lambda) y en las plantillas de políticas. No es un secreto, pero conviene reemplazarlo por `<account-id>`.
- **Parte de la evidencia está fuera del repositorio.** Los videos (cooldown, ciclos, latencia) están en una carpeta de Google Drive. La consigna (§6) pide los anexos dentro del repositorio, y solo se evalúa lo que está en él.
- **El informe no tiene portada:** no figuran el grupo, los integrantes ni la universidad.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Caso de uso original y práctico:** un SG90 presiona el pulsador de un ambientador. Se añade, como extensión, un **modo automático** que dispara un ciclo ante un nuevo flanco del PIR, con un *cooldown* configurable (5 s en pruebas y 60 s propuestos para producción). El modo se habilita por voz a través del Shadow.
- **Especificación del Shadow muy cuidada:** atributos con sección, tipo, valores y significado; QoS de cada tema; y comportamiento al reconectar (`/get` antes de aplicar). Un acierto de diseño: el uso de **`activationId`** (el `requestId` de Alexa) en lugar de un booleano, para que dos pulsaciones seguidas sean dos órdenes distintas, con deduplicación persistida en NVS y escritura versionada (bloqueo optimista).
- Hay tres diagramas de secuencia: vinculación, consulta del sensor y comando, con la nota de que la respuesta de voz y el `delta` pueden llegar en distinto orden.
- **Frecuencia de reporte justificada:** reporte por evento con un mínimo de 200 ms y respaldo cada 15 s, con el cálculo de mensajes por día y por mes. El umbral de datos antiguos (45 s) es coherente con el respaldo.

**Observaciones**
- **Requerimientos no funcionales sin valores medibles en varios atributos.** La exactitud del PIR es una "verificación funcional" (HIGH con la mano, LOW apuntando a una pared), sin un error ni una tasa declarada. El tiempo de respuesta declara ≤ 3 s, pero la columna de resultado solo dice "15 mediciones realizadas".
- **El diagrama de circuito es un `flowchart` de Mermaid**, no un esquema eléctrico: no muestra pines físicos, resistencias ni el condensador conectado. La tabla de conexiones está en `configuracion.md`, no en el informe.
- **Los diagramas de clases son mínimos:** una clase `Ambientador` en el firmware y una clase `AmbientadorSkill` en la Lambda. Es coherente con el código, pero muestra que no hay separación en módulos (ver §3.2).
- **El mecanismo de vinculación usa el nombre del Thing tal cual** (`thing`, `ambientador-dos`). El identificador es trivial de adivinar, y la "autorización" depende de que la política IAM de la Lambda liste a mano cada Thing permitido. Con esto, **la skill solo puede atender a los dos Things que están en la política**, lo que contradice la idea de que no esté atada a Things fijos. Además, `thing` no es un nombre adecuado para un Thing.
- El diseño de la skill tiene una sola frase de ejemplo por intent en el informe. Las demás solo están en el JSON del modelo.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- El firmware no bloquea: el ciclo del servo es una máquina de estados con `millis()` (`idle` → `pressing` → `returning`). La deduplicación de `activationId` se guarda en NVS (`Preferences`) antes de mover el servo, para que un reinicio no repita la orden.
- La Lambda valida el `applicationId`, usa lectura consistente en DynamoDB, valida el nombre del Thing con una expresión regular y reintenta la escritura versionada hasta 3 veces. Hay **22 pruebas unitarias** con clientes AWS simulados.
- **Las políticas exportadas son de mínimo privilegio:** el dispositivo solo se conecta como su client ID y publica o se suscribe a los temas de su Shadow; la Lambda solo tiene las acciones de IoT y de DynamoDB que usa. Se incluyen la *trust policy*, la definición de la tabla, el Shadow inicial y las variables de entorno de ejemplo.

**Observaciones**
- **Poca modularidad.** Todo el firmware está en una sola clase de 250 líneas que mezcla WiFi, MQTT, Shadow, sensor y servo. La Lambda es una clase con un método `handle()` de ≈ 85 líneas que despacha todos los intents con `if`. La consigna pide un diseño que pueda "extenderse con más sensores y actuadores sin rediseñar su arquitectura". Agregar un sensor hoy obliga a modificar la clase central.
- **El informe describe poco el código.** La §3 nombra cuatro "fragmentos clave" en una sola oración, sin explicarlos ni enlazarlos a líneas concretas.
- La configuración de AWS y Alexa se delega a `configuracion.md`. El informe no incluye una tabla de permisos asignados frente a mínimos (aquí coinciden, pero conviene decirlo de forma explícita).

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- `docs/pruebas.md` tiene un **plan bien diseñado**: casos P01–P18 y N01–N05, frases exactas, criterios de aceptación y método de medición (por ejemplo, video a 30 fps con la resolución de cada cuadro para la latencia).
- Las capturas del simulador (AL01) cubren la apertura de la skill, la activación, las consultas del sensor con y sin movimiento, el estado del motor y el modo automático.
- EP01 registra 15 min 43 s sin reinicios y 11 ciclos completos.

**Observaciones (bajan el nivel)**
- **El tiempo de respuesta no se midió con valores.** `evidencias/latencias.csv` **está vacío**: solo tiene la fila de encabezado. El propio `resultados.csv` dice: "15 mediciones realizadas; valores individuales no transcritos… Falta calcular máximo frente al umbral de 3 s". El RNF central de la práctica queda sin verificar.
- **El informe marca como "✅ Completada" pruebas que el registro marca como `PARCIAL`.** P02, N01, N03 y P13 figuran como parciales en `resultados.csv`, pero como completadas en la tabla de la §4.
- **La vinculación no tiene evidencia.** En `resultados.csv`, P06 dice "Usuario confirma que el comando se probó — **Falta captura** de respuesta y registro persistido DynamoDB". No hay pruebas documentadas de **cambiar** ni de **desvincular** (RF9), de un Thing inexistente, ni de **dos usuarios** con Things distintos (RF8).
- **La reconexión con estado pendiente no está demostrada.** R01 muestra que MQTT se reconecta y que se pide el `desired`, pero el propio registro dice "aplicación no confirmada". Falta la prueba de dar una orden con el objeto desconectado y verla aplicada al volver.
- La frecuencia de reporte no se midió: "Intervalos entre accepted no cuantificados".
- La exactitud del PIR se probó solo de 0 a 50 cm, con un objeto, una vez por distancia. No hay una métrica (tasa de detección, falsos positivos) ni un rango de trabajo declarado.
- Varias evidencias dicen "reportado por el usuario", "según el usuario" o "confirmación expresa del usuario". Este lenguaje deja ver que el texto se redactó a partir de lo que alguien contó, no de la evidencia, y la "confirmación" de un integrante no es evidencia verificable.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Se reportan los datos de compilación (RAM 14.2 %, flash 70.9 %), los 22/22 tests y la ventana de estabilidad de EP01.

**Observaciones**
- **Los resultados de los RNF no están cuantificados.** Latencia: "15 mediciones realizadas", sin media ni máximo. Frecuencia: "temporizador en operación". Reconexión: sin tiempo de recuperación. Exactitud: cualitativa. No hay una tabla de declarado frente a medido.
- **Las conclusiones son descriptivas y repiten la sección de resultados** ("se realizaron 15 mediciones…", "la infraestructura se encuentra desplegada y operativa"). No hay un análisis crítico de qué se cumplió y qué no.
- **Recomendaciones breves** (dos párrafos). Las extensiones hacia un uso real (nivel de consumible, realimentación mecánica, prueba de propiedad) se mencionan sin desarrollarlas.
- Los anexos remiten a una carpeta externa de Google Drive.

---

## 4. Recomendaciones para próximas entregas

1. **Completar las mediciones antes de redactar los resultados.** Transcribir las 15 latencias a `latencias.csv`, calcular la media y el máximo frente a los 3 s, y medir los intervalos de reporte y el tiempo de reconexión.
2. **Que el informe sea coherente con el registro de pruebas.** No marcar como completada una prueba que el propio registro declara parcial.
3. **Documentar la vinculación de punta a punta:** vincular, Thing inexistente, cambiar, desvincular, persistencia en una sesión nueva y dos usuarios con Things distintos, con capturas del simulador y de la tabla DynamoDB.
4. **Usar un identificador de vinculación no trivial** (código o número de serie) y no restringir la Lambda a una lista fija de Things, si se quiere que la skill atienda a cualquier usuario.
5. **Modularizar el firmware y la Lambda:** clases para el sensor, el actuador, el cliente del Shadow y el repositorio de vinculaciones, y un handler por intent.
6. **Incluir en el repositorio toda la evidencia** (videos comprimidos o fotogramas) y agregar una portada con los integrantes.
7. Redactar las evidencias en primera persona del grupo, con datos (fecha, hora, valores), y no como "confirmaciones del usuario".

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **`activationId` y deduplicación:** por qué un booleano no sirve para "pulsar dos veces", qué pasa si el ESP32 se reinicia en medio del ciclo, y por qué se guarda en NVS antes de mover el servo.
- **Bloqueo optimista:** qué hace el parámetro `version` en `UpdateThingShadow`, qué error devuelve AWS si hay conflicto y por qué se reintenta 3 veces.
- **Modo automático:** cómo se detecta el flanco, por qué mantener HIGH no repite ciclos, por qué no se ejecuta sin conexión, y en qué se diferencia el *cooldown* del firmware del retardo del potenciómetro del HC-SR501.
- **Vinculación y seguridad:** qué impide que un usuario vincule el Thing de otro, por qué la Lambda solo puede operar sobre `thing` y `ambientador-dos`, y qué habría que cambiar para atender a un tercer usuario.
- **QoS:** por qué las suscripciones son QoS 1 y las publicaciones QoS 0, y cómo compensa el reporte de respaldo la pérdida de un mensaje.
- **Tiempo de respuesta:** cómo se midieron las 15 latencias, qué valores dieron y si se cumplen los 3 s.
- **Circuito:** alimentación del PIR y del servo con la fuente externa de 5 V, tierra común, para qué sirve el condensador de 470 µF y por qué la salida del HC-SR501 es compatible con 3.3 V.
- **Aporte individual:** el firmware y la mayor parte del informe son de Jose Corrales; RashLop aportó el backend y la infraestructura, y Alejandro Bustamante los intents y las evidencias. Cada integrante debe poder explicar también las partes de los demás.
