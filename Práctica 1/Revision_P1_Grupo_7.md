# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/JesusxAriel/Grupo7_IoT_SensorUltraSonic
**Informe:** `doc/reporte-desarrollo.md`
**Autores de commits:** Jesus Ariel Mamani Gutierrez (JesusxAriel) · Luis Felipe Velasco Lopez (feliLuis) · Leonardo Koller (leoGKM)
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **2** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **2** / 4 |
| **Total** | **9 / 16** |
| **Nota del componente** = (9 / 16) × 100 | **56,25** |

---

## 2. Estado del repositorio en la fecha límite

Historial de *push* a `main` (registro público de eventos de GitHub, convertido a UTC-4):

| Commit | Push (UTC-4) | Autor | Contenido |
|---|---|---|---|
| `37c5fd4`, `da1a6f9` | 07-Sep 16:54 | Jesus Mamani / Luis Velasco | Commit inicial y explicación del código (`codigo.md`) |
| `b6d8917` (PR #1) | 07-Sep 19:12 | Jesus Mamani | Informe en `.md` |
| `ce728f7` | 08-Sep 00:46 | Luis Velasco | `diagrama-cableado.html` |
| `7e9960a` / `c9c2df2` | 11-Sep 12:01 | Jesus Mamani | Refactorización del código |
| `f179fbc`, `7928b60`, `c1c805f` | 15-Sep 15:02–15:14 | Jesus Mamani | Informe con casos de prueba, código explicado y README |
| `e0ee766` | **15-Sep 17:34** | Jesus Mamani | `reporte-desarrollo.html` |
| `c543316` | **15-Sep 18:26 (41 min tarde)** | Jesus Mamani | Colores del diagrama de cableado y línea de integrantes |

- **Se evalúa `e0ee766`** (17:34), el último estado dentro de plazo.
- El commit tardío `c543316` solo cambia la paleta y el texto de colores de `diagrama-cableado.html` y **agrega la línea "Integrantes: Mamani, Velasco y Koller"** al informe. **No modifica la calificación**: aunque se considerara, el contenido evaluado sería prácticamente el mismo.
- La rama `Reporte-desarrollo.html` (`fdb95a0`, Leonardo Koller, 08-Sep) **nunca se unió a `main`**. El único aporte de Koller visible en GitHub está en una rama aparte.
- Las evidencias fotográficas están en un **Google Docs externo**, fuera del repositorio. La consigna (§6) pide los anexos dentro del repositorio.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- RF bien definidos:
  - rangos contiguos con límites explícitos (Cerca `d ≤ 30`, Medio `30 < d ≤ 80`, Lejos `d > 80`);
  - comportamiento diferenciado: rojo a 6 Hz, naranja a 2 Hz y verde fijo;
  - estado de **error con los 3 LEDs parpadeando a 5 Hz** (RF4), en lugar de mostrar "zona libre" cuando falla el sensor.
- RF6 (no reiniciar el temporizador si el rango no cambia) muestra que entendieron el problema del parpadeo.
- Hay invariantes de diseño (AD-1 a AD-6), una máquina de estados y una tabla de pines.

**Por qué baja a 2**
- **Falta el diagrama estructural (clases) en el informe.** El informe no tiene diagrama de clases. Solo `doc/codigo.md` tiene un esquema de cajas `main → UltrasonicSensor / Semaforo → Led`, sin atributos, métodos ni tipos de relación. La consigna (§4.2) exige "diagramas estructurales y de comportamiento".
- **El diagrama de cableado contradice al informe.** `doc/diagrama-cableado.html` (en la versión entregada a tiempo) muestra **otros rangos y otros colores**: rojo `d ≤ 10 cm` a 4 Hz, amarillo `10 < d ≤ 25` fijo y naranja `d > 25` fijo. El informe y el código usan 30/80 cm, 6 Hz / 2 Hz / fijo. Además:
  - El HTML dibuja un **divisor de 1 kΩ / 2 kΩ en ECHO** y lo marca como "obligatorio", pero la tabla de pines del informe y el esquemático ASCII del Anexo A conectan **ECHO directo al GPIO26** y no lo mencionan.
  - El commit tardío corrige solo los colores del HTML, no los rangos ni la contradicción del divisor.
- **Los RNF no son medibles o no coinciden con el código:**
  - Estabilidad: "operación continua sin reinicios" **no tiene duración**.
  - Exactitud: dice "margen razonable (≤ ±3 cm)" pero no indica en qué rango de trabajo.
  - **Frecuencia de muestreo: se declaran ≥ 10 lecturas/s (100 ms) y AD-1 dice "~100 ms", pero `main.cpp` crea el sensor con un intervalo de 350 ms** (`UltrasonicSensor sensor(..., 350)`), es decir ≈ 2,9 lecturas/s. El propio código no cumple el RNF declarado.
- **Máquina de estados incompleta:** el diagrama ASCII no muestra las transiciones LEJOS → MEDIO, MEDIO → CERCA, ERROR → MEDIO ni ERROR → LEJOS, que el código sí permite.
- **Diagrama de arquitectura limitado:** es un ASCII solo de software. No muestra alimentación, GND, puerto serie ni PC.
- **No se analiza el riesgo de ECHO a 5 V** ni el uso de **GPIO12**, que es un pin de *strapping* del ESP32 (determina el voltaje de la flash al arrancar) y conviene evitarlo para salidas.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Buen diseño orientado a objetos:**
  - `Led` encapsula el pin y un modo `enum class` (Apagado / Sólido / Parpadeo) con parpadeo **no bloqueante** por `millis()`;
  - `Semaforo` **compone** tres objetos `Led` y contiene la máquina de estados;
  - `UltrasonicSensor` maneja su propia cadencia de muestreo con `millis()`;
  - `main.cpp` solo coordina.
- **`Led::blink()` no reinicia el parpadeo si se repite la misma orden:** solo lo reinicia si cambia el modo o la frecuencia. Además, `Semaforo::procesarDistancia()` sale antes si el rango no cambió. Los dos implementan RF6 correctamente.
- Pines en `namespace Pines` con `constexpr`, *timeout* de `pulseIn` de 25 ms y validación del rango 2–400 cm.
- `doc/codigo.md` explica la relación entre clases y documenta cada archivo.

**Por qué no llega a 4**
- **El código fuente casi no tiene comentarios:** los `.h` no tienen comentarios de clase ni de método y los `.cpp` solo tienen comentarios sueltos. La explicación está en `codigo.md`, no en el código.
- **La sección 3 del informe es muy corta:** solo enumera las clases en 4 viñetas, sin fragmentos de código, sin explicar el parpadeo no bloqueante y sin explicar la conversión.
- **El intervalo de muestreo contradice los requerimientos:** 350 ms en el código frente a los 100 ms declarados (ver §3.1).
- **Números mágicos:** los umbrales `30.0f` y `80.0f`, las frecuencias `6.0f`, `2.0f` y `5.0f`, la constante `0.0343f` y el *timeout* `25000` están escritos directamente en el código. Los límites `2.0f` y `400.0f` se repiten en `UltrasonicSensor.cpp` y `main.cpp`.
- **Nombres inconsistentes:** el informe habla de LED **naranja**, el código usa `amarillo_` y el HTML usa "amarillo" y "naranja" para LEDs distintos. Se mezclan `turnOn()`/`blink()` en inglés con `procesarDistancia()`/`realizarMedicion()` en español.
- `main.cpp` repite la decisión de error (`distancia < 0 || distancia < 2.0f || distancia > 400.0f`). Las tres condiciones son redundantes, porque el sensor ya devuelve −1 fuera de rango.
- `Led::update()` usa `ultimoCambioMs_ = tiempoActual` en lugar de sumar el intervalo, así que el periodo se va alargando poco a poco cuando `pulseIn` bloquea el ciclo.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- **16 casos de prueba físicos** con escenario, procedimiento, resultado esperado y observación. Cubren las tres zonas, los límites de 30 y 80 cm, el estado de error, la recuperación y los extremos < 2 cm, 2 cm y 400 cm.
- **Se reconocen las limitaciones:** la nota de TC-9 admite que **no se hizo la prueba de 10 minutos**, y se informa el uso de un LED de reemplazo.

**Por qué baja a 2**
- **Ninguno de los cuatro atributos no funcionales de la consigna se verificó con valores medidos:**
  - **Exactitud (±3 cm): no hay ninguna tabla que compare la distancia medida con la cinta métrica.** Las únicas lecturas informadas son "8,1 a 88,2 cm" (en movimiento) y "~232,8 cm".
  - **Frecuencia de muestreo: no se midió**, aunque el código difiere del valor declarado.
  - **Estabilidad: no se hizo** (lo reconoce TC-9).
  - **Tiempo de respuesta:** solo "poco menos de 1 s" en TC-12, sin método. Con un muestreo de 350 ms, debería ser ≲ 0,4 s; si realmente fue casi 1 s, eso merecía análisis.
- **Hay veredictos PASS que contradicen sus propios resultados:**
  - **TC-10:** a 6 Hz deberían verse **30 ciclos en 5 s**, pero se esperan "~20 (4–6 Hz)" y se observan ~20, es decir **4 Hz**. En el Google Docs el esperado decía "~4 Hz". El esperado se ajustó al resultado en lugar de investigar la diferencia con RF3.
  - **TC-11:** se esperaban 25 ciclos (5 Hz) y se registraron **20 (4 Hz)**, pero se informa "20 ciclos registrados en 5 s **(5 Hz)**", que es un error de cálculo.
  - **TC-15:** se esperaba una lectura válida a 400 cm y el sistema entró en error, pero se marca PASS.
- **Las observaciones son cualitativas** ("parpadea muy rápido"). Los límites de 30 y 80 cm se dan por probados "a 30 cm exactos" sin registrar la lectura del sensor, que es la que decide la zona.
- **TC-16 repite a TC-2**, y TC-6 se ejecutó con un LED distinto al especificado.
- **Las evidencias están fuera del repositorio** (Google Docs), así que no se puede verificar qué contenían en la fecha de entrega.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Las secciones existen y son coherentes en tono con el trabajo. Las recomendaciones incluyen un filtro de media móvil y una prueba formal de estabilidad.
- Se reconoce abiertamente que la prueba de 10 minutos no se hizo.

**Por qué baja a 2**
- **Resultados limitados y casi sin cifras:** tres viñetas sin tabla de exactitud, frecuencia, tiempo de respuesta ni estabilidad. El único dato numérico es la lectura de 232,8 cm.
- **Conclusiones sin sustento:**
  - "El sistema demostró ser **altamente reactivo**, registrando tiempos… por debajo de 1 segundo", cuando el único dato es "poco menos de 1 s", sin método.
  - "Permitió ejecutar el parpadeo a 2 Hz, 5 Hz y 6 Hz", cuando TC-10 y TC-11 midieron ≈ 4 Hz.
- **Recomendaciones pobres:**
  - Una es logística (conseguir el LED verde).
  - La de estabilidad justifica la prueba "para verificar la ausencia de desbordamiento en el contador de `millis()`", pero ese contador se desborda a los **~49,7 días**, no en 10 minutos.
  - No se recomienda nada sobre ECHO a 5 V, el intervalo de muestreo ni la diferencia de frecuencias.
- **Anexos:** el Anexo A es un esquemático ASCII **sin el divisor** que el HTML considera "obligatorio". El Anexo B remite a un Google Docs externo. El README dice que el informe incluye "pruebas de estabilidad", pero no se hicieron.
- **La lista de integrantes no está en el informe** entregado a tiempo; solo dice "Modalidad: Grupos de 2 o 3 integrantes".

---

## 4. Recomendaciones para próximas entregas

1. **Unificar la documentación:** una sola versión de rangos, frecuencias, colores y conexiones en el informe, el HTML de cableado, el README y el código. Decidir si hay divisor en ECHO (debería haberlo), implementarlo y dibujarlo en todos los diagramas.
2. **Agregar un diagrama de clases** (atributos, métodos, composición `Semaforo` ◆— `Led`) y completar la máquina de estados con todas las transiciones.
3. **Corregir el intervalo de muestreo** (350 → 100 ms) o el RNF, y **medirlo**: contar lecturas por segundo con marcas `millis()` en el puerto serie.
4. **Hacer la prueba de exactitud:** al menos 5 distancias con cinta métrica, varias lecturas por punto y el error absoluto en una tabla.
5. **Hacer la prueba de estabilidad** de ≥ 10 minutos y medir el tiempo de respuesta con un método explícito.
6. **No marcar PASS si el resultado no coincide con lo esperado.** Investigar por qué el parpadeo da ~4 Hz en lugar de 6 o 5 Hz; por ejemplo, contar con video o revisar `Led::update()`.
7. Guardar las evidencias (fotos y capturas) en `doc/` dentro del repositorio y enlazarlas en el informe.
8. Documentar las clases y métodos en los `.h`, centralizar umbrales y frecuencias en `constexpr` y usar un mismo idioma en los nombres.
9. Incluir los nombres completos de los integrantes en el informe.

---

## 5. Nota para la Defensa Individual

- **Jesus Ariel Mamani** concentra casi todos los commits en `main`. **Luis Felipe Velasco** aportó `codigo.md` y el HTML de cableado. **Leonardo Koller** solo tiene un commit en una rama que nunca se unió a `main`. Conviene comprobar el aporte de cada uno.
- Preguntas sugeridas:
  - ¿Por qué el código mide cada 350 ms si el requerimiento dice 100 ms?
  - ¿Por qué el HTML de cableado tiene otros rangos (10/25 cm) y un divisor que el informe no menciona? ¿El prototipo tiene divisor?
  - Si el rojo debe parpadear a 6 Hz, ¿cuántos ciclos deberían verse en 5 s y por qué contaron ~20?
  - ¿Cómo evita `Led::blink()` reiniciar el parpadeo en cada iteración?
  - ¿Qué riesgo tiene usar GPIO12 como salida en el ESP32?
