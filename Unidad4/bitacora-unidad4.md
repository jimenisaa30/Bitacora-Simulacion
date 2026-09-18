# Bitácora — Unidad 4: Oscilación
## Sistema audiovisual performativo con Kuramoto — Estanque Kuramoto

> Bitácora de diseño para la **Actividad 02: encargo de diseño** de la Unidad 4 (Oscilación) del curso
> [Simulación](https://juanferfranco.github.io/simulacion-2026-20/units/unit4/).

**Proyecto (p5.js Web Editor):** https://editor.p5js.org/jimenisaa/full/fAlLHuyUy

---

## 1. Síntesis del proyecto

**Estanque Kuramoto** es una experiencia audiovisual interactiva en la que 8 "hojas de loto" —agentes
gobernados por el modelo de Kuramoto— habitan un estanque. Cada agente pertenece a una de 4
**personalidades** con comportamiento, forma y sonido propios. El usuario puede modificar el
acoplamiento entre agentes y su velocidad de desplazamiento en tiempo real, perturbar agentes
individuales o agitar todo el sistema de un clic, y observar cómo el estanque transita entre el
desorden, la organización parcial y la sincronización estable — o cómo rompe esa estabilidad y se
reorganiza.

No es una visualización pasiva del modelo ni un secuenciador con ritmo fijo: el ritmo sonoro y visual
de la pieza nace directamente de cuándo cada agente completa un ciclo de fase, algo que depende
enteramente de la dinámica de Kuramoto y de las intervenciones del usuario.

---

## 2. Modelo obligatorio: Kuramoto

Se implementa el modelo de Kuramoto **sin modificar la ecuación de acoplamiento**. Con 8 agentes es
computacionalmente trivial hacer el acoplamiento **todos-con-todos** (no restringido por distancia),
que es la forma clásica del modelo para poblaciones pequeñas:

```
θᵢ' = ωᵢ + (K / N) · Σⱼ sin(θⱼ − θᵢ)      con N = 8, j ≠ i
```

**Qué representa cada variable en la aplicación:**

| Variable | Representación en el proyecto |
|---|---|
| `θᵢ` (fase) | El ciclo interno de cada agente. Cuando `θᵢ` completa una vuelta (`2π`), el agente dispara su manifestación visual (onda) y sonora. Es el "latido" propio de cada hoja. |
| `ωᵢ` (frecuencia natural) | El ritmo innato de cada agente, muestreado de una gaussiana angosta (`media 1.0, σ 0.07`, acotada entre `0.82` y `1.18`) al crear el agente. Representa que cada hoja "quiere" latir a un ritmo ligeramente distinto por sí misma. |
| `K` (acoplamiento) | Cuánto le importa a cada agente el desfase con el resto del estanque. Es la **variable obligatoria** que el usuario controla en tiempo real con el slider "Acoplamiento K". En `K` bajo, cada hoja sigue su propio ritmo (desorden); en `K` alto, el estanque tiende a sincronizarse. |
| `N` | Los 8 agentes simultáneos exigidos por el encargo. |
| `r` (parámetro de orden) | `r = (1/N)·\|Σᵢ e^{iθᵢ}\|`. Se calcula cada frame y es la métrica que determina el estado colectivo mostrado en la interfaz (desorden / organización parcial / sincronización estable) y el tinte del fondo del estanque. |

**Cómo producen el comportamiento observado:** en cada frame se calcula el término de acoplamiento de
cada agente promediando `sin(θⱼ − θᵢ)` contra los otros 7; ese término se suma a `ωᵢ` y define la
velocidad de fase real de ese instante. Cuando `K` es suficientemente alto respecto a la dispersión de
`ωᵢ`, el término de acoplamiento empieza a "arrastrar" las fases entre sí y `r` crece: los agentes
completan sus ciclos cada vez más cerca en el tiempo, lo que se traduce en ráfagas de ondas y sonidos
que se agrupan en el tiempo en vez de dispararse de forma dispersa.

**Variaciones respecto al modelo base:** no se modificó la ecuación de Kuramoto en sí (ni el término de
acoplamiento, ni la topología, que es todos-con-todos por diseño dado que `N` es pequeño). Lo que sí es
una decisión de diseño explícita es **qué hace cada agente con su propio `θᵢ` y con `r`** una vez que el
modelo los calcula: cada una de las 4 personalidades traduce la fase y el estado colectivo en un
comportamiento distinto (ver sección 4). Es decir, la extensión de diseño no está en el modelo
dinámico sino en el **mapeo** de sus salidas (`θᵢ`, `θᵢ'`, `r`) a movimiento, forma y sonido — que es
justamente el problema de diseño que plantea el encargo.

---

## 3. Cumplimiento de requisitos mínimos

| Requisito | Cómo se cumple |
|---|---|
| 8 agentes simultáneos gobernados por el sistema dinámico | 8 instancias de `Agent` (subclases), todas acopladas por Kuramoto todos-con-todos en `updateKuramoto()`. |
| 4 personalidades audiovisuales diferentes | `Deriva`, `Raíz`, `Espejo`, `Huidiza` — 2 agentes cada una. Ver sección 4. |
| Manifestación visual y sonora por agente, vinculada al comportamiento | Cada personalidad dispara su propio timbre/envolvente en `playPersonalitySound()` al completar un ciclo de fase (`onPhaseCycle()`), y su propia forma/movimiento en `display()` / `updateMovement()`. |
| ≥2 variables del modelo modificables en tiempo real, obligatorio `K` | Sliders **Acoplamiento K** y **Velocidad** en el panel, ambos activos durante la ejecución. |
| ≥2 formas de interacción performativa (global + individual) | **Global:** clic en agua vacía → `applyGlobalPerturbation()` agita la fase de los 8 agentes a la vez (más el salto decorativo de una rana). **Individual:** clic directo sobre un agente → `agent.perturb()` interviene solo ese agente. |
| ≥1 mecanismo de perturbación observable en el colectivo | `Agent.perturb(strength)`, usado tanto por el clic individual como por el aterrizaje de una rana; cada personalidad reacciona distinto (ver sección 4) y el efecto sobre `r` es observable en el indicador. |
| ≥3 estados colectivos reconocibles | Umbrales sobre `r`: `< 0.42` desorden, `0.42–0.78` organización parcial, `≥ 0.78` sincronización estable. |
| ≥1 forma perceptible de comunicar el estado colectivo | Indicador **ORDEN r** (barra + etiqueta de texto) y, además, el color de fondo del estanque se oscurece/enverdece progresivamente con `r` (doble canal: explícito + ambiental). |

---

## 4. Personalidad audiovisual

El encargo exige que las personalidades no se diferencien solo por color o altura musical. Cada una se
definió variando **movimiento, forma, timbre/envolvente, relación con la fase y reacción a las
intervenciones**:

- **Deriva** — vaga por el estanque con ruido de Perlin, forma orgánica que se deforma con la fase.
  Sonido sinusoidal breve en cada ciclo. Al ser perturbada, su fase salta y cambia de rumbo de golpe:
  "se despista".
- **Raíz** — casi inmóvil, anclada a su punto de origen (con una leve fuerza de retorno). Forma
  hexagonal rígida cuyo tamaño crece con su propia fase *y* con el parámetro de orden `r` del
  colectivo — es la personalidad que más "muestra" el estado global en su propio cuerpo. Timbre
  triangular, envolvente larga y resonante. Al perturbarla, responde con un pulso sonoro más largo e
  intenso.
- **Espejo** — su movimiento depende explícitamente de otro agente: mantiene una distancia deseada
  respecto a su vecino más cercano (se acerca o se aleja de él). Su forma se estira hacia ese vecino, y
  su afinación se desafina proporcionalmente al desfase de sus fases (más desfase → más disonante).
  Al ser perturbada, sincroniza su fase de golpe con la de ese vecino.
- **Huidiza** — casi quieta en reposo, pero ante una perturbación (o cuando el colectivo está muy
  sincronizado) huye con un arranque brusco: velocidad alta y forma espinosa mientras dura el impulso,
  con un sonido percusivo corto. En reposo apenas emite un tic sonoro tenue.

---

## 5. Experiencia performativa

El performer puede: (1) subir `K` gradualmente para llevar el estanque del desorden a la
sincronización estable y observarlo en el indicador; (2) romper esa estabilidad con un clic en el agua
(perturbación global) o tocando directamente un agente (perturbación individual) y ver cómo el
colectivo se reorganiza; (3) elegir cuántas ranas están activas, cuyo salto (decorativo) siempre viene
acompañado de la perturbación global; (4) ajustar la velocidad de desplazamiento para cambiar el
carácter espacial de la pieza sin tocar la dinámica de fase.

**¿Qué hace Kuramoto aquí que no podría resolver un reloj global o un secuenciador?** El momento exacto
en el que cada agente dispara su sonido/onda no está programado: depende de cuándo su fase individual
completa una vuelta, lo cual a su vez depende de `ωᵢ`, de `K` y del estado de fase de los otros 7
agentes en ese instante. Por eso el ritmo se "aprieta" en ráfagas cuando el sistema se sincroniza y se
dispersa de forma impredecible cuando no — un reloj global solo podría producir un patrón fijo o
aleatorio, nunca esta transición emergente y reversible entre orden y desorden.

---

## 6. Registro del proceso (avances, errores y decisiones)

- **v0 — versión preliminar (ver evidencia, sección 7):** ~120–200 "hojas" (una por identidad musical,
  8 identidades) sobre un canvas dependiente de la ventana (`windowWidth/windowHeight`), con
  sincronización Kuramoto por radio de interacción. Cumplía la mecánica básica pero no los requisitos
  del encargo (más de 8 agentes, personalidades que solo variaban en color/tono).
- **Depuración de audio:** el sonido no se escuchaba salvo el clic de confirmación. Causa raíz:
  `p5.sound` v2 (la versión que acompaña a p5.js 2.x) cambió `Oscillator.amp()` a un único parámetro
  (sin `rampTime`/`timeFromNow`), y luego `p5.Envelope` tampoco tiene `setRange()` en esta versión.
  Solución final: controlar directamente el `GainNode` nativo que expone cada nodo de p5.sound como
  `oscillator.output`, programando la envolvente con `setValueAtTime`/`linearRampToValueAtTime` de Web
  Audio — independiente de los cambios de API de alto nivel.
- **`TypeError: this.targetLeaf.perturb is not a function`:** faltaba el método `perturb()` en la clase
  de hoja, aunque ya se usaba desde `Frog.update()` y desde el render (`this.perturbation`). Se agregó.
- **Canvas y colisión física:** se corrigió el canvas a un tamaño fijo `1920×1080` (no dependiente de
  la ventana) y se añadió resolución física de colisiones entre agentes (separación por superposición +
  reflejo de velocidad relativa), ya que antes solo se detectaba el contacto para disparar sonido/onda
  pero los agentes podían superponerse.
- **Rediseño hacia los requisitos del encargo:** se redujo de ~120–200 hojas a **8 agentes fijos**, se
  sustituyeron las 8 identidades musicales (que solo variaban color y tono) por **4 personalidades**
  con reglas de movimiento, forma, timbre/envolvente y reacción a intervenciones distintas (`Deriva`,
  `Raíz`, `Espejo`, `Huidiza`), se cambió el acoplamiento a todos-con-todos, y se definieron
  explícitamente las dos interacciones performativas (global / individual) y el mecanismo de
  perturbación exigidos.

---

## 7. Evidencia del proceso

**Imagen 1.** Vista preliminar del estanque con el slider de "Acoplamiento K" en `0.80`. Corresponde a
una etapa temprana con muchas hojas dispersas, antes de reducir el sistema a 8 agentes y definir las 4
personalidades.

![Vista preliminar del estanque, K=0.80](./bitacora-assets/proceso-01-version-preliminar.png)

**Imagen 2.** Panel de control de la versión anterior (120 hojas), con `Acoplamiento K: 0.72` y
`ORDEN r: 0.99` — sistema en sincronización estable, con una rana saltando entre hojas. Corresponde a
la etapa en que todavía existía el slider "Hojas" (número de agentes ajustable), eliminado en la
versión final para cumplir el requisito de 8 agentes fijos.

![Panel con 120 hojas y r=0.99](./bitacora-assets/proceso-02-panel-120-hojas.png)

**Imagen 3.** Mismo estado del proyecto con velocidad reducida a `1.2 km/h` y `ORDEN r: 0.84`
(sincronización estable, agentes agrupándose en racimos). Evidencia del comportamiento del parámetro de
orden y de la agrupación espacial-visual asociada a un `K` alto, antes del rediseño a 8 agentes.

![Panel con velocidad 1.2 km/h y r=0.84](./bitacora-assets/proceso-03-panel-sincronizacion.png)

> Estas tres imágenes documentan una etapa intermedia del proceso (antes de ajustar el número de
> agentes y las personalidades a los requisitos mínimos del encargo) y se conservan aquí como evidencia
> del proceso de diseño e iteración, tal como pide la bitácora de la unidad.

---

## 8. Autoevaluación

Según los criterios de la Actividad 03 (presentación grupal):

- [ ] **1. Leí y verifiqué que mi proyecto cumple con los requisitos mínimos de la unidad.** (25 pts)
  _Ver la tabla de la sección 3: los 8 requisitos mínimos están mapeados a una implementación
  concreta._

- [ ] **2. Puedo explicar claramente qué representa cada variable del modelo de Kuramoto en mi
  proyecto.** (25 pts)
  _Ver la tabla de la sección 2 (`θᵢ`, `ωᵢ`, `K`, `N`, `r`)._

- [ ] **3. Puedo explicar claramente cómo las variables del modelo producen el comportamiento
  observado en mi proyecto.** (25 pts)
  _Ver el párrafo "Cómo producen el comportamiento observado" en la sección 2, y la respuesta a la
  pregunta central de diseño en la sección 5._

- [ ] **4. Puedo demostrar que mi proyecto cumple con los objetivos establecidos en la unidad.** (25 pts)
  _Demostrable en vivo: subir `K` para sincronizar, perturbar individual y globalmente, y observar la
  reorganización del colectivo — ver sección 5._

> Marcar cada casilla y ajustar la justificación antes de la presentación, según el estado real del
> proyecto en ese momento.

---

## 9. Enlaces

- **Proyecto (p5.js Web Editor):** https://editor.p5js.org/jimenisaa/full/fAlLHuyUy
- **Encargo de diseño (Unidad 4):** https://juanferfranco.github.io/simulacion-2026-20/units/unit4/#actividad-02-encargo-de-diseño
