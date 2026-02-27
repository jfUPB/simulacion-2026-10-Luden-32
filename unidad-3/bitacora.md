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
