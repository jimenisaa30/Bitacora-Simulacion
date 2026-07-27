// ---------- CONFIG ----------
let W = 400;
let H = 712;
let PINK = [250, 100, 200];
let PINK_GLOW = [250, 150, 200];

let REPEL_RADIUS = 90;      // que tan lejos "siente" el espiral al mouse
let REPEL_STRENGTH = 14;    // cuanto se mueve al acercarse el mouse

let GROW_FRAMES = 55;       // cuadros que tarda cada espiral en crecer
let LIFE_FRAMES = 420;      // cuanto dura visible antes de empezar a irse
let FADE_FRAMES = 60;       // cuadros que tarda en desvanecerse
let SPAWN_MIN = 35;         // separacion minima entre nacimientos (frames)
let SPAWN_MAX = 70;
let MAX_SPIRALS = 10;        // maximo de espirales visibles a la vez

let CENTER_X = W / 2;
let CENTER_Y = H / 2;
let CENTER_SPREAD_X = 70;   // que tan disperso alrededor del centro (x)
let CENTER_SPREAD_Y = 110;  // que tan disperso alrededor del centro (y)

let spirals = [];
let sparkles = [];
let nextSpawnFrame = 0;

function setup() {
  createCanvas(W, H);
  angleMode(RADIANS);

  for (let i = 0; i < 30; i++) {
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

function drawMouseAura() {
  noStroke();
  for (let r = 220; r > 0; r -= 20) {
    let a = map(r, 0, 220, 14, 0);
    fill(255, 120, 170, a * 0.35);
    ellipse(mouseX, mouseY, r, r);
  }
}

// ------------------------------------------------------------------
// Crea un nuevo espiral con tendencia a nacer cerca del centro del
// canvas (usando una distribucion gaussiana alrededor del centro).
// ------------------------------------------------------------------
function spawnSpiral() {
  let cx = randomGaussian(CENTER_X, CENTER_SPREAD_X);
  let cy = randomGaussian(CENTER_Y, CENTER_SPREAD_Y);
  cx = constrain(cx, 40, W - 40);
  cy = constrain(cy, 40, H - 40);

  let dir = random() < 0.5 ? 1 : -1;

  spirals.push({
    cx: cx,
    cy: cy,
    r: random(35, 75),
    turns: random(1.8, 2.8),
    startAngle: random(TWO_PI),
    dir: dir,
    bornFrame: frameCount
  });
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
    fill(255, 100, 200, min(alpha, 255));
    circle(s.x - sz * 0.4, s.y, sz);
    circle(s.x + sz * 0.4, s.y + sz * 0.3, sz * 0.8);
  }
}
