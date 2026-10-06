# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/Andres004/Ultrasonic_Led
**Informe:** `README.md` del proyecto
**Integrantes:** Ricardo Andres Andrade Vargas · Carlos Eduardo Iriarte Rodriguez · Dabner Eliezer Orozco Veizaga
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
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

Historial de *push* a `main` (registro público de eventos de GitHub, convertido a UTC-4):

| Commit | Push (UTC-4) | Autor | Contenido | ¿Se evalúa? |
|---|---|---|---|:---:|
| `6ed68b9` + `227cb1d` | 14-Sep 20:53 | Andres004 | Código modular (POO) y ajuste de la frecuencia de muestreo | Sí |
| `70913a4` | 15-Sep 17:18 | dahbner | `README.md` con el informe completo | Sí |
| `7861f70` (merge, incluye `e32908f`) | **15-Sep 17:49** | Andres004 | Comentarios en `UltrasonicSensor.h` y `UltrasonicSensor.cpp`; corrige `#endif.` | **No (4 min tarde)** |

- La revisión se hizo sobre el commit **`70913a4`**, que es lo que había en GitHub a las 17:45.
- El commit `e32908f` tiene fecha local 13:36, pero llegó al repositorio remoto recién con el *merge* de las 17:49. El commit `70913a4` se creó directamente sobre `227cb1d`, lo que confirma que `e32908f` no estaba en GitHub antes de la hora límite. Por eso **no se consideró la documentación de la clase `UltrasonicSensor`** agregada en ese commit.
- Los anexos (fotos, videos y tabla de pruebas) están en Google Drive y Google Sheets, **fuera del repositorio**. En §6 de la consigna se pide que estén dentro del repositorio. Se consultaron solo como apoyo, porque no tienen historial de versiones y no se puede comprobar qué contenían a la hora límite. Se revisaron el 05-Oct-2026.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- RF1–RF3 cumplen la consigna. Los tres rangos (`< 20`, `20 ≤ d < 40`, `≥ 40` cm) son contiguos, no se solapan y tienen un comportamiento distinto cada uno. Además se agrega **RF4** (exclusión mutua entre LEDs), que es una buena decisión.
- Los cuatro RNF tienen valores medibles (10 min, ±3 cm, ≤ 1 s, ≥ 2 lecturas/s) y cada uno indica con qué caso de prueba se verifica. Hay intención clara de trazabilidad.
- El **diagrama de arquitectura** (Mermaid) muestra alimentación, sensor, lógica, actuadores, UART y GND común.
- El **diagrama de circuito** es claro y correcto. Tiene leyenda de colores, resistencias de 220 Ω y un divisor de 3 × 1 kΩ en ECHO (5 V × 2/3 ≈ 3,33 V, nivel adecuado para el ESP32). La tabla de pines coincide con el código (27/26/25/33/32).
- El **diagrama de clases** coincide con el código (`Led`, `UltrasonicSensor`, composición desde `main.cpp`).

**Por qué no llega a 4**
- **Trazabilidad con errores:** según RNF2, la exactitud se verifica con *TC-1 a TC-3*, pero en §4.2.1 la prueba de exactitud es **TC-10** (20, 50, 100 y 200 cm). Además, RNF2 declara un rango de trabajo de **2 a 300 cm**, pero ninguna prueba planificada pasa de 200 cm ni baja de 10 cm.
- **No se muestra la secuencia de ejecución.** El diagrama de estados solo muestra las transiciones entre zonas. No aparece el ciclo `loop()` (medir → imprimir → clasificar → actuar → `delay(60)`) ni el subestado de parpadeo ON/OFF que maneja `millis()`. Tampoco hay transiciones de permanencia en la misma zona. Para el nivel 4 se pide que los diagramas permitan comprender *totalmente el sistema y su secuencia de ejecución*; faltaba un diagrama de secuencia o de actividad.
- **Decisión de diseño no analizada:** si no hay eco (sensor desconectado, objeto a menos de 2 cm o una superficie que absorbe el sonido), se devuelve 400 cm y el sistema pasa a la zona **"libre" (blanco)**. Es decir, una falla del sensor se ve como "sin peligro". Este riesgo debió discutirse en el diseño.
- Detalles de forma: "diagrama **dee** estados"; la tabla de pines dice `VIN (5V)` y el texto dice "pin de 5V".

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- El código es **modular y orientado a objetos**: clases `Led` y `UltrasonicSensor`, cada una con encapsulamiento (`private`) y archivos `.h`/`.cpp` separados. `main.cpp` funciona como orquestador.
- `Led::blink()` es **no bloqueante** (usa `millis()`), con la resta `now - lastToggleTime`, que funciona bien aunque `millis()` se desborde. Es una buena práctica.
- `pulseIn(..., 30000)` evita que el programa se bloquee cuando no hay eco. Antes de cada nueva medición hay una pausa de 60 ms.
- `Led.h`, `Led.cpp` y `main.cpp` tienen comentarios claros, y los comentarios de `main.cpp` hacen referencia a los RF.

**Por qué no llega a 4**
- **Documentación incompleta a la hora límite:** `UltrasonicSensor.h` no tenía ningún comentario y `UltrasonicSensor.cpp` tenía comentarios mínimos, en **español**, mientras que el resto del código está en inglés. La documentación de esta clase llegó en el commit de las 17:49, fuera de plazo.
- **Error de sintaxis en el preprocesador:** `#endif.` en `UltrasonicSensor.h:15`. Compila, pero genera la advertencia *extra tokens at end of #endif directive*.
- **Números mágicos:** los pines, los umbrales (20.0 y 40.0), los intervalos (100, 500 y 60 ms), el valor 400.0 y la constante 0.0343 están escritos directamente en el código, sin `constexpr` ni `#define` con nombre. Si se cambia un umbral, hay que tocar varios lugares y el informe deja de coincidir con el código.
- Condición redundante: `distance >= 20.0 &&` en la rama `else if`, que no es necesaria.
- `blink(int pauseTime)`: el nombre no aclara que es medio periodo. El comentario `// RF1: Sensor and actuator initialization` asigna a RF1 la inicialización de los actuadores, que no corresponde.
- En el informe (§3): "**Arduino IDE (PlatformIO)**" es incorrecto. El proyecto usa **PlatformIO** con el *framework* Arduino (ver `platformio.ini`), no el Arduino IDE. La sección §3 no incluye fragmentos de código ni enlaces a los archivos fuente.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El plan está bien pensado. Tiene 10 casos de prueba, con escenario, resultado esperado y requerimiento que cubre cada uno. Incluye **pruebas de frontera** (TC-4 a 20 cm y TC-5 a 40 cm), barridos (TC-6) y 3 repeticiones por cada caso de distancia.
- Explica la estrategia y los instrumentos (cinta métrica, cronómetro, monitor serie).

**Por qué baja a 2**
- **El informe no documenta los resultados de las pruebas.** Las tablas de §4 solo tienen la columna *Resultado esperado*. Los resultados se envían a un "**Anexo [N]**", que quedó como marcador sin completar, y a una hoja externa que no está en el repositorio.
- **Validación de RNF incompleta:**
  - **RNF3 (≤ 1 s):** no hay ninguna medición en milisegundos. En la hoja de anexos se registra como "*sin cronometraje exacto*, Éxito (cualitativo)". Además, medir ≤ 1 s con cronómetro manual no es suficiente, porque el tiempo de reacción humano (~200–300 ms) es del mismo orden que lo que se quiere medir.
  - **RNF4 (≥ 2 lecturas/s):** el método planificado (TC-8, contar líneas `Distance:` en el monitor serie) no aparece en el informe. En su lugar, §5 deduce la frecuencia de muestreo a partir de los **ciclos de parpadeo** (17 en 5 s), lo cual no es válido (ver §3.4). La hoja de anexos registra 19,7 lecturas en 10 s, es decir **≈ 1,97 lecturas/s, por debajo del mínimo de 2**, y aun así marca "Éxito" con la nota "*sin conteo exacto*".
  - **RNF1 (estabilidad):** la hoja de anexos registra la prueba de 15 min como "**(Simulado)** 15 min operando, 0 reinicios". Si la prueba fue simulada, el requerimiento no se verificó en hardware.
  - **TC-10 (exactitud a 20/50/100/200 cm) no se ejecutó.** Los datos de exactitud que se reportan son de 10, 20, 30, 40 y 60 cm (TC-1 a TC-5). El rango declarado de 2 a 300 cm queda sin validar por encima de 60 cm.
- **Las pruebas de frontera no prueban realmente la frontera:** en TC-4 todas las lecturas quedaron por encima de 20 cm (20,15–20,61) y en TC-5 por encima de 40 cm (40,02–40,28). Por eso no se comprobó qué pasa con una lectura menor al umbral estando el objeto colocado en él. Las conclusiones (punto 2) reconocen esta limitación, pero §5 dice que las pruebas se hicieron "exactamente a 20.0 cm y 40.0 cm".
- **No hay análisis de errores:** el resultado de TC-7 se apartó de lo esperado (17 ciclos frente a ≈25 según el README y ≈20 según la hoja) y no se explica por qué. La causa es el `delay(60)`: el LED solo puede cambiar de estado al pasar por `loop()`, así que el medio periodo real es ~2 iteraciones (≈ 120–150 ms) y no 100 ms. No se informa la dispersión entre repeticiones ni el error máximo individual (por ejemplo, 59,12 cm a 60 cm da un error de 0,88 cm, que el promedio oculta).

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay resultados cuantitativos para la exactitud (error absoluto promedio entre 0,03 y 0,47 cm) y para la estabilidad (15 min, 0 reinicios).
- Las **recomendaciones son técnicas y pertinentes**: extraer la clasificación a una función pura para probarla con valores sintéticos, compensar la velocidad del sonido según la temperatura, ampliar los casos de borde (2 y 300 cm, superficies que no reflejan bien) y registrar telemetría automatizada o analizar video para medir el tiempo de respuesta. En este aspecto el informe supera el nivel 2.

**Por qué baja a 2**
- **Algunas conclusiones no se sostienen con la evidencia:**
  - *Conclusión 3 / §5:* "17 ciclos completos en 5 s → más de 3 lecturas por segundo". El parpadeo no mide el muestreo. Con un `loop()` de ≈ 61–90 ms (60 ms de `delay` + `pulseIn` + `Serial`), el sistema debería hacer **≈ 11–16 lecturas/s**. Este valor no coincide ni con las ">3 lecturas/s" del informe ni con las 1,97 lecturas/s de la hoja. Esto muestra que la frecuencia de muestreo nunca se midió realmente.
  - *Conclusión 4 / §5:* se dice que el cambio de estado es "*virtualmente instantáneo*" y que RNF3 se valida "*en un tiempo comprobado menor*", pero no se presenta ningún valor medido. Además, el informe afirma que RNF3 queda validado "como consecuencia directa" de RNF4, lo cual es una inferencia y no una medición.
  - *§5, Estabilidad:* el informe dice "*demostrando empíricamente*", mientras que el anexo registra la prueba como **simulada**.
  - *Conclusión 4:* que el *timeout* de 30 ms "evitó bloqueos en los temporizadores internos del ESP32" no tiene sustento técnico. El *timeout* solo limita la espera de `pulseIn`.
- El análisis crítico es limitado. El error relativo de **4,7 % a 10 cm** (el más alto) no se discute, aunque es justo en la zona de mayor importancia (peligro).
- **Anexos fuera del repositorio**, sin fotografías dentro del repo y con el marcador "Anexo [N]" sin completar. Para el nivel 4 se piden "anexos pertinentes" y §6 exige incluirlos en el repositorio.

---

## 4. Recomendaciones para próximas entregas

1. **Hacer push antes de la hora límite** y comprobar en GitHub que el último commit quedó publicado. En esta entrega, la documentación de `UltrasonicSensor` quedó fuera por 4 minutos.
2. Escribir los **resultados obtenidos directamente en las tablas del README** (columnas *Resultado observado* y *Estado*) y guardar las evidencias (CSV, fotos, capturas del monitor serie) en una carpeta `docs/` o `anexos/` dentro del repositorio.
3. Medir el muestreo y el tiempo de respuesta **desde el firmware**: registrar `millis()` en cada lectura y en cada cambio de zona, enviarlo por `Serial` y calcular con esos datos la frecuencia (lecturas/s) y la latencia (ms).
4. No dar por validado un requerimiento con una prueba **simulada** o "cualitativa". Si una prueba no se pudo hacer, hay que decirlo en el informe.
5. Revisar la coherencia interna del informe: que cada RNF apunte al TC correcto, que el rango declarado coincida con el rango probado y que los valores esperados sean los mismos en el README y en los anexos.
6. En el código: reemplazar los números mágicos por constantes con nombre, documentar todas las clases con el mismo estilo e idioma, y reconsiderar el comportamiento cuando no hay eco (por ejemplo, un estado de **error** en lugar de "zona libre").
7. Incluir un diagrama de **secuencia o de actividad** del `loop()` para mostrar el orden de ejecución.
