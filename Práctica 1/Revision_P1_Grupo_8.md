# Revisión del Informe Técnico — Práctica 1

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de sensores y actuadores en un objeto inteligente
**Repositorio:** https://github.com/santiagotorresv/ultrasonic-sensors-leds-g8
**Informe:** `INFORME.md`
**Integrantes (según el informe):** Santiago Javier Torres Vacaflores · Ariel Adrian Mercado Alegre (Grupo 8)
**Fecha límite:** 15-Sep-2026, 17:45 (UTC-4)
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **4** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **3** / 4 |
| **Total** | **13 / 16** |
| **Nota del componente** = (13 / 16) × 100 | **81,25** |

---

## 2. Estado del repositorio en la fecha límite

| Commit | Push (UTC-4) | Contenido |
|---|---|---|
| `e1f0ff2`, `a2c95c3` | 13-Sep 22:10 | Prototipo y refactorización a clases con parpadeo no bloqueante |
| `e9127e7` | 14-Sep 19:10 | Informe y diagrama del circuito |
| `9573da2`, `ffdf620`, `5a26a46` | 14-Sep 20:21–21:08 | Resultados de pruebas y evidencias |
| `010b3f9` | **14-Sep 23:06** | Informe final |

- **Entrega dentro de plazo**, casi 19 horas antes del límite. Se evalúa `010b3f9`, que es el estado actual.
- Las pruebas se hicieron con el firmware `a2c95c3`. Hasta la versión final solo se agregaron **6 líneas de comentarios**, sin cambios de lógica, así que **el firmware probado es el mismo que se entregó**.
- Todas las evidencias (PDF con capturas del monitor serie, fotos y esquema SVG) están **dentro del repositorio**.
- **Los 7 commits son de Santiago Torres.** Ariel Mercado fue agregado como colaborador el 15-Sep a las 11:24, después del último commit.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Requerimientos completos y medibles:**
  - cada RF tiene criterio de aceptación y **elemento responsable** (clase o archivo);
  - cada RNF tiene un valor medible y un **método de verificación previsto**;
  - RNF2 delimita el intervalo validado (3–30 cm) en lugar de afirmar exactitud en todo el rango del sensor.
- **Rangos explícitos y justificados:** `(0,5]`, `(5,15]`, `(15,25]` y `> 25`, con una aclaración sobre a qué rango pertenece cada límite y un estado `Invalido` con los LEDs apagados.
- **Matriz de trazabilidad** (§2.6) que vincula cada requisito con el diseño, la implementación y la evidencia.
- **El divisor de tensión de ECHO está diseñado, calculado e implementado:**
  - 2,2 kΩ arriba y 2 × 2,2 kΩ abajo;
  - 5 V × 4,4 / 6,6 ≈ 3,33 V;
  - aparece en la descripción del sistema, la tabla de conexiones, el diagrama Mermaid, el esquema SVG y el README.

  Es el único grupo revisado que resolvió el nivel de ECHO en el diseño **y en el montaje**, con documentación coherente.
- **Diagramas completos y coherentes con el código:**
  - arquitectura, que incluye el monitor serie;
  - circuito: Mermaid más el **esquema SVG** (`docs/diagramas/circuito.svg`) con GND común, VIN, divisor y resistencias de 220 Ω;
  - **clases**, que coincide con los atributos y métodos reales;
  - **secuencia** del ciclo principal, que muestra la medición cada 250 ms y la actualización continua del controlador.
- §2.1 analiza el problema en cuatro etapas antes del diseño.

**Observaciones (no cambian el nivel)**
- **Elección de señalización discutible:** en el rango `Lejos` (> 25 cm) **parpadean los tres LEDs juntos**, un patrón que suele asociarse a una alarma o error. El estado `Invalido` (todos apagados) no se distingue a simple vista de la fase apagada del parpadeo ni de un equipo sin energía. Faltó justificar estas decisiones.
- **`Lejos` no tiene límite superior** (cualquier distancia hasta el *timeout*, ~5 m), mientras que la exactitud solo se valida hasta 30 cm.
- El diagrama de clases no incluye los constructores (que reciben los pines) y menciona `ProgramaPrincipal` sin definirlo. No hay diagrama de estados del ciclo de parpadeo (encendido/apagado y cambio pendiente).

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Código limpio y modular:**
  - `SensorUltrasonico` y `ControladorLeds` reciben sus pines por constructor y usan lista de inicialización;
  - `enum class RangoDistancia`;
  - `main.cpp` solo coordina, con funciones auxiliares en un *namespace* anónimo.
- **Sin números mágicos:** pines, intervalos, *timeout*, velocidad del sonido, duración del pulso y límites de rango son `constexpr` con nombres descriptivos.
- **Control de tiempo no bloqueante:** muestreo cada 250 ms y parpadeo de 150 ms con `millis()`, y resta sin signo que funciona aunque `millis()` se desborde.
- **Lectura inválida bien tratada:** el sensor devuelve `NAN` y el controlador la clasifica con `isfinite()`, sin valores especiales como −1 o 400.
- `establecerDistancia()` solo reinicia el patrón si cambia el rango, y la bandera `cambioPendiente_` aplica el nuevo estado de inmediato en la siguiente actualización.
- El informe explica claramente la medición, la clasificación, el parpadeo y la coordinación (§3.3 a §3.6), además de las buenas prácticas aplicadas (§3.7).

**Por qué no llega a 4**
- **Poca documentación en el código:** solo hay 5 comentarios en todo el proyecto. Los `.h` no documentan la interfaz pública: no explican que `medirDistanciaCm()` devuelve `NAN` ante *timeout*, qué unidad espera `actualizar()` ni cómo se usa `cambioPendiente_`. RNF5 declara "comentarios breves y útiles", pero el contrato de las clases no está escrito en ninguna parte del código.
- **El informe no incluye fragmentos de código**, solo enlaces en el Anexo A. La consigna (§4.2) pide "código fuente documentado" en la sección 3.
- **Detalles menores:**
  - `actualizar()` usa `ultimoCambioMs_ = tiempoActual`, así que el periodo se alarga un poco cada vez que `pulseIn` bloquea el ciclo;
  - en estado `Invalido`, `apagarTodos()` escribe los tres GPIO en cada iteración de `loop()`;
  - el sensor no aplica un rango mínimo válido (por ejemplo, < 2 cm se acepta si `pulseIn` devuelve un valor).

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Sesión documentada:** fecha, integrantes, **commit del firmware** (`a2c95c3`), alimentación, instrumento de referencia, condiciones y forma de medir la distancia (desde la cara de los transductores).
- **Pruebas funcionales en 10 casos**, incluida la lectura inválida (sensor apuntando al techo), con la lectura promedio y el rango observado.
- **Exactitud con 45 lecturas** (5 por cada una de 9 distancias), con mínimo, máximo, promedio y **error máximo absoluto individual**, no solo el del promedio. El error absoluto promedio global (0,245 cm) está calculado de forma explícita.
- **Evidencia verificable en el repositorio:** el PDF tiene capturas del monitor serie con marcas de tiempo para cada distancia y para la lectura inválida, y coinciden con las tablas del informe.
- **Estabilidad inferida con buen criterio:** las marcas de `millis()` crecen de forma continua desde 142 752 ms hasta 1 298 003 ms. Como un reinicio habría vuelto `millis()` a cero, esto respalda que no hubo reinicios en esa ventana de 19 min 15 s.
- **Las limitaciones se informan con honestidad:** RNF3 y RNF4 se declaran expresamente como "validación lógica, no medición física".

**Por qué no llega a 4**
- **RNF3 y RNF4 no se midieron, aunque los datos estaban disponibles.** Las propias capturas muestran lecturas cada **250 ms** (142 752 → 143 002 → 143 252…), es decir **4 lecturas/s medidas**. El informe pudo usarlas para verificar RNF4 y aun así se limitó al cálculo teórico. Tampoco se midió el tiempo de respuesta, que la consigna (§3.2) pide verificar "en la sección de Pruebas y Validaciones".
- **Las pruebas de límite no prueban el límite:** se afirma que se probaron "los límites exactos de 5, 15 y 25 cm", pero las lecturas fueron **4,94, 14,75 y 24,61 cm**. Todas quedaron por debajo del umbral, así que no se comprobó el comportamiento con una lectura igual o apenas superior (por ejemplo, 5,01 cm → Amarillo). Faltó colocar el objeto de modo que la lectura cruzara cada umbral.
- **La estabilidad no tiene un registro continuo:** el PDF tiene 10 capturas de 5 líneas con huecos de varios minutos entre ellas. La ausencia de reinicios queda bien respaldada por `millis()`, pero no se pueden comprobar "bloqueos" o "interferencias" en esos huecos. Tampoco fue una prueba dedicada con condiciones constantes.
- **El parpadeo no se midió:** el informe reconoce que una foto no lo demuestra, pero no cuenta ciclos ni lo verifica de otra forma.
- La primera captura del PDF (3 cm) está **duplicada**.

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Resultados cuantificables** en una tabla (§5): rangos, lectura inválida, 19 min 15 s de estabilidad, error máximo de 0,63 cm y promedio de 0,245 cm, 280 ms en el peor caso teórico y 4 o 3,57 lecturas/s.
- **Conclusiones rigurosas:** no generalizan la exactitud fuera del intervalo probado y distinguen siempre lo medido de lo calculado. Esto muestra un criterio crítico poco habitual en las entregas revisadas.
- **Anexos completos y bien organizados en el repositorio:** código, diagramas, evidencias con índice (`docs/evidencias/README.md`, que explica qué demuestra cada archivo) y la consigna.

**Por qué no llega a 4**
- **Las recomendaciones no son técnicas:** casi todas son de procedimiento ("conservar los datos", "repetir las pruebas si se modifica el hardware", "registrar el instrumento"). Una incluso recomienda **"mantener sin filtros ni calibraciones adicionales"**. No hay propuestas de mejora del sistema, como:
  - histéresis en los umbrales 5/15/25 cm;
  - un límite superior para `Lejos`;
  - distinguir visualmente `Invalido` de `Lejos`;
  - documentar el código;
  - medir RNF3 y RNF4 con las marcas de tiempo que ya tienen.
- **El análisis no aborda las limitaciones de las pruebas de límite** (lecturas por debajo del umbral) ni las del diseño de señalización (§3.1).

---

## 4. Recomendaciones para próximas entregas

1. **Medir RNF3 y RNF4 con el registro serie:** la frecuencia sale directamente de las marcas de tiempo, y el tiempo de respuesta se puede medir moviendo el objeto entre zonas y registrando el instante del cambio.
2. **Hacer que la lectura cruce cada umbral** en las pruebas de límite (por ejemplo, 4,9 → 5,1 cm) y registrar las dos lecturas con su rango.
3. Hacer una prueba de estabilidad **dedicada**, con un registro serie continuo de ≥ 10 minutos guardado como archivo `.txt` o `.csv`.
4. Documentar la interfaz pública en los `.h` (contrato de `NAN`, unidades y uso de `actualizar()`) e incluir fragmentos de código en la sección 3 del informe.
5. Reconsiderar la señalización: diferenciar `Lejos` de un patrón de alarma, distinguir `Invalido` de "apagado" y definir un límite superior para el rango válido.
6. Escribir recomendaciones **técnicas** de mejora del sistema, no solo de procedimiento.

---

## 5. Nota para la Defensa Individual

- **Todos los commits son de Santiago Torres.** **Ariel Mercado** fue agregado al repositorio después del último commit. El informe lo menciona como responsable de las pruebas, pero conviene comprobar su aporte.
- Preguntas sugeridas:
  - ¿Cómo se calcula el divisor 2,2 kΩ / 4,4 kΩ y por qué es necesario?
  - ¿Por qué el tiempo de respuesta en el peor caso es ≈ 280 ms? ¿Cómo lo medirían?
  - ¿Qué hace `cambioPendiente_` y por qué no se reinicia el parpadeo en cada lectura?
  - ¿Cómo distingue un usuario el estado `Invalido` de la fase apagada del parpadeo?
  - ¿Por qué el rango `Lejos` hace parpadear los tres LEDs?
