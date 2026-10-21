# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/JustAdrianMercado/Integracion-de-Objetos-Inteligentes-con-TCP-IP
**Informe:** `INFORME.md`
**Integrantes (según el informe):** Santiago Javier Torres Vacaflores · Ariel Adrian Mercado Alegre
**Fecha límite:** martes 13-Oct-2026
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **1** / 4 |
| Resultados, conclusiones y recomendaciones | **1** / 4 |
| **Total** | **8 / 16** |
| **Nota del componente** = (8 / 16) × 100 | **50** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 16:42 (UTC-4), merge `b008e9e` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Santiago Torres: 13 (documentación) · Adrián Mercado: 11 (código) |

- Se evaluó el commit **`b008e9e`**.
- El trabajo se organizó con ramas y *pull requests* (`ft/application-protocol`, `ft/tcp-server`, `ft/sensor-client`, `ft/actuator-client`, `desarrollo-informe`), lo cual es una buena práctica.
- **El informe se entregó sin terminar.** El encabezado dice: "evidencias, integración y validación física **pendientes de acreditación**". El documento tiene **112 marcadores** `[PENDIENTE DE MEDICIÓN]`, `[PENDIENTE DE EVIDENCIA]` o `[PENDIENTE DE IMPLEMENTACIÓN]`, y la §4 indica: "**No se ejecutaron pruebas en esta iteración ni se presentan resultados físicos obtenidos**".
- Las evidencias de `docs/evidencias/` (fotos y `Pruebas de rangos.pdf`) son **de la Práctica 1**, y el propio informe aclara que no se usan como prueba de la Práctica 2.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- RF1–RF4 con criterio de aceptación, componente responsable y estado. RNF1–RNF5 con criterio, **método de verificación detallado** y alcance. Para RNF3 se aclara correctamente que **no se pueden restar *timestamps* de relojes no sincronizados**.
- Rangos contiguos y sin solapamiento (`(0,50)`, `[50,100)`, `[100,+∞)`), con comportamientos distintos (4 Hz, 2 Hz y fijo) y frecuencias de parpadeo definidas con precisión.
- **Circuito del sensor correcto:** divisor de ECHO con R1 = 2.2 kΩ y R2 + R3 = 4.4 kΩ, con el cálculo (≈ 3.33 V). Es de los pocos grupos que lo incorporan en el diseño. Esquemas en SVG.
- Diagramas de arquitectura, de clases (servidor Python, firmware y biblioteca C++), de secuencia (funcionamiento normal y fallas/recuperación) y matriz de trazabilidad.
- **Protocolo bien especificado:** codificación, delimitación, tamaño máximo (128 bytes), reconstrucción del flujo, mensajes con ejemplos, representación decimal y políticas de sesión.

**Por qué no llega a 4**
- El informe reconoce **diferencias no resueltas entre el diseño y la implementación**. Por ejemplo, el plazo de 2000 ms desde el último `POST` completo "no equivale" al `settimeout()` por operación del servidor. Hay una "diferencia de supervisión pendiente de revisión técnica". El nivel 4 exige que el diseño explique totalmente el sistema que se implementó.
- Los circuitos se presentan como "**propuestos**" y "no acreditan el montaje físicamente verificado".
- El estado de lectura inválida (todos los LEDs apagados) no se distingue de un actuador sin alimentación.
- La redacción, muy cargada de matices ("según declaración del equipo", "sin equipararlas con el cumplimiento", "no acredita") dificulta la lectura. Un informe técnico debe afirmar lo que se hizo.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Buena arquitectura de código:** biblioteca C++ compartida (`lib/Compartido/`: `ProtocoloAplicacion`, `ReceptorLineas`, `Tipos.h`) usada por los dos firmwares sin duplicar código; `ClienteTcp` común; `SensorUltrasonico` y `ControladorLeds` encapsulan pines y temporización.
- Servidor Python separado en `dominio.py` (clasificación), `protocolo.py` y `servidor.py`.
- Constantes con nombre (`PIN_DISPARO`, `INTERVALO_MEDICION_MS`), credenciales fuera del repositorio (`ConfiguracionRed.example.h` + `__has_include`) y un único proyecto PlatformIO con dos entornos.
- **Pruebas automatizadas extensas:** 191 comprobaciones C++ nativas con *fakes* de Arduino, WiFi y sockets, y 26 métodos Python.

**Por qué no llega a 4**
- **La implementación no cubre todos los requerimientos declarados**, según el propio informe: la supervisión del servidor difiere del diseño, `sendall()` se ejecuta con el bloqueo del estado tomado (un cliente lento puede retrasar a los demás) y el actuador puede descartar estados intermedios.
- **Documentación del código escasa:** `ReceptorLineas.cpp` no tiene comentarios, y `PrincipalSensor.cpp` y `SensorUltrasonico.cpp` tienen 2 cada uno.
- No hay evidencia de ejecución de las pruebas: el informe dice que "no encontró registros verificables" del reporte 7/7 del equipo.

### 3.3 Pruebas y validaciones — **1 / 4 (Necesita mejorar)**

**Lo que hay**
- Un **plan de pruebas bien diseñado**: 16 casos funcionales (PV-01 a PV-16), 4 ensayos no funcionales (PN-01 a PN-04) con criterio y método, y tablas preparadas para registrar los resultados.
- Inventario de las pruebas automatizadas disponibles (191 comprobaciones C++ y 26 métodos Python).

**Por qué baja a 1**
- **No se ejecutó ninguna prueba física.** Todas las celdas de resultado de PV-01 a PV-16 y de PN-01 a PN-04 dicen `[PENDIENTE DE MEDICIÓN]`. No hay datos de estabilidad, exactitud, tiempo de respuesta ni frecuencia de muestreo.
- **Tampoco hay registros de la ejecución de las pruebas automatizadas**: el 7/7 se cita como "reporte del equipo sin registros verificables".
- Un plan sin ejecución no permite verificar ningún requerimiento. Corresponde al nivel "pruebas insuficientes". No baja a 0 porque el plan y el código de pruebas sí se entregaron.

### 3.4 Resultados, conclusiones y recomendaciones — **1 / 4 (Necesita mejorar)**

**Lo que hay**
- La tabla de cumplimiento (§5.1) y el análisis de limitaciones (§5.2) identifican con honestidad los riesgos de la implementación (*timeout* por operación, envíos con bloqueo, último `SET` pendiente).
- Las recomendaciones se basan en los hallazgos y tienen prioridad y validación propuesta.

**Por qué baja a 1**
- **No hay resultados medibles.** Todos los RF y RNF quedan como "pendiente" o "validación física pendiente", y el informe lo declara: "**No se declara ningún RF ni RNF plenamente validado**".
- Las conclusiones describen la estructura del software y repiten que "no acredita todavía un prototipo físicamente validado". La sección de conclusiones experimentales es un marcador `[PENDIENTE]`.
- La mayoría de las recomendaciones son, en realidad, **tareas pendientes de esta misma práctica** ("realizar PN-01 a PN-04", "verificar cableado", "conservar salida completa de las suites"), no mejoras técnicas a futuro.
- El índice de evidencias de la Práctica 2 (§8.2) contiene solo "identificadores reservados" que, según el informe, "**no representan archivos existentes**".

---

## 4. Recomendaciones para próximas entregas

1. **Planificar el tiempo para las pruebas físicas.** El código se terminó el mismo día de la entrega (13-Oct, 12:33) y no quedó tiempo para medir. El montaje y las pruebas deberían empezar al menos 2 o 3 días antes.
2. **Ejecutar el plan que ya está diseñado.** PV-01 a PV-16 y PN-01 a PN-04 están bien definidos; solo falta ejecutarlos y registrar los datos y las evidencias en `docs/evidencias/p2/`.
3. **Guardar la salida de las pruebas automatizadas** (`ctest`, `pytest`) en el repositorio, con la fecha y el commit.
4. **Resolver las diferencias entre el diseño y la implementación** antes de entregar: o se ajusta el código (*timeout* desde el último mensaje completo, enviar fuera del *lock*) o se actualiza el diseño justificando el cambio.
5. **No entregar un informe con marcadores de texto pendiente.** Si algo no se hizo, basta con decirlo una vez con claridad; 112 marcadores dificultan la lectura y no aportan información.
6. Redactar en primera persona del equipo y con afirmaciones directas sobre lo realizado.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

Como el informe no tiene resultados físicos, en la defensa se preguntará **si el sistema funciona realmente** y cómo lo comprobaron. Se recomienda preparar:

- **Estado real del prototipo:** si el sistema completo funcionó (sensor → servidor → actuador por WiFi), cuándo, en qué red y con qué resultado. Si se hizo alguna medición después de la entrega, se puede mostrar, pero no modifica la calificación del informe.
- **Divisor de ECHO:** cálculo con 2.2 kΩ y 4.4 kΩ; por qué hace falta y qué pasaría sin él.
- **Protocolo:** formato de `REGISTER`, `POST` y `SET`; por qué la distancia inválida se codifica como `invalid` y no como 0; cómo `ReceptorLineas` reconstruye mensajes fragmentados o agrupados; por qué el límite de 128 bytes.
- **Servidor:** cómo se maneja la concurrencia; qué problema tiene llamar a `sendall()` con el bloqueo tomado; en qué se diferencia `settimeout()` por operación de un plazo desde el último mensaje completo.
- **Actuador:** por qué consume solo el último `SET` completo pendiente y qué consecuencias tiene; cómo `ControladorLeds` conserva la fase del parpadeo ante comandos repetidos.
- **Pruebas nativas C++:** cómo funcionan los *fakes* de `Arduino.h`, `WiFi.h` y `lwip/sockets.h`.
- **Aporte individual:** según el historial, Adrián Mercado escribió todo el código (11 commits) y Santiago Torres toda la documentación (13 commits). Por eso **Santiago debe demostrar en la defensa que puede explicar y modificar el código** (firmware y servidor), y **Adrián debe poder explicar los diagramas, el protocolo y el diseño del circuito**.
