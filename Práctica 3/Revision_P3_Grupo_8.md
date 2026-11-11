# Revisión del Informe Técnico — Práctica 3

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de un objeto inteligente con Alexa mediante MQTT y AWS IoT Core
**Repositorio:** https://github.com/JustAdrianMercado/Practica3
**Informe:** `INFORME.md`
**Integrantes (según el informe):** Santiago Torres · Adrián Mercado · Mateo Rojas · Lucía Escobar
**Caso de uso:** "Caja de los Enigmas" (Escape Room) — sensor magnético *reed switch* + servomotor SG90 como cerradura
**Fecha límite:** martes 10-Nov-2026, 17:45
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **4** / 4 |
| Desarrollo e implementación (código y documentación) | **4** / 4 |
| Pruebas y validaciones | **2** / 4 |
| Resultados, conclusiones y recomendaciones | **2** / 4 |
| **Total** | **12 / 16** |
| **Nota del componente** = (12 / 16) × 100 | **75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 10-Nov 17:41 (UTC-4), commit `176abf6` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | Adrián Mercado: 11 (estructura, firmware de conectividad, informe) · Santiago Torres: 7 (hardware, servo, sensor, circuito) · Mateo Rojas: 4 (Lambda, pruebas, evidencias AWS) · `unknown` (correo de Lucía Escobar): 1 (modelo de Alexa) |

- Se evaluó el commit **`176abf6`**.
- **Buena organización del trabajo:**
  - Planificación por iteraciones y por integrante (`docs/planificacion/`).
  - Contratos compartidos entre capas (`docs/contratos.md`).
  - Ramas por área (`feat/hardware`, `feat/firmware-aws`, `feat/lambda`, `lucia`) y *pull requests*.
- **El aporte de Lucía Escobar en el historial es un solo commit** (el modelo de Alexa), y aparece con autor `unknown` porque Git no tenía configurado el nombre de usuario. El informe le atribuye "pruebas de frases y coordinación de la redacción del informe", pero los dos commits del informe son de Adrián Mercado.
- El repositorio incluye `AGENTS.md` con instrucciones para agentes de IA (Codex, Claude Code) y reglas de código limpio. Usar herramientas es válido, pero varias frases del informe se escribieron desde el punto de vista del agente, por ejemplo "Las respuestas comunicadas por el usuario…" (§3.5) o "esta revisión documental no cambió la lógica del proyecto" (§4.8).
- No se encontraron certificados, claves privadas, credenciales WiFi ni *access keys* en el historial.
- **Se repite la observación de la Práctica 2:** el diseño y el código son de buen nivel, pero la entrega no incluye mediciones.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **4 / 4 (Excelente)**

**Fortalezas**
- **Caso de uso original y coherente:** el coordinador de un *Escape Room* controla la cerradura de un compartimento y sabe si el participante abrió la tapa. Se distinguen bien **tapa** (sensor) y **cerradura** (actuador): destrabar no significa que se haya abierto.
- **RF1–RF9 con un criterio de aceptación concreto** cada uno, y una **matriz de trazabilidad** (§2.9) que une requisito, diseño e implementación con la prueba y el resultado.
- **RNF declarados con valores medibles:** 10 min, 30 ciclos (60 estados), 10 órdenes ≤ 3 s, evento ≤ 1 s y respaldo de 60 s, 3 cortes de WiFi y 3 de Internet.
- **Todos los diagramas exigidos:** arquitectura, circuito en SVG (alineado con los ángulos del firmware), clases del firmware y de la Lambda, y tres secuencias (vinculación, consulta de la tapa, comando con confirmación).
- **El mejor mecanismo de vinculación del curso en cuanto a seguridad:**
  - El ESP32 genera un **código de 4 dígitos, lo guarda en NVS y lo publica en su Shadow**.
  - La Lambda exige el número de caja y el código, valida el formato, comprueba `DescribeThing` y **compara el código con el del Shadow** antes de guardar.
  - Es justo lo que la consigna recomienda para evitar que un usuario vincule un objeto ajeno.
- **Shadow y skill bien especificados:** atributos con responsable, temas MQTT, 11 intents, 48 frases y slots obligatorios con delegación del diálogo.

**Observaciones menores (no bajan el nivel)**
- La tabla de RNF (§1.4) mezcla el objetivo con un "resultado cualitativo". El diseño de los RNF es correcto; lo que falla es su verificación (ver §3.3).

### 3.2 Desarrollo e implementación — **4 / 4 (Excelente)**

**Fortalezas**
- **Firmware modular**, con una clase por responsabilidad: `SensorTapa` (antirrebote de 50 ms), `CerraduraServo` (validación de estados), `CodigoVinculacion` (NVS), `ConexionWifi`, `ConexionAwsIot` (TLS mutuo, NTP antes de TLS), `EstadoShadow` (con 17 casos de prueba nativos) y `ConsolaSerial`. Los nombres y las convenciones son claros y consistentes.
- **Lambda organizada** en `manejadores/` (un archivo por intent, más errores, identidad y sesión) y `servicios/` (repositorio de vinculaciones, registro y Shadow). Tiene **76 pruebas** que ejecutan el *dispatcher* real del ASK SDK con la red bloqueada, y dependencias fijadas.
- **Seguridad bien resuelta:**
  - Política de IoT con `iot:Connection.Thing.IsAttached`.
  - Rol de la Lambda limitado a `thing/caja-enigmas-*` y a la tabla, con la justificación de cada permiso.
  - Escape XML de los datos que se insertan en SSML.
  - Identidad tomada solo de `context.System.user.userId`.
  - Hash del paquete desplegado verificado contra el documentado.
- **Configuración de AWS y Alexa verificada con AWS CLI y ASK CLI** (región, runtime, *timeout*, variables, disparador restringido al Skill ID, endpoint del manifiesto).
- Incidencias del backend documentadas con su corrección y la prueba de regresión correspondiente.

**Observaciones menores**
- El firmware conserva los comandos Serial `abrir` y `cerrar` como canal de diagnóstico. El informe lo declara, pero conviene desactivarlos en el firmware de producción.
- El *timeout* de la Lambda es de 3 s, el valor por defecto. Es ajustado si la Lambda alguna vez tuviera que esperar la confirmación del objeto.

### 3.3 Pruebas y validaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- Hay un plan de pruebas por requisito (PF01–PF13, PN01–PN05, PS01), con procedimiento y resultado esperado, y un plan de frases (`docs/PLAN_PRUEBAS.md`).
- **76 pruebas automáticas** con salida verificable.
- Hay un registro del Shadow que muestra las órdenes "abre la cerradura" y "cierra la cerradura" (versiones 115 y 118) con `desired` y `reported` coincidentes.
- El informe distingue con honestidad lo automatizado, lo manual y lo cualitativo, sin presentar parámetros del código como mediciones.

**Observaciones (bajan el nivel)**
- **Ningún RNF se midió.** El propio informe dice: "No se elaboró una bitácora con cantidades de ejecuciones, duración de las sesiones, tiempos cronometrados ni porcentajes de éxito".
  - Estabilidad: "verificado cualitativamente", sin duración.
  - Exactitud del reed: sin ciclos ni distancia de activación.
  - Tiempo de respuesta: "no se informan promedio ni máximo".
  - Frecuencia de reporte: verificada solo en el código.
  - Reconexión con una orden pendiente: "no se le atribuye una ejecución física".
- **Los RF se aprobaron por la palabra del equipo.** Cada RF figura como "Aprobado — validación manual reportada por el equipo", con una sola foto del simulador y una del montaje como evidencia. No hay capturas del simulador por frase, ni de DynamoDB al cambiar o desvincular, ni video del servo.
- **Las "48 frases confirmadas manualmente" no tienen transcripción ni captura** ("sin inventar transcripciones individuales"). La consigna pide verificar las frases de la skill.
- La reconexión con una orden emitida sin conexión, que es un punto clave de la rúbrica de la demo, **no se probó**.

### 3.4 Resultados, conclusiones y recomendaciones — **2 / 4 (Satisfactorio con recomendaciones)**

**Fortalezas**
- El análisis es honesto sobre lo que los datos permiten y no permiten afirmar.
- Las conclusiones son técnicas: separar tapa de cerradura y `desired` de `reported`, la utilidad de los contratos compartidos y la importancia de cargar un binario con la configuración vigente.
- Las recomendaciones incluyen extensiones hacia un uso real: pestillo con sensor de posición, límite de intentos de vinculación, antigüedad de los reportes, acertijos y nuevos sensores para el *Escape Room*.

**Observaciones**
- **Los resultados no son cuantificables.** La tabla de §5 resume aprobaciones manuales, 76 tests y "RNF observados cualitativamente". No hay ningún valor medido frente a un valor declarado.
- Las conclusiones 4 y 5 reconocen que no se puede afirmar el cumplimiento numérico de ningún RNF. El informe lo dice con claridad, pero no alcanza el nivel de "resultados medibles" que pide la rúbrica.
- Los anexos son limitados: dos fotos, siete capturas de AWS de la Lambda y ninguna evidencia del simulador por frase.

---

## 4. Recomendaciones para próximas entregas

1. **Ejecutar los protocolos que ustedes mismos diseñaron.** `docs/PRUEBAS_FISICAS.md` ya define cómo medir cada RNF (30 ciclos del reed, 10 órdenes cronometradas, 3 cortes de red). Con 1–2 horas de pruebas y un celular para grabar, el criterio habría subido de nivel. Esta es la misma observación de la Práctica 2.
2. **Capturar la evidencia mientras se prueba:** una captura del simulador por frase, la tabla DynamoDB antes y después de vincular, cambiar y desvincular, y un video del servo junto al simulador. "Confirmado por el equipo" no es evidencia verificable.
3. **Probar la orden pendiente con el objeto desconectado** (dar la orden con el WiFi cortado y verla aplicada al reconectar). Es un requisito explícito de la consigna y de la demo.
4. Configurar `git config user.name` en todos los equipos, y asegurarse de que el aporte de cada integrante (redacción, pruebas de frases) quede en el historial.
5. Revisar el texto final para quitar frases redactadas desde el punto de vista de una herramienta ("respuestas comunicadas por el usuario", "esta revisión documental…").

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Vinculación:** cómo genera el ESP32 el código, por qué se guarda en NVS, por qué se publica en `reported`, qué compara la Lambda antes de guardar y qué protección ofrece frente a un número de caja adivinado. ¿Qué pasaría con 10 000 intentos?
- **Firmware:** responsabilidad de cada clase (`EstadoShadow`, `ConexionAwsIot`, `CodigoVinculacion`), por qué se pide la hora por NTP antes de TLS, cómo se procesa `state.delta` en `get/accepted` al reconectar y qué pasa con un valor inválido de `cerradura`.
- **Lambda:** estructura en `manejadores/` y `servicios/`, cómo se distingue una orden sin confirmar (`desired` ≠ `reported`), por qué no se usa `session.user` como alternativa y por qué se escapa el SSML.
- **Políticas:** qué hace `iot:Connection.Thing.IsAttached` y por qué el rol se limita a `thing/caja-enigmas-*`.
- **Circuito:** *reed switch* con `INPUT_PULLUP` (LOW = cerrada), antirrebote de 50 ms, servo desde VIN sin fuente externa, y ángulos de 45° y 135°.
- **Pruebas:** cómo se mediría cada RNF con los protocolos de `PRUEBAS_FISICAS.md` y por qué no se ejecutaron.
- **Aporte individual:** Lucía Escobar tiene un solo commit, con autor `unknown`. En la defensa debe demostrar su aporte, y todos deben poder explicar el firmware, la Lambda y la configuración de AWS.
