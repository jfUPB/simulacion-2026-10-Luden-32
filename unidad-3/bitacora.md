# Unidad 3
```jsx
let movers = [];
let liquid;
let spawnTimer = 0;
let spawnDelay = 30;

function setup() {
  createCanvas(640, 360);
  noiseDetail(3, 0.4);
  liquid = new Liquid(0, height / 2, width, height / 2, 0.1);
}

function draw() {
  background(255);
  liquid.show();

  // Spawn con mouse
  if (mouseIsPressed) {
    if (spawnTimer >= spawnDelay) {
      movers.push(new Mover(mouseX, mouseY, random(0.5, 3)));
      spawnTimer = 0;
    }
    spawnTimer++;
  } else {
    spawnTimer = spawnDelay;
  }

  // 🔁 Atracción entre partículas (solo en agua)
  for (let i = 0; i < movers.length; i++) {
    for (let j = i + 1; j < movers.length; j++) {
      let a = movers[i];
      let b = movers[j];

      if (liquid.contains(a) && liquid.contains(b)) {
        let force = a.attract(b);
        a.applyForce(force);
        b.applyForce(force.mult(-1));
      }
    }
  }

  for (let m of movers) {
    // Gravedad
    let gravity = createVector(0, 0.1 * m.mass);
    m.applyForce(gravity);

    if (liquid.contains(m)) {
      let dragForce = liquid.calculateDrag(m);
      m.applyForce(dragForce);

      m.grow();

      // Ruido 0–20 (perceptual)
      let targetDeform = map(
        pow(m.mass, 1.4),
        pow(0.5, 1.4),
        pow(6, 1.4),
        0,
        20,
        true
      );
      m.deform = lerp(m.deform, targetDeform, 0.05);
    }

    m.update();
    m.show();
    m.checkEdges();
  }
}

// ================= LIQUID =================

class Liquid {
  constructor(x, y, w, h, c) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.c = c;
  }

  contains(m) {
    return (
      m.position.x > this.x &&
      m.position.x < this.x + this.w &&
      m.position.y > this.y &&
      m.position.y < this.y + this.h
    );
  }

  calculateDrag(m) {
    let speed = m.velocity.mag();
    let dragMag = this.c * speed * speed;
    let drag = m.velocity.copy();
    drag.mult(-1).normalize().mult(dragMag);
    return drag;
  }

  show() {
    noStroke();
    fill(180, 210, 240, 200);
    rect(this.x, this.y, this.w, this.h);
  }
}

// ================= MOVER =================

class Mover {
  constructor(x, y, mass) {
    this.mass = mass;
    this.radius = mass * 8;

    this.position = createVector(x, y);
    this.velocity = createVector(0, 0);
    this.acceleration = createVector(0, 0);

    this.noiseOffset = random(1000);
    this.deform = 0;
  }

  applyForce(force) {
    this.acceleration.add(p5.Vector.div(force, this.mass));
  }

  // 🔑 Atracción lenta y estable
  attract(other) {
    let force = p5.Vector.sub(other.position, this.position);
    let d = constrain(force.mag(), 20, 120);
    force.normalize();

    let G = 3; // 👈 MUY bajo = lento
    let strength = (G * this.mass * other.mass) / (d * d);
    force.mult(strength);

    return force;
  }

  grow() {
    this.mass += 0.005;
    this.radius = this.mass * 8;
  }

  update() {
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
    this.acceleration.mult(0);
  }

  show() {
    push();
    translate(this.position.x, this.position.y);

    fill(247, 101, 93, 200);
    noStroke(247, 101, 93, 200);
    strokeWeight(2);

    let step = map(this.radius, 5, 60, 0.35, 0.08, true);

    beginShape();
    for (let a = 0; a < TWO_PI; a += step) {
      let nx = cos(a) + 1 + this.noiseOffset;
      let ny = sin(a) + 1;
      let n = noise(nx * 1.2, ny * 1.2);

      let r = this.radius + map(n, 0, 1, -this.deform, this.deform);
      vertex(r * cos(a), r * sin(a));
    }
    endShape(CLOSE);

    pop();

    if (this.deform > 0.01) {
      this.noiseOffset += 0.01;
    }
  }

  checkEdges() {
    if (this.position.y > height - this.radius) {
      this.velocity.y = 0;
      this.position.y = height - this.radius;
    }
  }
}
```

> 
> 
> 
> Esta obra generativa representa la **contaminación de los océanos** como un proceso acumulativo y silencioso, donde pequeños elementos individuales terminan formando una **masa problemática** imposible de ignorar.
> 
> Las partículas circulares simbolizan residuos, micro plásticos o desechos que ingresan al océano de forma dispersa. Al inicio son pequeños, casi inofensivos, pero al entrar en el “agua” comienzan a **crecer**, tanto en tamaño como en complejidad visual. Este crecimiento no es solo físico: el aumento del **ruido orgánico en sus bordes** representa cómo la contaminación se vuelve cada vez más caótica, irregular y difícil de controlar a medida que se acumula.
> 
> El uso del **ruido de Perlin** en la forma de las partículas no busca realismo, sino transmitir inestabilidad. A mayor tamaño, mayor deformación: visualmente, lo que parecía simple se vuelve incómodo, agresivo y desordenado. Esto refleja cómo la contaminación, al concentrarse, deja de ser un problema invisible y se transforma en una amenaza evidente.
> 
> Dentro del agua, las partículas también **se atraen entre sí de manera lenta**, simulando cómo los residuos tienden a agruparse por corrientes marinas, zonas de acumulación y dinámicas naturales del océano. Esta atracción no es inmediata ni violenta: es progresiva, casi inevitable. La masa resultante representa la idea de que el problema no es un solo objeto, sino la **suma de muchos**, que al juntarse generan consecuencias mayores.
> 
> La aceleración, las fuerzas aplicadas y la deformación visual están directamente ligadas a esta narrativa:
> 
> - El agua no es solo un fondo, es el medio que **transforma**.
> - El crecimiento y el ruido simbolizan el **deterioro acumulativo**.
> - La atracción entre partículas refuerza la idea de que la contaminación no permanece aislada, sino que se concentra y se agrava.
> 
> En conjunto, el sistema de reglas da vida a una composición donde el caos no aparece de golpe, sino que **emerge lentamente**, reflejando cómo la contaminación oceánica se construye con el tiempo hasta convertirse en una masa densa, compleja y difícil de revertir.
> 

<img width="777" height="457" alt="image" src="https://github.com/user-attachments/assets/4f600600-51d2-4188-949c-32a3d4143e57" />


<aside>
🔗

https://editor.p5js.org/Luden-32/sketches/ITrCM-MKY

</aside>


### Reflect

**¿Que es el motion 101?**

Motion 101 es una forma básica de entender cómo se mueve algo a partir de cuatro cosas clave: posición, velocidad, aceleración y fuerza. En programación, por ejemplo en p5.js, esto se ve muy directo usando vectores: en cada fotograma la velocidad se suma a la posición para que el objeto se mueva. Desde la física clásica, la idea es que las fuerzas generan aceleración, la aceleración modifica la velocidad y, como resultado, la velocidad cambia la posición del objeto en el espacio.

<aside>
🔗

https://editor.p5js.org/Luden-32/sketches/NmPbjHTc6
https://youtu.be/VewUU7mYzTk?si=X4ESOQ179GkLko52&t=1844
<img width="879" height="567" alt="image" src="https://github.com/user-attachments/assets/5a311d75-649c-46a0-bc5e-e0785b44f7f2" />


</aside>

```jsx
let stems = [];
let windT = 0;

// espiral (Motion 101 angular)
let spiralRot = 0;
let spiralVel = 0;
let spiralAcc = 0;

// mouse
let prevMouse;

function setup() {
  createCanvas(700, 450);
  noiseDetail(2, 0.5);

  prevMouse = createVector(mouseX, mouseY);

  let count = 6;
  for (let i = 0; i < count; i++) {
    let x = map(i, 0, count - 1, 100, width - 100);
    stems.push(new Stem(x));
  }
}

function draw() {
  background(206, 213, 192);

  // ================= MOUSE SPEED =================
  let mouseNow = createVector(mouseX, mouseY);
  let mouseVel = p5.Vector.sub(mouseNow, prevMouse);
  let mouseSpeed = constrain(mouseVel.mag(), 0, 30);
  prevMouse = mouseNow.copy();

  drawRedSun();
  drawSpiral(mouseSpeed);

  for (let s of stems) {
    s.applyForces(mouseSpeed);
    s.update();
    s.show();
  }

  windT += 0.01;
}

// ================= TALLO (MOVER) =================

class Stem {
  constructor(x) {
    this.baseX = x;
    this.h = random(height * 0.55, height * 0.8);
    this.noiseOffset = random(1000);

    // Motion 101
    this.pos = createVector(0, 0); // desplazamiento lateral
    this.vel = createVector(0, 0);
    this.acc = createVector(0, 0);

    // hoja
    this.leafAngle = random(-PI / 10, PI / 10);
    this.leafLength = random(45, 75);
    this.leafWidth = random(12, 26);
  }

  applyForce(force) {
    // Nature of Code
    this.acc.add(force);
  }

  applyForces(mouseSpeed) {
    // viento base (ruido)
    let windStrength = map(
      noise(windT + this.noiseOffset),
      0, 1,
      -0.04, 0.04
    );
    let wind = createVector(windStrength, 0);
    this.applyForce(wind);

    // mouse = más viento
    if (mouseSpeed > 0) {
      let mouseWind = createVector(mouseSpeed * 0.0006, 0);
      this.applyForce(mouseWind);
    }

    // fuerza de retorno (tipo resorte)
    let restore = createVector(-this.pos.x * 0.02, 0);
    this.applyForce(restore);
  }

  update() {
    // Motion 101 clásico
    this.vel.add(this.acc);
    this.vel.mult(0.95); // fricción
    this.pos.add(this.vel);
    this.acc.mult(0);
  }

  show() {
    stroke(120);
    strokeWeight(2);
    noFill();

    let steps = 20;
    beginShape();
    for (let i = 0; i <= steps; i++) {
      let y = map(i, 0, steps, height, height - this.h);
      let bend = map(i, 0, steps, 0, this.pos.x * 120);
      let n = noise(i * 0.2, this.noiseOffset);
      let jitter = map(n, 0, 1, -4, 4);

      vertex(this.baseX + bend + jitter, y);
    }
    endShape();

    let tipX = this.baseX + this.pos.x * 120;
    let tipY = height - this.h;

    drawLeaf(
      tipX,
      tipY,
      this.leafAngle + this.pos.x * 0.3,
      this.leafLength,
      this.leafWidth
    );
  }
}

// ================= HOJA =================

function drawLeaf(x, y, angle, len, wid) {
  push();
  translate(x, y);
  rotate(angle);

  stroke(121, 122, 117);
  strokeWeight(3);
  fill(121, 122, 117);

  beginShape();
  vertex(0, 0);
  vertex(-wid * 0.5 + noise(x) * 2, -len);
  vertex(wid * 0.5 + noise(y) * 2, -len);
  endShape(CLOSE);

  pop();
}

// ================= CÍRCULO ROJO =================

function drawRedSun() {
  noStroke();
  fill(180, 40, 30);
  circle(width * 0.35, height * 0.55, 210);
}

// ================= ESPIRAL =================

function drawSpiral(mouseSpeed) {
  push();
  translate(width * 0.65, height * 0.6);

  // Motion 101 angular (igual al Mover)
  spiralAcc += mouseSpeed * 0.0004;
  spiralVel += spiralAcc;
  spiralVel *= 0.96;
  spiralRot += spiralVel;
  spiralAcc = 0;

  rotate(spiralRot);

  noFill();
  stroke(20);

  let r = 6;
  for (let a = 0; a < TWO_PI * 2; a += 0.12) {
    let w = map(a, 0, TWO_PI * 2, 10, 1);
    strokeWeight(w);

    let x1 = cos(a) * r;
    let y1 = sin(a) * r;
    let x2 = cos(a + 0.12) * (r + 1);
    let y2 = sin(a + 0.12) * (r + 1);

    line(x1, y1, x2, y2);
    r += 1;
  }

  pop();
}
```
