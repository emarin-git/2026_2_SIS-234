# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/HenryRiveraM/smartpark-practica-3
**Informe:** `docs/informe.md`
**Integrantes (según el informe):** Carlos Eduardo Iriarte Rodriguez · Isabella Catherine Montes Iñiguez · Henry Alejandro Rivera Mendez
**Caso de uso:** SmartPark — barrera de parqueo con sensor ultrasónico HC-SR04 + servomotor SG90
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **4** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **4** / 4 |
| **Total** | **15 / 16** |
| **Nota del componente** = (15 / 16) × 100 | **93.75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 16:18 (UTC-4), commit `8f98f7a` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Carlos Iriarte (ciriarte-dev y su nombre completo): 16 (informe, diagramas) · Henry Rivera: 10 (firmware, Lambda, AWS) · Isabella Montes: 7 (pruebas funcionales y evidencias) |

- Se evaluó el commit **`8f98f7a`**.
- **Buena organización:** ramas por integrante o por tarea (`henry`, `Pruebas`, `docs/...`) integradas con 15 *pull requests*. Cada prueba funcional tiene su carpeta con README, hoja de cálculo, fotos y video.
- **División del trabajo marcada:** **todo el código** (firmware, Lambda y pruebas automáticas) es de Henry Rivera; Carlos Iriarte escribió el informe e Isabella Montes ejecutó y documentó las pruebas.
- No se encontraron certificados, claves privadas, credenciales WiFi ni *access keys* en el historial. Se versionaron archivos `.DS_Store`, como reconoce el propio informe (§7.5).
- **Uso del ultrasónico:** la consigna lo permite solo si el caso de uso lo requiere. **El informe lo justifica bien**, con una comparación frente al IR, el reed, el lazo inductivo y el ToF: se necesita una distancia sin contacto y un umbral configurable para detectar un vehículo de cualquier tipo.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Caso de uso real y bien delimitado:** actores (usuario, administrador, objeto, nube), flujo general, un escenario paso a paso con tiempos característicos y una tabla de "el sistema sí / el sistema no".
- **Sensor y actuador justificados con tablas comparativas** de candidatos (magnitud, rango, contacto, costo, interfaz y adecuación). Se incluye el divisor 1 kΩ/2 kΩ para ECHO y la fuente externa de 5 V para el servo.
- **Cada RF tiene un enlace a la línea de código que lo implementa** y un criterio de aceptación observable, con la prueba de banco y la prueba automática que lo verifican.
- **RNF con valor declarado, justificación, método de medición y riesgo si no se cumplen.** Se agrega un RNF de consumo de mensajes, con una cuota de 1000 solicitudes por día contada en NVS y 200 reservadas para control y recuperación.
- **Diagramas completos:**
  - Arquitectura (imagen y Mermaid).
  - Circuito.
  - Varios diagramas de clases del firmware y de la Lambda.
  - Secuencias de vinculación, consulta y comando, con las ramas de "sin vinculación" y "desconectado".
- **Mecanismo de vinculación excelente:** un **administrador genera un PIN temporal** (300 s, 3 intentos, un solo uso) para un `userId` y un Thing concretos, con una Lambda administrativa sin disparador público. El PIN se consume con una transacción en DynamoDB, que es segura frente a la concurrencia. Así, el nombre del Thing nunca se toma de la voz del usuario.
- La política de reporte considera la dinámica de la variable (presencia estable en 3 muestras, Δ ≥ 10 cm, separación mínima de 10 s, respaldo de 30 s) y la frescura del dato en la Lambda (≤ 180 s).

**Observaciones menores (no bajan el nivel)**
- Quedan restos de redacción en borrador: "Los valores marcados como **propuesta** deben ser confirmados por el grupo antes de medir", y "Criterio propuesto… Es una meta por verificar experimentalmente, no un resultado" junto a un resultado ya medido.
- El informe es **muy extenso** (≈ 2300 líneas) y algunas ideas se repiten entre §1.4, §2.6 y §5.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Firmware modular:** `UltrasonicSensor` (espera acotada a 30 ms y clasificación válida, *timeout* o fuera de rango), `BarrierServo`, `AwsIoTManager` (tarea de red separada con colas FreeRTOS, *backoff* exponencial con *jitter* y GET de recuperación), `ShadowBudget` (cuota en NVS) y `SerialCommands`.
- **Pruebas del firmware en el PC** con dobles de WiFi, PubSubClient, NVS, FreeRTOS y TLS, compiladas con `-Wall -Wextra -Werror -pedantic`.
- **Lambda bien estructurada:** `handlers/` (barrera, vinculación, sensor, sistema), `services/` (DynamoDB, Shadow, validación, política de PIN y errores) y `administration/` (servicio de emparejamiento). Tiene **91 pruebas**, incluidas pruebas de concurrencia del PIN, aislamiento entre paquetes y logs sin datos sensibles, más eventos JSON de prueba y scripts de empaquetado y verificación.
- **Seguridad del backend:** validación del Skill ID y de la antigüedad de la solicitud (≤ 150 s), *timeouts* acotados en las llamadas a AWS y la Lambda administrativa sin disparador ni URL pública.
- **Configuración de AWS y Alexa documentada** con un procedimiento de despliegue reproducible en 12 pasos y capturas del Thing, el Shadow, el certificado, la política, las variables de las dos Lambdas, el rol IAM, la invocación, los intents y el simulador.

**Observaciones menores**
- **La política de IoT desplegada es `iot:*` sobre `*`.** El informe lo reconoce ("la política capturada tiene permisos más amplios que el ejemplo de mínimo privilegio") y entrega la política mínima, pero **no explica por qué** se usó la amplia, como pide la consigna (§3.2). La recomendación §7.5 sugiere que también podrían quedar políticas `FullAccess` en los roles.
- La compilación y la suite de la Lambda no se volvieron a ejecutar sobre la versión final (el informe lo declara).

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Pruebas funcionales con evidencia real en el repositorio:** fotos, hojas de cálculo y **videos** de PF-01 a PF-08 (medición, histéresis, Shadow, abrir y cerrar por voz, consultas, persistencia entre sesiones, PIN inválido).
- **Exactitud del sensor bien medida:** 35 lecturas entre 5 y 50 cm con cinta métrica y un error máximo de 1.0 cm. La histéresis se probó a 10, 16, 17 y 18 cm, y se explica correctamente por qué a 17 cm el resultado depende del estado anterior.
- **Honestidad en el reporte:** PNF-01 (3 reinicios en 10 min) y PNF-03 (una medición de 4 s) se declaran **"No cumple"**, y PNF-04 como "No concluyente". Además, se corrigen las discrepancias de la hoja original (máximo de 5 s frente a 4 s; minutos rotulados como segundos) sin alterar el PDF.
- Hay una matriz de trazabilidad y una tabla de casos negativos cubiertos por las pruebas automáticas (PIN vencido, intentos en paralelo, dato desactualizado, solicitud antigua, *timeout*).

**Observaciones (bajan el nivel)**
- **El tiempo de respuesta se midió con una resolución de 1 s** (valores 1, 2, 3, 4) y **sin el video cronometrado** que el propio protocolo exigía ("No incluido: impide auditar el inicio y fin de cada medición"). Para un límite de 3 s, esa resolución no permite saber si 3 s cumple o no.
- **Los datos de reconexión (PNF-04) son de mala calidad:** están en minutos (1 a 4 min), no se registró la duración del corte y no se sabe si el tiempo de MQTT es acumulado o adicional. El resultado no se puede comparar con el criterio de ≤ 60 s.
- **La estabilidad falló y no se volvió a medir.** Hubo 3 reinicios sin registrar su causa (`RESET_REASON`), y la frecuencia de reporte no se pudo calcular porque faltaron marcas de tiempo.
- **RF9 no se probó en el simulador:** el cambio de vinculación, la desvinculación y el Thing inexistente solo tienen cobertura automática.

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Resultados cuantificados y comparados con lo declarado:** resumen de cumplimiento (8/8 funcionales; 1 aprobada, 2 fallidas y 1 no concluyente en las no funcionales), tablas de exactitud, latencia, reconexión y estabilidad con lo esperado frente a lo medido, y recursos del firmware.
- **Análisis crítico real:** distingue implementación, validación y resultado medido. Explica qué no se puede concluir de los 10 UPDATE aceptados y señala la evidencia que falta (video de PNF-03, salidas de las suites).
- **Conclusiones fundamentadas que incluyen los incumplimientos.** Por ejemplo: "una función puede ejecutarse correctamente y, al mismo tiempo, incumplir un requisito no funcional".
- **Recomendaciones priorizadas y ligadas a cada resultado, con su criterio de verificación:** investigar los reinicios (causa, memoria, alimentación del SG90), medir la latencia por etapas, finales de carrera o sensores Hall, detección de obstáculos, mínimo privilegio, pruebas repetibles. También se proponen **extensiones hacia un parqueo real** (sensor de salida, contador de plazas, panel administrativo, varias barreras).
- Anexos completos, con glosario, referencias y evidencias organizadas por prueba.

---

## 4. Recomendaciones para próximas entregas

1. **Instrumentar las pruebas de tiempo:** grabar el video (≥ 30 fps) junto con las mediciones, o registrar marcas de tiempo en el monitor serie y en CloudWatch, y reportar con resolución de décimas de segundo.
2. **Cuando un RNF falla, diagnosticar y volver a medir.** Los 3 reinicios de PNF-01 son la prioridad: registrar `esp_reset_reason()`, el heap libre y la tensión del servo, corregir y repetir la prueba. Un resultado "No cumple" bien documentado es valioso, pero cerrar el ciclo lo es más.
3. **Repetir PNF-04 con un protocolo claro:** hora del corte, duración, hora de la restauración, `WIFI_CONNECTED`, `MQTT_CONNECTED`, GET y `reported`, todo en segundos y desde el mismo origen.
4. **Probar RF9 en el simulador:** cambiar de Thing, desvincular y vincular con un Thing inexistente, con capturas de DynamoDB.
5. **Distribuir el trabajo técnico.** Todo el código es de un solo integrante. Para la defensa individual (§5.4), los demás deben haber trabajado también sobre el firmware o la Lambda.
6. Aplicar la política de IoT mínima que ya está escrita, o justificar la amplia, y quitar los `.DS_Store` del repositorio.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

Como el código es de Henry Rivera, **Carlos Iriarte e Isabella Montes deben demostrar que dominan el firmware y la Lambda**:

- **Firmware:** por qué la red corre en una tarea separada con colas FreeRTOS, cómo funciona el *backoff* exponencial con *jitter* y qué pasa al agotar los 10 intentos, qué hace `ShadowBudget` y por qué reserva 200 solicitudes, y cómo se recupera una orden dada sin conexión (el `delta` no se reenvía, por eso el GET).
- **HC-SR04:** por qué se usa el divisor 1 kΩ/2 kΩ en ECHO (cálculo de la tensión), qué es la histéresis 15/18 cm, por qué se exigen 3 muestras iguales y qué significa un *timeout* de 30 ms en distancia.
- **Vinculación con PIN:** el flujo administrador → PIN → usuario, por qué el PIN vence a los 300 s y admite 3 intentos, cómo evita la transacción de DynamoDB que dos solicitudes en paralelo consuman el mismo PIN, y por qué la Lambda administrativa no tiene disparador.
- **Lambda:** el control de frescura (180 s por campo), la validación de antigüedad de la solicitud (150 s) y por qué la Lambda no espera la confirmación del dispositivo.
- **Seguridad:** qué permite la política `iot:*` desplegada frente a la mínima del repositorio.
- **Resultados:** posibles causas de los 3 reinicios (alimentación del servo, memoria, *watchdog*) y cómo se mediría la latencia por etapas.
- **Aporte individual:** el historial muestra a Henry en el código, a Carlos en el informe y a Isabella en las pruebas. Cada integrante debe poder explicar las tres partes.
