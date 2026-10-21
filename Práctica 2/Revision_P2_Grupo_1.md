# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/Andres004/SecondProject-IoT
**Informe:** `Informe.md`
**Integrantes (según el informe):** Ricardo Andres Andrade Vargas · Dabner Eliezer Orozco Veizaga
**Fecha límite:** martes 13-Oct-2026
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
| Último push a `main` | 13-Oct 17:43 (UTC-4), commit `b3dfa23` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor | dahbner: 24 · Ricardo Andrade / Andres004 / `unknown` (mismo correo): 12 |

- Se evaluó el commit **`b3dfa23`**, el último publicado antes del cierre.
- Los anexos (videos, fotos, CSV, logs y salidas de los scripts) **están dentro del repositorio**, en `docs/evidencias/<ID>/`, como pide §6.
- Hay 5 commits del servidor con autor **`unknown`**: Git no tenía configurado el nombre de usuario. El correo coincide con el de Ricardo Andrade, pero para la defensa individual conviene que cada integrante tenga su identidad bien configurada (`git config user.name`).
- Casi toda la evidencia de pruebas (A01 a B07) se subió el mismo 13-Oct, entre las 11:53 y las 17:42. El trabajo está completo, pero se concentró en el último día.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Requerimientos completos y medibles.** RF1–RF4 cubren la consigna, y RF5–RF7 (histéresis, lecturas inválidas, falla segura y reconexión) agregan robustez. Cada RF tiene un criterio de aceptación verificable (por ejemplo: "desde NEAR, 21.9 cm mantiene NEAR y 22.0 cm cambia a MEDIUM").
- **Rangos bien definidos y demostrados.** La Tabla 1 define cuatro estados (NEAR, MEDIUM, FAR y ERROR), con los límites asignados de forma explícita, y se demuestra formalmente que los intervalos son contiguos y no se solapan (§1.3). El estado **ERROR** tiene un patrón propio (los tres LEDs parpadeando), así que una falla no se confunde con "objeto lejos". Esto corrige justamente la debilidad observada en la Práctica 1.
- **RNF con valor, método de verificación y prueba asignada** (RNF1–RNF5), y la justificación de cada valor declarado. La **matriz de trazabilidad** (§1.6) une requisito → diseño → código → prueba → resultado.
- **Todos los diagramas exigidos**, y con buena calidad:
  - Arquitectura en dos vistas: red física 802.11 en modo infraestructura (Fig. 2a) y componentes con las conexiones TCP (Fig. 2b), más el recorrido de un mensaje por las cuatro capas TCP/IP.
  - Circuitos SVG de OI-1 y OI-2, con el cálculo del divisor de ECHO (3.33 V) y la corriente de cada LED, la justificación de los GPIO elegidos (no *strapping*, no flash) y una foto del montaje real.
  - Diagramas de clases de la librería compartida, de OI-1, de OI-2 y del servidor, que coinciden con el código.
  - Tres diagramas de secuencia (registro, ciclo de control, fallas y recuperación), con el *handshake* TCP y la asociación 802.11.
  - Máquina de estados con histéresis (Fig. 1).
- **Protocolo de aplicación completo** (§2.5): transporte, formato de cada mensaje, secuencia, números de secuencia, cuándo se envía un comando, códigos de error y una sesión real capturada. Distingue bien el ACK de TCP del `ACK` de aplicación.
- Justifica las decisiones de diseño (TCP frente a otras opciones, quién inicia la conexión, lógica en el servidor) y reconoce el costo: el servidor es un punto único de falla.

**Observaciones menores (no bajan el nivel)**
- El informe es **muy extenso** (≈ 1700 líneas). Para el lector, conviene un resumen ejecutivo de una página al inicio.
- La numeración "2.2.4 Montaje real" aparece con nivel `###` en lugar de `####`.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Código modular y orientado a objetos** en los tres programas: `UltrasonicSensor`, `LedIndicator`, `TcpLink` (librería compartida entre los dos firmwares), `EventLog`, `ZoneController` y `ControlServer`. Las constantes están en `SensorConfig`, `ActuatorConfig` y `config.py`; no hay números mágicos.
- **Diseño sin bloqueos:** la medición del eco usa una **interrupción** (`attachInterruptArg` + `IRAM_ATTR`, con atributos `volatile`), y la reconexión y el parpadeo usan `millis()`. El informe explica la temporización de cada lazo (§3.7).
- **Servidor concurrente y seguro:** un hilo por cliente, `threading.Lock` para el estado compartido, `TCP_NODELAY` (Nagle desactivado), *timeout* de 1 s en el socket del sensor y rechazo de mensajes de conexiones reemplazadas (`_is_current()`).
- **Framing correcto** por `\n`, con límite de 32 caracteres por línea y validación de cada mensaje.
- Credenciales fuera del repositorio (`Secrets.example.h` + `.gitignore`).
- Comentarios claros y consistentes, en inglés, en todos los archivos. El informe explica los fragmentos clave **con enlaces al código** (§3.6), sin copiarlo entero, como pide la consigna.
- Logs con `t_ms` en los tres nodos, que después usa `analyze_log.py` para calcular cada requisito: el código se diseñó pensando en cómo verificarlo.

**Observaciones menores**
- `parseSet()` usa `sscanf` y acepta texto adicional después del modo (`SET 5 RED ON xyz` se toma como válido). No afecta el funcionamiento, pero el protocolo podría ser más estricto en el actuador.
- `TcpLink::send()` puede bloquear `loop()` cuando el buffer de envío está lleno (1.66 s en A06). El propio informe lo reconoce y propone la solución en §7.

### 3.3 Pruebas y validaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Plan de pruebas completo** con 15 pruebas: 8 físicas (A01–A08) y 7 automáticas (B01–B07). Cada una indica requisito, procedimiento, criterio de éxito, resultado observado, estado y enlace a su evidencia.
- **Todos los RNF se midieron con instrumentos objetivos**, no a ojo:
  - RNF1: corrida de 12 min, 7202 lecturas, 0 reinicios, intervalo máximo 926.9 ms.
  - RNF2: 70 lecturas en 7 puntos (5–100 cm) con mínimo, máximo, promedio, error individual y error del promedio.
  - RNF3: 42 cambios de zona medidos con el reloj del servidor (`POST` → `SET` → `ACK`).
  - RNF4: lecturas por ventana de 10 s (10.0 lecturas/s).
  - RNF5: reconexión tras reiniciar el servidor (máx. 1927 ms).
- **Pruebas de frontera reales:** 2 cm, 20 cm, 40 cm, 200 cm, zona ciega y más allá del alcance. La histéresis se verifica con el objeto quieto en el umbral (0 `SET` durante 55 s).
- **Pruebas de falla:** cable ECHO desconectado, sensor desenchufado, punto de acceso apagado y reinicio del servidor.
- **Pruebas automáticas** del servidor (17 unitarias + 14 de sistema) y del firmware con un servidor de prueba (`client_check.py`).
- **Análisis de errores sobresaliente** (§4.4): identifica el error sistemático que crece con la distancia, explica las lecturas sueltas (la mano dentro del haz), atribuye con evidencia los retardos a la red WiFi y no al firmware, y documenta un defecto encontrado y corregido en la revisión (`_is_current()`).
- Honestidad en el reporte: RNF3 se declara **"no cumple" en 5 cambios con la red degradada**, en lugar de ocultarlo.

**Observaciones menores**
- El término de 100 ms de RNF3 (espera hasta la siguiente lectura) es una **cota calculada, no medida**. El informe lo reconoce (§5.2) y propone filmar a 60 fps.
- Todas las pruebas físicas se hicieron en una sola sesión y una sola red (13-Oct). Para RNF1, el margen fue de solo 73 ms en una única corrida.

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Tabla consolidada** (Tabla 11) con valor declarado, valor medido, prueba, tipo de evidencia y veredicto para cada requisito. Incluye gráficos del error frente a la distancia y de la latencia máxima por prueba.
- **Análisis crítico real:** separa lo que los datos permiten afirmar de lo que **no** permiten afirmar (exactitud por encima de 100 cm, margen de RNF1, término de 100 ms, una sola sesión). Es el tipo de análisis que pide el nivel 4.
- **Conclusiones fundamentadas en números** (10.0 lecturas/s, 42 cambios con 8.9–140.8 ms, ACK 0 en 1033–1187 ms, etc.), incluida una sobre lo aprendido del propio TCP (flujo de bytes, *framing*, ACK de transporte frente a ACK de aplicación, Nagle).
- **Recomendaciones técnicas, cada una vinculada a la prueba que la motiva** (calibrar `CM_PER_ECHO_US`, mediana de 3 lecturas, *heartbeat*/keepalive, envío no bloqueante, red dedicada), más una lista de limitaciones conocidas del diseño con su posible solución.
- **Anexos pertinentes y dentro del repositorio**, con un README por carpeta de evidencia.

**Observaciones menores**
- El error sistemático de A02 se diagnosticó, pero no se corrigió. Se recomienda calibrar, aunque no se hizo antes de la entrega. Con el margen de 0.4 cm a 100 cm, convenía aplicar la calibración y repetir A02.

---

## 4. Recomendaciones para próximas entregas

1. **Distribuir el trabajo de pruebas en el tiempo.** Toda la evidencia se generó el último día. Una falla de hardware o de red ese día habría dejado sin validar varios requisitos.
2. **Configurar la identidad de Git** en cada equipo (`user.name` y `user.email`) para que ningún commit aparezca como `unknown`.
3. **Cerrar el ciclo diagnóstico → corrección:** cuando el análisis de errores identifica una causa corregible (el factor de escala de A02), aplicar la corrección y repetir la prueba para mostrar la mejora.
4. Agregar un **resumen ejecutivo** de una página al inicio del informe (arquitectura, protocolo y tabla de cumplimiento), para que el lector no tenga que leer 1700 líneas para conocer el resultado.
5. Repetir las pruebas de tiempo (RNF1 y RNF3) en más de una sesión o en una red dedicada, para tener un margen estadístico y no un único valor.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

El informe tiene un nivel de detalle muy alto. En la defensa, **cada integrante debe poder explicar cualquier parte**, no solo la que escribió. Se recomienda preparar:

- **Protocolo y TCP:** por qué hace falta el *framing* con `\n` (TCP es un flujo de bytes), qué diferencia hay entre el ACK de TCP y el `ACK <seq>` de aplicación, qué hace el algoritmo de Nagle y por qué se desactivó, y qué pasa si llegan dos mensajes en el mismo segmento.
- **Medición del eco por interrupción:** por qué `IRAM_ATTR`, por qué los atributos son `volatile`, qué riesgo hay al leer variables compartidas con una ISR y por qué se eligió un *timeout* de 25 ms (≈ 4.3 m de ida y vuelta).
- **Algoritmo de control:** la Tabla 2 de histéresis (por qué 2 cm, cómo se justificó con A02/A04), qué pasa al salir de ERROR y por qué la tercera lectura inválida seguida llega a los 200 ms.
- **Servidor concurrente:** un hilo por cliente, qué protege el `Lock`, cómo se detecta la pérdida del sensor (*timeout* de 1 s) y qué corrige `_is_current()`.
- **Cálculo del tiempo de respuesta:** por qué se mide con el reloj del servidor y no con los ESP32, y qué parte de RNF3 es medida y qué parte es calculada.
- **Circuito:** cálculo del divisor de ECHO (5 V × 2/3 ≈ 3.33 V), corriente de cada LED y por qué el blanco brilla menos.
- **Análisis de errores:** cómo se concluyó que los retardos de 3.6 s y 5.7 s eran de la red WiFi y no del firmware.
- **Aporte individual:** la mayoría de los commits del firmware y de las pruebas son de dahbner, y los del servidor y el informe, de Ricardo Andrade. Cada uno debe poder explicar también la parte del otro (por ejemplo, quien hizo el servidor debe poder explicar `TcpLink` y la ISR, y viceversa).
