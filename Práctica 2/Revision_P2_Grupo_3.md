# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/Lucas-RodriguezAncieta/IoT-Practica-2
**Informe:** `informe.md`
**Integrantes:** **no figuran en el informe.** Según el historial de commits: Lucas Rodriguez Ancieta · Pablo Raya · Josue Camacho
**Fecha límite:** martes 13-Oct-2026
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
| **Nota del componente** = (10 / 16) × 100 | **62,5** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 17:37 (UTC-4), commit `9b9d4da` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor | pabloraya: 46 · Josue Camacho: 8 · Lucas Rodriguez Ancieta: 5 |

- Se evaluó el commit **`9b9d4da`**.
- **Todo el proyecto se construyó en menos de 18 horas** (primer commit: 13-Oct 00:26). El código del sensor se subió entre las 14:34 y las 14:50 en ~25 commits creados desde la interfaz web de GitHub ("Create…", "Delete…", "Update .gitkeep"), uno por archivo. Ese historial no muestra un desarrollo incremental, sino la subida de archivos ya terminados.
- El informe **no lista a los integrantes** e indica que "las fuentes disponibles no vinculan estas áreas con integrantes identificados individualmente". La portada de un informe técnico debe identificar a sus autores y su aporte.
- No hay fotografías del montaje, videos ni logs crudos en el repositorio. La propia carpeta de evidencias aclara que "no afirma la existencia de fotografías, vídeos, logs u otras evidencias".

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- La tabla de requerimientos (RF1–RF7, RNF1–RNF4) incluye responsable, prueba prevista y estado. Los rangos (`[2,20)`, `[20,50)`, `[50,100]`, fuera de rango = INVALID) son contiguos, no se solapan y tienen los límites asignados de forma explícita.
- RF6 (el servidor es la única autoridad de clasificación) y RF7 (manejo de mensajes malformados) son buenos requisitos de diseño.
- La **especificación del protocolo es completa**: registro por rol, `DISTANCE`/`INVALID`, `SET`, `ACK`, códigos `ERROR`, gramática del valor numérico, límite de 128 bytes, comportamiento ante líneas largas y publicación periódica cada 200 ms.
- Hay diagramas de clases de los tres programas, un diagrama de secuencia con los mensajes y los casos de desconexión, y circuitos SVG con el **divisor de tensión 1 kΩ / 2 kΩ** en ECHO.
- Explica correctamente la diferencia entre `0.0.0.0` (dirección de escucha) y la IP real de la PC.

**Por qué no llega a 4**
- El **diagrama de arquitectura del informe** solo tiene tres cajas y dos flechas. No muestra el punto de acceso, la red 802.11, las direcciones y puertos ni la pila TCP/IP. `diagramas/arquitectura.md` tiene más detalle, pero el informe no lo integra.
- Los circuitos se presentan como "circuito **lógico**", con la advertencia de "verificar las etiquetas GPIO antes de energizar". La §2.8 dice además que el documento "**no afirma que exista un montaje probado**". Es un lenguaje de diseño *previo* al montaje, y contradice la afirmación de que hubo pruebas físicas.
- Los diagramas de clases omiten los atributos "deliberadamente", y los de los clientes TCP no muestran su estado interno (buffer, registro, temporizadores).
- La tabla de RNF no indica el **método de medición** (con qué instrumento y cómo se toma el tiempo de respuesta). Solo remite a casos M-xx.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Estructura modular y orientada a objetos: `UltrasonicSensor` (con `Measurement` y `MeasurementStatus`), `WiFiConnection`, `TcpSensorClient`, `SensorProtocol`, `LedController`, `TcpActuatorClient`; en el servidor, `TcpServer`, `DistanceController` y `protocol.py`.
- Constantes con nombre (`constexpr`), muestreo con `millis()`, reconexión no bloqueante y credenciales fuera del repositorio (`WiFiSecrets.example.h` + `__has_include`).
- El servidor usa hilos, `RLock` y un *lock* de envío que serializa las escrituras en el socket. Tiene pruebas automatizadas (24 pytest) y un script que verifica la configuración de credenciales.
- El parpadeo mantiene la fase ante un `SET` repetido.

**Por qué no llega a 4**
- **Documentación del código casi inexistente.** La mayoría de los `.cpp` y `.h` del firmware no tienen ningún comentario (`TcpSensorClient.cpp`, `UltrasonicSensor.cpp`, `LedController.cpp`, `TcpActuatorClient.cpp` [256 líneas], `WiFiConnection.cpp`). En Python, solo hay *docstrings* en algunas clases. La consigna pide código documentado.
- **Idioma mezclado:** la mayoría del código está en inglés y algunos mensajes y *docstrings* están en español.
- `pulseIn(..., 30000)` bloquea hasta 30 ms en cada medición. Con un periodo de 200 ms no es crítico, pero no se analiza.
- La sección §3 del informe describe el código en prosa densa y no explica fragmentos clave (por ejemplo, el *framing* o la publicación periódica).
- Las pruebas Unity "en placa" se mencionan como **pendientes**.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay un plan de pruebas manuales (M-01 a M-11 en `tests/manual/README.md`) y una tabla que asigna cada CSV a su requisito.
- Las pruebas automatizadas del servidor (24 pytest) cubren clasificación, *parser*, roles, fragmentación, expiración y mensajes inválidos.
- Los CSV tienen una estructura útil (referencia, medida, error, límite, veredicto).

**Por qué baja a 2**
- **Los CSV de las pruebas físicas tienen inconsistencias técnicas que impiden aceptarlos como evidencia válida:**
  - **RNF4 (muestreo):** el CSV registra **307 lecturas en 60 s (5.117 lecturas/s)**, con segundos de 6 lecturas. El firmware mide cuando `now - lastSampleAtMs >= 200` y luego hace `lastSampleAtMs = now`. Con este código el periodo **nunca puede ser menor que 200 ms** (más el tiempo de `pulseIn` lo alarga), así que el máximo teórico es 5.0 lecturas/s. El valor reportado no es posible con el código entregado.
  - **RNF3 (latencia):** la latencia se calcula restando *timestamps* del sensor, del servidor y del actuador (`timestamp_sensor_ms`, `timestamp_servidor_rx_ms`, `timestamp_actuador_aplica_ms`). Son **tres relojes independientes y no sincronizados** (`millis()` de cada ESP32 y el reloj de la PC). Restarlos no da una latencia. Además, los *timestamps* del sensor son múltiplos exactos de 2200 ms (12200, 14400, 16600…), algo poco verosímil en una prueba física manual. El informe no explica cómo se sincronizaron los relojes.
  - **RNF1 (estabilidad):** el CSV tiene una fila por minuto con "Operación normal" y ceros. No hay log del servidor ni del monitor serie que lo respalde.
  - **RF4 y RF6:** los CSV describen pasos del protocolo con resultados esperados y obtenidos idénticos, sin *timestamps* ni capturas reales.
- El propio informe se distancia de los datos: dice que los CSV son registros "**según confirmación del equipo**", que "esta revisión no reprodujo el montaje ni los ensayos físicos" y que el documento "no afirma que exista un montaje probado". Un informe técnico no puede validar requisitos con datos que sus mismos autores no respaldan.
- **No hay evidencias primarias** (logs crudos, fotos, videos) que permitan verificar los CSV.
- No hay análisis de errores: las 6 lecturas OUT_OF_RANGE a 2 cm se mencionan, pero "el archivo no identifica su causa".

Las pruebas automatizadas del software son correctas. La validación de los RNF con el hardware real, que es lo que exige el nivel 4, no se puede dar por demostrada.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay resultados numéricos (error máximo de 0.66 cm, latencia máxima de 72 ms, 5.117 lecturas/s, 600 s de estabilidad) y una tabla de compilación con el uso de RAM y flash.
- Se reconoce que la validez de la lectura y la exactitud son métricas distintas.

**Por qué baja a 2**
- Los resultados dependen de los CSV observados en §3.3. Al menos dos de ellos (RNF3 y RNF4) son técnicamente inconsistentes con el diseño, así que las conclusiones que se apoyan en esos números **no están fundamentadas**.
- **No hay análisis crítico:** no se discute por qué la latencia sería de 40–72 ms, qué la compone ni cuál es la incertidumbre de la medición.
- Las **recomendaciones son generales** ("conservar logs crudos, fotografías o vídeos", "mantener el divisor de tensión", "completar Unity en placa"). No proponen mejoras técnicas al sistema.
- Los anexos son enlaces a documentos de procedimiento. No hay fotos del prototipo ni evidencias de la ejecución.

---

## 4. Recomendaciones para próximas entregas

1. **Identificar a los integrantes** en la portada del informe e indicar el aporte de cada uno.
2. **Medir la latencia con un solo reloj.** Por ejemplo, en el servidor: el tiempo desde que llega el `DISTANCE` que cambia el rango hasta que llega el `ACK` del actuador. O bien filmar sensor y LEDs a la vez. Nunca restar *timestamps* de dispositivos con relojes no sincronizados.
3. **Verificar que los resultados sean coherentes con el código:** con un periodo mínimo de 200 ms no pueden obtenerse más de 5 lecturas/s. Si un dato contradice el diseño, hay que explicarlo o repetir la prueba.
4. **Guardar evidencia primaria:** el log del servidor con *timestamps* y la captura del monitor serie de cada prueba, a partir de los cuales se generan los CSV. Agregar fotos del montaje y videos cortos.
5. **Documentar el código:** comentar cada clase y método público, y explicar lo no obvio (temporización, *framing*, reconexión).
6. **Trabajar de forma incremental con Git** (commits locales con `git commit`, ramas por componente) en lugar de subir los archivos uno por uno desde la web el último día.
7. Redactar el informe en primera persona del equipo, afirmando lo que se hizo y se midió. Evitar frases como "según confirmación del equipo" o "este documento no afirma que exista un montaje probado".

---

## 5. Aspectos a tomar en cuenta para la defensa individual

El informe muestra rasgos de haber sido redactado con asistencia de IA sin una revisión final del equipo (se refiere a "esta revisión", "las fuentes disponibles" y "confirmación del equipo"). En la defensa se verificará que cada integrante conozca el sistema. Se recomienda preparar:

- **Pruebas físicas:** cómo se tomó cada dato de los CSV, con qué instrumento, en qué fecha y quién lo hizo. En particular:
  - cómo se obtuvieron 307 lecturas en 60 s si el periodo mínimo del firmware es de 200 ms;
  - cómo se sincronizaron los relojes del sensor, del servidor y del actuador para calcular la latencia de RNF3.
- **Protocolo:** el flujo `REGISTER` → `OK` → `DISTANCE` → `SET` → `ACK`; por qué el servidor republica el estado cada 200 ms; qué pasa si el sensor deja de enviar durante 1 s; diferencia entre `ERROR INVALID_DISTANCE` y `SET INVALID`.
- **Servidor:** por qué hay un `RLock` y un *lock* de envío separado; qué pasa si llega un segundo cliente con el mismo rol.
- **Firmware:** cómo funcionan `__has_include` y `kUsingExampleWiFiSecrets`; cómo se reconecta el WiFi sin bloquear; por qué se divide entre 58.0 para obtener cm.
- **Circuito:** cálculo del divisor 1 kΩ / 2 kΩ (tensión resultante en GPIO19) y corriente de los LEDs con 220 Ω.
- **Aporte individual:** Pablo Raya tiene 46 commits, en su mayoría subidas de archivos y documentación por la web. Josue Camacho tiene 8, del actuador. Lucas Rodriguez tiene 5, del servidor, el informe inicial y los "últimos cambios". Cada integrante debe poder explicar el componente de los demás, y en especial el **servidor TCP**, que se incorporó prácticamente completo en un solo commit (`c91c2a5`).
