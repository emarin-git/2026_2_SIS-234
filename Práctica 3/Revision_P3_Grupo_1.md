# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/Andres004/ThirdProject-AWS-IoT
**Informe:** `Informe.md`
**Integrantes (según el informe):** Ricardo Andres Andrade Vargas · Dabner Eliezer Orozco Veizaga
**Caso de uso:** puerta de un monoambiente — sensor PIR AM312 + servomotor SG90
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **4** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **4** / 4 |
| Resultados, conclusiones y recomendaciones | **4** / 4 |
| **Total** | **16 / 16** |
| **Nota del componente** = (16 / 16) × 100 | **100** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 17:44 (UTC-4), commit `73fed14` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor | Andres (Ricardo Andrade): 34 · dahbner (Dabner Orozco): 10 |

- Se evaluó el commit **`73fed14`**, publicado un minuto antes del cierre.
- Los anexos (logs, CSV de CloudWatch, salidas de los scripts, capturas y videos) **están dentro del repositorio**, en `docs/evidencias/<ID>/`, cada carpeta con su `README.md`.
- No se encontraron certificados, claves privadas, credenciales WiFi ni *access keys* en los archivos ni en el historial. Solo se publica la plantilla `include/secrets.example.h`.
- **Todo el proyecto se hizo en dos días** (9 y 10-Nov). Las 17 evidencias de prueba se subieron en un mismo minuto (10-Nov 00:46).
- **División del trabajo muy marcada.** Todo el código, las herramientas de prueba y las evidencias son de Andres. Los 10 commits de dahbner son solo del informe: ≈ 2000 líneas escritas en una secuencia de commits por sección entre las 11:32 y las 12:55 del 10-Nov. Esto se tomará en cuenta en la defensa individual (ver §5).
- El commit `f7337c7` ("removing guides and extra files"), 3 minutos antes del cierre, borró `docs/PLAN_DE_PRUEBAS.md`. **El informe lo sigue enlazando** en §4 y en los Anexos, y ese enlace quedó roto.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Caso de uso real y bien acotado:** saber si hay alguien en la puerta, y abrirla o cerrarla a distancia. Se justifica por qué se eligieron el PIR y el servo, y por qué se descartaron el ultrasónico, el LED y el solenoide. El LED del circuito es solo un indicador de conexión y el informe lo aclara.
- **Limitaciones del sensor tratadas en el diseño:** retención de 10 s, porque el PIR detecta movimiento y no presencia quieta; 30 s de calentamiento; *pull-down* para que un cable suelto no dé presencias falsas. Dos máquinas de estados (sensor y puerta).
- **Requerimientos completos y medibles.** RF1–RF9 cubren la consigna. RF10–RF12 (confirmar contra `reported`, avisar si el objeto está desconectado y rechazar órdenes inválidas o repetidas) agregan robustez. Cada RF tiene un criterio de aceptación, y cada RNF un valor declarado justificado, su método de verificación y la prueba que lo verifica. La **matriz de trazabilidad** (§1.5) une requisito → diseño → código → prueba → resultado.
- **Todos los diagramas exigidos en §4.2:**
  - Arquitectura en dos vistas (despliegue y componentes), con una tabla de nodos y el recorrido de un comando por las capas.
  - Circuito en SVG, con la tabla de pines, el cálculo del pulso del servo y de la corriente del LED, la justificación de los GPIO y una foto del montaje.
  - Diagramas de clases del firmware y de la Lambda.
  - Cuatro diagramas de secuencia: vinculación, consulta, comando y reconexión con estado pendiente.
- **Especificación del Shadow completa** (§2.5): temas MQTT, atributos con tipo y valores válidos, uso de `desired`, `reported` y `delta`, manejo de versiones, errores y una sesión real.
- **Mecanismo de vinculación bien pensado** (§2.6). El código de 6 dígitos sin cero inicial se justifica: no es trivial de adivinar, y `AMAZON.NUMBER` perdería el cero a la izquierda. El informe incluye el modelo de datos, las validaciones en orden y sus limitaciones (no prueba la propiedad del objeto, y el `userId` cambia si se deshabilita la skill).
- **Frecuencia de reporte justificada** por evento + heartbeat de 60 s, con el cálculo del consumo frente a la capa gratuita (Tabla 3). Muestra que un heartbeat de 10 s la habría superado.

**Observaciones menores (no bajan el nivel)**
- **La política de AWS IoT usada es `Allow * *`.** Está documentada con su riesgo y con la política mínima, como permite la consigna. Aun así, convenía aplicar la mínima, que ya estaba escrita (`aws/iot-policy.json`).

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Firmware modular y extensible.** La interfaz `DeviceComponent` permite agregar un sensor o actuador sin tocar el código del Shadow: `DoorActuator` y `PresenceSensor` se registran en `components[]`. La abstracción `ShadowTransport` tiene dos implementaciones (`ShadowClient` por MQTT/TLS y `SerialShadowClient` para pruebas), y las clases de soporte son `ShadowReporter`, `WifiManager`, `HealthMonitor`, `EventLog` y `TimeSync`. Las constantes están en `config.h`.
- **Lambda orientada a objetos** con el ASK SDK: `LinkRepository` (DynamoDB), `ThingGateway` (IoT), `ShadowState`, una jerarquía de handlers (`LinkedThingHandler` → `DoorCommandHandler` → `OpenDoorHandler` y `CloseDoorHandler`) e interceptores que registran una línea `SKILL_REQUEST` por solicitud, con el `userId` reducido a un hash.
- **Decisiones técnicas finas, y explicadas:** filtro de versiones del `delta` (`STALE_IGNORED`); `reported.door` se publica recién al terminar los 400 ms del servo; la Lambda dice "Listo" solo al leer la confirmación del objeto (RF10); reconexión no bloqueante y estado seguro (puerta cerrada) al arrancar.
- **Configuración de AWS y Alexa bien documentada** (§3.4–3.6): parámetros de la Lambda (timeout de 7 s, justificado frente a los 8 s de Alexa), verificación del Skill ID en el disparador, tabla de permisos **asignados frente a mínimos**, con el riesgo de cada uno, y tabla de secretos con cómo se protege cada uno.
- Se exportan todas las configuraciones que pide §6: modelo de interacción, política de IoT usada y mínima, política del rol, definición de la tabla y plantilla de secretos.

**Observaciones menores**
- El rol de la Lambda usa `AWSIoTFullAccess` y `AmazonDynamoDBFullAccess`. Está documentado y aceptado por la consigna, pero, como en la política de IoT, la versión mínima ya estaba escrita y no se aplicó.
- La conexión MQTT sigue siendo sincrónica: bloquea `loop()` ≈ 2 s al conectar. El informe lo reconoce (§4.5) y propone moverla a una tarea de FreeRTOS.

### 3.3 Pruebas y validaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Plan completo** con 10 pruebas físicas (A01–A10) y 3 automáticas (B01–B03), cada una con requisitos, procedimiento, criterio de éxito, resultado observado, estado y evidencia.
- **Frases de la skill verificadas una por una.** A01 recorre 13 frases en orden: sin vincular, código inválido, Thing inexistente, vincular, repetir, cambiar, desvincular y persistencia en una sesión nueva. Cada frase tiene su respuesta y su `result` de CloudWatch.
- **RNF verificados con números y herramientas propias.** `analyze_log.py` une el log del objeto con CloudWatch por la versión del Shadow. Resultados: 28 comandos para RNF3 (mediana 919 ms, máx. 1691 ms), 27.3 min de estabilidad, caída de WiFi y corte de energía con orden pendiente (3.3–3.9 s) y valores inválidos publicados desde la consola.
- **Las pruebas encontraron defectos reales, que se corrigieron** (§4.4): la confirmación prematura del servo por una resta sin signo (D1) y el bloqueo de `loop()` de 7 s con WiFi sin Internet (D2). Se guardan los logs de antes y de después de cada corrección.
- **Análisis de errores sobresaliente** (§4.5): explica por qué el peor caso es de 804 ms frente a 254 ms, separa el tiempo de la nube del tiempo del servo y calcula la cota de confianza del PIR (15/15 → 81.9 %). También dice qué tramo no se midió (voz → Lambda).

**Observaciones menores**
- **El segundo usuario se simuló con un evento de prueba de la Lambda** (`test-user-2`), no con otra cuenta de Alexa. El informe lo reconoce. En la demo conviene mostrarlo con dos cuentas reales.
- El plan detallado (`docs/PLAN_DE_PRUEBAS.md`) se borró antes de la entrega y su enlace quedó roto. El resumen del plan sí está en el informe.

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Tabla consolidada** (Tabla 10) con valor declarado, medido, prueba, tipo de evidencia y veredicto de cada RF y RNF. Dos gráficos: el margen de cada requisito de tiempo y la composición de un comando (Lambda 109 ms, nube 96 ms, servo 411 ms, reporte y consulta 303 ms).
- **Análisis crítico real:** separa lo que los datos permiten afirmar de lo que **no**, como el error del servo por debajo de 3°, la tasa del PIR ≥ 95 %, la estabilidad por días o la generalidad de los tiempos con una sola red.
- **Conclusiones fundamentadas en las mediciones.** Por ejemplo: "confirmar con `reported` cuesta ≈ 300 ms, a cambio de que la skill no mienta".
- **Recomendaciones técnicas ligadas a la prueba que las motiva** (MQTT asíncrono, borrar el `desired` inválido, aplicar los permisos mínimos, *reed switch* para la posición real, fuente aparte), y **extensiones hacia un uso real** (cerradura de 12 V con MOSFET, eventos proactivos de Alexa, historial de presencia, varias puertas por usuario, código de un solo uso).
- Anexos completos e indexados.

**Observaciones menores**
- Todas las pruebas se hicieron en una noche, con una sola red (hotspot) y una sola placa. El informe lo reconoce como limitación.

---

## 4. Recomendaciones para próximas entregas

1. **Equilibrar el trabajo técnico entre los integrantes.** El historial muestra que un integrante hizo todo el código y las pruebas, y el otro solo el informe. La consigna (§5.4) evalúa el aporte de cada uno en el historial de commits.
2. **No hacer limpiezas de último minuto.** El commit de las 17:42 borró un archivo que el informe enlaza. Antes de entregar, conviene comprobar que todos los enlaces internos existan.
3. **Aplicar las políticas mínimas que ya se escribieron.** El grupo documentó las políticas de mínimo privilegio, pero dejó `Allow * *` y las políticas `FullAccess`. Aplicarlas y repetir las pruebas cuesta poco y elimina el riesgo descrito en el propio informe.
4. **Distribuir el proyecto en el tiempo.** Todo se desarrolló, probó y documentó en 48 horas, y las evidencias se subieron en un solo minuto. Un fallo de hardware o de red el último día habría dejado requisitos sin validar.
5. Probar la vinculación de varios usuarios con **dos cuentas de Alexa reales**, no solo con un evento de prueba de la Lambda.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

El informe tiene un nivel de detalle muy alto. Como el código es de un integrante y el informe del otro, **en la defensa cada integrante debe demostrar dominio de la parte que no hizo**:

- **Quien escribió el informe (dahbner)** debe poder explicar el código: la interfaz `DeviceComponent` y cómo se agregaría un sensor nuevo, `ShadowTransport` y por qué tiene una implementación serie, el filtro de versiones (`acceptVersion`), el defecto D1 (por qué la resta sin signo daba ≈ 2³²) y el D2 (por qué el intervalo entre reintentos se cuenta desde el final del intento).
- **Quien hizo el código (Andres)** debe poder justificar el diseño del informe: la matriz de trazabilidad, la cota de confianza del 81.9 % y el cálculo del consumo de la capa gratuita.
- **Shadow:** diferencia entre `desired`, `reported` y `delta`; por qué AWS reenvía el `delta` mientras el `desired` inválido sigue en el Shadow (A09); qué temas suscribe el objeto y con qué QoS; qué hace el objeto al reconectarse (`/get`).
- **Lambda y confirmación:** cómo funciona `wait_for_door()` (consulta cada 250 ms durante 3 s), por qué el timeout de la Lambda es de 7 s y qué responde si el objeto está desconectado (umbral de 150 s).
- **Vinculación:** por qué el código no empieza en 0, por qué se usa `DescribeThing` y qué permiso IAM requiere, qué pasa si dos usuarios vinculan el mismo Thing, y por qué se usa `PutItem` para "cambiar".
- **Seguridad:** qué permitiría un atacante con el certificado y la política `Allow * *`, frente a la política mínima, y cómo se protegieron los secretos en el repositorio.
- **Circuito:** alimentación del servo desde VIN, riesgo de *brownout*, por qué el AM312 no necesita divisor de tensión y cálculo del pulso de 1450 µs para 90°.
