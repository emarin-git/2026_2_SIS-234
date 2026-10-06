# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/Matuidich/Practica1-IoT
**Informe:** `doc/informe.md`
**Integrantes:** Mateo Rojas Campos (Matuidich) · Lucia Escobar Galaburda (LuEscobarG)
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

> **Nota sobre esta revisión:** primero se calificó el repositorio según su estado a la hora límite (§6 de la consigna), con **5/16 (31,25)**. Por decisión del docente, **se reevaluó sin considerar la hora de presentación**, tomando el estado final del repositorio (`36e8837`, 15-Sep 18:23). **La calificación vigente es la de la reevaluación: 9/16 (56,25).** La evaluación original se conserva en el §7 como referencia.

---

## 1. Resumen de la calificación

| Criterio | Evaluación original (a las 17:45) | **Reevaluación (estado final)** |
|---|:---:|:---:|
| Requerimientos, análisis y diseño | 2 | **2** / 4 |
| Desarrollo e implementación (código y documentación) | 2 | **2** / 4 |
| Pruebas y validaciones | 1 | **2** / 4 |
| Resultados, conclusiones y recomendaciones | 0 | **3** / 4 |
| **Total** | 5 / 16 | **9 / 16** |
| **Nota del componente** | 31,25 | **56,25** |

La nota sube porque los commits posteriores a la hora límite agregaron:
- la ejecución de las pruebas;
- las secciones de Resultados, Conclusiones y Recomendaciones;
- 12 imágenes de evidencia (fotos del montaje y capturas del monitor serie);
- los nombres reales de los integrantes.

Las secciones 1 a 3 del informe y el código son casi iguales a la versión original, por eso esos criterios no cambian.

---

## 2. Historial del repositorio

Historial de *push* a `main` (registro público de eventos de GitHub, convertido a UTC-4):

| Commit | Push (UTC-4) | Autor | Contenido |
|---|---|---|---|
| `336f22c` | 11-Sep 19:34 | Mateo Rojas Campos | Código e informe inicial (pruebas en `PENDIENTE`, sin secciones 5 a 8) |
| `f700f1a` | 15-Sep 18:05 (20 min tarde) | Lucia Escobar | Pruebas ejecutadas, resultados, conclusiones, recomendaciones, 12 imágenes y nombres de los integrantes |
| `36e8837` | 15-Sep 18:23 (38 min tarde) | Lucia Escobar | "fin": cambios en `main.cpp` y `platformio.ini` |

Lucia Escobar fue agregada como colaboradora del repositorio el 15-Sep a las 18:04.

---

## 3. Observaciones por criterio — Reevaluación (estado final `36e8837`)

### 3.1 Requerimientos, análisis y diseño — **2 / 4 (Satisfactorio con recomendaciones)**

La única novedad en esta sección es la lista de integrantes con nombres reales. Las observaciones del §7.1 siguen vigentes:

- **Requerimientos imprecisos en las fronteras:** RF2 dice "entre 20 y 40 cm" y "mayor a 40 cm" sin aclarar si los límites se incluyen; recién se aclara en §2.7 y §3.5. Los RNF no indican cómo se verifican y no hay matriz de trazabilidad.
- **Diagrama de circuito incompleto:** no muestra VCC 5 V, GND común, **resistencias limitadoras de los LEDs** (no se mencionan en todo el informe), polaridad ni el nivel de 5 V de ECHO sobre el GPIO18, que trabaja a 3,3 V.
- **Diagrama de arquitectura mínimo**, casi igual al de comportamiento.
- **Diagrama de clases con errores:** muestra **composición** entre `UltrasonicSensor` y `Measurement`, cuando en realidad `Measurement` es una clase anidada que se devuelve como valor. No incluye las funciones de `main.cpp` ni el enum `DistanceState`.
- No hay diagrama de secuencia ni de estados. El flujo de §2.6 no muestra el tiempo bloqueado por `delay()` en el parpadeo ni el envío por puerto serie.

### 3.2 Desarrollo e implementación — **2 / 4 (Satisfactorio con recomendaciones)**

Las observaciones del §7.2 siguen vigentes:
- solo el sensor es POO; los LEDs se manejan con funciones sueltas y `#define`;
- se mezclan `#define` y `constexpr`;
- la constante `0.034f` está escrita directamente en el código;
- el parpadeo es bloqueante (`delay(125)` × 2);
- **el código no tiene ningún comentario**;
- `upload_port` está fijo (ahora `COM5`).

A esto se suman dos problemas nuevos:

- **El último commit (`36e8837`, "fin") rompe el firmware.**
  - Elimina `sensor.begin()` de `setup()`, así que el pin TRIG (GPIO5) **nunca se configura como salida** y el sensor no puede dispararse.
  - Reemplaza `applyLedState(DistanceState::Invalid)` por encender los tres LEDs al arrancar, lo que contradice el estado `Invalid` del diseño.
  - El código entregado ya no es el que se usó en las pruebas, y el informe (§3.3) sigue describiendo `begin()` como parte del arranque.
  - Este problema deja el criterio **en el límite con el nivel 1** ("código que no corresponde a los requerimientos declarados"). Se mantiene en 2 porque la estructura del código sigue siendo razonable.
- **No hay intervalo entre mediciones en los estados `Mid` y `Far`.** En esos estados `loop()` no tiene ninguna espera, y el sensor se dispara apenas termina la medición anterior. El HC-SR04 necesita unos **60 ms entre disparos** para que se disipen los ecos. Las propias capturas del equipo muestran el efecto (ver §3.3).

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Las pruebas **se ejecutaron**:
  - 6 distancias (10 a 60 cm) con 2 lecturas cada una;
  - tabla de exactitud con promedio y error absoluto;
  - prueba de la frontera de 40 cm (39,97 cm → amarillo; ~40,1 cm → verde);
  - estabilidad de 10 minutos;
  - tiempo de respuesta;
  - comportamiento de los actuadores.
- Se informó con honestidad una **falla alrededor del minuto 11** de la prueba de estabilidad (el monitor dejó de mostrar datos y aparecieron mediciones inválidas), en lugar de ocultarla.
- Hay 12 imágenes en `doc/imgs/`: fotos del montaje con cinta métrica y capturas del monitor serie.

**Por qué no llega a 3**
- **Las propias capturas muestran un 50 % de lecturas inválidas, y el equipo no lo notó.** En todas las capturas de los estados `Mid` y `Far` (21,51 cm, 31,94 cm, 39,51–39,97 cm, 49,81 cm y 59,18 cm), las líneas **alternan una por una** entre `Distance: … cm` e `Invalid measurement`. En `Near` (10,57 cm) no pasa, porque ahí el `delay(125)` × 2 deja tiempo entre disparos. Esto tiene dos consecuencias:
  - Cada lectura inválida ejecuta `setLedOutputs(false, false, false)`. Por eso **el LED amarillo o verde se apaga y se enciende sin parar**, lo que contradice el "Resultado observado: LED amarillo/verde encendido" de PF2 a PF6 y la afirmación de "encendido constante" de §4.5.
  - Es la explicación más probable de la falla del minuto 11. Al no haber espera, `loop()` envía mensajes por el puerto serie sin pausa y dispara el sensor sin respetar su ciclo. La recomendación de "revisar conexiones" apunta a otro lado.
- **RNF4 (frecuencia de muestreo ≥ 2 lecturas/s) no se verificó.** El plan original tenía una prueba de frecuencia (§4.5), pero se reemplazó por "frecuencia de los actuadores", que solo **calcula** los 4 Hz del parpadeo a partir de la configuración (125 + 125 ms) sin medirlos. No hay ningún conteo de lecturas por segundo, aunque el monitor serie lo permitía.
- **El tiempo de respuesta (0,5–0,8 s) no tiene método ni coincide con el código.** No se explica cómo se midió (cronómetro, video o registro). Según el código, en `Mid`/`Far` el ciclo dura pocos milisegundos y en `Near` unos 250 ms, así que el cambio de estado debería verse casi de inmediato. El rango informado parece una estimación visual.
- **Se quitaron los casos de frontera del plan original:** en la versión final ya no están PF2 (19 cm), PF3 (20 cm) ni PF6 (41 cm). A 20 cm de referencia el sensor midió 21,5–22,4 cm, así que la frontera de 20 cm **nunca se probó**.
- **Muestra muy pequeña:** solo 2 lecturas por distancia. La estabilidad no tiene registro (horas, cantidad de lecturas ni conteo de inválidas).
- **El informe no enlaza las evidencias:** §4.7 lista lo que "se adjuntó", pero ninguna imagen aparece en el informe ni se indica qué imagen respalda cada prueba. Los archivos tienen nombres generados automáticamente (UUID). Además, la §4 tiene el título duplicado ("## 4." y "# 4.").

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Resultados medibles:** tabla de exactitud con error máximo de 1,94 cm (a 20 cm), frontera de 40 cm verificada con valores, tiempo de respuesta máximo de ~0,8 s y estabilidad de 10 minutos.
- Las conclusiones siguen la secuencia de las pruebas y **reconocen la anomalía** del minuto 11 como una limitación.
- **Hay recomendaciones**, algunas técnicas: filtrar o promediar lecturas cerca de los límites, recuperar el sistema ante lecturas inválidas seguidas, pruebas prolongadas y revisión de la comunicación serie.

**Por qué no llega a 4**
- **El análisis crítico es limitado.** Los resultados no detectan el patrón de lecturas inválidas alternadas que se ve en sus propias evidencias, ni lo relacionan con la falla del minuto 11. Afirman que los LEDs amarillo y verde están "encendidos de manera constante", cuando el código y las capturas indican lo contrario.
- **Conclusión de estabilidad débil:** se dice que el sistema "cumple formalmente" RNF1 porque la falla ocurrió en el minuto 11. Un sistema que deja de medir bien al minuto 11 no es estable; el umbral de 10 minutos es un mínimo, no un tiempo después del cual se aceptan fallas.
- **Faltan recomendaciones clave:** agregar un intervalo mínimo entre mediciones (≥ 60 ms), hacer el parpadeo no bloqueante y adaptar el nivel de ECHO a 3,3 V. Varias recomendaciones son genéricas ("mantener el sensor alineado", "evitar superficies inclinadas").
- **No hay sección 8 (Anexos)**, aunque la consigna (§4.2) la exige. Las imágenes están en el repositorio pero el informe no las enlaza ni las describe.

---

## 4. Recomendaciones para próximas entregas

1. **Agregar un intervalo fijo entre mediciones** (por ejemplo, medir cada 100 ms con `millis()`) y **hacer el parpadeo no bloqueante**. Con eso deberían desaparecer las lecturas inválidas alternadas y el muestreo sería uniforme en todos los estados.
2. **Corregir el último commit:** volver a poner `sensor.begin()` y el arranque con los LEDs apagados. Volver a probar con el firmware final antes de la Demo.
3. **Medir en lugar de calcular:**
   - la frecuencia de muestreo, contando líneas del monitor serie en un intervalo conocido o imprimiendo `millis()`;
   - el tiempo de respuesta, con marcas de tiempo o video;
   - el parpadeo, contando ciclos en 10 s.
4. **Leer las evidencias propias con atención:** las capturas mostraban el problema principal del sistema.
5. Volver a incluir los casos de frontera (19, 20, 41 cm), usar más de 2 lecturas por punto e informar el error individual máximo.
6. Crear la sección **8. Anexos** con las imágenes enlazadas, renombradas con nombres descriptivos (`pf1_10cm.jpg`, `serial_mid.jpg`…) y con una explicación de qué demuestra cada una.
7. Completar el esquema eléctrico (VCC, GND, resistencias, divisor en ECHO), corregir el diagrama de clases y agregar una clase `LedIndicator`.
8. **Publicar con margen antes de la hora límite** y verificar en GitHub que el último commit aparezca a tiempo.

---

## 5. Nota para la Defensa Individual

- Hasta la hora límite, el repositorio solo tenía el trabajo de **Mateo Rojas Campos** (11-Sep). La ejecución de las pruebas, los resultados y las conclusiones son de **Lucia Escobar Galaburda** (15-Sep, después de las 17:45). Conviene comprobar qué hizo cada integrante.
- Preguntas sugeridas:
  - ¿Por qué en las capturas de `Mid` y `Far` se alternan lecturas válidas e `Invalid measurement`, pero en `Near` no?
  - ¿Qué efecto tiene eso sobre los LEDs?
  - ¿Por qué el último commit hace que el sensor deje de funcionar?
  - ¿Por qué `Measurement` es una clase anidada con constructor privado?
  - ¿Qué riesgo tiene conectar ECHO (5 V) directo al GPIO?

---

## 6. Diferencias entre la evaluación original y la reevaluación

| Criterio | Original | Reevaluación | Motivo del cambio |
|---|:---:|:---:|---|
| Requerimientos, análisis y diseño | 2 | 2 | Sin cambios de contenido en §1 y §2, salvo los nombres de los integrantes. |
| Desarrollo e implementación | 2 | 2 | El código casi no cambió. El último commit lo empeora (elimina `sensor.begin()`), pero se mantiene el nivel. |
| Pruebas y validaciones | 1 | 2 | Las pruebas se ejecutaron con valores medidos, pero RNF4 no se verificó, el tiempo de respuesta no tiene método y no se detectó el 50 % de lecturas inválidas en sus propias capturas. |
| Resultados, conclusiones y recomendaciones | 0 | 3 | Las secciones 5, 6 y 7 ahora existen, con resultados medibles y recomendaciones. Falta la sección 8 (Anexos) y el análisis crítico es limitado. |
| **Total** | **5 / 16 (31,25)** | **9 / 16 (56,25)** | |

---

## 7. Evaluación original (según el estado a la hora límite, `336f22c`)

> Se conserva como registro. Esta fue la primera calificación, hecha según §6 de la consigna ("solo se evalúa el contenido existente en el repositorio hasta esa fecha"). Fue **reemplazada por la reevaluación** de las secciones anteriores.

**Calificación original: 5/16 = 31,25**, menor que la reevaluación. Los dos *push* con el contenido de pruebas, resultados y evidencias (`f700f1a` a las 18:05 y `36e8837` a las 18:23) llegaron después de las 17:45, por lo que no se consideraron.

### 7.1 Requerimientos, análisis y diseño — 2 / 4

- RF1 a RF3 y RNF1 a RNF5 definidos, con valores medibles. Tabla de pines y de lógica de control. Diagramas de arquitectura, circuito, clases y flujo.
- Fronteras imprecisas en RF2 (20/40 cm); sin método de verificación de los RNF ni trazabilidad.
- Circuito sin VCC, GND, resistencias, polaridad ni tratamiento del nivel de ECHO.
- Arquitectura mínima. Diagrama de clases con composición incorrecta. Sin diagrama de secuencia ni de estados.
- Lista de integrantes con texto de ejemplo ("Nombre integrante 1/2").

### 7.2 Desarrollo e implementación — 2 / 4

- Fortalezas: clase `UltrasonicSensor` con clase anidada `Measurement` (no se puede usar una distancia inválida por error), constantes `constexpr` con `static_assert`, `enum class DistanceState`, *timeout* en `pulseIn`, y §3 del informe que explica la implementación.
- Debilidades: POO solo para el sensor (LEDs con funciones sueltas y `#define`), convenciones mezcladas, `0.034f` escrita directamente, parpadeo bloqueante que afecta RNF3/RNF4, código sin comentarios, validación repetida sin límite superior y `upload_port = COM3` fijo.

### 7.3 Pruebas y validaciones — 1 / 4

- Había un plan de pruebas bien orientado, con casos de frontera en 19, 20, 40 y 41 cm, pero **todos los resultados estaban en `PENDIENTE`** y §4.6 "Evidencias" estaba escrita en futuro. No había ninguna prueba ejecutada ni evidencia en el repositorio.

### 7.4 Resultados, conclusiones y recomendaciones — 0 / 4

- El informe terminaba en §4.6. **No existían** las secciones 5 (Resultados), 6 (Conclusiones), 7 (Recomendaciones) ni 8 (Anexos).
