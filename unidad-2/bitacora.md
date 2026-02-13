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
https://editor.p5js.org/Luden-32/sketches/229ghFr4m

