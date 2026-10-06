# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/Ruben-Cordero/IoT-LEDs
**Informe:** `doc/informe.md`
**Autores de commits:** Rubén Cordero · Alejandro Machaca (alous12) · Luciana Leaño (lucianaaaaaaa)
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **4** / 4 |
| **Total** | **13 / 16** |
| **Nota del componente** = (13 / 16) × 100 | **81,25** |

---

## 2. Estado del repositorio en la fecha límite

- **Entrega dentro de plazo.** El último *push* a `main` (`9f613f6`, "Anexos") fue el **14-Sep a las 20:25**, casi un día antes del límite. Se evalúa ese estado, que es el actual.
- **Trabajo colaborativo visible:** los tres integrantes tienen commits. Cada uno trabajó en su rama (`Sensor`, `A-Lu`, `Rama_Ale`) y luego las ramas se unieron en `main`. Los módulos se repartieron por persona: sensor, LEDs y clasificador. Es una buena práctica de trabajo en equipo.
- Los anexos (fotografías de los casos I01 a I05) y las pruebas automatizadas están **dentro del repositorio**.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Requerimientos de muy buena calidad.**
  - RF1 a RF3 tienen criterios de aceptación explícitos.
  - Los rangos son contiguos y no se solapan: CERCANO `2 ≤ d < 20`, MEDIO `20 ≤ d < 40`, LEJANO `40 ≤ d ≤ 200`. Se aclara a qué rango pertenecen los valores exactos de 20 y 40 cm.
  - Hay un estado `ERROR` con todos los LEDs apagados.
- **Histéresis de 2 cm bien diseñada y documentada.** Una tabla explica cuándo se confirma cada transición, sin crear un cuarto rango. Es una mejora real frente al problema típico de parpadeo en los umbrales.
- **RNF1 a RNF5 medibles, con método de verificación**, y una **matriz de trazabilidad** (§1.3) que vincula cada requerimiento con el diseño, el código y la prueba.
- **Conjunto de diagramas completo y bien explicado:**
  - arquitectura, con subgrafos de entrada, MCU y salida, y una tabla de datos intercambiados;
  - circuito, con tabla de conexiones;
  - clases, que coincide con el código real (atributos, firmas y relaciones);
  - **actividad**;
  - **secuencia**.
  Se entiende tanto la estructura como el orden de ejecución.
- §2.5 justifica las decisiones de diseño (separación de responsabilidades, fallo seguro, observabilidad).

**Por qué no llega a 4**
- **ECHO a 5 V conectado directo al GPIO26, sin ningún análisis.** El informe dice que "la señal ECHO del HC-SR04 se conecta directamente al GPIO26 del ESP32, de acuerdo con el montaje utilizado". Sin embargo, no menciona en ningún lado que el HC-SR04 se alimenta con 5 V y que su ECHO entrega ~5 V, mientras que los GPIO del ESP32 trabajan a 3,3 V (máximo absoluto ≈ 3,6 V). El diseño no tiene divisor de tensión ni conversor de nivel, y el problema no aparece en riesgos, conclusiones ni recomendaciones. En un análisis de diseño esta es una omisión importante.
- **El diagrama de circuito es un flujo de Mermaid, no un esquema eléctrico.** Muestra qué se conecta con qué, pero no muestra niveles de tensión, la alimentación del ESP32 (USB) ni la polaridad de los LEDs. La conexión `GND --- SGND` y los retornos de los LEDs a `GND` se entienden solo por la tabla.
- **No se analiza el efecto del estado `ERROR` sobre la histéresis.** Por diseño, después de `ERROR` la siguiente lectura se clasifica con los límites base (20 y 40 cm). Una sola lectura inválida aislada (frecuente en el HC-SR04) **borra la memoria de la histéresis** y además apaga los LEDs durante un ciclo. Esta consecuencia no se discute, y es la causa más probable de la falla H04 (ver §3.3).
- El informe no tiene título, carátula ni lista de integrantes.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Diseño modular claro:**
  - `SensorUltrasonico` es una clase con pines configurables por constructor.
  - `IndicadorLeds` es una clase con método privado `apagarTodos()`.
  - `clasificarDistancia()` es una **función pura sin dependencia de Arduino**, con su función auxiliar en un *namespace* anónimo.
  - `LecturaDistancia` es una estructura compartida.
  - Se usan `enum class` para rangos y estados.
  - `main.cpp` solo coordina: no contiene `pinMode`, `digitalWrite` ni `pulseIn`.
- **Configuración centralizada** en `Config.h` con `constexpr`, más `static_assert` en las pruebas para asegurar que los umbrales estén en orden. Esto evita números mágicos en la lógica.
- Las lecturas inválidas se manejan con el par `distanciaCm` + `valida`, sin valores especiales como 400 o 0.
- **§3 del informe es excelente:** explica cada módulo con fragmentos de código, la tabla de conversión rango → estado y el formato del registro serie con un ejemplo.

**Por qué no llega a 4**
- **El código fuente no tiene comentarios:** ningún archivo de `src/` tiene comentarios de clase, de método ni de contrato. La consigna (§3.2) y la rúbrica piden código **documentado**. La explicación está en el informe y en `MD/`, pero no en el código.
- **La documentación de `MD/` está desactualizada y contradice el código y el informe:**
  - `MD/ClasificadorDistancia.md` menciona `include/Config.h`, `lib/ClasificadorDistancia/` y el tipo `LecturaSensor`, que ya no existen.
  - `MD/IndicadorLeds.md` indica **resistencias de 330 Ω**, mientras que el informe dice 220 Ω.
- **Configuración inconsistente entre clases:** el sensor recibe sus pines por constructor, pero `IndicadorLeds` los tiene fijos como `static constexpr` privados. No se pueden cambiar sin editar la clase.
- **El rango se valida dos veces con límites distintos:** el sensor acepta 2–400 cm y el clasificador 2–200 cm. Funciona, pero hay dos fuentes de verdad para el mismo concepto.
- `README.md` solo contiene el título `# IoT-LEDs`. No orienta hacia el informe, el código ni las instrucciones de compilación.

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Plan de 10 pruebas** con tipo, requisitos cubiertos, objetivo y estado. Separa claramente las pruebas de software, las experimentales y las analíticas.
- **20 pruebas Unity reales y verificables en el repositorio** (`test/test_clasificacion/test_main.cpp`) con entorno `native`. Cubren límites exactos (1,99; 2,00; 19,99; 20,00; 39,99; 40,00; 200,00; 200,01) y todas las transiciones de la histéresis.
- **Exactitud en 8 distancias, de 10 a 200 cm, cubriendo todo el rango declarado**, con 3 lecturas por punto, promedio y error absoluto. Ninguno de los otros grupos revisados validó el rango completo.
- **Muestreo medido** (74 lecturas en 10 s = 7,4 lecturas/s) y contrastado con el cálculo teórico (130 ms frente a 135 ms observados).
- **Estabilidad de 10 minutos** con 6 puntos de control en distintos rangos.
- **Casos del indicador con evidencia fotográfica** (Anexos 1 a 5), incluidos fuera de rango y sin eco.
- **Se informó un resultado negativo (H04: "No aprobada")** en lugar de ocultarlo. Es una muestra de rigor.

**Por qué no llega a 4**
- **RNF3 (tiempo de respuesta < 1 s) no se midió.** Se da por cumplido con el razonamiento "periodo de 135 ms → respuesta < 1 s". Pero el periodo de muestreo no es el tiempo de respuesta. Con histéresis, el cambio solo se confirma cuando la lectura supera el margen, y eso no se cuantificó. Bastaba con usar `tiempo_ms` del registro serie para medir el tiempo entre la primera lectura en la zona nueva y el cambio de rango.
- **El análisis de H04 no es coherente con los propios datos.** El informe atribuye la oscilación a "variaciones físicas de la lectura que podrían atravesar el límite de 42 cm". Para oscilar entre amarillo y verde con histéresis, las lecturas tendrían que llegar a **≥ 42 cm y luego bajar de < 38 cm**, es decir, variar más de 4 cm. Sin embargo, la propia prueba de exactitud a 40 cm muestra lecturas de 40,26–40,51 cm. La explicación más probable es la del punto anterior: **una lectura inválida aislada lleva a `ERROR`**, y la siguiente lectura (41 o 39 cm) se clasifica **sin histéresis**, como verde o amarillo. Esto coincide con el caso C20 de las propias pruebas. Revisar el registro serie habría confirmado o descartado esta hipótesis.
- **No hay registros crudos en el repositorio:** ni la salida serie de las pruebas de exactitud, muestreo y estabilidad, ni el log de compilación (RAM y flash se informan pero no hay archivo de respaldo). Los valores no se pueden verificar.
- **Pocas muestras y error informado como promedio:** con 3 lecturas por punto, se reporta como "máximo" el **error del promedio** (1,63 cm). El error individual más alto es **2,19 cm** (22,19 cm a 20 cm de referencia), sigue dentro de ±3 cm y no se menciona.
- **No se cumplieron las condiciones de ensayo declaradas:** §4.5.2 pide un "objeto plano y estable, colocado perpendicularmente", pero en el Anexo 1 se usa un objeto **esférico**.

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Resultados cuantificables** consolidados en una tabla (§5.1): error máximo 1,63 cm, error medio 0,64 cm, 7,4 lecturas/s, periodo de 135 ms, 10 min de estabilidad, 20/20 pruebas automatizadas, 5/5 casos del indicador, 5/6 secuencias de histéresis, RAM y flash.
- **Hay análisis crítico (§5.2):**
  - se relaciona la frecuencia observada con la teórica;
  - se ubica dónde ocurre el mayor error;
  - se distingue entre la lógica del clasificador, validada en aislamiento, y el comportamiento del sistema completo con el sensor real.
- **Conclusiones fundamentadas y honestas:** la conclusión 5 reconoce que la histéresis no elimina todas las oscilaciones y que eso debe tenerse en cuenta en el uso.
- **Recomendaciones técnicas y accionables:**
  - registrar una serie continua entre 38 y 42 cm para diagnosticar H04;
  - evaluar un filtro de mediana o promedio y ajustar el margen;
  - repetir la prueba de exactitud si cambian las condiciones;
  - ejecutar las pruebas de regresión después de cada cambio.
- **Anexos pertinentes:** cada fotografía corresponde a un caso de prueba (I01 a I05) y se explica qué demuestra.

**Observaciones (no cambian el nivel)**
- RNF3 se presenta en los resultados como "compatible con respuesta < 1 s" a partir del periodo de muestreo. Faltó aclarar que se infirió y no se midió.
- Las recomendaciones no tratan el nivel de tensión de ECHO (ver §3.1), que es la mejora de hardware más urgente.
- Algunas recomendaciones son genéricas, como "Mantener las resistencias de 220 Ω" o "Conservar la salida serie".

---

## 4. Recomendaciones para próximas entregas

1. **Agregar un divisor de tensión en ECHO** (por ejemplo 1 kΩ en serie y 2 kΩ a GND, ≈ 3,3 V) o un conversor de nivel. Dibujarlo en el esquema y justificarlo en el análisis de diseño.
2. **Revisar cómo interactúan `ERROR` y la histéresis.** Por ejemplo, conservar el último rango válido durante N lecturas inválidas seguidas antes de pasar a `ERROR`. Agregar una prueba Unity con la secuencia `MEDIO → inválida → 41 cm` y verificar si se reproduce H04.
3. **Medir RNF3** con las marcas `tiempo_ms` del registro: el tiempo entre que el objeto cruza el umbral (más el margen) y el cambio de rango.
4. **Guardar en el repositorio los registros crudos** (salida serie de cada ensayo y log de `platformio run`/`test`), informar el error individual máximo y usar más de 3 lecturas por punto.
5. **Documentar el código fuente:** comentarios de clase y de método en los `.h`, sobre todo los contratos de `medirDistanciaCm()` y `clasificarDistancia()`. Actualizar o eliminar los archivos de `MD/` que contradicen el informe.
6. Hacer que `IndicadorLeds` reciba sus pines por constructor, igual que el sensor.
7. Agregar título, integrantes y fecha al informe, y un `README.md` que enlace al informe y explique cómo compilar y probar.

---

## 5. Nota para la Defensa Individual

El historial muestra una división del trabajo por módulos: Rubén Cordero el sensor, Alejandro Machaca los LEDs, la integración y la mayor parte del informe, y Luciana Leaño el análisis, el diseño y las pruebas. Conviene comprobar que cada integrante pueda explicar también los módulos de los demás, en especial:

- la lógica de histéresis y su relación con el estado `ERROR` (caso H04);
- el riesgo de conectar ECHO a 5 V directamente al GPIO;
- cómo se ejecutan las pruebas `native` sin hardware.
