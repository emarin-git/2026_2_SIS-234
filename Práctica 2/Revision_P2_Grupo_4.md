# Revisión del Informe Técnico — Práctica 2

**Asignatura:** Internet de las Cosas [SIS-234] — 2-2026
**Actividad:** Integración de Objetos Inteligentes con TCP/IP
**Repositorio:** https://github.com/lucianaaaaaaa/proyecto2
**Informe:** `doc/informe.md`
**Integrantes (según el informe):** Alejandro Machaca Villena · Rubén Cordero Martinez · Luciana Leaño Guerrero
**Fecha límite:** martes 13-Oct-2026
**Componente evaluado:** Informe Técnico (1/3 de la nota). La Demo y la Defensa Individual se califican por separado.

---

## 1. Resumen de la calificación

| Criterio | Puntaje |
|---|:---:|
| Requerimientos, análisis y diseño | **3** / 4 |
| Desarrollo e implementación (código y documentación) | **3** / 4 |
| Pruebas y validaciones | **3** / 4 |
| Resultados, conclusiones y recomendaciones | **3** / 4 |
| **Total** | **12 / 16** |
| **Nota del componente** = (12 / 16) × 100 | **75** |

---

## 2. Estado del repositorio en la fecha límite

| Dato | Valor |
|---|---|
| Último push a `main` | 13-Oct 17:22 (UTC-4), commit `0ef7626` |
| Commits posteriores a la fecha límite | Ninguno |
| Commits por autor (sin *merges*) | alous12 (Alejandro Machaca): 4 · lucianaaaaaaa (Luciana Leaño): 4 · Rubén Cordero: 2 |

- Se evaluó el commit **`0ef7626`**.
- **Todo el código se subió el 10-Oct, en unos 30 minutos (17:29–17:56)**, en un commit grande por integrante: actuador (Luciana), sensor (Alejandro), red y servidor (Rubén). Después de eso el código no se modificó, salvo `ConfigRed.h`. El informe se subió el 13-Oct entre las 17:02 y las 17:22.
- **Credenciales en el repositorio:** `src/cliente/ConfigRed.h` contiene el **SSID y la contraseña reales** de la red WiFi. El propio informe lo reconoce (Recomendación 4), pero no lo corrigió. Se recomienda **cambiar esa contraseña**, porque quedó publicada en el historial de un repositorio público.
- Las fotografías del prototipo están en `doc/anexos/`, dentro del repositorio.

---

## 3. Observaciones por criterio

### 3.1 Requerimientos, análisis y diseño — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Requerimientos muy completos:** nueve RF, cada uno con un criterio de aceptación verificable, y nueve RNF con valor medible y forma de verificación (incluye latencia del servidor, recuperación, detección de fallos y uso de recursos).
- **Matriz de trazabilidad** requisito → diseño → implementación (con enlaces) → prueba.
- Rangos contiguos y sin solapamiento, **histéresis de 2 cm** y **filtro de 3 lecturas inválidas**. Se justifica bien por qué hace falta el filtro: sin él, una lectura falsa borraría la histéresis.
- **Diagramas completos:** ubicación en las capas TCP/IP, arquitectura con direcciones, clases, secuencia de la operación completa, **estados** del algoritmo de control y **actividad** de cada firmware.
- **Protocolo** con gramática ABNF, tabla de mensajes, reglas de procesamiento y errores, y secuencia.
- La §2.7 analiza decisiones de diseño y riesgos, entre ellos la limitación de detectar fallos solo con el cierre TCP.

**Por qué no llega a 4**
- **El circuito del sensor tiene un error de diseño:** ECHO (5 V) va directo a GPIO26, sin divisor. La tabla de "Niveles de tensión de las señales" de §2.3 **omite justamente la fila de ECHO**, que es la única que no cumple. El problema se analiza bien en la Recomendación 1 (diodo de protección, límite de VDD + 0,3 V), pero el diseño entregado y el montaje real siguen sin corregirse.
- **El informe no coincide con la configuración del código entregado.** El informe (§2.1, §2.2 y §3.8.1, y la prueba PH-02) describe la red `192.168.0.0/24` con el servidor en `.9`. Pero `ConfigRed.h`, modificado el 13-Oct, usa la red `172.20.10.0/28` (punto de acceso de un celular), con el servidor en `172.20.10.7`. Antes de ese cambio, el servidor estaba en `.26`, no en `.9`. Los diagramas de arquitectura describen una red distinta de la que se usó.
- Los diagramas de circuito están en texto ASCII. Se entienden, pero son menos claros que un esquemático (Fritzing, KiCad o un SVG).
- §2.1 afirma que, como TCP garantiza el orden y la entrega, "el protocolo no necesita [...] confirmaciones propias". El ACK de TCP solo confirma que los bytes llegaron a la pila del receptor, no que el actuador aplicó el comando. Faltó discutir esa diferencia.

### 3.2 Desarrollo e implementación — **3 / 4 (Satisfactorio)**

**Fortalezas**
- Firmware modular y orientado a objetos: `SensorUltrasonico` (con `LecturaDistancia`), `IndicadorLeds`, `ClienteTCP` compartido por los dos objetos, y `Config.h` y `ConfigRed.h` con las constantes. Un solo proyecto PlatformIO con dos entornos (`build_src_filter`).
- Servidor con `ThreadingTCPServer`, `threading.Lock`, constantes con nombre y *docstrings*. Validación de mensajes y límite de 192 caracteres.
- Pruebas automatizadas nativas en C++ (29 comprobaciones, con *mocks* de Arduino) y del servidor (14 pruebas con sockets reales).
- El informe describe cada módulo con enlaces al código (§3.3–§3.6) y hace una revisión estática propia (PS-06), con observaciones.

**Por qué no llega a 4**
- **Credenciales WiFi reales versionadas** en `ConfigRed.h`, sin un archivo `.example` ni `.gitignore`. Es una mala práctica explícita.
- **Servidor en un solo archivo** (`server.py`, 233 líneas) con el algoritmo, el protocolo, la atención de clientes y el `main`. Separarlo en módulos facilitaría las pruebas y la lectura.
- El ciclo del sensor usa `delay(INTERVALO_MEDICION_MS)` y `pulseIn`, ambos bloqueantes. Funciona, pero el periodo real depende del tiempo de eco (de 7.5 a 9.7 lecturas/s según PH-06).
- El propio informe deja pendientes de código (E08: líneas largas; faltan *docstrings* en constructores y en `principal()`; falta comentar `enviarLinea()`).
- No hay un registro de que el código se haya revisado después del 10-Oct: las pruebas de software son de ese día.

### 3.3 Pruebas y validaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Plan de pruebas completo** con 6 pruebas de software (PS-01 a PS-06) y 9 de hardware (PH-01 a PH-09). Cada una tiene objetivo, requisitos, procedimiento, criterio y estado.
- **Pruebas de hardware con valores:** exactitud en 8 puntos de 10 a 200 cm (mínimo, máximo, promedio, desviación estándar y error individual máximo); frecuencia de muestreo en tres condiciones, comparada con el valor teórico; 10 mediciones del tiempo de extremo a extremo; tiempos de fallo seguro y reconexión; y 10 min de estabilidad.
- **Pruebas de frontera** (19, 21, 39 y 41 cm desde ERROR) y de histéresis (H01–H08), con resultados coherentes con la Tabla 1.1.2.
- Se registran los resultados negativos: F05 (corte abrupto del sensor) queda como "observación", no se oculta.
- PS-04 mide la latencia del servidor en 6000 mensajes (media, p99 y máximo), y el script está en el Anexo B.

**Por qué no llega a 4**
- **Los valores de hardware se declaran "aproximados"** y **no hay registros crudos**. PH-03 dice que "las lecturas crudas de cada referencia se incluyen en el Anexo C", pero el Anexo C solo tiene 8 líneas de consola y el texto pendiente "*Pegar aquí la salida del monitor serie del sensor en PH-03 [...] PH-07 [...] PH-08 y PH-09*". Por eso, los datos de PH-03, PH-07 y PH-09 no se pueden verificar.
- **Inconsistencia en PH-07:** el método descrito resta *timestamps* del monitor serie (`pio device monitor -f time`), con resolución de 1 ms. Sin embargo, **los diez tiempos reportados son múltiplos exactos de 1/240 s (4.167 ms)** (162.5 = 39 × 4.167; 216.7 = 52 × 4.167; 370.8 = 89 × 4.167…). Ese patrón corresponde a contar cuadros de un video en cámara lenta a 240 fps, no al método documentado. Si se usó video, es un método válido, pero hay que describirlo y adjuntarlo.
- **PH-02 reporta direcciones `192.168.0.100` y `192.168.0.101`**, que no corresponden a la configuración `172.20.10.x` del código entregado. Hay que aclarar con qué configuración se hicieron realmente las pruebas.
- En PH-03 el error crece casi linealmente con la distancia y la desviación estándar crece de forma monótona. Sería esperable analizarlo como un error sistemático (calibración de la velocidad del sonido), pero no se analiza.

### 3.4 Resultados, conclusiones y recomendaciones — **3 / 4 (Satisfactorio)**

**Fortalezas**
- **Tabla de resultados cuantificables** (§5.1), donde se distingue el tiempo de respuesta **medido** del **estimado**, con un presupuesto teórico por etapa.
- El análisis del rendimiento del servidor es bueno (9 de 6000 mensajes > 10 ms, con su posible causa).
- **Las recomendaciones son técnicas y concretas:** divisor de ECHO con cálculo y explicación del diodo de protección, *timeout* de 1 s en `handle()` o `SO_KEEPALIVE` para la caída abrupta del sensor, desactivar Nagle, sacar las credenciales del repositorio, mediana de lecturas, CI con GitHub Actions y mensaje de latido.
- Hay fotografías del prototipo en cada rango y de la consola del servidor.

**Por qué no llega a 4**
- **La sección quedó incompleta.** El análisis (§5.2) termina con el texto pendiente "*Completar con el análisis de los resultados de hardware [...]*". La conclusión 7 es otro texto pendiente: "*Completar con las conclusiones de las pruebas de hardware (exactitud, tiempo de respuesta de extremo a extremo, estabilidad y reconexión)*". Es decir, **no hay análisis ni conclusiones de las pruebas físicas**, que son las que validan los RNF.
- Las conclusiones 3 a 5 se apoyan solo en pruebas de software ("se verificó a nivel lógico").
- Los anexos están incompletos (Anexo C con el texto pendiente).

---

## 4. Recomendaciones para próximas entregas

1. **Revisar el informe completo antes de entregar** y buscar los marcadores pendientes ("Completar…", "Pegar aquí…"). En esta entrega quedaron tres.
2. **Aplicar las correcciones que el propio informe recomienda:** el divisor de ECHO y la eliminación de las credenciales del repositorio estaban identificados y eran de bajo costo.
3. **Mantener el informe sincronizado con el código:** si cambia la red, actualizar los diagramas, las tablas de configuración y las pruebas, o documentar en qué red se hizo cada prueba.
4. **Adjuntar los registros crudos** (salida del monitor serie y de la consola del servidor, o el video de PH-07) y describir el método de medición que se usó realmente.
5. Analizar el error sistemático de PH-03 y, si corresponde, calibrar la constante de velocidad del sonido.
6. Desarrollar de forma incremental. Un commit grande por persona el mismo día no muestra cómo evolucionó el código ni facilita la integración.

---

## 5. Aspectos a tomar en cuenta para la defensa individual

- **Circuito:** por qué ECHO necesita un divisor, cómo se calcula (5 V × 2 kΩ / 3 kΩ ≈ 3,33 V), qué hace el diodo de protección interno y por qué no es una solución. Por qué la tabla de niveles de tensión no incluye ECHO.
- **Red y configuración:** en qué red se hicieron las pruebas (192.168.0.x o el punto de acceso del celular 172.20.10.x), qué significa la máscara `255.255.255.240` y por qué se usan IP fijas.
- **Medición del tiempo de respuesta (PH-07):** cómo se obtuvieron exactamente los valores y por qué todos son múltiplos de 1/240 s.
- **Algoritmo de control:** explicar la tabla de histéresis y el filtro de 3 lecturas con un ejemplo concreto (qué pasa con `39.77 → 204.12 → 39.77`).
- **Servidor:** cómo funciona `ThreadingTCPServer` (un hilo por cliente), qué protege el `Lock` y por qué una caída abrupta del sensor (F05) no se detecta; cómo la resolvería `settimeout(1.0)` o `SO_KEEPALIVE`.
- **Protocolo:** el formato `POST distance=…` / `SET redLed=…, yellowLed=…, greenLed=…`, el *framing* por línea y el límite de 192 caracteres. Diferencia entre el ACK de TCP y una confirmación de aplicación.
- **Pruebas nativas en C++:** cómo se simulan `Arduino.h` y `pulseIn` para probar `SensorUltrasonico` en la PC.
- **Aporte individual:** según el historial, Luciana hizo el actuador, Alejandro el sensor y Rubén la red y el servidor, cada uno en un solo commit grande. Cada integrante debe poder explicar los otros dos componentes, en especial `ClienteTCP` y `server.py`, que concentran la comunicación TCP.
