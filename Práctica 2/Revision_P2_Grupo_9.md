# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/ciriarte-dev/IoT-practice-2-smart-objects-integration-over-TCP-IP-with-ESP32
**Informe:** `docs/informe.md`
**Integrantes (según el informe):** Carlos Eduardo Iriarte Rodriguez · Isabella Catherine Montes Iñiguez · Henry Alejandro Rivera Mendez
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
| Último push a `main` | 13-Oct 17:10 (UTC-4), commit `0662921` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Carlos Iriarte (dos cuentas): 11 · Henry Rivera: 6 · Isabella Montes: 4 |

- Se evaluó el commit **`0662921`**.
- El trabajo se organizó con ramas y *pull requests* por componente (`feature/ultrasonico`, `feature/server`, `LEDS`, `feature/integracion`, `docs/...`), y el código se desarrolló de forma incremental. Es una buena práctica.
- Las fotografías y los esquemas están dentro del repositorio (`docs/images/`).
- Carlos Iriarte hace commits con dos identidades (`Carlos Eduardo Iriarte Rodriguez` y `ciriarte-dev`), con el mismo correo. No es un problema, pero conviene unificarlas.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **13 RF concretos y medibles**, con valores explícitos (500 ms de muestreo, rango válido de 2–400 cm, 2 s y 3 s de seguridad, reconexión cada 2 s) y el componente responsable de cada uno.
- Rangos contiguos y sin solapamiento (`< 20`, `20 a 40 inclusive`, `> 40`), con los límites asignados. RNF-04 exige que la clasificación respete los umbrales sin errores de redondeo (19.99 / 40.01).
- RNF de seguridad física (LEDs apagados ante cualquier falla), no bloqueo, robustez de *framing* (128 bytes), confidencialidad de credenciales y verificabilidad sin hardware.
- **Diagramas completos y claros:** arquitectura (despliegue) con clases por nodo y GPIO, circuitos de los dos objetos como imágenes (con **divisor de tensión en ECHO** y GPIO 35 como entrada), diagrama de clases, diagrama de secuencia y **máquinas de estado** de los dos clientes ESP32 (Anexo B).
- **Protocolo bien especificado:** transporte, formato, mensajes, secuencia obligatoria, errores, comportamiento de los clientes y temporización.

**Por qué no llega a 4**
- **Faltan los RNF que exige la consigna (§3.2).** No se declaran valores medibles de **estabilidad** (tiempo mínimo de operación continua), **exactitud de medición** frente a una referencia física (RNF-04 trata de la exactitud de la *clasificación*, no de la *medición*), **tiempo de respuesta** de extremo a extremo ni **frecuencia de muestreo** como atributo verificable. RF-01 fija 500 ms (exactamente 2 lecturas/s, el mínimo de referencia, sin margen), pero no se declara como RNF ni se verifica.
- No hay **trazabilidad** de requisito → prueba.
- El divisor de ECHO aparece como "R1/R2" sin sus valores ni el cálculo de la tensión resultante.
- El LED amarillo usa **GPIO 2**, conectado al LED azul de la placa, y el verde usa **GPIO 15**, un pin de arranque (*strapping*). El primer problema se detectó y se recomienda corregirlo (Recomendación 1), pero debió evitarse en el diseño.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Código orientado a objetos y modular: `UltrasonicSensor`, `SensorClient` y `SensorConfig`; `Led`, `CommandParser`, `ActuatorManager`, `ActuatorClient` y `ActuatorConfig`; en el servidor, `controller.py`, `protocol.py` y `tcp_server.py`.
- **Servidor con `selectors`** (E/S multiplexada, sin hilos) y clientes ESP32 con envío no bloqueante. La elección está justificada en las conclusiones.
- Máquinas de estado explícitas en los clientes, muestreo con `millis()` resistente al desbordamiento y credenciales en `secrets.h` fuera del repositorio (`secrets.example.h`).
- **Pruebas automatizadas** del servidor (`test_tcp_server.py`, 449 líneas) y de los dos firmwares con *fakes* de Arduino, WiFi y sockets.
- El informe describe cada componente (§3.3–§3.5).

**Por qué no llega a 4**
- **Lógica de conexión duplicada** entre `SensorClient` y `ActuatorClient` (WiFi, TCP, identificación). El propio informe lo reconoce (Recomendación 4).
- Los umbrales de 20 y 40 cm y el rango de 2–400 cm **no están definidos como constantes** (Recomendación 5 del propio informe).
- La documentación del código es básica: `Led.cpp` tiene 1 comentario y `CommandParser.cpp`, `ActuatorManager.cpp` y `UltrasonicSensor.cpp` tienen 2 cada uno. Los comentarios están en español y los identificadores en inglés.
- El informe **no reporta la ejecución de las pruebas automatizadas** (cuántas hay ni el resultado), aunque RNF-08 las declara como requisito.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Prueba de integración del servidor real con clientes simulados, con el **log de consola** incluido.
- Mediciones de exactitud en 8 distancias (5–100 cm), **repetibilidad en las fronteras** de 20 y 40 cm (3 intentos) y casos de lectura inválida, incluido el alcance máximo observado (330.97 cm).

**Por qué baja a 2**
- **La validación del sistema completo se pospone a la demostración** (§4.4 y §5.4): "el funcionamiento [...] se verifica en la demostración". La tabla de §4.4 solo tiene "resultado esperado", sin resultado obtenido. El informe debe documentar las pruebas ya realizadas; la demo se califica por separado.
- **Ningún atributo temporal se midió:** no hay prueba de estabilidad (≥ 10 min), ni de tiempo de respuesta sensor → servidor → actuador, ni de la frecuencia de muestreo real, ni de los plazos de seguridad (2 s y 3 s) ni de la reconexión.
- **Las mediciones de exactitud se hicieron "con una versión anterior del firmware del sensor"**, con una sola lectura por distancia (salvo las fronteras). No se validó la exactitud con el firmware entregado.
- La prueba de integración simulada tiene solo 4 mensajes y no incluye valores en las fronteras exactas (20.00 / 40.00), ni mensajes inválidos, fragmentados o agrupados (RF-12 y RNF-03).
- No se reporta la ejecución de las pruebas automatizadas del servidor y del firmware.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- **Conclusiones auténticas y bien razonadas**, escritas desde la experiencia del equipo: acordar el protocolo antes de programar, el *framing* necesario porque TCP no separa mensajes, el diseño pensando en la falla y la elección de `selectors` en lugar de hilos.
- **Recomendaciones técnicas basadas en problemas observados de verdad:** GPIO 2 enciende el LED azul de la placa; `SO_REUSEADDR` en Windows permite dos instancias en el mismo puerto (una observación fina y correcta); histéresis en los umbrales a partir de la variación medida; mediana de lecturas y compensación por temperatura; unificar el código de conexión; WiFiManager; MQTT si el sistema crece.
- Resultados de exactitud resumidos (error promedio de 0.13 cm, máximo de 0.30 cm, variación en las fronteras) y de compilación (RAM/flash).
- Fotografías del montaje y de los LEDs en distintas zonas.

**Por qué baja a 2**
- **Los resultados son limitados:** solo hay datos de exactitud (de un firmware anterior) y de la simulación de 4 mensajes. No hay resultados del sistema integrado ni de ninguno de los atributos de estabilidad, tiempo de respuesta o frecuencia que pide la consigna.
- La conclusión 1 ("logramos que dos ESP32 trabajen juntos") no tiene ningún resultado medido en el informe que la respalde, más allá de las fotografías.
- Falta un análisis crítico de los límites: por ejemplo, qué pasa con el muestreo de exactamente 2 lecturas/s si `pulseIn` se demora, o por qué el error relativo es mayor a 10 cm.

---

## 4. Recomendaciones para próximas entregas

1. **Declarar los cuatro RNF de la consigna con valores medibles** (estabilidad, exactitud frente a una cinta métrica, tiempo de respuesta de extremo a extremo y frecuencia de muestreo) y **verificarlos con datos** en la sección de pruebas.
2. **No posponer pruebas a la demostración.** El informe debe documentar las pruebas del sistema completo que ya se hicieron, con sus resultados y evidencias.
3. **Medir los tiempos:** por ejemplo, con el log del servidor (`time` de cada `DIST` y de cada `LED`) y con un video en cámara lenta para el tiempo físico de extremo a extremo; contar las lecturas por minuto; cronometrar los plazos de 2 s y 3 s y la reconexión.
4. Repetir las mediciones de exactitud **con el firmware final** y con varias lecturas por punto.
5. Agregar una **matriz de trazabilidad** requisito → prueba → resultado y reportar la ejecución de las pruebas automatizadas (cantidad y resultado).
6. Indicar los valores del divisor de ECHO y el cálculo de la tensión; evitar los pines GPIO 2 y GPIO 15 para los actuadores.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Servidor con `selectors`:** cómo funciona la E/S multiplexada, en qué se diferencia de usar un hilo por cliente y cómo se evita que un cliente lento afecte al otro.
- ***Framing* y protocolo:** cómo se reconstruyen los mensajes partidos o agrupados; qué pasa con un mensaje de más de 128 bytes; secuencia `HELLO` → `OK` → `DIST`/`INVALID` → `LED ...`; por qué el servidor repite la orden cada 500 ms.
- **Seguridad ante fallas:** cómo se implementan los plazos de 2 s (servidor) y 3 s (actuador), y qué pasa en cada estado de la máquina de estados del Anexo B.
- **Circuito:** valores de R1/R2 del divisor y la tensión resultante; por qué GPIO 35 sirve para ECHO (solo entrada); qué problema causan GPIO 2 y GPIO 15.
- **Mediciones:** con qué versión del firmware se midió la exactitud y por qué no se repitió con la versión final; cuál es la frecuencia de muestreo real si el periodo es de 500 ms y `pulseIn` puede tardar hasta su *timeout*.
- **`SO_REUSEADDR` en Windows:** explicar el problema detectado y su solución.
- **Pruebas con *fakes*:** cómo se prueban `SensorClient` y `ActuatorClient` sin hardware (`run_tests.py`, `fakes/Arduino.h`, `fakes/WiFi.h`).
- **Aporte individual:** según el historial, Henry Rivera desarrolló el sensor, el servidor y la integración; Isabella Montes la clase `Led`, el *parser* de comandos, `ActuatorManager` y el cliente del actuador; Carlos Iriarte los circuitos, las fotografías y el informe. Carlos debe estar preparado para explicar el código del servidor y del firmware, e Isabella y Henry para explicar los diagramas y el protocolo.
