# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/JesusxAriel/Practica2IoTGrupo7
**Informe:** `README.md`
**Integrantes (según el informe):** Jesús Ariel Mamani Gutierrez · Leonardo Koller Mejia · Luis Felipe Velasco Lopez
**Fecha límite:** martes 13-Oct-2026
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **3** / 4 |
| **Total** | **12 / 16** |
| **Nota del componente** = (12 / 16) × 100 | **75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 18:38 (UTC-4), commit `c76e1a2` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Jesús Ariel Mamani: 6 · Luis Felipe Velasco: 2 · Leonardo Koller: 1 |

- Se evaluó el commit **`c76e1a2`**.
- **Las evidencias están fuera del repositorio:** todas las fotos, videos y capturas de Wireshark están enlazadas a Google Drive / `googleusercontent.com`. La carpeta `docs/evidencias/` solo contiene un `.gitkeep`. §6 de la consigna pide que los anexos estén **dentro del repositorio**. Además, los enlaces externos no tienen historial y no se puede comprobar qué contenían en la fecha límite.
- **Falta un archivo citado:** el informe enlaza en varias partes a `tools/analizar_log.py` (PT-06, §4.1 y Anexo C), pero **ese archivo no existe en el repositorio**.
- **Se versionó la carpeta `.pio/`** (318 archivos de compilación y de la biblioteca ArduinoJson). El `.gitignore` la excluye, pero se agregó antes de crear esa regla y nunca se eliminó del índice (`git rm -r --cached .pio`).
- El servidor completo (923 líneas, con las pruebas) entró en **un solo commit** de Leonardo Koller (`cd8bdde`), y el actuador en un commit de Luis Felipe Velasco. El historial no muestra cómo evolucionó cada componente.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- RF1–RF7 indican qué elemento de diseño los cubre. Los RNF tienen valor declarado y prueba asociada, y **se acota explícitamente el alcance** (por ejemplo, la exactitud solo se declara en 5–100 cm).
- Rangos contiguos con los límites asignados (15 y 40 cm pertenecen al rango superior), con un comportamiento distinto por rango (parpadeo de 150 ms, parpadeo de 500 ms y fijo).
- **Estado `FAULT` distinguible** (los tres LEDs parpadeando), separado de los rangos de distancia, con sus causas enumeradas. El tratamiento de las lecturas inválidas está bien razonado: una lectura aislada no activa `FAULT`.
- **Protocolo SDP bien especificado:** JSON por línea, límite de 256 bytes, campos comunes (`seq`, `ts`), tabla de mensajes, ejemplos, reglas (reemplazo de sesión, estado al reconectar, `PING/PONG`, *timeout* de 15 s) y códigos de error.
- El diagrama de secuencia incluye los caminos de falla (*watchdog*, *heartbeat*, caída del servidor con reintentos de espera creciente). Hay diagrama de clases y tabla de parámetros de configuración.

**Por qué no llega a 4**
- **Los "diagramas de circuito" son diagramas de flujo Mermaid** con líneas entre pines, no esquemáticos eléctricos.
- **ECHO a 5 V conectado directo a GPIO 18**, "sin resistencias ni divisor de tensión". El informe lo reconoce en la Recomendación 9, pero el diseño entregado no lo corrige.
- El rango `CLOSE` se define como `0 ≤ d < 15 cm`, pero las lecturas por debajo de 2 cm se consideran inválidas. El límite inferior real del rango es 2 cm.
- El diagrama de arquitectura y el de secuencia no muestran el punto de acceso (*hotspot*) como nodo de la red, aunque el texto lo describe.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Firmware orientado a objetos, con una clase por par `.h`/`.cpp`: `HCSR04Sensor` (mediana de 3 lecturas), `WiFiManager`, `TCPClientManager`, `SDPMessage`, `LedActuator` y `ActuatorController`.
- Servidor Python modular (`control_engine.py`, `protocol.py`, `tcp_server.py`, `event_logger.py`), con *watchdog*, reemplazo de sesión y registro CSV con marcas de tiempo.
- Reconexión con espera creciente, `PING/PONG` y parpadeo no bloqueante. Credenciales fuera del repositorio (`config.example.h`).
- El informe explica las decisiones de diseño (§3.4) y fragmentos clave (§3.3).

**Por qué no llega a 4**
- **Código duplicado:** `SDPMessage`, `TCPClientManager` y `WiFiManager` están copiados de forma idéntica en `esp32_sensor/` y `esp32_actuator/` (≈ 260 líneas duplicadas). El propio informe recomienda moverlos a `lib/`; era fácil hacerlo en esta entrega.
- **Documentación del código mínima:** la mayoría de los `.cpp` tienen entre 1 y 5 comentarios (`ActuatorController.cpp`, `LedActuator.cpp` y `WiFiManager.cpp` tienen 1 cada uno).
- Se versionaron artefactos de compilación (`.pio/`) y falta el script de análisis citado.
- PT-12 dice "11 pruebas ejecutadas", pero el resumen dice "14" y `test_integration.py` tiene 14 métodos de prueba. Hay que mantener la coherencia entre el informe y el código.

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Plan de 12 casos (PT-01 a PT-12)** con procedimiento, resultado esperado, criterio de aprobación, evidencia y resultado. Hay una tabla de trazabilidad requisito → prueba.
- Variedad de instrumentos: flexómetro, **Wireshark** para verificar el protocolo en la red real, registro del servidor, monitor serie y video en cámara lenta para el tiempo de respuesta.
- Pruebas de falla (desconectar ECHO, apagar el sensor, reiniciar el servidor) y de lecturas inválidas (ausencia breve y prolongada de eco, objeto a menos de 2 cm).
- Presupuesto de tiempo teórico para RNF3, presentado como estimación previa a la medición.

**Por qué no llega a 4**
- **Faltan los datos detallados:**
  - PT-02 promete "10 lecturas por cada distancia" en 9 distancias, pero solo informa el **error máximo** (1,03 cm). No hay una tabla por distancia.
  - PT-06 dice explícitamente que "los valores numéricos no se incluyen en este informe", y aun así se marca "Aprobado" sin un criterio numérico.
  - PT-07 informa solo el máximo (0,3 s) de 20 mediciones, sin tabularlas.
- La **evidencia está en Google Drive**, fuera del repositorio, y el script de análisis no está.
- La prueba de estabilidad (PT-08) tiene **10 min de funcionamiento normal exactos**, sin margen y sin un log que muestre la continuidad.

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Resultados medibles (error máximo de 1,03 cm, 3,05 lecturas/s, 0,3 s, reconexión ≈ 3 s, `FAULT` ≈ 3 s) con una tabla que indica el **alcance y las limitaciones** de cada uno.
- **Buen análisis crítico y honesto:** se distingue frecuencia de latencia, se reconoce que los tiempos son manuales y aproximados, se descarta como evidencia una "observación puntual no documentada", y se discute la diferencia de 3,1 cm a 60 cm en PT-01.
- **Recomendaciones técnicas pertinentes:** histéresis, distinguir "sin eco" de "sensor desconectado", compensar la temperatura, sincronizar relojes, biblioteca compartida, TLS y proteger ECHO.

**Por qué no llega a 4**
- Los resultados se presentan como valores resumen (máximos) sin los datos que los respaldan. En PT-06 el informe reconoce que no reporta los valores.
- **Anexos fuera del repositorio** y un anexo (C) que apunta a un archivo inexistente.
- La conclusión 1 afirma que "los 12 casos quedaron aprobados", pero PT-06 no tenía un criterio numérico.

---

## 4. Recomendaciones para próximas entregas

1. **Guardar todas las evidencias dentro del repositorio** (`docs/evidencias/PT-xx/`): fotos, videos cortos o comprimidos, capturas de Wireshark, logs CSV del servidor y la salida de las pruebas.
2. **Verificar antes de entregar que cada enlace del informe apunte a un archivo existente** (en esta entrega faltó `tools/analizar_log.py`).
3. **Tabular las mediciones individuales:** exactitud por distancia (mínimo, máximo, promedio y error máximo), las 20 mediciones de PT-07 y las latencias `DATA → SET_STATE_SENT` y `SET_STATE → ACK` del log.
4. Quitar `.pio/` del índice de Git (`git rm -r --cached .pio`) y verificar con `git status` que no se suban artefactos generados.
5. Mover el código común de los firmwares a `lib/` y documentar cada clase y método público.
6. Agregar el divisor de tensión en ECHO y presentar los circuitos como esquemáticos eléctricos.
7. Repartir los commits según el trabajo real de cada integrante: el historial es la evidencia de su aporte.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Protocolo SDP:** por qué se eligió JSON por línea, qué pasa con un mensaje de más de 256 bytes, cómo se usa `seq` para medir el RTT `SET_STATE → ACK`, para qué sirve `PING/PONG` y qué hace el servidor si llega un `INIT` con un `id` que ya tiene sesión.
- **Watchdog y `FAULT`:** cómo se llega a ≈ 3,5 s como máximo (3 s + 0,5 s), por qué una lectura inválida aislada no activa `FAULT` y en qué casos el actuador entra en `FAULT` por sí mismo.
- **Sensor:** por qué se toma la mediana de 3 lecturas, qué significa el *timeout* de 25 ms (≈ 4,3 m) y cómo afecta al periodo real (3,05 frente a 3,3 lecturas/s).
- **Wireshark:** cómo se identificaron los mensajes SDP en la captura, qué es *Follow TCP Stream* y qué se ve del *handshake* TCP.
- **Circuito:** por qué ECHO necesita un divisor de tensión y cómo se calcula; por qué se cambió el LED verde de GPIO 5 a GPIO 21 (*strapping pin*).
- **Medición del tiempo de respuesta:** cómo se midieron los 0,3 s en el video en cámara lenta (fps de la cámara, desde qué evento hasta cuál).
- **Aporte individual:** el informe asigna el servidor a Leonardo Koller, el sensor a Jesús Ariel Mamani y el actuador a Luis Felipe Velasco. En el historial, Leonardo tiene 1 commit (el servidor completo, 923 líneas), Luis 2 (el actuador y el README) y Jesús 6 (el sensor, el informe y la entrega final). **Cada integrante debe demostrar con solvencia su aporte** y poder explicar los módulos de los otros dos.
