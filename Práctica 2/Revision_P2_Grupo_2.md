# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/JoseCorralesAgreda/Proyecto-Practica-2-IOT
**Informe:** `informe.md` (con detalle en `docs/*.md`)
**Integrantes (según el informe):** Bustamante Alejandro · Corrales Jose · Lopez Rasshid
**Fecha límite:** martes 13-Oct-2026
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **2** / 4 |
| **Total** | **11 / 16** |
| **Nota del componente** = (11 / 16) × 100 | **68,75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 16:48 (UTC-4), commit `617fa9c` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin contar *merges*) | Alejandro Bustamante: 5 · RashLop: 3 · Jose Corrales: 2 |

- Se evaluó el commit **`617fa9c`**.
- El trabajo se organizó con ramas y *pull requests* (`server`, `rash_sensor`, `alejandro_actuator`, `update-docs`), lo cual es una buena práctica.
- El repositorio incluye un `AGENTS.md` y una carpeta `plans/`: el desarrollo se hizo con un **agente de IA** guiado por planes. Esto no está prohibido, pero hace todavía más importante la defensa individual (ver §5).
- **No hay ninguna evidencia de las pruebas físicas en el repositorio** (fotos, videos, logs ni CSV). El informe dice expresamente que "no se requiere adjuntar fotografías ni evidencias adicionales", lo cual contradice §6 de la consigna, que pide incluir en el repositorio los *anexos (diagramas, esquemas, fotografías y evidencias de las pruebas)*.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- RF1–RF4 cumplen la consigna. Los rangos (`2 ≤ d < 50`, `50 ≤ d < 100`, `100 ≤ d ≤ 200`) son contiguos, no se solapan y tienen un comportamiento distinto cada uno.
- Hay un **manejo de errores** bien definido: tolerancia de dos lecturas inválidas, silencio del sensor `max(2 s, 3 × PERIOD)`, *watchdog* de 5 s en el actuador y reintento cada 3 s.
- Están todos los diagramas pedidos: arquitectura con la tabla de capas TCP/IP, circuito de cada objeto (Mermaid y esquemático en `docs/assets/circuits.html`), clases y secuencia.
- La **especificación del protocolo es muy buena**: gramática formal (estilo ABNF), longitud máxima de 32 bytes, terminador LF, estados `0/S/B` y una tabla de parámetros temporales. El mensaje `PERIOD`, con el que el servidor configura el muestreo del sensor, es una idea interesante.

**Por qué no llega a 4**
- **Error de diseño en el circuito del sensor:** el ECHO del HC-SR04 (5 V) va **directo** a GPIO 19, sin divisor ni adaptador de nivel. El informe lo justifica diciendo que "el equipo verificó experimentalmente que no daña componentes". Esa justificación no es válida: la tensión máxima absoluta de un GPIO del ESP32 es ≈ 3.6 V. Que la placa no falle en una prueba corta no demuestra que el diseño sea seguro, porque el daño puede ser acumulativo. Es una decisión que se debía corregir, no justificar.
- **RNF sin método de verificación ni trazabilidad:** la tabla de RNF solo tiene "objetivo" y "estado"; no dice cómo se mide cada uno ni con qué prueba se valida.
- **Diagrama de clases mínimo:** muestra solo los métodos públicos, sin atributos, sin `config` y sin los tipos de las relaciones (agregación o composición). Con este diagrama no se entiende cómo `SensorClient` usa a `UltrasonicSensor` ni dónde vive el estado.
- El diagrama de secuencia no muestra los caminos de falla (tercer `DIST ERR`, silencio del sensor, *watchdog* del actuador). Solo están descritos en texto.
- **Estado de error poco distinguible:** ante la falla, los LEDs se apagan (`000`). Para un observador, "todo apagado" se confunde con un actuador sin alimentación. Esta decisión debió analizarse en el diseño.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Código limpio, modular y orientado a objetos:** `DistanceController` (sin sockets, fácil de probar), `TcpServer`, `UltrasonicSensor`, `SensorClient`, `LedController` y `ActuatorClient`, con `.h`/`.cpp` separados y constantes centralizadas en `config.py` y `config.h`.
- El servidor usa `asyncio` y reconstruye líneas por LF. Valida la gramática, restringe el control al socket registrado como sensor y maneja el reemplazo de conexiones.
- **Parpadeo local que conserva la fase:** los refrescos del servidor no reinician el ciclo, y la aritmética con `millis()` tolera el desbordamiento.
- Credenciales fuera del repositorio (`config.h.example`), código formateado con `clang-format` y comentarios de documentación en inglés.
- El informe describe los fragmentos clave **con enlaces** al código, sin copiarlo.

**Observaciones menores**
- `UltrasonicSensor::readDistance()` usa `pulseIn`, que es bloqueante (hasta el *timeout*). Con un periodo de 250 ms no es grave, pero conviene mencionarlo.
- Las banderas `SENSOR_WIRING_CONFIRMED` y `LED_WIRING_CONFIRMED` son poco habituales. Si se olvidan en `false`, el firmware compila y no hace nada, y solo lo avisa por el monitor serie.
- Las compilaciones registradas son del 11-Oct con el perfil `esp32dev`, pero el proyecto final usa `esp32doit-devkit-v1`. La versión final no tiene registro de compilación.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- **18 pruebas automatizadas del servidor** con sockets TCP reales en *loopback*. Cubren los límites de cada rango (49.9, 50, 50.1, 99.9, 100, 100.1), mensajes fragmentados y concatenados, entradas inválidas, roles, reemplazo de conexiones, *timeouts* y cierre. Es un trabajo de buena calidad.
- Las tablas de prueba del sensor y del actuador tienen **procedimiento y resultado esperado** bien pensados (fragmentación, línea larga, silencio, tráfico inválido continuo).
- Se documenta un defecto encontrado y corregido (bloqueo al cerrar el *listener* en Python 3.14).

**Por qué baja a 2**
- **Las pruebas físicas no tienen datos.** Todas las filas dicen solo "Correcto". No hay ninguna medición:
  - **Exactitud (RNF2):** "Medir referencias de 2 a 200 cm → Correcto". No se indica qué distancias se usaron, cuántas lecturas, el valor medido ni el error.
  - **Tiempo de respuesta (RNF3):** "≤ 1 s → Correcto", sin ningún valor en ms ni el método de medición.
  - **Frecuencia (RNF4):** "contar `DIST` en ventanas de 10 s → Correcto", sin el conteo obtenido.
  - **Estabilidad (RNF1):** "≥ 10 min → Correcto", sin la duración real, la hora ni el log.
- **No hay evidencias** (fotos, videos, logs del servidor o del monitor serie). La afirmación "no se requiere adjuntar fotografías ni evidencias adicionales" va en contra de lo que pide la consigna.
- No hay pruebas de frontera con el sensor real (por ejemplo, el objeto colocado a 50 y 100 cm), solo con valores sintéticos en el servidor.
- No hay análisis de errores de medición.

Las pruebas de software son de nivel 4, pero los **valores declarados en los RNF no se verificaron con datos**, que es lo que exige el nivel 4 ("verifica [...] los valores declarados en los no funcionales"). Por eso la validación es parcial.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Las **conclusiones son técnicamente interesantes y bien redactadas**: parpadeo generado localmente para aislarlo del *jitter* de la red, la distinción entre `DIST ERR` (error de medición válido) y una línea malformada, la lógica concentrada en el servidor, y la lección de que compilar o pasar pruebas en *loopback* no demuestra el comportamiento físico.
- Las recomendaciones son técnicas: IP fija o reserva DHCP, prueba automatizada de latencia y TLS si hiciera falta confidencialidad.

**Por qué baja a 2**
- **Los únicos resultados cuantitativos son de software** (18/18 pruebas, RAM y flash de la compilación). Para el prototipo físico, la tabla de resultados solo dice "Cumplido" en cada métrica, sin ningún número. Un resultado "Cumplido" sin valor medido no se puede verificar.
- Las conclusiones no se apoyan en datos del sistema real, porque no hay ninguno. La propia conclusión 4 dice que la compilación y el *loopback* "no acreditan el comportamiento físico", pero después el informe no presenta la evidencia física.
- No hay análisis crítico de límites ni de fuentes de error (exactitud a distancias largas, latencia de la red, efecto de `pulseIn`).
- **Los anexos se limitan a enlaces a la documentación**; no hay fotografías del montaje ni evidencias de las pruebas.
- La recomendación de *no* agregar ACK ni *heartbeat* es defendible, pero la conclusión de que el refresco de 1 s "actúa como latido implícito" solo vale en un sentido (servidor → actuador). El servidor no tiene forma de saber si el actuador aplicó la orden.

---

## 4. Recomendaciones para próximas entregas

1. **Corregir el circuito del sensor:** poner un divisor de tensión (por ejemplo 1 kΩ / 2 kΩ) o un adaptador de nivel en ECHO. No justificar un diseño que excede los valores máximos del fabricante.
2. **Medir y registrar valores numéricos en cada prueba física:** tabla de distancia de referencia frente a distancia medida (con varias lecturas por punto), latencia en ms (por ejemplo, con *timestamps* en el log del servidor desde el `DIST` que cambia el rango hasta el envío del `LED`), conteo de lecturas por ventana y duración real de la prueba de estabilidad.
3. **Guardar las evidencias en el repositorio** (`docs/evidencias/`): logs del servidor, capturas del monitor serie, fotos del montaje y videos cortos de cada rango.
4. Agregar a la tabla de RNF una columna **"método de verificación"** y otra **"prueba"**, y una matriz de trazabilidad requisito → prueba → resultado.
5. Completar el diagrama de clases con los atributos y el tipo de cada relación, y agregar un diagrama de secuencia para los caminos de falla.
6. Elegir un patrón de error **distinguible** de "sin alimentación" (por ejemplo, los tres LEDs parpadeando).

---

## 5. Aspectos a tomar en cuenta para la defensa individual

El repositorio muestra que el trabajo se apoyó en un agente de IA (`AGENTS.md`, `plans/`). En la defensa, cada integrante debe demostrar que **entiende y puede justificar** lo que el agente produjo. Se recomienda preparar:

- **Circuito del sensor:** por qué el ECHO a 5 V no debe ir directo a un GPIO del ESP32, cuál es la tensión máxima admitida y cómo se calcula un divisor de tensión. Explicar la afirmación del informe de que "se verificó experimentalmente".
- **Protocolo:** leer y explicar la gramática (`dist = "DIST" SP ...`), por qué el límite de 32 bytes, qué pasa con un mensaje fragmentado o con dos mensajes en el mismo segmento, y para qué sirve `PERIOD`.
- **Servidor `asyncio`:** qué es un *event loop*, en qué se diferencia de usar un hilo por cliente, y cómo `_maintain_clients()` detecta el silencio del sensor.
- **Manejo de errores:** por qué `DIST ERR` renueva el plazo del sensor y una línea malformada no; qué pasa en el tercer error consecutivo; qué hace el *watchdog* de 5 s.
- **Parpadeo con preservación de fase:** cómo funciona la aritmética modular sobre `millis()` y por qué tolera el desbordamiento.
- **RNF:** cómo midieron realmente la exactitud, el tiempo de respuesta y la frecuencia de muestreo, con qué instrumento y qué valores obtuvieron. Estos datos **no están en el informe** y se pedirán en la defensa.
- **Aporte individual:** Jose Corrales tiene 2 commits (servidor inicial), RashLop 3 (sensor, dependencias y logs) y Alejandro Bustamante 5 (actuador y toda la documentación). Cada integrante debe poder explicar también las partes que no hizo.
