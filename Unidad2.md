
const BEE = 0; // índice del tipo "abeja"
const NUM_TYPES = 5; // 1 abeja + 4 colores de flores
const TYPE_COLORS = ['#f4d35e', '#e63946', '#52b788', '#4361ee', '#9d4edd'];

let particles = [];
let matrix = [];

const MAX_SPEED = 6;   // límite de velocidad para evitar explosiones numéricas
const MIN_DIST = 2;    // distancia mínima usada al calcular fuerzas, evita división inestable

// controles UI (creados con funciones propias de p5, sin HTML externo)
let uiBee, uiFlower, uiRMax, uiForce, uiRep, uiAttr, uiFric, uiChase, resetBtn;
let labelDiv;

function setup() {
  createCanvas(windowWidth, windowHeight);
  pixelDensity(1);

  createControls();
  resetParticles();
}

function createControls() {
  const panel = createDiv().style('position', 'fixed')
    .style('top', '0').style('left', '0').style('right', '0')
    .style('background', 'rgba(10,10,18,0.85)')
    .style('padding', '10px 16px')
    .style('display', 'flex').style('flex-wrap', 'wrap')
    .style('gap', '18px').style('align-items', 'flex-end')
    .style('font-family', 'sans-serif').style('font-size', '12px')
    .style('color', '#eee').style('z-index', '10')
    .style('border-bottom', '1px solid #333');

  uiBee = addSlider(panel, 'Abejas', 1, 40, 15, 1);
  uiFlower = addSlider(panel, 'Flores por color', 1, 40, 25, 1);
  uiRMax = addSlider(panel, 'Radio de interacción', 40, 220, 110, 1);
  uiForce = addSlider(panel, 'Fuerza general', 0.01, 2, 0.6, 0.01);
  uiRep = addSlider(panel, 'Repulsión de la abeja', 0, 2, 1.0, 0.01);
  uiAttr = addSlider(panel, 'Cohesión entre flores', 0, 1, 0.4, 0.01);
  uiFric = addSlider(panel, 'Amortiguación', 0.7, 0.99, 0.9, 0.01);

  const chaseWrap = createDiv().parent(panel)
    .style('display', 'flex').style('align-items', 'center').style('gap', '6px');
  uiChase = createCheckbox('', false).parent(chaseWrap);
  createSpan('Las flores persiguen a la abeja').parent(chaseWrap);

  resetBtn = createButton('Reiniciar').parent(panel)
    .style('padding', '8px 16px').style('border-radius', '8px')
    .style('border', 'none').style('background', '#f4d35e')
    .style('color', '#111').style('font-weight', 'bold')
    .style('cursor', 'pointer').style('height', '32px');
  resetBtn.mousePressed(resetParticles);
}

function addSlider(parent, label, min, max, def, step) {
  const wrap = createDiv().parent(parent)
    .style('display', 'flex').style('flex-direction', 'column').style('gap', '4px');
  createSpan(label).parent(wrap).style('color', '#ccc');
  const s = createSlider(min, max, def, step).parent(wrap).style('width', '140px');
  return s;
}

function resetParticles() {
  particles = [];
  const beeN = int(uiBee.value());
  const flowerN = int(uiFlower.value());

  for (let k = 0; k < beeN; k++) particles.push(makeParticle(BEE));
  for (let t = 1; t < NUM_TYPES; t++) {
    for (let k = 0; k < flowerN; k++) particles.push(makeParticle(t));
  }
}

function makeParticle(type) {
  return { x: random(width), y: random(height), vx: 0, vy: 0, type: type };
}

// Construye la matriz de colores (reglas de atracción/repulsión)
function buildMatrix() {
  const rep = uiRep.value();       // qué tanto huye la abeja de las flores
  const attr = uiAttr.value();     // cuánto se agrupan las flores del mismo color
  const chase = uiChase.checked();

  matrix = [];
  for (let i = 0; i < NUM_TYPES; i++) {
    matrix[i] = [];
    for (let j = 0; j < NUM_TYPES; j++) {
      if (i === BEE && j === BEE) {
        matrix[i][j] = -0.3;                  // las abejas no se amontonan entre sí
      } else if (i === BEE && j !== BEE) {
        matrix[i][j] = -rep;                   // la abeja es repelida por CUALQUIER flor
      } else if (i !== BEE && j === BEE) {
        matrix[i][j] = chase ? 0.25 : 0;        // opcional: las flores "persiguen" a la abeja
      } else if (i === j) {
        matrix[i][j] = attr;                    // flores del mismo color se atraen (cohesión)
      } else {
        matrix[i][j] = -0.15;                   // flores de distinto color se separan un poco
      }
    }
  }
}

// Curva de fuerza estándar de particle-life (repulsión dura cerca, luego atracción/repulsión suave)
function forceFn(r, a) {
  const beta = 0.3;
  if (r < beta) {
    return r / beta - 1;
  } else if (r < 1) {
    return a * (1 - Math.abs(2 * r - 1 - beta) / (1 - beta));
  }
  return 0;
}

function draw() {
  background(11, 11, 18, 55); // estela translúcida

  buildMatrix();

  const rMax = int(uiRMax.value());
  const forceScale = uiForce.value();
  const friction = uiFric.value();
  const n = particles.length;

  // 1) calcular fuerzas
  for (let i = 0; i < n; i++) {
    const p = particles[i];
    let ax = 0, ay = 0;
    for (let j = 0; j < n; j++) {
      if (i === j) continue;
      const q = particles[j];
      let dx = q.x - p.x;
      let dy = q.y - p.y;
      // distancia mínima toroidal (el mundo se envuelve en los bordes)
      if (dx > width / 2) dx -= width;
      if (dx < -width / 2) dx += width;
      if (dy > height / 2) dy -= height;
      if (dy < -height / 2) dy += height;

      const dSq = dx * dx + dy * dy;
      const minDSq = MIN_DIST * MIN_DIST;
      if (dSq > 0 && dSq < rMax * rMax) {
        const d = Math.sqrt(Math.max(dSq, minDSq)); // evita división inestable si están casi encimadas
        const r = d / rMax;
        const a = matrix[p.type][q.type];
        const f = forceFn(r, a);
        ax += (dx / d) * f;
        ay += (dy / d) * f;
      }
    }
    p.ax = ax * forceScale;
    p.ay = ay * forceScale;
  }

  // 2) integrar movimiento
  for (let i = 0; i < n; i++) {
    const p = particles[i];
    p.vx = (p.vx + p.ax) * friction;
    p.vy = (p.vy + p.ay) * friction;

    // límite de velocidad máxima: evita que la simulación explote o llegue a NaN
    const speed = Math.sqrt(p.vx * p.vx + p.vy * p.vy);
    if (speed > MAX_SPEED) {
      p.vx = (p.vx / speed) * MAX_SPEED;
      p.vy = (p.vy / speed) * MAX_SPEED;
    }

    p.x += p.vx;
    p.y += p.vy;
    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;
  }

  // 3) dibujar
  noStroke();
  for (let i = 0; i < n; i++) {
    const p = particles[i];
    if (p.type === BEE) {
      drawBee(p);
    } else {
      drawFlower(p);
    }
  }
}

function drawBee(p) {
  const ang = Math.atan2(p.vy, p.vx);
  push();
  translate(p.x, p.y);
  rotate(ang);
  // aura suave
  fill(244, 211, 94, 40);
  ellipse(0, 0, 14, 14);
  // cuerpo
  fill('#f4d35e');
  ellipse(0, 0, 6, 5);
  // rayas
  fill(20);
  rect(-1, -2.5, 1.4, 5);
  rect(1.4, -2.5, 1.4, 5);
  // alitas
  fill(255, 255, 255, 120);
  ellipse(-1, -4, 4, 3);
  ellipse(2, -4, 4, 3);
  pop();
}

function drawFlower(p) {
  const c = color(TYPE_COLORS[p.type]);
  push();
  translate(p.x, p.y);
  // pétalos
  fill(red(c), green(c), blue(c), 160);
  for (let k = 0; k < 5; k++) {
    push();
    rotate((k * TWO_PI) / 5 + p.type); // pequeño offset por tipo para variedad visual
    ellipse(3.2, 0, 4.5, 2.6);
    pop();
  }
  // centro
  fill(255, 240, 200);
  ellipse(0, 0, 3, 3);
  pop();
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
