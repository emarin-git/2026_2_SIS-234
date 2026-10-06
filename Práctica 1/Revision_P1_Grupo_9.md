# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/HenryRiveraM/PracticaUnoIOT
**Informe:** `Practica_1.md`
**Autor de commits:** Henry Alejandro Rivera Mendez (HenryRiveraM). Colaboradora agregada: Isabella12-bit, sin commits.
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **3** / 4 |
| **Total** | **12 / 16** |
| **Nota del componente** = (12 / 16) × 100 | **75,00** |

---

## 2. Estado del repositorio en la fecha límite

| Commit | Push (UTC-4) | Contenido |
|---|---|---|
| `0f87a7f` | 11-Sep 12:36 | Proyecto completo: código, informe, diagramas, pruebas de host, registros Excel y fotos |
| `4eeccb6` | **15-Sep 17:50 (5 min tarde)** | "Remove simulation mode": quita el modo simulación del firmware, del informe y de las pruebas |

- **Se evalúa `0f87a7f`**, el último estado dentro de plazo.
- El commit tardío **solo elimina el modo simulación** (`SIMULATION_MODE`, el entorno `[env:simulacion]` y `test_simulation.cpp`) y ajusta las menciones en el informe. **No cambia la calificación**: aunque se considerara, el contenido evaluado sería prácticamente el mismo.
- **Hay un solo autor de commits.** El informe dice "Autor: Henry Rivera Mendez" y **no tiene lista de integrantes**. Isabella12-bit fue agregada como colaboradora el 14-Sep, pero no tiene commits.
- Todos los anexos (diagramas, fotos, registro Excel y evidencia de software) están dentro del repositorio.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **RNF propios, medibles y más exigentes que la referencia:**
  - error ≤ 2 cm entre 5 y 100 cm;
  - tiempo de respuesta ≤ 300 ms;
  - 10 ± 1 intentos/s;
  - parpadeo de 4 Hz ± 5 % (38 a 42 ciclos en 10 s);
  - 10 minutos de estabilidad.

  Cada uno tiene **criterio de aceptación**, por ejemplo "diez intentos por punto" o "90 a 110 intentos en 10 s".
- **Rangos sin ambigüedad:** `< 20` rojo a 4 Hz, `20–40` amarillo fijo (límites incluidos) y `> 40` verde fijo, más un estado inválido con los tres LEDs apagados. Se aclara que la clasificación usa la distancia **sin redondear**.
- **Diagramas completos:**
  - arquitectura;
  - circuito (`circuito.svg`) y tabla de conexiones;
  - clases;
  - actividad.

  En los anexos también están los diagramas de **estados** de la adquisición (Idle / WaitRise / WaitFall) y de **secuencia**.
- **Divisor de ECHO diseñado y calculado** (5 V × 440 / 660 = 3,33 V), con la razón explicada.

**Por qué no llega a 4**
- **RF6 a RF10 no son requerimientos del sistema, sino de la entrega:** "entregar informe", "incluir diagramas", "documentar", "probar", "presentar resultados". Se mezcla qué debe hacer el objeto inteligente con qué debe contener el informe.
- **El esquema de circuito es poco legible y está marcado como "previsto":**
  - las líneas se cruzan sobre las resistencias (la línea de TRIG atraviesa las tres resistencias de los LEDs);
  - las etiquetas "5V / VIN" y "GND" se superponen;
  - el título dice "Esquema de conexiones **previsto**" y la nota pide "confirmar… antes de conectar".

  No queda claro que represente el montaje real.
- **El divisor usa resistencias de 220 Ω** (220 + 440 = 660 Ω), lo que exige ≈ 7,6 mA a la salida ECHO. Funciona en lo ideal, pero es una carga alta para esa salida; lo habitual son valores de kΩ (por ejemplo 1 kΩ / 2 kΩ). No se justifica la elección.
- **Pines de *strapping* sin análisis:** el LED amarillo está en **GPIO2** (que además comparte el LED azul integrado de la DevKit V1) y el verde en **GPIO15**. Los dos intervienen en el arranque del ESP32 y no se discute el riesgo.
- **No hay matriz de trazabilidad:** la relación RNF → P08 a P12 se deduce, pero no está explícita.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Adquisición completamente no bloqueante:**
  - `Ultrasonic::poll()` implementa una **máquina de estados** (Idle → WaitRise → WaitFall) **sin `pulseIn`**;
  - lanza un disparo cada 100 ms y tiene *timeout* de 30 ms en ambas esperas;
  - **detecta un ECHO que quedó en alto** antes de un disparo nuevo;
  - devuelve `true` una sola vez por resultado.

  Es la implementación de adquisición más sólida de los grupos revisados.
- **`Led` con parpadeo no bloqueante bien pensado:**
  - recupera la fase si el ciclo se retrasa (`steps = (now − lastToggle_) / 125`, `lastToggle_ += steps × 125`), así que el periodo no se desvía;
  - funciona aunque `millis()` se desborde;
  - `blink()` no reinicia el parpadeo si ya estaba activo;
  - cuenta los ciclos completos.
- **Lectura con validez explícita** (`struct Reading { float cm; bool valid; }`), con validación de 2–400 cm y `isfinite()`.
- **Documentación en el código:** los encabezados tienen comentarios de contrato (`/** Devuelve true una sola vez por resultado… */`, `/** Inicia 4 Hz… sin reiniciar si ya parpadea */`) y `main.cpp` tiene comentarios en las funciones clave.
- **Pruebas automáticas en la PC** (`test/host/`) con un doble de `Arduino.h`. **Se ejecutaron en esta revisión y pasan:** rangos, invalidez, exclusividad, 40 ciclos en 10 s, *timeouts*, recuperación y desbordamiento.
- **El modo simulación estaba bien separado:** el monitor serie marcaba los mensajes como `SIMULACION` y no como `SENSOR`, para no confundir datos generados con mediciones reales.

**Observaciones (no cambian el nivel)**
- **Números mágicos:** `20`, `40`, `58.0f`, `100000U`, `30000U` y `125U` están escritos directamente en el código. Los pines tienen valores por defecto en el constructor de `Ultrasonic` (19, 18).
- **La clasificación está duplicada** en `applyReading()` y `stateLabel()`; si cambia un umbral, hay que modificarlo en dos lugares.
- `Led` solo admite 4 Hz (125 ms fijos); no se puede configurar la frecuencia.
- `main.cpp` tiene bastante lógica propia (ventana de conteo del parpadeo y reportes), además de coordinar.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- **Plan P01 a P13** con el resultado esperado de cada prueba, incluidos los límites exactos de 20 y 40 cm.
- **Pruebas automáticas reales y reproducibles** (ver §3.2), además de la compilación de los dos entornos.
- **Pruebas de frontera repetidas:** 3 intentos en 20 cm y 3 en 40 cm, con la lectura serie y el LED observado. Así se ve el comportamiento al cruzar el umbral (39,95 → amarillo, 40,05 → verde).
- **Lectura inválida probada en 3 condiciones**, incluido el paso de 330,97 cm (verde) a inválida.
- **Registro en Excel dentro del repositorio** y conteo automático del parpadeo por puerto serie.
- **Las limitaciones se informan con honestidad:** respuesta, frecuencia y estabilidad se marcan como "observacional" o "parcial", y no como cumplidas.

**Por qué baja a 2**
- **De los cuatro atributos no funcionales de la consigna, solo se verificó la exactitud:**
  - **Tiempo de respuesta (≤ 300 ms): no medido.**
  - **Frecuencia (10 ± 1 intentos/s): no medida.** El firmware ya imprime `t_ms` en cada lectura, así que bastaba contar líneas en 10 s.
  - **Estabilidad (10 min): "no se ejecutó corrida formal".**
- **La exactitud no siguió su propio criterio:** RNF1 exige "**diez intentos** en 5, 10, 20, 30, 40, 60 y 100 cm", pero la tabla tiene **una sola lectura por distancia**. El error máximo de 0,30 cm sale de 8 lecturas únicas.
- **El parpadeo se verificó con el contador del firmware, no físicamente.** `red.cycles()` cuenta los cambios de estado que hace el propio programa, así que por construcción siempre dará 40 en 10 s. Es útil como prueba de software, pero no muestra el comportamiento real del LED. El Excel dice que se aceptó un margen "por la dificultad de conteo visual manual", pero ese conteo visual no se hizo.
- **Hay anomalías sin analizar en las fronteras:**
  - En la tabla de exactitud, **20,10 cm se registró con LED rojo**, cuando la lógica indica amarillo para cualquier valor ≥ 20. El Excel lo reconoce ("la lectura serial indicaba rango amarillo, pero se observó rojo"), pero el informe solo dice "se repitió aparte".
  - En la repetición, **"20.00 cm" → rojo** se explica como "variación observada en frontera". La causa más probable es que el valor mostrado está **redondeado a 2 decimales** (por ejemplo, 19,996 se muestra como 20.00 y se clasifica como rojo). No se identifica.
- **Hay discrepancias entre el registro y el informe:** en el Excel, la columna "Cumple" dice **"Parcialmente"** para todas las distancias, y en los casos de 20 y 40 cm está vacía. En el informe todo figura como "Cumple exactitud y rango".
- **Faltan fotos de los LEDs encendidos en cada rango:** solo hay una foto del sensor con la cinta y otra de un "montaje parcial".

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Los resultados separan bien los tipos de evidencia:** software (host), compilación, observación manual y medición física, cada uno con su veredicto ("Si, para software", "Observacional", "Parcial por software").
- **Resultados medibles:** error máximo de 0,30 cm en 8 distancias de 5 a 100 cm, 40 ciclos en 3 ventanas de 10 s y lecturas de frontera con valores.
- **Conclusiones coherentes y honestas:** reconocen que el tiempo de respuesta y la estabilidad "quedaron como observaciones de funcionamiento y no como mediciones instrumentadas".
- **Anexos completos y organizados** en `docs/anexos/` con índice, más el diseño técnico y el plan de pruebas aparte.

**Por qué no llega a 4**
- **El análisis crítico no cubre las anomalías propias** (20,10 cm → rojo; redondeo en 20,00) ni el hecho de que el conteo de parpadeo es autorreferencial.
- **Recomendaciones casi solo de procedimiento:** masa común, revisar resistencias y polaridad, usar un objeto plano. La única técnica (histéresis o filtrado) se menciona y se descarta en la misma frase. Faltan, por ejemplo:
  - medir la frecuencia y el tiempo de respuesta con `t_ms`;
  - hacer la prueba de estabilidad;
  - revisar los valores del divisor;
  - evitar los pines de *strapping*;
  - centralizar los umbrales.
- **El repositorio tiene restos de texto generado por un agente y documentos desactualizados:**
  - el informe termina con "*El repositorio GitHub aún no se ha publicado desde esta sesión*" y usa frases como "*La placa confirmada por Henry*";
  - `verificacion-software.md` dice que los RNF "permanecen pendientes hasta que Henry ejecute el plan manual";
  - el `README.md` dice que "las pruebas físicas… se completarán al final".

---

## 4. Recomendaciones para próximas entregas

1. **Medir la frecuencia, el tiempo de respuesta y la estabilidad** con las marcas `t_ms` que el firmware ya imprime: contar líneas en 10 s, registrar el instante del cambio de zona y guardar un log de ≥ 10 minutos.
2. **Cumplir el criterio propio de exactitud** (10 intentos por distancia) e informar mínimo, máximo, promedio y error máximo individual.
3. **Verificar el parpadeo físicamente** (video con conteo de cuadros, o un fotodiodo o analizador lógico sobre el GPIO). El contador del firmware no basta.
4. **Investigar la anomalía de 20,10 cm → rojo** e imprimir la distancia con más decimales en las pruebas de frontera.
5. Separar los requerimientos del sistema (RF1 a RF5) de los requisitos de la entrega (RF6 a RF10) y agregar una matriz de trazabilidad.
6. Rehacer el esquema de circuito según el montaje real, sin cruces de líneas. Revisar el divisor (valores en kΩ) y mover los LEDs fuera de GPIO2 y GPIO15.
7. Centralizar umbrales, tiempos y constantes en `constexpr` y eliminar la clasificación duplicada.
8. Incluir la lista de integrantes y quitar el texto residual del agente y los documentos desactualizados.

---

## 5. Nota para la Defensa Individual

- **Hay un solo autor de commits y el informe no lista integrantes.** Si el grupo tiene más miembros (por ejemplo, Isabella12-bit, colaboradora sin commits), conviene comprobar el aporte de cada uno.
- El repositorio muestra un trabajo apoyado en agentes (BMAD: PRD, arquitectura y *specs*). Preguntas sugeridas:
  - ¿Cómo funciona la máquina de estados de `Ultrasonic::poll()` y por qué no usa `pulseIn`?
  - ¿Cómo recupera `Led::update()` la fase si el ciclo se retrasa?
  - ¿Por qué con 20,10 cm se observó el LED rojo? ¿Y por qué "20.00" puede quedar en rojo?
  - ¿Qué corriente circula por el divisor de 220 Ω y por qué no usar kΩ?
  - ¿Qué riesgo tiene usar GPIO2 y GPIO15 para los LEDs?
