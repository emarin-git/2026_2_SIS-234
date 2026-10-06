# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/mjlozada2003/UltrasonicSensorLeds2
**Informe:** `INFORME_TECNICO.md`
**Integrantes (según el informe):** María Jesús Lozada Peralta · Samuel Jarro Rodriguez · Katherine Montaño Mejia
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **3** / 4 |
| **Total** | **13 / 16** |
| **Nota del componente** = (13 / 16) × 100 | **81,25** |

---

## 2. Estado del repositorio en la fecha límite

Historial de *push* (registro público de eventos de GitHub, convertido a UTC-4):

| Commit | Push (UTC-4) | Autor | Contenido |
|---|---|---|---|
| `1df9d2c` … `3794015` | 05/06-Sep | María Lozada | Base del código, BMAD, PRD, arquitectura, refactorización a POO y pruebas unitarias |
| `3d88f19` | 08-Sep 00:06 | María Lozada | Informe corregido según la rúbrica |
| `d46e303` | 14-Sep 19:05 (rama `sam`) | Samuel Jarro | Código mejorado: `millis()` y control de valores negativos |
| `408b395` | 14-Sep 20:15 (rama `sam`) | María Lozada | Corrección de pruebas y reestructuración del informe |
| `b03dcb5` | 15-Sep 13:51 (rama `sam`) | Samuel Jarro | Fotografías de anexos |
| `0f136c5` | **15-Sep 16:52** (`main`) | Samuel Jarro | Merge de `sam` en `main` |

- **Entrega dentro de plazo.** Se evalúa `0f136c5`, que es el estado actual de `main`.
- Hay commits de **dos** integrantes: María Lozada y Samuel Jarro. **Katherine Montaño Mejia no tiene ningún commit.**

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Requerimientos en tablas con ID y criterio. Se agregan RF4 (envío de datos por el puerto serie) y RF5 (modo de depuración) como requerimientos propios.
- **Rangos con límites explícitos y sin ambigüedad:** Near `0 ≤ d ≤ 10`, Medium `10 < d ≤ 20`, Far `20 < d ≤ 30`, más un estado OutOfRange con los LEDs apagados (también cuando no hay eco).
- RNF con **valores propios más exigentes** que la referencia de la consigna: tiempo de respuesta ≤ 150 ms y muestreo de ≈ 10 lecturas/s.
- **Conjunto de diagramas completo:**
  - arquitectura, con entrada, procesamiento, salida y PC;
  - circuito, con resistencias de 220 Ω, VIN 5 V, GND común y pines;
  - clases, que **coincide con el código real**;
  - **máquina de estados**;
  - **secuencia** del ciclo de muestreo.
  Cada diagrama tiene su explicación.
- **Se reconoce el riesgo de ECHO a 5 V** en una nota técnica (§2.2) y se propone un divisor de 1 kΩ / 2 kΩ. Es mejor que omitir el tema, aunque no se haya implementado.

**Por qué no llega a 4**
- **Hay una inconsistencia entre los rangos y el rango de trabajo.** La zona Near empieza en **0 cm**, pero RNF2 declara un rango de trabajo de **2 a 30 cm** y §4.4 reconoce que por debajo de 2 cm el sensor está en su "zona ciega". Una lectura de 0–2 cm se trata como válida y enciende el rojo, cuando debería tratarse como no confiable.
- **Algunos requerimientos describen la implementación, no el comportamiento:** RF1 menciona `pulseIn()` y "devuelve `-1.0f`", RF2 nombra la función `evaluateDistanceZone()` y RNF1 justifica el cumplimiento con detalles de diseño (`millis()`, watchdog). Esto mezcla el qué con el cómo.
- **No hay matriz de trazabilidad.** La relación entre requerimientos y pruebas solo se puede deducir por los nombres (RF1 → P-RF1). No se vinculan con componentes ni con resultados.
- **El diagrama de circuito es un flujo de Mermaid**, no un esquema eléctrico: no se ven niveles de tensión ni la polaridad de los LEDs.
- **ECHO conectado directo al GPIO27:** el riesgo se documenta, pero el prototipo sigue con 5 V sobre un pin de 3,3 V.

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Diseño modular orientado a objetos:**
  - `UltrasonicSensor` y `DistanceIndicator` son clases con pines y *timeout* recibidos por constructor.
  - `evaluateDistanceZone()` es una **función pura sin dependencia de Arduino** y se puede probar en la PC.
  - `enum class DistanceZone` evita números mágicos para los estados.
  - `main.cpp` solo coordina.
- **Control de tiempo no bloqueante:** se cambió `delay(100)` por un control con `millis()` (`ahora - ultimaLecturaMs >= intervaloLecturaMs`), con un intervalo fijo de 100 ms entre disparos. Ese intervalo respeta el ciclo del HC-SR04.
- **Código documentado:** los encabezados tienen comentarios Doxygen (`@file`, `@brief`, `@param`, `@return`) y los `.cpp` tienen comentarios por paso. Es el código mejor documentado de los grupos revisados.
- `DistanceIndicator::update()` asegura que nunca haya dos LEDs encendidos a la vez escribiendo los tres pines en cada caso.
- El informe incluye **el código fuente completo** en §3.2 y explica las decisiones de diseño (§3.1).
- Pruebas automáticas de la lógica pura con Unity en entorno `native`, y logs de compilación y de pruebas en el repositorio.

**Observaciones (no cambian el nivel)**
- **Números mágicos en la lógica:** los umbrales `10.0f`, `20.0f` y `30.0f` están escritos directamente en `DistanceZone.cpp`, y `30.0f` se repite en `main.cpp` para decidir el mensaje "FUERA DE ALCANCE". La constante `0.01723f` también está escrita directamente. Faltaría centralizarlos como `constexpr`.
- **Nombres en dos idiomas:** `ledRojo`, `pinTrigger`, `ultimaLecturaMs` en `main.cpp` frente a `redPin`, `triggerPin` en las clases.
- **Lógica de salida repetida:** `main.cpp` vuelve a decidir si el valor está "fuera de alcance" (`cm < 0` / `cm > 30`) en lugar de usar la zona ya calculada. Para valores mayores a 30 cm imprime dos líneas (la distancia y "FUERA DE ALCANCE").
- **Los enlaces del informe al código no funcionan:** apuntan a rutas locales `file:///c:/Users/Maria/OneDrive/...`, que no se abren desde GitHub.
- El repositorio incluye **unos 150 archivos de herramientas de agentes** (`.agent/`, `_bmad/`, `_bmad-output/`) que no forman parte del proyecto.

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Plan formal** para RF (P-RF1 a P-RF4) y RNF (P-NF1 a P-NF4), con procedimiento y criterio de aceptación para cada uno.
- **5 pruebas Unity sobre las fronteras exactas** (−0,01; 0,0; 9,99; 10,0; 10,01; 19,99; 20,0; 20,01; 29,99; 30,0; 30,01; 400). El código y el log de ejecución están en el repositorio (`docs/anexos/logs-pruebas-unitarias.txt`).
- **§4.4 anticipa las fuentes de error esperadas:** zona ciega < 2 cm, pérdida de eco y efecto de la temperatura sobre la velocidad del sonido.
- **Evidencia útil de las fronteras:** las fotos de `docs/anexos/` combinan el montaje con una captura del monitor serie:
  - 9,84 cm → rojo y 10,15 cm → amarillo (frontera de 10 cm);
  - 19,94 cm → amarillo y 20,04 cm → verde (frontera de 20 cm).

**Por qué no llega a 4**
- **Exactitud con una sola lectura por punto.** La tabla de P-NF2 tiene un valor por distancia, sin repeticiones, dispersión ni captura que lo respalde. Los errores son muy parejos (+0,12 / −0,12, +0,21 / −0,21). Además, la columna "Error absoluto" lleva signo (−0,12, −0,21), cuando el error absoluto es siempre positivo.
- **El tiempo de respuesta no tiene método.** Se declara "≈ 110 a 130 ms medido" sin decir con qué instrumento. P-NF3 dice "medir el tiempo transcurrido", pero no es posible distinguir 110 ms de 130 ms a simple vista ni con cronómetro manual.
- **Afirmaciones sin evidencia:**
  - P-RF2 dice que las transiciones ocurrieron "exactamente en 10,0 cm, 20,0 cm y 30,0 cm". Las fotos respaldan 10 y 20 cm, pero **no hay evidencia de la frontera de 30 cm ni del estado OutOfRange**.
  - La estabilidad ("15 minutos, más de 8.500 lecturas, 0 reinicios") y el muestreo ("97 líneas en 10 s") **no tienen log ni captura**.
- **Las evidencias no están vinculadas al informe.**
  - El Anexo B dice que en `docs/anexos/` "se encuentran **reservadas** las ubicaciones para las fotografías", aunque las 4 fotos ya están en el repositorio. El informe no las enlaza ni explica qué demuestra cada una.
  - `docs/anexos/README.md` mantiene la lista de evidencias pendiente (`[ ]`), con nombres de archivo que no coinciden con los subidos. Faltan la foto general, la de fuera de rango y la del monitor serie con "FUERA DE ALCANCE".
- **No hay análisis de los errores medidos:** §4.4 trata los errores *esperados*, pero no se comparan con los obtenidos. Tampoco se explica la diferencia entre la tabla (10,15 cm a 10 cm y 20,21 cm a 20 cm) y las fotos (9,84/10,15 y 19,94/20,04).

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Resultados cuantificados:**
  - tabla de exactitud con error absoluto y relativo (máximo 0,28 cm);
  - estabilidad de 15 minutos;
  - 9,7 lecturas/s;
  - tiempo de respuesta de 110–130 ms;
  - 5 de 5 pruebas unitarias.
- **Conclusiones organizadas por tipo de saber** (conceptual, procedimental y actitudinal), coherentes con el trabajo realizado.
- **Recomendaciones técnicas y pertinentes:**
  - divisor o conversor de nivel en ECHO;
  - histéresis o mediana móvil en las fronteras;
  - fijación mecánica del sensor;
  - extensión al servomotor (variante B) reutilizando `DistanceZone`.
- Anexos C y D con la salida de pruebas y de compilación.

**Por qué no llega a 4**
- **El análisis crítico es limitado.** Los resultados se presentan como confirmaciones ("diez veces por debajo del límite", "reacción inmediata"), sin discutir limitaciones: muestra de una lectura, rango probado solo hasta 30 cm, método del tiempo de respuesta y frontera de 30 cm no verificada. Las conclusiones hablan sobre todo de lo aprendido y poco sobre lo que los datos permiten afirmar.
- **Anexos incompletos o inconsistentes:**
  - El Anexo A remite al diagrama Mermaid de §2.2 en lugar de un esquema.
  - El Anexo B describe las fotos como "reservadas".
  - Los anexos C y D se presentan como "Salida **Real**", pero **no coinciden con los logs del repositorio**: 1,30 s frente a 2,96 s en las pruebas, y 270 669 B / 20,7 % frente a 270 649 B / 20,6 % en la compilación.
- **Inconsistencias entre README e informe:** el README dice 9,5 lecturas/s y el informe 9,7. El README indica "Fecha de Entrega: 10 de Septiembre" y enlaza `docs/documentacion-tecnica-sistema.md`, un archivo que no existe en esa ruta.

---

## 4. Recomendaciones para próximas entregas

1. **Respaldar cada número con evidencia en el repositorio:** captura o log del monitor serie para exactitud (con al menos 5 lecturas por punto), muestreo (con marcas `millis()`) y estabilidad (inicio, fin y número de lecturas).
2. **Medir el tiempo de respuesta con un método explícito**, por ejemplo marcas de tiempo en el puerto serie al cambiar de zona, o video a 60 fps contando cuadros.
3. **Enlazar las fotos en el Anexo B**, una por caso, con su explicación. Completar las evidencias de la frontera de 30 cm y del estado OutOfRange.
4. **Corregir el rango de trabajo:** tratar `d < 2 cm` como inválida (OutOfRange) para que coincida con RNF2 y con la zona ciega del sensor.
5. Centralizar umbrales y constantes físicas en `constexpr`, usar la zona calculada para el mensaje "FUERA DE ALCANCE" y unificar el idioma de los nombres.
6. Implementar el divisor de ECHO en el prototipo, no solo recomendarlo.
7. Usar rutas relativas en los enlaces del informe y sacar del repositorio las herramientas de agentes.

---

## 5. Nota para la Defensa Individual

- **Katherine Montaño Mejia no tiene commits.** Conviene comprobar su aporte en la defensa.
- El repositorio y el informe muestran un flujo de trabajo apoyado en agentes (BMAD: *brief*, PRD, arquitectura). Conviene comprobar que cada integrante pueda explicar:
  - de dónde sale `0.01723` (velocidad del sonido / 2 en cm/µs);
  - por qué `millis()` en lugar de `delay()`, y qué pasaría con menos de ~60 ms entre disparos;
  - cómo se midió el tiempo de respuesta de 110–130 ms;
  - qué pasa con una lectura de 1 cm según el código actual;
  - el riesgo de conectar ECHO (5 V) directo al GPIO27.
