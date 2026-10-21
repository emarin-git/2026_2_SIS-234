# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/Sam-J-R/IoT_practica_2
**Informe:** `INFORME_TECNICO.md`
**Integrantes (según el informe):** María Jesús Lozada Peralta · Samuel Jarro Rodriguez · Katherine Montaño Mejia
**Fecha límite:** martes 13-Oct-2026
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

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 16:42 (UTC-4), merge `6731796` (PR #1 `maria-correcciones`) |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Sam-J-R (Samuel Jarro): 6 · Maria Lozada: 4 · KathM00 (Katherine Montaño): 3 |

- Se evaluó el commit **`6731796`**.
- Están en el repositorio las evidencias de software (salidas de pytest y Unity, compilaciones) y **5 fotografías** del prototipo. **No hay registros de la sesión física** (el JSONL del servidor ni el monitor serie) que respalden RNF1, RNF3 y RNF4. El único `.jsonl` del repositorio es de una prueba de software con `ncat`.
- Hay cuatro carpetas de evidencia de software casi duplicadas (`2026-10-13`, `2026-10-13-cierre-codigo`, `20261013T174238…`, `20261013T174415…`), una de ellas con un resultado `FALLIDO` que hay que leer junto con su reintento. Conviene dejar solo la evidencia final.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **14 RF y 12 RNF**, cada uno con su criterio de aceptación, y una **matriz de trazabilidad** con componente, prueba y resultado.
- Rangos contiguos y sin solapamiento, con los límites asignados de forma explícita (`[2,10]`, `(10,20]`, `(20,30]`). La zona ciega (< 2 cm) se corrigió respecto de la Práctica 1.
- **Protocolo muy bien especificado:** formato `COMANDO clave=valor`, reglas de espacios, límite de 128 bytes, números de secuencia, confirmaciones `RECIBIDO` (lectura procesada) y `APLICADO` (orden aplicada en los GPIO) y 14 códigos de error.
- Arquitectura con el punto de acceso y los GPIO, circuitos en SVG, diagramas de clases de C++ y Python, secuencia normal y secuencia de excepción. Se analizan la concurrencia y las temporizaciones (§2.5, §2.7 y §2.8).
- El Anexo C muestra cómo se aplicaron las observaciones de la Práctica 1.

**Por qué no llega a 4**
- **ECHO a 5 V conectado directo a GPIO 27**, sin divisor. El propio SVG se llama "Circuito del sensor **sin resistor en Echo**". El informe reconoce el riesgo, pero lo deja como recomendación "para despliegues definitivos". Al mismo tiempo, **RNF11 (seguridad eléctrica) se declara "verificado" y "cumplido"**, lo cual es contradictorio: el pin recibe una tensión mayor que la máxima del fabricante.
- **Estado de error indistinguible:** fuera de rango (> 30 cm), sin eco, falla del sensor y caída del servidor terminan todos en `APAGADO` (los tres LEDs apagados). Para un observador, "objeto lejos", "sensor dañado" y "actuador sin alimentación" se ven igual. Faltó analizar esta decisión.
- El rango de trabajo es muy estrecho (2–30 cm) y no se justifica. Más allá de 30 cm, el sistema no informa nada.
- Muchos RF y RNF (RF9 a RF14, RNF6 a RNF12) describen la **implementación** (colas, contadores de 32 bits, hilos, tamaño de tramas) más que comportamientos observables del sistema, y diluyen los requerimientos principales.
- El Anexo E autoevalúa los diagramas como "Nivel 4 — Excelente". La calificación corresponde al docente; ese anexo no aporta al informe técnico.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Servidor Python bien estructurado** en módulos (`control.py`, `protocolo.py`, `registro.py`, `servidor.py`). Usa hilos separados para recepción, transmisión, supervisión y registro, colas acotadas y `RLock`. La orden `APAGADO` tiene prioridad sobre las órdenes pendientes.
- **Filtro de mediana de 3 muestras** en el servidor, que se vacía con un dato inválido.
- **Pruebas automatizadas amplias:** 27 en Python y 17 en Unity, con dobles de Arduino, WiFi, reloj y GPIO. Hay un analizador de registros JSONL.
- *Fail-safe* local de 3 s en el actuador y reconexión automática. Credenciales fuera del repositorio (`ConfiguracionRedEjemplo.h` + `__has_include`).

**Por qué no llega a 4**
- **El firmware no sigue el mismo nivel de diseño que el servidor:**
  - `firmware/src/sensor/main.cpp` tiene números mágicos (`SensorUltrasonico sensor(14, 27)`) y varias variables globales de estado (`numeroLectura`, `ultimaRecibida`, `inicioEsperaConfirmacion`, `estabaRegistrado`) que deberían estar encapsuladas en una clase.
  - `ClienteTcp` está implementado por completo dentro del `.h` (161 líneas), sin separación `.h`/`.cpp`.
- **Documentación del firmware escasa:** `IndicadorLeds.cpp` no tiene comentarios, y `main.cpp` del sensor y del actuador tienen 2 cada uno.
- La §4.1 dice que las pruebas se ejecutaron "sin agregar casos ni modificar código", una frase propia de un agente automático, no de un equipo que describe su trabajo.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Pruebas de software muy completas (44 casos) con sus registros de ejecución en el repositorio.
- Plan de validación física (F1 a F5) con procedimiento y criterio de aceptación, y definición formal del error firmado, absoluto y relativo.
- Fotografías del prototipo con regla en cada punto medido.

**Por qué baja a 2**
- **Exactitud (RNF2) con una sola lectura por punto.** Hay 5 mediciones en total (2, 5, 10.1, 15 y 20 cm). Con una lectura por punto no se puede conocer la dispersión del sensor ni el error máximo real.
- **No se probó el rango VERDE (20–30 cm).** Ningún punto de medición ni fotografía muestra el LED verde. Tampoco se probaron las fronteras exactas de 10 cm (se usó 10.1 cm) ni de 30 cm. RF2 y RF3 quedan sin verificar en uno de los tres rangos.
- **Tiempo de respuesta (RNF3) sin datos:** solo se declara "< 400 ms". No hay una tabla de mediciones individuales ni un log. Además, el método (§4.5) correlaciona "marcas de tiempo monotónicas" del sensor, del servidor y del actuador, que son **relojes independientes no sincronizados**. El desglose (~100 ms + 100–200 ms + ~10 ms + ~1 ms) es una **estimación**, no una medición.
- **Estabilidad (RNF1) y muestreo (RNF4) sin evidencia:** se afirma una "sesión continua registrada en JSONL" de 15 min y "≥ 8,5 lecturas válidas/s", pero ese JSONL no está en el repositorio y no se muestra ningún conteo.
- **Recuperación (F5)** se describe como "comportamiento verificado en consola serie", sin tiempos medidos ni registros.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay una tabla de resultados de exactitud con error absoluto y relativo, y una tabla resumen de los RNF.
- Las recomendaciones son técnicas: nivel lógico en ECHO, histéresis en las fronteras, fijación mecánica del sensor y TLS.

**Por qué baja a 2**
- **Las conclusiones exceden la evidencia:** la conclusión 4 afirma que el prototipo "cumple **plenamente** con todos los requerimientos funcionales (RF1–RF14) y no funcionales (RNF1–RNF12)". Sin embargo, el rango verde no se probó, RNF3 no tiene mediciones, RNF1 y RNF4 no tienen registros y RNF11 se contradice con el circuito.
- **No hay análisis crítico.** El error es siempre positivo y crece con la distancia (+0.12 → +0.28 cm), lo que sugiere un error sistemático, pero no se analiza. Tampoco se discute el error relativo de 6 % a 2 cm.
- Se usa un lenguaje promocional ("alta resiliencia operativa", "validación rigurosa", "más de diez veces por debajo") en lugar de un análisis de los límites del sistema.
- La recomendación 2 propone una histéresis de ±0.5 cm sin haber medido la dispersión del sensor (con una lectura por punto, no hay datos para dimensionarla).

---

## 4. Recomendaciones para próximas entregas

1. **Probar los tres rangos y todas las fronteras** (10, 20 y 30 cm exactos, y valores a ambos lados), con evidencia de cada LED.
2. **Tomar varias lecturas por punto** (al menos 10) e informar mínimo, máximo, promedio y error máximo individual.
3. **Medir el tiempo de respuesta con un solo reloj** (por ejemplo, el servidor: desde la `LECTURA` que cambia el rango hasta el `APLICADO`) y presentar los valores de cada repetición.
4. **Subir al repositorio el JSONL de la sesión física** y usar `analizar_registro.py`, que ya existe, para calcular la frecuencia y la estabilidad a partir de él.
5. **Corregir el circuito (divisor en ECHO)** antes de declarar cumplida la seguridad eléctrica.
6. Elegir un estado de **error distinguible** del estado "fuera de rango".
7. Llevar el firmware al mismo nivel que el servidor: encapsular el estado global en clases, separar `.h`/`.cpp`, usar constantes con nombre para los pines y comentar las clases.
8. Escribir conclusiones proporcionales a la evidencia y dejar fuera del informe la autoevaluación con la rúbrica.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Servidor concurrente:** explicar los hilos de recepción, transmisión, supervisión y registro; qué protege el `RLock`; por qué las colas son acotadas; cómo la orden `APAGADO` adelanta a las órdenes pendientes.
- **Protocolo:** diferencia entre `RECIBIDO` y `APLICADO`; qué pasa si llega un `APLICADO` con un número de secuencia incorrecto; por qué el límite de 128 bytes; para qué sirve reiniciar la sesión antes del desbordamiento del contador de 32 bits.
- **Filtro de mediana:** cómo funciona con 3 muestras, por qué un dato inválido vacía la ventana y cuánto retardo agrega.
- **Circuito:** por qué se conectó ECHO directo a 5 V; qué dice la hoja de datos del ESP32 sobre la tensión máxima; cómo se calcula un divisor 1 kΩ / 2 kΩ.
- **Pruebas físicas:** cómo se midió el tiempo de respuesta de "< 400 ms" (instrumento y valores) y dónde está el registro de la sesión de 15 min. Por qué no se probó el LED verde.
- **Pruebas Unity:** qué es un "doble" de Arduino o WiFi y cómo permite probar el firmware en la PC.
- **Aporte individual:** María Lozada (4 commits) subió la implementación inicial con las pruebas Unity (≈ 3000 líneas en un commit), la refactorización a `.h`/`.cpp` y varias correcciones; Samuel Jarro (6) hizo la versión del servidor con hilos, la unificación en un solo proyecto y la limpieza; Katherine Montaño (3) hizo el informe y las fotografías. Cada integrante debe poder explicar también el código de los demás, en especial el servidor (410 líneas en `servidor.py`) y `ClienteTcp`.
