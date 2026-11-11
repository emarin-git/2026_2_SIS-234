# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/Alous12/Practica_3_IOT
**Informe:** `docs/INFORME.md`
**Integrantes (según el informe):** Alejandro Machaca · Ruben Cordero · Luciana Leaño
**Caso de uso:** "Vigilante inteligente" — PIR HC-SR501 + servomotor SG90 como pestillo
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
| Último push a `main` | 10-Nov 17:36 (UTC-4), commit `42d1b8d` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Luciana Leaño: 11 · Rubén Cordero: 7 · Alejandro Machaca (alous12): 6 |

- Se evaluó el commit **`42d1b8d`**.
- **Buena organización del trabajo:**
  - El informe declara qué capa tuvo a cargo cada integrante (nube, hardware y comunicación), y los commits lo confirman.
  - Se usaron ramas (`lu`, `hardware`, `Rama_Ale`).
  - Hay un *workflow* de GitHub Actions que ejecuta las pruebas de la Lambda, compila los tres firmwares y corre las pruebas C++.
- No se encontraron certificados, claves privadas, credenciales WiFi ni *access keys* en el historial (`secrets.h` no se subió nunca). El informe reconoce que **el ID de cuenta apareció en una primera versión de las capturas** (commit `b26bc6b`) y que se tapó después sin reescribir el historial. Es correcto declararlo, aunque el ID sigue visible en ese commit.
- Las evidencias están en `docs/anexos/` (capturas de AWS y Alexa, CSV, foto del montaje y circuito en 2D y 3D). **No hay videos** del servo ni de las mediciones de latencia en el repositorio.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Caso de uso real** (vigilar un objeto valioso y trabar su puerta), con una tabla de **alternativas descartadas** (barrera láser, *reed switch*, ultrasónico, solenoide, buzzer) y la razón de cada una. La extensión futura (barrera láser y buzzer como un `Sensor` y un `Actuator` más) es coherente con la arquitectura.
- **Requerimientos trazables:** cada RF indica la clase o handler que lo implementa y la prueba que lo verifica. **Cada RNF tiene un valor medible** (≥ 30 min; ≥ 95 % de detección hasta 3 m y 0 falsos positivos; ≤ 3 s; evento ≤ 1 s y latido de 60 s ± 1 s; reconexión ≤ 30 s), junto con su método de verificación.
- **La métrica de exactitud de un sensor binario está bien justificada:** detecciones perdidas frente a pasadas controladas con cinta métrica, y falsos positivos con la sala vacía. La frecuencia de reporte se justifica por la dinámica del HC-SR501 (pulso de ≈ 1.2 s y bloqueo de ≈ 5 s).
- **Diagramas completos:**
  - Arquitectura.
  - Circuito (diagrama 2D, vista 3D en SVG y foto del montaje), con lista de componentes, conexiones, justificación de los pines y ajustes del PIR.
  - Diagramas de clases del firmware y de la Lambda (con un diagrama de dependencias entre módulos).
  - Secuencias de vinculación (vincular, rechazar, cambiar y desvincular), consulta y comando con reconexión.
  - Ciclo de vida de una vinculación y flujo de la conversación.
- **Mecanismo de vinculación sólido:** código de 6 dígitos asociado al Thing `vigilante-<código>`, validación con `DescribeThing`, confirmación por diálogo ("¿sí?") antes de vincular o desvincular, y limitaciones documentadas.
- **Diseño de la skill** con intents, slots, frases, delegación automática del diálogo y manejo de la sesión.

**Observaciones menores (no bajan el nivel)**
- Los códigos de demostración (`123456`, `654321`) son triviales de adivinar. El informe lo reconoce y propone códigos aleatorios publicados por el ESP32 en su Shadow.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Firmware con una arquitectura en capas muy clara:**
  - `core/` define las interfaces `Sensor`, `Actuator` y `ShadowPort`.
  - `sensors/PirSensor`, `actuators/DoorLock`, `net/` (`WifiConnection`, `ShadowClient`, `TimeSync`) y `app/SmartObject` las implementan y coordinan.
  - Con `ShadowPort`, el mismo código corre con AWS o por USB en modo local, y se prueba en la PC con dobles de hardware (`test/luciana`, `test/ruben`).
- **Lambda organizada como paquete** (`vigilante/`): `link_repository`, `device_gateway`, `device_state`, `handlers`, `speech`, `config` y `skill`, con dependencias inyectadas. **61 pruebas** con `pytest`, incluidas pruebas de contrato entre capas y un guion completo de la skill. También hay eventos de prueba JSON (`lambda/events/01…09`) y scripts de empaquetado.
- **Seguridad bien resuelta:**
  - Política de IoT con `${iot:Connection.Thing.ThingName}`, **verificada en vivo**: un cliente con el certificado del dispositivo no puede escribir el Shadow de otro Thing ni conectarse con otro client ID.
  - Rol de la Lambda con una política propia de mínimo privilegio (`vigilante-lambda-minimo`).
  - Alerta de facturación y estimación del costo (≈ USD 0.11/mes).
- **Configuración de AWS y Alexa documentada con capturas** (facturación, política, Things, certificado, Shadow, tabla, ítems, rol, Lambda, variables, invocación, intents, endpoint, *build* y *test*), con el ID de cuenta, el endpoint y el Skill ID tapados.
- Detalles de robustez: filtro de 50 ms y *pull-down* en el PIR, calentamiento de 60 s, se suelta el PWM tras 700 ms para que el servo no vibre, estado seguro (cerrada) al encender, y alimentación de potencia separada (4 × AA) con tierra común.

**Observaciones menores**
- `reported.puerta` es la posición ordenada, no una medición. El informe lo reconoce y propone un *reed switch*.

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Plan completo** (P01–P21): requisito, procedimiento, criterio de aceptación, responsable, resultado y evidencia de cada prueba.
- **Guion del simulador de 16 pasos** que recorre RF4–RF9: sin vincular, código inexistente, vincular, comando con un Thing sin hardware, cambiar, consultar el vínculo, sensor con y sin presencia, cerrar, consultar, abrir, segundo usuario, desvincular con "no" y con "sí". El mismo guion está automatizado.
- **RNF medidos con valores:**
  - RNF1: 30 min con `uptime_s` y RSSI.
  - RNF2: 20 pasadas por distancia (98.3 % entre 1 y 3 m) y 0 falsos positivos en 10 min.
  - RNF3: 10 comandos a 60 fps (promedio 1.60 s, máximo 2.12 s).
  - RNF4: latido de 60.0 s ± 0.1 s; eventos en 0.31 s.
  - RNF5: 3 cortes de WiFi, con reconexión en 7.0–9.7 s y la orden pendiente aplicada.
- **Verificación en vivo de la política de IoT** con `paho-mqtt` y el certificado real, y **log del arranque del ESP32** aplicando una orden pendiente dada con la placa apagada.
- **Análisis de errores del montaje muy valioso:** un SG90 de rotación continua confundido con uno de 180°, falsos contactos en GPIO 13, interferencias al cargar el firmware y caídas con un cargador. Cada problema se diagnosticó separando el software del hardware.

**Observaciones (bajan el nivel)**
- **La evidencia de RNF3, RNF4 y RNF5 son solo tablas transcritas.** No hay en el repositorio el video a 60 fps del que salen los números de cuadro, ni la exportación de CloudWatch con `latencia_ms`, ni los logs del cliente MQTT o del monitor serie de los latidos, los eventos y los cortes de WiFi. `pruebas-comunicacion.md` repite las mismas tablas, sin registros crudos. Los valores no se pueden verificar.
- **Las respuestas del guion del simulador no tienen capturas paso a paso.** Solo hay una captura de la pestaña *Test* (`alexa-05`). La evidencia que se cita es la prueba automática, que verifica la Lambda pero no el simulador.
- **El segundo usuario solo se probó de forma automatizada** (`test_guion_segundo_usuario…` y el evento `09_vincular_segundo_usuario.json`). El paso 13 del guion dice "Opción A u B", sin indicar cuál se usó, y no hay captura con otra cuenta de Alexa.
- La estabilidad (RNF1) se probó con órdenes desde la consola de AWS IoT, no desde Alexa. La corrida es una sola, con 0 de margen sobre los 30 min declarados (≈ 1826 s).

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Tabla de resultados** con valor declarado frente a obtenido para cada RF y RNF, un gráfico de detección por distancia y la estimación del costo.
- **Análisis crítico:** explica la diferencia de ≈ 1 s entre el Δt de punta a punta y `latencia_ms` (Alexa e invocación de la Lambda), el peor caso por arranque en frío, el riesgo de la red WiFi, y una sección de **"lo que los datos no permiten afirmar"** (estabilidad > 30 min, ningún margen a 3 m, una sola sala, posición real del pestillo, RNF3 con mala señal).
- **Conclusiones por capa y por integrante,** apoyadas en los datos, y una conclusión valiosa sobre el trabajo en equipo: las pruebas de contrato detectaron incompatibilidades al unir las ramas.
- **Recomendaciones técnicas concretas** de hardware, firmware y nube (*reed switch*, solenoide de 12 V con MOSFET, jumper H, OTA, *backoff*, códigos aleatorios, Account Linking, notificaciones proactivas, infraestructura como código, confirmación sin sondeo).
- Anexos organizados por responsable.

---

## 4. Recomendaciones para próximas entregas

1. **Guardar la evidencia cruda de las mediciones:** el video (o al menos los fotogramas de inicio y fin) de RNF3, la exportación de CloudWatch y los logs del monitor serie o del cliente MQTT de RNF4 y RNF5. Una tabla transcrita sin su fuente no se puede verificar.
2. **Capturar cada paso del guion del simulador,** con *Skill I/O* visible, y probar el segundo usuario con otra cuenta de Alexa real.
3. **Dar margen a los valores declarados:** la estabilidad se declaró en 30 min y se midieron ≈ 30.4 min. Una corrida de 45–60 min daría un margen real.
4. **Cuidar las capturas antes del primer commit.** El ID de cuenta quedó en el historial; conviene revisar las imágenes antes de subirlas.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

El informe atribuye cada capa a un integrante. En la defensa **cada uno debe poder explicar también las capas de los otros**:

- **Firmware:** qué es `ShadowPort` y cómo permite ejecutar el firmware sin AWS; cómo se agregaría la barrera láser como un `Sensor`; qué hace `SmartObject::handleDelta`; por qué la separación mínima de 500 ms tiene excepciones (confirmación del actuador, primer reporte tras reconectar).
- **Hardware:** cómo se distingue un SG90 de 180° de uno de 360°; por qué se separa la alimentación del servo (pilas) de la del ESP32 (USB) con tierra común; qué hacen el jumper L/H y los potenciómetros Sx y Tx del HC-SR501; por qué se movió el servo a GPIO 25.
- **Lambda:** cómo funciona la espera de la confirmación (consulta cada 250 ms hasta `CONFIRM_TIMEOUT_S`), qué responde la skill si el ESP32 no confirma y cómo se valida el código con `DescribeThing`.
- **Seguridad:** qué significa `${iot:Connection.Thing.ThingName}` en la política, qué se comprobó con `paho-mqtt` y por qué el ID de cuenta no es una credencial, pero conviene ocultarlo.
- **Mediciones:** cómo se calculó Δt con los cuadros del video a 60 fps, qué incluye y qué no incluye `latencia_ms`, y por qué el intento 6 fue el más lento.
- **Vinculación:** por qué se pide confirmación antes de vincular o desvincular, qué pasa si dos usuarios vinculan el mismo Thing y cómo se haría el código no adivinable.
