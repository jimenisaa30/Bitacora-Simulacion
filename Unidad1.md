// ---------- CONFIG ----------
let W = 400;
let H = 712;
let PINK = [255, 105, 200];
let PINK_GLOW = [255, 205, 225];

let REPEL_RADIUS = 90;      // que tan lejos "siente" el espiral al mouse
let REPEL_STRENGTH = 16;    // cuanto se mueve al acercarse el mouse

let GROW_FRAMES = 55;       // cuadros que tarda cada espiral en crecer
let LIFE_FRAMES = 420;      // cuanto dura visible antes de empezar a irse
let FADE_FRAMES = 60;       // cuadros que tarda en desvanecerse
let SPAWN_MIN = 35;         // separacion minima entre nacimientos (frames)
let SPAWN_MAX = 70;
let MAX_SPIRALS = 15;        // maximo de espirales visibles a la vez

let CENTER_X = W / 2;
let CENTER_Y = H / 2;
let CENTER_SPREAD_X = 110;  // que tan disperso alrededor del centro (x)
let CENTER_SPREAD_Y = 220;  // que tan disperso alrededor del centro (y)

let NOISE_DRIFT = 45;       // que tanto "flota" el centro de cada espiral (Perlin)
let NOISE_SPEED = 0.003;    // que tan rapido cambia el ruido Perlin

let LEVY_SCALE = 7;         // tamano base de los saltos del Levy flight
let PULL_TO_CENTER = 0.025; // que tanto se "recuerda" volver al centro

let spirals = [];
let sparkles = [];
let nextSpawnFrame = 0;

// caminante que usa Levy flight para decidir donde nace cada
// espiral automatico: pasos casi siempre cortos, con saltos largos
// ocasionales, tirando suavemente de vuelta al centro
let walkerX = CENTER_X;
let walkerY = CENTER_Y;

function setup() {
  createCanvas(W, H);
  angleMode(RADIANS);

  for (let i = 0; i < 50; i++) {
    sparkles.push({ x: random(W), y: random(H), phase: random(TWO_PI) });
  }

  nextSpawnFrame = 20;
}

function draw() {
  background(10);

  drawMouseAura();

  if (frameCount >= nextSpawnFrame && spirals.length < MAX_SPIRALS) {
    spawnSpiral();
    nextSpawnFrame = frameCount + floor(random(SPAWN_MIN, SPAWN_MAX));
  }

  updateAndDrawSpirals();
  drawSparkles();
}

// al hacer click, nace un espiral justo ahi (con un pequeno salto
// tipo Levy flight para que no se sienta mecanico)
function mousePressed() {
  let jump = levyFlightStep(6);
  spawnSpiralAt(mouseX + jump.dx, mouseY + jump.dy);
}

function drawMouseAura() {
  noStroke();
  for (let r = 220; r > 0; r -= 20) {
    let a = map(r, 0, 220, 14, 0);
    fill(255, 120, 170, a * 0.35);
    ellipse(mouseX, mouseY, r, r);
  }
}

// ------------------------------------------------------------------
// Mueve el caminante con un paso de Levy flight (mayoria de pasos
// cortos, con saltos largos ocasionales) y lo jala suavemente hacia
// el centro, para que los espirales automaticos tiendan a nacer ahi.
// ------------------------------------------------------------------
function spawnSpiral() {
  let step = levyFlightStep(LEVY_SCALE);
  walkerX += step.dx;
  walkerY += step.dy;

  walkerX = lerp(walkerX, CENTER_X, PULL_TO_CENTER);
  walkerY = lerp(walkerY, CENTER_Y, PULL_TO_CENTER);

  // dispersion aleatoria (gaussiana) encima del caminante: esto es
  // lo que permite que, ademas de la tendencia al centro, los
  // espirales puedan aparecer en cualquier parte de la pantalla
  let scatterX = randomGaussian(0, CENTER_SPREAD_X);
  let scatterY = randomGaussian(0, CENTER_SPREAD_Y);

  let cx = constrain(walkerX + scatterX, 30, W - 30);
  let cy = constrain(walkerY + scatterY, 30, H - 30);

  spawnSpiralAt(cx, cy);
}

// crea un espiral nuevo en (x, y). Usada tanto por el nacimiento
// automatico (Levy flight) como por el click del mouse.
function spawnSpiralAt(x, y) {
  let cx = constrain(x, 30, W - 30);
  let cy = constrain(y, 30, H - 30);
  let dir = random() < 0.5 ? 1 : -1;

  if (spirals.length >= MAX_SPIRALS) {
    spirals.shift(); // hace espacio quitando el mas viejo
  }

  spirals.push({
    baseCx: cx,
    baseCy: cy,
    cx: cx,
    cy: cy,
    r: random(35, 75),
    turns: random(1.8, 2.8),
    startAngle: random(TWO_PI),
    dir: dir,
    bornFrame: frameCount,
    noiseOffX: random(1000),
    noiseOffY: random(1000)
  });
}

// ------------------------------------------------------------------
// Paso de Levy flight (algoritmo de Mantegna): la mayoria de los
// pasos son cortos, pero ocasionalmente aparece un salto grande,
// dando un movimiento organico e impredecible.
// ------------------------------------------------------------------
function levyFlightStep(scale) {
  let beta = 1.5;
  let gamma1 = 1.3293403881791370; // Gamma(1 + beta)
  let gamma2 = 0.9064024770554770; // Gamma((1 + beta) / 2)

  let sigma = pow(
    (gamma1 * sin(PI * beta / 2)) / (gamma2 * beta * pow(2, (beta - 1) / 2)),
    1 / beta
  );

  let u = randomGaussian(0, sigma);
  let v = randomGaussian(0, 1);
  let stepLen = u / pow(abs(v), 1 / beta);

  let angle = random(TWO_PI);
  return {
    dx: cos(angle) * stepLen * scale,
    dy: sin(angle) * stepLen * scale
  };
}

function updateAndDrawSpirals() {
  for (let i = spirals.length - 1; i >= 0; i--) {
    let sp = spirals[i];
    let age = frameCount - sp.bornFrame;

    let growT = constrain(age / GROW_FRAMES, 0, 1);
    let fadeT = 1;
    if (age > GROW_FRAMES + LIFE_FRAMES) {
      fadeT = map(age - GROW_FRAMES - LIFE_FRAMES, 0, FADE_FRAMES, 1, 0, true);
    }

    if (fadeT <= 0) {
      spirals.splice(i, 1);
      continue;
    }

    // ruido Perlin: el centro del espiral "flota" suavemente en
    // vez de quedarse fijo, usando noise() en lugar de aleatoriedad
    // pura para que el movimiento sea continuo y organico
    let nX = noise(sp.noiseOffX + frameCount * NOISE_SPEED);
    let nY = noise(sp.noiseOffY + frameCount * NOISE_SPEED);
    sp.cx = sp.baseCx + (nX - 0.5) * NOISE_DRIFT;
    sp.cy = sp.baseCy + (nY - 0.5) * NOISE_DRIFT;

    let pts = spiralPointsLive(sp);
    let count = max(2, floor(growT * pts.length));
    drawPath(pts.slice(0, count), fadeT);
  }
}

// genera los puntos del espiral EN VIVO, del centro hacia afuera
// (para que la animacion de crecimiento se vea como que brota),
// con una rotacion continua que nunca se detiene.
function spiralPointsLive(sp) {
  let pts = [];
  let steps = 60;
  let rotOffset = frameCount * 0.006 * sp.dir;
  for (let i = 0; i <= steps; i++) {
    let t = i / steps;
    let a = sp.startAngle + rotOffset + sp.dir * t * sp.turns * TWO_PI;
    let r = sp.r * t;
    let x = sp.cx + cos(a) * r;
    let y = sp.cy + sin(a) * r;
    pts.push(applyRepel(x, y));
  }
  return pts;
}

// empuja un punto lejos del mouse si esta cerca (asi el espiral se
// "mueve" al pasar el cursor, ademas de brillar)
function applyRepel(x, y) {
  let dx = x - mouseX;
  let dy = y - mouseY;
  let d = sqrt(dx * dx + dy * dy);
  if (d < REPEL_RADIUS && d > 0.001) {
    let push = map(d, 0, REPEL_RADIUS, REPEL_STRENGTH, 0);
    x += (dx / d) * push;
    y += (dy / d) * push;
  }
  return { x: x, y: y };
}

function drawPath(pts, alphaMul) {
  if (pts.length < 2) return;
  noFill();
  let dmin = distToMouseFromPath(pts);
  let glow = map(dmin, 0, 130, 1, 0, true);
  let col = lerpColor(color(PINK[0], PINK[1], PINK[2]), color(PINK_GLOW[0], PINK_GLOW[1], PINK_GLOW[2]), glow);
  strokeWeight(lerp(2, 3.6, glow));
  stroke(red(col), green(col), blue(col), 235 * alphaMul);
  beginShape();
  for (let i = 0; i < pts.length; i++) {
    curveVertex(pts[i].x, pts[i].y);
  }
  endShape();
}

function distToMouseFromPath(pts) {
  let m = Infinity;
  for (let i = 0; i < pts.length; i += 3) {
    let dd = dist(mouseX, mouseY, pts[i].x, pts[i].y);
    if (dd < m) m = dd;
  }
  return m;
}

function drawSparkles() {
  noStroke();
  for (let i = 0; i < sparkles.length; i++) {
    let s = sparkles[i];
    let d = dist(mouseX, mouseY, s.x, s.y);
    let hoverAmt = map(d, 0, 100, 1, 0, true);
    let tw = (sin(frameCount * 0.05 + s.phase) + 1) / 2;
    let alpha = 60 + tw * 100 + hoverAmt * 120;
    let sz = 3 + hoverAmt * 4 + tw * 1.5;
    fill(30, 230, 140, min(alpha, 255));
    circle(s.x - sz * 0.7, s.y, sz);
    circle(s.x + sz * 0.5, s.y + sz * 0.4, sz * 0.9);
  }
}
