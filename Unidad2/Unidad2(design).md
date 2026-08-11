# Diseño del sistema: diseño generativo — simulación

## Tensión

Quiero explorar la tensión de la lógica de polinización de las flores y las abejas, y cómo aplicar la ironía para mantenerlas separadas entre ellas: en vez de que la abeja busque las flores, las flores son las que se acercan y la abeja huye constantemente de ellas.

---

## 1. Tipos de partículas

- **Abeja** (amarillo).
- **Flores**: rojas, verdes, azules y moradas.

**Justificación:** el sistema necesita al menos dos roles enfrentados para que la tensión (abeja que huye / flores que se agrupan) sea perceptible. Un solo tipo de partícula no genera conflicto visual. Cuatro colores de flor, en vez de uno solo, permiten mostrar además una segunda dinámica dentro del mismo sistema: la formación de parches por color, que compite visualmente con la huida de la abeja.

```js
const BEE = 0; // índice del tipo "abeja"
const NUM_TYPES = 5; // 1 abeja + 4 colores de flores
const TYPE_COLORS = ['#f4d35e', '#e63946', '#52b788', '#4361ee', '#9d4edd'];
```

---

## 2. Cantidad de partículas por tipo

La cantidad no es un valor fijo en el código: se lee en tiempo real desde los sliders `Abejas` y `Flores por color` mediante la función `resetParticles()`.

```js
function resetParticles() {
  particles = [];
  const beeN = int(uiBee.value());       // slider "Abejas" (rango 1–40)
  const flowerN = int(uiFlower.value());  // slider "Flores por color" (rango 1–40)

  for (let k = 0; k < beeN; k++) particles.push(makeParticle(BEE));
  for (let t = 1; t < NUM_TYPES; t++) {
    for (let k = 0; k < flowerN; k++) particles.push(makeParticle(t));
  }
}
```

**Justificación:** `flowerN` se aplica por igual a los cuatro colores de flor (no un total repartido), para que ningún color tenga ventaja numérica sobre otro y la comparación entre parches sea justa. El total de partículas es siempre `beeN + flowerN * 4`. Dejar ambas cantidades como parámetros ajustables (en vez de constantes) permite comparar directamente cómo cambia el comportamiento cuando la abeja está en clara minoría frente a las flores, que es el escenario que mejor representa la tensión buscada.

---

## 3. Matriz de atracción, repulsión e indiferencia


```js
function buildMatrix() {
  const rep = uiRep.value();             // repulsión de la abeja
  const attr = uiAttr.value();           // cohesión entre flores del mismo color
  const crossAttr = uiCrossAttr.value(); // atracción mínima entre distintos colores de flores
  const chase = uiChase.checked();

  matrix = [];
  for (let i = 0; i < NUM_TYPES; i++) {
    matrix[i] = [];
    for (let j = 0; j < NUM_TYPES; j++) {
      if (i === BEE && j === BEE) {
        matrix[i][j] = -0.3;                 // abeja-abeja: leve repulsión, para no amontonarse
      } else if (i === BEE && j !== BEE) {
        matrix[i][j] = -rep;                  // abeja-flor: repulsión fuerte (huida)
      } else if (i !== BEE && j === BEE) {
        matrix[i][j] = chase ? 0.25 : 0;       // flor-abeja: opcional, "persecución"
      } else if (i === j) {
        matrix[i][j] = attr;                   // flor-flor mismo color: cohesión
      } else {
        matrix[i][j] = crossAttr;              // flor-flor distinto color: atracción mínima 0.1
      }
    }
  }
}
```

**Justificación:**
- **Abeja repelida por todas las flores:** es la traducción directa de la tensión conceptual. La fila de la abeja es negativa frente a los cuatro colores por igual, sin favoritismos, para que la huida no dependa del color sino de la condición de "ser flor".
- **Abeja-abeja levemente repulsiva:** evita que las abejas colapsen en un solo punto y se lean como una única masa; conserva su identidad individual mientras huyen.
- **Flor-flor mismo color con atracción (cohesión):** es lo que permite que emerjan "parches" reconocibles de cada color, en vez de una nube uniforme de puntos.
- **Flor-flor distinto color con atracción mínima (0.1) en vez de repulsión:** se ajustó a pedido para que los parches no queden completamente aislados entre sí, generando fronteras más difusas y orgánicas entre colores, sin perder la cohesión interna de cada color (que sigue siendo más fuerte, 0.4).
- **Relación flor→abeja opcional (checkbox "persigue"):** se dejó como parámetro y no como constante porque permite comparar dos lecturas distintas de la misma tensión: huida pura, o huida con una leve persecución que la intensifica.

---

## 4. Intensidad y alcance de cada relación

La intensidad de cada relación no es solo el valor de la matriz: depende de una **curva de fuerza** que combina ese valor con la distancia entre partículas.

```js
function forceFn(r, a) {
  const beta = 0.3;
  if (r < beta) {
    return r / beta - 1;              // repulsión dura de corto alcance, igual para todos los tipos
  } else if (r < 1) {
    return a * (1 - Math.abs(2 * r - 1 - beta) / (1 - beta)); // atracción/repulsión según la matriz
  }
  return 0;                            // fuera del radio de interacción, no hay fuerza
}
```

**Justificación:** se usa la curva estándar de particle-life (repulsión de contacto + zona intermedia dependiente de la matriz) en vez de una fuerza puramente lineal, porque garantiza que ningún par de partículas pueda superponerse indefinidamente (siempre hay una repulsión mínima a muy corta distancia, con `beta = 0.3` fija para todos los tipos), mientras que la atracción o repulsión "de comportamiento" (huida, cohesión) solo actúa en la zona intermedia. Esto separa la física de contacto (constante) de la intención de diseño (variable, vía matriz).

---

## 5. Distancias de interacción

```js
const rMax = int(uiRMax.value()); // slider "Radio de interacción", rango 40–220 px, por defecto 110
const MIN_DIST = 2;               // distancia mínima usada al calcular fuerzas
```

**Justificación:** `rMax` define hasta qué distancia una partícula "nota" a otra; más allá de ese radio, la fuerza es cero. Se dejó como parámetro ajustable porque cambia directamente la escala de los parches: un radio pequeño genera muchos grupos pequeños y dispersos, uno grande genera pocos grupos grandes. `MIN_DIST` es un piso técnico (no conceptual): evita que la distancia entre dos partículas casi superpuestas llegue a valores extremadamente pequeños que dispararían la fuerza y desestabilizarían la simulación.

---

## 6. Fricción y velocidad máxima

```js
const MAX_SPEED = 6; // fijo, límite de velocidad para evitar explosiones numéricas
const friction = uiFric.value(); // slider "Amortiguación", rango 0.70–0.99, por defecto 0.90
```

**Justificación:** la velocidad máxima es **constante (6)** porque su función es puramente de estabilidad numérica, no de diseño: sin este límite, cuando muchas partículas quedan muy próximas entre sí la fuerza resultante puede acumularse y producir velocidades absurdas o valores `NaN` que "congelan" visualmente la simulación. La fricción, en cambio, **sí es un parámetro variable** porque afecta directamente la lectura del sistema: valores altos (cerca de 0.99) mantienen el movimiento vivo por más tiempo; valores bajos disipan energía rápido y el sistema converge antes a un estado de equilibrio (parches estables, poco movimiento).

---

## 7. Distribución inicial

```js
function makeParticle(type) {
  return { x: random(width), y: random(height), vx: 0, vy: 0, type: type };
}
```

**Justificación:** todas las partículas nacen en una posición aleatoria uniforme sobre el lienzo y con velocidad inicial en cero. Se eligió una distribución uniforme, sin agrupar por color desde el inicio, para que cualquier agrupamiento que aparezca (parches de flores, huida de la abeja) sea resultado visible de las reglas del sistema y no esté ya insinuado por la posición inicial. El mundo es toroidal (los bordes se envuelven), así que tampoco hay un centro ni bordes privilegiados que sesguen la distribución.

---

## 8. Parámetros constantes y variables

**Invariantes (constantes del sistema, no cambian en ejecución):**

| Invariante | Valor | Rol |
|---|---|---|
| Color de cada tipo | `TYPE_COLORS` | identidad visual de cada especie de partícula |
| Repulsión abeja-abeja | `-0.3` | evita colapso de las abejas entre sí |
| `beta` de la curva de fuerza | `0.3` | umbral de repulsión dura por contacto |
| Velocidad máxima | `6` | estabilidad numérica |
| Distancia mínima | `2` | estabilidad numérica |

**Variables (parámetros ajustables por sliders, cambian la lectura del sistema en cada ejecución):**

| Variable | Rango | Por defecto |
|---|---|---|
| Cantidad de abejas | 1–40 | 15 |
| Cantidad de flores por color | 1–40 | 25 |
| Radio de interacción (`rMax`) | 40–220 | 110 |
| Fuerza general (`forceScale`) | 0.01–2 | 0.6 |
| Repulsión de la abeja | 0–2 | 1.0 |
| Cohesión entre flores (mismo color) | 0–1 | 0.4 |
| Atracción entre colores (distinto color) | 0.1–1 | 0.1 |
| Amortiguación (fricción) | 0.70–0.99 | 0.90 |
| Persecución flor→abeja | on/off | off |

**Justificación:** la separación entre invariante y variable sigue un criterio de diseño, no técnico: todo lo que define la *identidad* de cada tipo de partícula (su color, y las reglas mínimas de estabilidad física) se mantiene constante, mientras que todo lo que define la *intensidad del comportamiento* (qué tanto huye, qué tanto se agrupa, qué tan lejos interactúan) queda expuesto como parámetro. Esto permite explorar muchas variaciones de la misma tensión conceptual sin tener que tocar código.

---

## 9. Apariencia e interacción

- **Abeja:** cuerpo ovalado amarillo con rayas negras y un par de "alas" translúcidas, rotada según la dirección de su velocidad (`atan2(vy, vx)`), más un aura suave alrededor. La rotación hace visible hacia dónde está huyendo en cada instante.
- **Flor:** forma de cinco pétalos alrededor de un centro claro, dibujada con el color propio de su tipo (`TYPE_COLORS`), con un pequeño desfase de rotación por tipo para variar ligeramente su apariencia.
- **Estela:** el fondo se redibuja con transparencia parcial cada frame (`background(11,11,18,55)`), dejando un rastro que muestra las trayectorias recientes de las partículas.
- **Interacción del usuario:** sliders para cada parámetro variable (sección 8), un checkbox para activar la persecución flor→abeja, y un botón "Reiniciar" que vuelve a poblar el sistema con las cantidades actuales de los sliders.

**Justificación:** se evitó dibujar las partículas como simples puntos porque la metáfora del sistema (abeja / flor) necesita ser legible de un vistazo, sin depender del texto. Rotar la abeja según su velocidad refuerza la lectura de "huida" en vez de simple movimiento aleatorio. La estela ayuda a percibir la dirección y velocidad del sistema en conjunto, no solo la posición instantánea.

---

## Evidencia / capturas

**Versión normal, valores por defecto:**

![Versión normal, valores por defecto](assets/version-normal.png)

**Cantidad de abejas al máximo:**

![Cantidad de abejas al máximo](assets/abejas-maximo.png)

**Condición de repulsión invertida:**

![Condición de repulsión invertida](assets/repulsion-invertida.png)

---

## Autoevaluación

| Criterio | Peso | Valoración | Aporte |
|---|---|---|---|
| La intención es clara y perceptible en el comportamiento. | 20 | 4 | 0.8 |
| Los tipos, cantidades, matriz y parámetros están justificados desde la intención. | 25 | 4.5 | 1.125 |
| Comprendo y puedo modificar el funcionamiento técnico del sistema. | 20 | 4.8 | 0.96 |
| El sistema produce variaciones con una identidad reconocible. | 15 | 4 | 0.6 |
| Experimenté, comparé, seleccioné y descarté con criterios claros. | 10 | 4.8 | 0.48 |
| Puedo distinguir y sustentar lo diseñado y lo emergente. | 10 | 4.8 | 0.48 |
| **Total** | **100** | | **4.45** |
