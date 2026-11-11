# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/Sam-J-R/IoT_practica3
**Informe:** `docs/INFORME.md`
**Integrantes (según el informe):** Samuel Jarro Rodriguez · María Jesús Lozada Peralta · Katherine Montaño Mejia
**Caso de uso:** tendedero inteligente — sensor de agua MH (analógico) + servomotor MG90S para la cubierta
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **4** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **4** / 4 |
| Resultados, conclusiones y recomendaciones | **4** / 4 |
| **Total** | **16 / 16** |
| **Nota del componente** = (16 / 16) × 100 | **100** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 14:52 (UTC-4), commit `2ebb758` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | KathM00 (Katherine Montaño): 10 · Sam-J-R (Samuel Jarro): 4 · Maria Lozada: 3 |

- Se evaluó el commit **`2ebb758`**, entregado casi 3 horas antes del cierre.
- El trabajo se organizó con ramas por integrante (`maria-aws`, `kath-alexa`) y *pull requests*. El informe declara qué parte hizo cada uno, y el historial lo confirma: el firmware y las herramientas de ensayo son de Samuel, la Lambda y AWS de María, y Alexa, la integración y el informe de Katherine.
- No se encontraron certificados, claves privadas, credenciales WiFi ni *access keys* en el historial. `.gitignore` excluye también la calibración local y los originales de las capturas sin redactar.
- **Los anexos están completos dentro del repositorio:** 17 capturas de Alexa, 5 paneles de AWS, el CSV de CloudWatch, 2 videos de integración, el video de RNF3 y **los logs crudos de cada ensayo RNF, incluidas las sesiones fallidas** (`resultados/`).
- La raíz del repositorio tiene cuatro scripts de ensayo sueltos (`ensayo_rnf.py`, `grabar_rnf.py`, `analizar_rnf.py`, `orden_offline.py`). Conviene moverlos a una carpeta `tools/`.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Caso de uso real y bien delimitado:** proteger la ropa de la lluvia. El informe aclara qué mide el sensor MH (conductividad clasificada como seco o agua) y qué **no** mide (intensidad ni porcentaje de lluvia). También aclara que el montaje es de banco y no demuestra una cubierta bajo carga.
- **Requerimientos trazables** a la implementación y a la evidencia (capturas AX, videos INT, ensayos RNF).
- **RNF medibles, declarados antes de los ensayos** ("no se modificaron después de ver los resultados"), con criterios propios justificados: ≤ 5 % de error en ≥ 20 observaciones, respaldo ≤ 6 s, evento ≤ 1 s y reconexión ≤ 30 s.
- **Todos los diagramas exigidos:**
  - Arquitectura.
  - Circuito en SVG, con la justificación de cada conexión (ADC1 en GPIO34 para no chocar con el WiFi; el servo desde VIN y no desde el regulador de 3V3) y una foto del montaje.
  - Diagramas de clases del firmware y de la Lambda.
  - **Cuatro diagramas de secuencia:** vinculación, consulta, comando y recuperación del estado deseado.
- **Shadow especificado con precisión:** quién escribe cada atributo, `clientToken` para correlacionar respuestas, temas MQTT, y que `null` elimina un atributo y no equivale a "seco".
- **Vinculación** con alias normalizados y validación con `DescribeThing` antes de escribir. Se reconoce que no prueba la posesión física y se propone un código temporal.

**Observaciones menores (no bajan el nivel)**
- **El respaldo periódico de 5 s es alto para el caso de uso.** Da ≈ 17 000 reportes por día, y por dispositivo superaría la capa gratuita de mensajes en un mes de operación continua. El propio informe lo reconoce en §5.3 y propone 30–60 s. Lo correcto era declarar ese valor desde el diseño.
- La redacción es muy condensada, casi telegráfica en algunas tablas, y eso dificulta la lectura (por ejemplo, "Certificado→política y certificado→Thing son asociaciones distintas").

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Firmware modular:** `SensorAgua` (ADC con histéresis 200/800), `ServoCubierta` (asentamiento de 1 s), `ClienteShadow` (MQTT/TLS, GET al conectar, `delta`, `clientToken`, descarte por versión) y `Tendedero` como programa principal. Hay perfiles de prueba aislados (`SensorTest`, `ServoTest`, `SensorShadowTest`) y uno de ensayo (`tendedero_ensayo`). La calibración local queda fuera del repositorio.
- **Lambda en clases:** `BindingRepository` (DynamoDB con `ConsistentRead`), `ShadowService` (actualización parcial de `desired`, sin escribir `reported`) y handlers del ASK SDK, con `_run_safe` y un `ErrorHandler` global. El ZIP de despliegue se verificó contra el código fuente.
- **Configuración de AWS bien documentada:**
  - Política IoT de mínimo privilegio, que distingue `topic` de `topicfilter`.
  - Rol de la Lambda con la tabla exacta en DynamoDB y comodín en IoT. **El comodín se justifica** (el analizador de políticas de la cuenta mostraba acceso nulo con los ARN restringidos) y se indica el mínimo para producción (`thing/tendedero-*`), como pide la consigna.
  - Variables de entorno y permiso de invocación restringido al Skill ID.
- **Incidente del alias documentado:** el slot llegaba como "prueba 2" y no como "prueba dos". Se resolvió con variantes en `THING_ALIASES_JSON`, sin fijar un Thing.
- 12 pruebas locales (contrato estático, importación con el SDK del ZIP e integración con dos usuarios y dos Things).

**Observaciones menores**
- La resolución de nombres depende de un mapa de alias en una variable de entorno. Para cada Thing nuevo con un nombre hablado distinto hay que editar la configuración de la Lambda.
- `requirements.txt` no fija versiones. El propio informe lo recomienda en §7.

### 3.3 Pruebas y validaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **El informe distingue qué permite concluir cada tipo de evidencia** (§4.1): inspección, prueba local, evento directo, captura, video y ensayo automatizado.
- **Frases de la skill verificadas con capturas:** inicio, sin vínculo, vincular, persistencia entre conversaciones, Thing inexistente, desvincular, consultas de agua (seco y con agua), abrir y cerrar, y cambio válido.
- **Todos los RNF se midieron con datos crudos conservados:**
  - RNF1: 15 min con 15 órdenes y 14 consultas.
  - RNF2: 20 observaciones independientes con matriz de confusión.
  - RNF3: **10 órdenes desde la skill, grabadas en el mismo video que el servo y analizadas cuadro a cuadro**, con un promedio de 1.70 s y un máximo de 1.80 s.
  - RNF4: 54 intervalos.
  - RNF5: 3 reinicios y 3 pérdidas de WiFi, con el `desired` pendiente aplicado.
- **Honestidad en el reporte:** el reporte por evento se declara "**no cumple**" (1 de 15 tardó 1.033 s frente a ≤ 1 s), y la primera orden tras inactividad (6.47 s por arranque en frío) se informa aparte, sin descartarla.
- **Análisis de errores con las sesiones fallidas conservadas:** ensayo de WiFi interrumpido, reinicio sin punto de acceso, log cortado por disco lleno, intervalos de 6.97 s causados por las consultas del propio ensayo, y monitor MQTT rechazado por la política.

**Observaciones menores**
- Salvo RNF3, los ensayos se hicieron con el firmware de ensayo, donde **el propio ESP32 publica las órdenes** (sin pasar por Alexa ni Lambda), y la pérdida de red se simuló con una desasociación provocada. El informe lo declara como adaptación, pero RNF1, RNF4 y RNF5 no se verificaron con la cadena completa ni con el firmware final.
- Las pruebas se hicieron por texto, porque el micrófono no funcionó, y el multiusuario se probó con eventos directos, no con dos cuentas de Alexa. Ambas limitaciones están declaradas.

### 3.4 Resultados, conclusiones y recomendaciones — **4 / 4 (Excelente)**

**Fortalezas**
- **Resultados cuantificados** en una tabla de objetivo frente a medido, más el cálculo sobre el CSV de CloudWatch (`Duration` frente a `Init Duration`, sin confundirlo con el tiempo de punta a punta) y una figura cuadro a cuadro de RNF3.
- **Análisis crítico:** un balance de lo que cada evidencia sustenta y lo que no (voz, cubierta bajo carga, posición estimada), y un análisis del consumo de mensajes con la propuesta de reducirlo entre 6 y 12 veces.
- **Conclusiones fundamentadas**, que incluyen los incumplimientos (evento de 1.033 s, arranque en frío).
- **Recomendaciones técnicas y de extensión a un uso real:** calibrar con gotas y humedad leve; concurrencia aprovisionada o *warm-up* de la Lambda; finales de carrera y fuerza con la cubierta real; antigüedad del reporte; prueba de posesión; corrosión del sensor y protección ambiental; fijar las dependencias; alerta de facturación.
- La tabla de correspondencia con el enunciado (§8.6) facilita la revisión.

---

## 4. Recomendaciones para próximas entregas

1. **Validar los RNF con la cadena completa y el firmware final.** El firmware de ensayo es útil para automatizar, pero al menos una corrida de estabilidad y una de reconexión deberían usar órdenes reales desde la skill y cortar el punto de acceso (no una desasociación provocada).
2. **Declarar desde el diseño una frecuencia sostenible.** Si el propio análisis concluye que 30–60 s es lo adecuado, conviene implementarlo y medirlo, no solo recomendarlo.
3. **Mejorar la legibilidad.** El informe es técnicamente sólido, pero varias tablas y párrafos están tan condensados que cuesta leerlos. Frases completas y algo más de contexto ayudan al lector.
4. Probar el multiusuario con **dos cuentas de Alexa reales** y, si es posible, con la voz.
5. Configurar la alerta de facturación (el informe dice que no hay evidencia de que exista).

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Sensor:** por qué se usa GPIO34 (ADC1) y no un pin del ADC2, qué es la histéresis 200/800 y por qué no se reporta cada fluctuación del ADC, y qué significa la atenuación de 11 dB.
- **Servo:** por qué se alimenta desde VIN, qué es el asentamiento de 1000 ms y por qué `reported.cubierta` es una "posición estimada".
- **Shadow:** para qué sirve el `clientToken`, cómo se descartan los `delta` antiguos por versión, qué hace el ESP32 al reconectar y qué pasa si se escribe `null` en un atributo.
- **Lambda y permisos:** por qué se usó el comodín en las acciones de IoT, qué mostraba el analizador de políticas, cuál sería el mínimo y por qué `ConsistentRead=True`.
- **Alias:** qué pasó con "prueba dos" frente a "prueba 2", cómo funciona la normalización y por qué se valida con `DescribeThing` antes del `PutItem`.
- **Mediciones:** cómo se midió RNF3 cuadro a cuadro, por qué la primera orden tardó 6.47 s y cómo se interpreta el evento de 1.033 s.
- **Aporte individual:** la mayoría de los commits son de Katherine (informe e integración), pero el firmware es de Samuel y la Lambda de María. Cada integrante debe poder explicar las partes de los otros dos.
