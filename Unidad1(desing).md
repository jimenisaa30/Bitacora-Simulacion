# Justificación de cumplimiento — Rúbrica del encargo

| Criterio | Cumplo | No cumplo |
|---|:---:|:---:|
| Encargo completo | ✅ | |
| Simulación con intención | ✅ | |
| Interacción significativa | ✅ | |
| Prototipo funcional | ✅ | |
| Proceso documentado | | ❌ |

---

## 1. Encargo completo

**Cumplo.**

El sistema interpreta los cinco momentos definidos en el encargo dentro de un
mismo sistema visual coherente (misma paleta, misma lógica de espirales), sin
cambiar de lenguaje visual entre uno y otro:

1. **Posibilidad** — *"todas las direcciones parecen posibles"*. Cada espiral
   nace con un ángulo inicial completamente aleatorio (`startAngle =
   random(TWO_PI)`) y un sentido de giro también aleatorio (`dir = random() <
   0.5 ? 1 : -1`). Antes de aplicar cualquier sesgo, no hay ninguna dirección
   privilegiada: todas son igual de probables.

2. **Tendencia** — *"una pequeña preferencia repetida termina construyendo una
   dirección"*. El caminante (`walkerX`, `walkerY`) recibe en cada spawn un
   empujón mínimo hacia el centro (`lerp(walkerX, CENTER_X, PULL_TO_CENTER)`,
   con `PULL_TO_CENTER = 0.025`). Ese jalón es diminuto en un solo paso, pero
   al repetirse en cada nacimiento va construyendo, con el tiempo, una
   dirección clara: la tendencia del sistema a converger hacia el centro del
   lienzo.

3. **Normalidad** — *"la mayoría de los recorridos permanece cerca de lo
   habitual"*. La posición final de cada espiral se calcula sumando un
   desplazamiento con **distribución gaussiana** (`randomGaussian(0,
   CENTER_SPREAD_X)` / `CENTER_SPREAD_Y`) sobre la posición del caminante. Por
   definición, una distribución gaussiana concentra la mayor parte de sus
   valores cerca de la media: la mayoría de los espirales aparece cerca del
   comportamiento habitual (cerca del caminante/centro), y solo unos pocos se
   alejan.

4. **Excepción** — *"un evento improbable permite descubrir un territorio
   nuevo"*. El paso del caminante no se mueve con pasos uniformes, sino con un
   **Levy flight** (`levyFlightStep()`, algoritmo de Mantegna): casi todos los
   pasos son cortos, pero su distribución tiene "cola pesada", lo que hace que
   ocasionalmente ocurra un salto grande e improbable. Ese salto excepcional es
   justamente lo que permite que el sistema visite zonas del lienzo que un
   recorrido puramente gaussiano/normal jamás alcanzaría.

5. **Influencia** — *"la presencia del visitante transforma lo que puede
   ocurrir"*. La posición del mouse afecta directamente el comportamiento del
   sistema: los espirales cercanos al cursor se apartan de él (repulsión en
   `applyRepel()`) y cambian de color/grosor (`glow` en `drawPath()`). Además,
   el click del visitante genera un espiral nuevo exactamente en ese punto
   (`mousePressed()` → `spawnSpiralAt()`), alterando directamente dónde y con
   qué probabilidad aparece la siguiente forma. El sistema deja de depender
   solo de su propia lógica interna en cuanto hay un visitante presente.

---

## 2. Simulación con intención

**Cumplo.**

Utilizo más de tres conceptos de la unidad, aplicados con una intención
comunicativa clara (transmitir organicidad, imprevisibilidad controlada y
comportamiento "vivo"):

- **Ruido Perlin (`noise()`)**: hace que el centro de cada espiral flote de
  forma continua y orgánica en vez de saltar aleatoriamente, comunicando
  movimiento natural.
- **Levy flight (algoritmo de Mantegna)**: determina dónde nace cada espiral
  automático — mayoría de pasos cortos con saltos largos ocasionales,
  comunicando un patrón de aparición impredecible pero no caótico (el momento
  de "Excepción").
- **Interpolación (`lerp` / `lerpColor`)**: se usa tanto para el color (mezcla
  gradual entre el tono base y el tono de brillo al pasar el mouse) como para
  el "jalón" del caminante hacia el centro del lienzo (el momento de
  "Tendencia").
- **Distribución gaussiana (`randomGaussian`)**: dispersa los nacimientos
  alrededor del centro con mayor densidad cerca de él, sin eliminar la
  posibilidad de aparecer en cualquier zona del lienzo (el momento de
  "Normalidad").

Cada concepto se eligió para reforzar uno de los cinco momentos del encargo,
no como decoración técnica aislada.

---

## 3. Interacción significativa

**Cumplo.**

La interacción **modifica el comportamiento del sistema**, que **también
funciona de forma autónoma sin intervención**:

- **Sin intervención**: el sistema genera espirales por sí solo de manera
  continua (temporizador `SPAWN_MIN`–`SPAWN_MAX`), con su propio ciclo de
  nacimiento, crecimiento, vida y desvanecimiento.
- **Con intervención (hover)**: al acercar el mouse a un espiral, este cambia
  de color (de tono base a tono brillante) y se aparta físicamente del cursor
  (repulsión), alterando su trayectoria en tiempo real.
- **Con intervención (click)**: al hacer click, se genera un espiral nuevo
  directamente en esa posición (con una pequeña variación por Levy flight),
  alterando la probabilidad/densidad de espirales en esa zona del lienzo de
  forma directa. Esto corresponde exactamente al momento de "Influencia" del
  encargo.

Es decir, el usuario no solo "activa" el sistema: modifica su comportamiento
espacial y visual, y el sistema conserva su lógica propia incluso sin que el
usuario interactúe.

---

## 4. Prototipo funcional

**Cumplo.**

El sketch se ejecuta de principio a fin sin errores que impidan comprenderlo:
`setup()` inicializa el lienzo y los sparkles, `draw()` corre el ciclo continuo
de spawn/actualización/dibujo, y las funciones de interacción (`mousePressed`)
están correctamente registradas por p5.js. El prototipo puede recorrerse
completo (ver el ciclo de vida de varios espirales, interactuar por hover y por
click) sin necesidad de pasos adicionales ni de corregir errores en consola.

---

## 5. Proceso documentado

**No cumplo.**

Actualmente no cuento con una bitácora que evidencie el avance, las decisiones
tomadas, las dificultades encontradas, las soluciones aplicadas, el uso de IA
durante el desarrollo, ni el enlace al prototipo. Este documento es un primer
paso hacia ese registro, pero falta construir la bitácora completa del proceso
(por ejemplo: capturas o iteraciones del sketch en distintas etapas, explicación
de por qué se eligieron Levy flight y ruido Perlin para representar cada uno de
los cinco momentos, errores encontrados como el de `curveVertex is not
defined` y su solución, y el enlace final al prototipo funcionando).

**Plan para completarlo:** crear `bitacora.md` con una entrada por sesión de
trabajo, documentando decisiones y problemas técnicos a medida que ocurren, y
enlazar el prototipo (por ejemplo alojado en el editor de p5.js o en GitHub
Pages) al final del documento.
