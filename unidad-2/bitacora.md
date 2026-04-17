### Actividad 9:

> **Concepto**
Representa la tensión entre la autonomía y la atracción. Las partículas habitan un vacío absoluto y solo reaccionan cuando un ente externo (el cursor) invade su espacio personal, creando una danza de órbitas y persecuciones.
**Regla de Aceleración: Inversa a la Distancia**
Aplicamos una lógica basada en la **Ley de Gravitación**:
  $a = \frac{G}{d}$
• **Por qué:** Para romper la linealidad. Esto genera un comportamiento orgánico donde los objetos ignoran al usuario si está lejos (paz), pero caen violentamente hacia él si se acerca (caos).
**Exploración Artística**
• **Diseño:** Sustituí el rastro "sucio" por un historial geométrico para evocar una **memoria efímera**.
• **Sensación:** La obra me evoca un microcosmos o sistemas estelares. El movimiento lento a la distancia sugiere **desapego**, mientras que el frenesí cercano sugiere **obsesión** o dependencia gravitatoria.
> 

```jsx
let movers = [];
let totalMovers = 100;
let sunRadius = 10; // Tamaño del "sol"

function setup() {
  createCanvas(windowWidth, windowHeight);
  resetMovers();
}

function draw() {
  background(0);
  displayInstructions();

  // Dibujamos el "Sol" en la posición del mouse
  fill(253,253,150);
  stroke(255, 100);
  circle(mouseX, mouseY, sunRadius *2 );
  fill(255, 30);
  circle(mouseX, mouseY, sunRadius*2);

  for (let mover of movers) {
    mover.update();
    mover.show();
  }
}

function keyPressed() {
  if (key === ' ' || keyCode === 32) {
    resetMovers();
  }
}

function resetMovers() {
  movers = [];
  for (let i = 0; i < totalMovers; i++) {
    movers[i] = new Mover();
  }
}

function displayInstructions() {
  fill(255, 100);
  noStroke();
  textSize(14);
  textAlign(LEFT, TOP);
  text("Gravedad Sólida: Los objetos no atraviesan el Sol", 20, 20);
  text("ESPACIO para reiniciar", 20, 40);
}

class Mover {
  constructor() {
    this.position = createVector(random(width), random(height));
    this.velocity = createVector(0, 0);
    this.acceleration = createVector(0, 0);
    this.topspeed = 12;
    this.history = []; 
  }

  update() {
    let mouse = createVector(mouseX, mouseY);
    let force = p5.Vector.sub(mouse, this.position);
    let distance = force.mag();

    // --- Lógica de Colisión (El Sol como cuerpo sólido) ---
    if (distance < sunRadius) {
      // Si entra al radio, lo empujamos al borde
      let overlap = sunRadius - distance;
      let escapeDirection = p5.Vector.sub(this.position, mouse);
      escapeDirection.normalize();
      escapeDirection.mult(overlap);
      this.position.add(escapeDirection);
      
      // Rebotamos un poco la velocidad para que no se pegue
      this.velocity.reflect(escapeDirection);
      this.velocity.mult(0.5); 
      
      distance = sunRadius; // Ajustamos distancia para el cálculo de fuerza
    }

    // --- Lógica de Atracción Inversa ---
    distance = constrain(distance, 5, 500);
    force.normalize();
    let strength = 50 / distance; 
    force.mult(strength); 
    
    this.acceleration = force;
    this.velocity.add(this.acceleration);
    this.velocity.limit(this.topspeed);
    this.position.add(this.velocity);
    this.velocity.mult(0.98);

    this.history.push(this.position.copy());
    if (this.history.length > 15) {
      this.history.splice(0, 1);
    }
  }

  show() {
    noFill();
    strokeWeight(2);
    beginShape();
    for (let i = 0; i < this.history.length; i++) {
      let pos = this.history[i];
      let alpha = map(i, 0, this.history.length, 0, 200);
      stroke(255, 50, 100, alpha); 
      vertex(pos.x, pos.y);
    }
    endShape();

    fill(255);
    noStroke();
    circle(this.position.x, this.position.y, 4);
  }
}
```

<img width="837" height="851" alt="image" src="https://github.com/user-attachments/assets/ca7127c2-2efe-4c70-b9f6-32d6a6f99a10" />



[https://editor.p5js.org/natureofcode/full/VQfwqpDlv](https://editor.p5js.org/Luden-32/sketches/wMVDBpivh)


### Actividad 10

```jsx
let suns = [];
let particles = [];

let totalSuns = 5;
let particlesPerSun = 25;

// ─── SOL DEL CURSOR ───
let cursorSunRadius = 6;
let cursorStrength = 1.4;
let cursorRange = 180;

function setup() {
  createCanvas(windowWidth, windowHeight);

  // Crear soles
  for (let i = 0; i < totalSuns; i++) {
    suns.push(new Sun(random(width), random(height)));
  }

  // Crear partículas orbitando
  for (let sun of suns) {
    for (let i = 0; i < particlesPerSun; i++) {
      particles.push(new Particle(sun));
    }
  }
}

function draw() {
  background(0);

  let cursorSun = createVector(mouseX, mouseY);

  // Dibujar sol del cursor (pequeño pero poderoso)
  noStroke();
  fill(253, 253, 150);
  circle(cursorSun.x, cursorSun.y, cursorSunRadius * 2);

  // Actualizar soles
  for (let sun of suns) {
    sun.update(cursorSun);
    sun.show();
  }

  // Actualizar partículas
  for (let p of particles) {
    p.update(cursorSun);
    p.show();
  }
}

/* =========================
          SUN
========================= */
class Sun {
  constructor(x, y) {
    this.pos = createVector(x, y);
    this.mass = random(35, 60);
    this.radius = this.mass * 0.3;
    this.minOrbit = this.radius + 14;
  }

  update(cursorSun) {
    // Repulsión suave del cursor
    let force = p5.Vector.sub(this.pos, cursorSun);
    let d = force.mag();

    if (d < cursorRange) {
      d = constrain(d, 30, cursorRange);
      force.normalize();
      force.mult(35 / d);
      this.pos.add(force);
    }
  }

  show() {
    noStroke();
    fill(255, 180);
    circle(this.pos.x, this.pos.y, this.radius * 2);
  }
}

/* =========================
        PARTICLE
========================= */
class Particle {
  constructor(sun) {
    this.sun = sun;

    let angle = random(TWO_PI);
    let r = random(sun.minOrbit + 10, sun.minOrbit + 60);

    this.pos = p5.Vector.fromAngle(angle).mult(r).add(sun.pos);
    this.vel = p5.Vector.fromAngle(angle + HALF_PI).mult(random(1.2, 2));
  }

  update(cursorSun) {
    let toSun = p5.Vector.sub(this.sun.pos, this.pos);
    let dSun = toSun.mag();

    // ─── ORBITA ESTABLE ───
    if (dSun < this.sun.minOrbit) {
      // empuje tangencial (evita colisión)
      let tangent = createVector(-toSun.y, toSun.x);
      tangent.normalize();
      tangent.mult(0.6);
      this.vel.add(tangent);
    } else {
      toSun.normalize();
      toSun.mult(this.sun.mass / (dSun * 0.9));
      this.vel.add(toSun);
    }

    // ─── INFLUENCIA DEL CURSOR (LIMITADA) ───
    let toCursor = p5.Vector.sub(cursorSun, this.pos);
    let dCursor = toCursor.mag();

    if (dCursor < cursorRange) {
      let influence = map(dCursor, 0, cursorRange, cursorStrength, 0);
      toCursor.normalize();
      toCursor.mult(influence);
      this.vel.add(toCursor);
    }

    this.vel.limit(4);
    this.pos.add(this.vel);
    this.vel.mult(0.995);
  }

  show() {
    noStroke();
    fill(255, 120, 180);
    circle(this.pos.x, this.pos.y, 3);
  }
}

```

> 
> 
> 
> Mi obra es un sistema de soles y partículas que simula cómo funcionan las influencias.
> Cada sol atrae partículas que giran a su alrededor y forman órbitas estables. Todo está en equilibrio… hasta que aparece otro sol.
> 
> El cursor soy yo: un sol más pequeño, pero más fuerte. No destruye el sistema, solo lo altera. Cuando me acerco, las partículas cambian su comportamiento, pierden su órbita original y se ven atraídas por una nueva fuerza.
> 
> La obra habla de cómo una presencia puede cambiar dinámicas ya establecidas sin necesidad de tocar nada directamente. Todo sigue moviéndose, pero ya no de la misma forma.
> 

https://editor.p5js.org/Luden-32/sketches/229ghFr4m
<img width="923" height="852" alt="image" src="https://github.com/user-attachments/assets/9e5d7c37-497e-48f3-991b-7ac744da57f5" />




