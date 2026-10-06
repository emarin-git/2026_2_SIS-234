# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/JoseCorralesAgreda/PROYECTO-IOT-SENSOR
**Informe:** `README.md` del proyecto
**Autores de commits:** Jose Franz Corrales Agreda · Alejandro Bustamante · RashLop
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **2** / 4 |
| **Total** | **11 / 16** |
| **Nota del componente** = (11 / 16) × 100 | **68,75** |

---

## 2. Estado del repositorio en la fecha límite

Historial de *push* a `main` (registro público de eventos de GitHub, convertido a UTC-4):

| Commit | Push (UTC-4) | Autor | Contenido |
|---|---|---|---|
| `4dea33c`, `d4484a7` | 08-Sep 17:51 | Jose Franz Corrales | Primer commit y código |
| `ab5ea5c` | 14-Sep 20:25 | Jose Franz Corrales | Herramientas BMAD (`.agents/`, `_bmad/`) |
| `9d2a911` | 14-Sep 21:25 | Jose Franz Corrales | Plan de verificación y pruebas Unity |
| `67e98ed` | 15-Sep 13:01 | RashLop | Mensajes de distancia por `Serial` |
| `194ec75` | 15-Sep 14:21 | Alejandro Bustamante | README, circuito, plan de pruebas |
| `011107c` | 15-Sep 15:48 | Jose Franz Corrales | Ajuste al README |
| `f80aa64` | **15-Sep 17:05** | Jose Franz Corrales | *fix*: tolerar lecturas inválidas aisladas |

- **Entrega dentro de plazo.** El último *push* fue a las 17:05. Se evalúa `f80aa64`, que es el estado actual de `main`.
- Todos los anexos (circuito, plan de pruebas, PRD, arquitectura y registros de verificación) están **dentro del repositorio**, como pide §6 de la consigna.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Los rangos cumplen RF1–RF3: rojo `2 ≤ d < 10`, amarillo `10 ≤ d < 30`, verde `30 ≤ d ≤ 400` cm. Son contiguos, no se solapan y tienen un comportamiento distinto cada uno. Además se define un **estado de error** (parpadeo conjunto a 2 Hz) para lecturas ausentes, que no son números finitos o que quedan fuera de rango. Con esto se evita el problema habitual de mostrar "zona libre" cuando el sensor falla.
- La tabla §1.1 declara los RNF con valores medibles y más exigentes que la referencia (≥ 30 min, ≤ 3 cm, ≤ 100 ms, 10 lecturas/s), e indica el caso de prueba que verifica cada uno.
- El conjunto de diagramas es completo:
  - **Arquitectura:** diagrama de componentes de software y GPIO.
  - **Circuito:** SVG con valores de componentes, tabla de conexiones y esquema en texto.
  - **Clases.**
  - **Secuencia** del `loop()`.
  - **Estados** de `DistanceIndicator`.
  Con ellos se entiende la estructura y el orden de ejecución.
- El divisor de Echo está bien calculado (5 V × 2k/3k = 3,33 V ≤ 3,6 V) y se citan fuentes (hoja de datos del HC-SR04 y documentación de Espressif).

**Por qué no llega a 4**
- **Los requerimientos funcionales no están en el informe.** La §1 remite al PRD (`_bmad-output/.../prd.md`) para RF-01 a RF-07 y RNF-01 a RNF-06. El README solo resume los rangos. Según §4.2 de la consigna, la sección 1 del informe debe contener los requerimientos.
- **El diagrama de estados no coincide con el firmware entregado.** El diagrama (§2.5) y RF-05 indican que *una* lectura inválida lleva al estado `Invalido`. Pero el commit `f80aa64` agregó `kInvalidReadingsBeforeError = 2` (`DistanceIndicator.h:32`), así que ahora hacen falta **dos lecturas inválidas seguidas**. Ni el diagrama, ni el PRD, ni el README se actualizaron.
- **El diagrama de circuito no representa el montaje que se probó.** Según §3.3, todas las pruebas se hicieron **sin divisor** (Echo conectado directo a GPIO19). El diagrama muestra el divisor, y el montaje real solo aparece como nota.
- **Claridad del circuito:** en el SVG, la línea de Trigger (GPIO18) baja junto al símbolo de GND y puede leerse como si estuvieran conectados. Faltaba separar los trazos o marcar el cruce sin unión.
- El "diagrama de arquitectura" es un flujo de software con GPIO. No es un diagrama de bloques o de despliegue del sistema físico (alimentación, MCU, sensor, actuadores, PC/serial).
- Se enlaza `GUIA-DEL-CODIGO.md` (§2.5), pero ese archivo **no existe** en el repositorio.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Arquitectura destacada:**
  - Separa adquisición (`UltrasonicSensor`, `EchoCapture`), decisión (`DistanceIndicator`) y salida (`LedDriver`).
  - Los tipos `Reading` y `LedOutput` no dependen de Arduino, por eso el núcleo se puede probar sin hardware.
- **Adquisición no bloqueante por interrupción** (`attachInterruptArg` con `CHANGE`):
  - La ISR y la tarea comparten una sección crítica `portMUX_TYPE`.
  - El *timeout* de 30 ms se maneja sin `pulseIn`.
  - Las restas sin signo funcionan aunque `micros()`/`millis()` se desborden.
  - Un resultado se consume una sola vez.
- C++17 bien usado: `std::optional`, `enum class`, `constexpr`, miembros `const`, `explicit` y espacio de nombres anónimo en `main.cpp`.
- Versiones fijadas en `platformio.ini`, lo que hace la compilación reproducible.

**Por qué no llega a 4**
- **El código casi no tiene documentación.** Ningún archivo `.h` tiene comentarios de clase ni de método. `DistanceIndicator`, `EchoCapture` y `UltrasonicSensor` tienen lógica delicada (fases de parpadeo, rechazo de flancos con `UINT32_MAX / 2`, la bandera `invalidPending_`, ISR) sin ninguna explicación en el código. Los únicos comentarios útiles están en `main.cpp` y en una línea de `EchoCapture.h`. El RNF-03 del propio equipo ("comentarios únicamente cuando aporten información") no justifica dejar sin comentar justamente las partes no evidentes. Además, la guía del código que debía compensar esto (`GUIA-DEL-CODIGO.md`) no está en el repositorio.
- **Números mágicos:** `100000` (período en µs) en `UltrasonicSensor.cpp:23`, `250` (media fase) en `DistanceIndicator.h:26-27`, `58.0f`, los umbrales `2/10/30/400` y los números de pin en `main.cpp`. Solo `kTimeoutUs` y `kInvalidReadingsBeforeError` tienen nombre.
- **Código duplicado e inconsistente en `loop()`:** el bloque de impresión por `Serial` aparece dos veces, y la segunda copia no imprime `INVALID` (`main.cpp:49-57`). Faltaba una función auxiliar.
- **Cambio de comportamiento a última hora:** `f80aa64` (17:05) cambió la lógica de error. Se actualizaron dos pruebas Unity, pero no se registró una nueva ejecución de las pruebas: `verification-native.txt` sigue fechado el 14-Sep, con la huella de las fuentes anteriores.
- `src/main.cpp` empieza con un BOM UTF-8 (`﻿`).
- El repositorio incluye unos **200 archivos de herramientas de agentes** (`.agents/skills/`, `_bmad/`) que no tienen relación con el objeto inteligente y hacen difícil revisarlo. Hay **dos versiones distintas de `PLAN-DE-PRUEBAS.md`**: la de la raíz está desactualizada (dice que P-01 *bloquea* la conexión de Echo).

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **15 pruebas Unity automatizadas** que usan las clases reales, con dobles de Arduino para reloj, GPIO e interrupciones. Cubren fronteras (2/10/30/400 cm), NaN/infinito, *timeout*, flancos fuera de orden y desbordamiento de `uint32_t`. Hay evidencia guardada (`verification-native.txt`, `verification-esp32.txt`) con salida, código de retorno y SHA-256.
- **Plan manual muy completo (M-01 a M-10):** cada caso tiene seguridad, preparación, pasos, resultado esperado y criterio de aprobación cuantitativo (por ejemplo, 250 ± 10 ms, sesgo ≤ 1 ms, ≥ 100 ms entre inicios). También hay una matriz de trazabilidad entre requerimientos, pruebas y estado.
- La tabla §4.3 distingue bien qué demuestra y qué no demuestra cada nivel de evidencia.

**Por qué no llega a 4**
- **El registro manual no tiene ningún valor medido.** Para M-01 a M-10 solo se anota "conforme" o "dentro del objetivo" y el veredicto "Aprobado". El plan pide registrar valores (`R_medida`, `V_nodo,max`, medias fases, retraso en ms, hora de inicio y fin), pero en el registro no aparece ninguno. Además, el plan dice expresamente que "no se exige conservar fotografías, videos, capturas", aunque §1.1 anuncia la verificación de M-09 "con bitácora y video". **Los RNF declarados quedan sin valores que los respalden.**
- **M-02 se contradice con el propio informe.** El registro dice "tensión dentro del límite del GPIO" en un montaje **sin divisor**. Pero §3.3 afirma que Echo llega a ~5 V y que el GPIO tolera como máximo 3,6 V, y no se informa ninguna tensión medida. Con eso, RNF-06 ("no conectar una señal que exceda la tolerancia del GPIO") **no se cumplió en el banco**, y aun así figura como "Aprobado".
- **El firmware probado no es el que se entregó.** Los ensayos M se hicieron "sobre el firmware verificado el 2026-09-14". Después se agregaron los mensajes por `Serial` (15-Sep 13:01) y la tolerancia a lecturas inválidas (15-Sep 17:05). Esta última cambia directamente lo que miden M-05, M-06, M-07 ("estado inválido entre 30 y 40 ms después de Trigger") y M-08. Con dos lecturas inválidas necesarias y un período de 100 ms, el paso válido → inválido tarda ≈ 130–230 ms. Eso **supera el "≤ 100 ms en el peor caso"** declarado, aunque sigue por debajo de 1 s. No se repitió ninguna prueba manual ni Unity después del cambio.
- **La exactitud (M-10) no se verificó como se declaró.**
  - El RNF dice "≤ 3 cm en **todo el rango de 2 a 400 cm**", pero M-10 solo prueba 5–100 cm y los puntos a ±3 cm de las fronteras.
  - El método por color de LED solo limita el error en un sentido cerca de cada umbral. No mide el error en 50 o 100 cm.
  - El firmware de prueba (14-Sep) no mostraba la distancia por `Serial`, así que la frase "error absoluto dentro de 3 cm" no tiene ningún valor numérico detrás. La propia recomendación de §8 ("Exponer la distancia… habilita la ejecución de M-10") lo confirma.
- **Frecuencia de muestreo (10 lecturas/s):** M-07 solo comprueba que la separación sea ≥ 100 ms, que es un mínimo. No se mide la frecuencia real lograda.
- **M-09 (≥ 30 min):** no se anotan hora de inicio ni de fin, ni duración real.

> Aunque estas observaciones son importantes, la calidad del plan y de las pruebas automatizadas, con evidencia reproducible dentro del repositorio, supera claramente el nivel 2 ("plan incompleto con validaciones parciales"). Lo que falla es el registro de la ejecución física.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El informe presenta una tabla de resultados con estado y fuente de evidencia por requerimiento, y separa la evidencia simulada de la física.
- Las conclusiones reconocen límites técnicos reales: P-03 (ecos espurios) y las fallas de rebote por debajo de 2 cm.
- Las **recomendaciones son muy buenas**, técnicas y bien organizadas por áreas: histéresis en los umbrales, filtro de mediana con ventana limitada, calibración del factor 58 µs/cm, compensación por temperatura, repetibilidad, CI de pruebas nativas, etiquetar la versión de firmware probada y PCB. En este aspecto el informe alcanza el nivel 4.

**Por qué baja a 2**
- **No hay resultados cuantificables.** §6 y el Anexo B solo tienen veredictos. No se informa ni un error en cm, una latencia en ms, una frecuencia en lecturas/s, una tensión en V ni una duración en minutos. Para el nivel 3 se piden "resultados **medibles**" y para el 4, "cuantificables".
- **Algunas conclusiones no se sostienen con la evidencia:**
  - "Los tres requerimientos funcionales mínimos quedan cubiertos y **verificados con 15 pruebas nativas**". Las pruebas nativas no pueden verificar RF1 (medición física), como admite el propio §4.3.
  - "RNF-06 compatibilidad eléctrica: **Aprobado**", con el montaje probado sin divisor (ver §3.3 de esta revisión).
  - "El repositorio reúne… **el código fuente documentado**". El código casi no tiene comentarios y la guía enlazada no existe.
  - "Exactitud: error absoluto ≤ 3 cm — Aprobado", sin ningún valor medido y sin cubrir el rango declarado.
- **No hay análisis crítico de los cambios de última hora.** La tolerancia a lecturas inválidas (`f80aa64`) se agregó para corregir un parpadeo observado en el banco, lo que es un resultado físico relevante. Sin embargo, no se informa ni se analiza en §6 ni en §7.
- **Falta la lista de integrantes.** El encabezado solo dice "Autor del proyecto: JOSE FRANZ", y la propia recomendación "Incorporar la tabla de integrantes y su aporte individual" reconoce que falta.

---

## 4. Recomendaciones para próximas entregas

1. **Registrar valores, no solo veredictos.** Para cada RNF: error en cm por distancia (con media y desviación de las repeticiones), latencia medida en ms, frecuencia real en lecturas/s, tensión de Echo en V y hora de inicio y fin del ensayo de estabilidad. Guardar capturas del osciloscopio y del monitor serie en el repositorio.
2. **Repetir las pruebas después de cada cambio de firmware.** Si se cambia el código después de probarlo, hay que volver a ejecutar las pruebas o decir explícitamente cuáles resultados dejaron de valer, y actualizar los diagramas y requerimientos afectados.
3. **Que el informe coincida con el montaje real.** Si la prueba fue sin divisor, el diagrama debe mostrarlo, y no se debe marcar como aprobado un requerimiento eléctrico que no se cumplió. Lo mejor es montar el divisor, que solo son dos resistencias.
4. Escribir los RF y RNF **dentro del README**. Los documentos auxiliares deben complementar el informe, no reemplazarlo.
5. Documentar el código: comentarios de clase y método en los `.h`, sobre todo en la ISR, la sección crítica y la lógica de fases. Usar constantes con nombre para períodos, umbrales y pines, y eliminar la duplicación en `loop()`.
6. Mantener el repositorio limpio: sacar del control de versiones las herramientas de agentes (`.agents/`, `_bmad/`), eliminar el `PLAN-DE-PRUEBAS.md` duplicado y corregir los enlaces rotos.
7. Poner la lista de integrantes en el encabezado del informe.

---

## 5. Nota para la Defensa Individual

El informe y la estructura del repositorio muestran un flujo de trabajo apoyado fuertemente en agentes (BMAD: PRD, *architecture spine*, revisiones automáticas). El resultado técnico es sólido, pero conviene comprobar en la defensa que **cada integrante** pueda explicar:

- la ISR y por qué hace falta `portMUX_TYPE`;
- la aritmética sin signo y el desbordamiento;
- el protocolo `start` / `onEdge` / `expire` / `takePulse` de `EchoCapture`;
- el motivo del cambio `kInvalidReadingsBeforeError = 2`;
- el riesgo eléctrico de conectar Echo sin divisor.
