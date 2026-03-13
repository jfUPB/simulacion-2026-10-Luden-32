## Actividad
<aside>
🔗

**LINK**: https://editor.p5js.org/Luden-32/full/e90KVWKvf


</aside>

```jsx
let t = 0;
let movers = [];
let totalMovers = 15;
let liquid;
let oscillators = [];
let mode = "ATTRACTION";

function setup() {
  createCanvas(windowWidth, windowHeight);
  noiseDetail(3, 0.4);
  
  liquid = new Liquid(0, 0, 0, 0, 0.5);
  
  for (let i = 0; i < 5; i++) {
    oscillators.push(new Oscillator(i));
  }
  
  resetMovers();
}

function draw() {
  background(255, 235, 240);
  
  push();
  translate(width/2, height/2);
  
  let rBase = 180;
  let deform = 25 + map(mouseX, 0, width, 0, 50);
  
  liquid.x = width/2 - rBase - deform;
  liquid.y = height/2 - rBase - deform;
  liquid.w = (rBase + deform) * 2;
  liquid.h = (rBase + deform) * 2;
  
  noStroke();
  fill(180, 210, 240, 120);
  
  beginShape();
  for (let a = 0; a < TWO_PI; a += 0.05) {
    let nx = cos(a) + 1 + t * 0.3;
    let ny = sin(a) + 1;
    let n = noise(nx * 1.5, ny * 1.5);
    let r = rBase + map(n, 0, 1, -deform, deform);
    vertex(r * cos(a), r * sin(a));
  }
  endShape(CLOSE);
  
  stroke(180, 210, 240, 200);
  strokeWeight(5);
  noFill();
  
  beginShape();
  for (let a = 0; a < TWO_PI; a += 0.05) {
    let nx = cos(a) + 1 + t * 0.3;
    let ny = sin(a) + 1;
    let n = noise(nx * 1.5, ny * 1.5);
    let r = rBase + map(n, 0, 1, -deform, deform);
    vertex(r * cos(a), r * sin(a));
  }
  endShape(CLOSE);
  
  pop();
  
  for (let osc of oscillators) {
    osc.update();
    osc.show();
  }
  
  for (let i = movers.length - 1; i >= 0; i--) {
    let mover = movers[i];
    
    let attraction = createVector(mouseX, mouseY);
    let force = p5.Vector.sub(attraction, mover.position);
    let distance = force.mag();
    
    distance = constrain(distance, 10, 300);
    force.normalize();
    let strength = map(distance, 10, 300, 5, 0.5);
    force.mult(strength);
    mover.applyForce(force);
    
    let centerX = width/2;
    let centerY = height/2;
    let dx = mover.position.x - centerX;
    let dy = mover.position.y - centerY;
    let distToCenter = sqrt(dx*dx + dy*dy);
    
    let angle = atan2(dy, dx);
    let nx = cos(angle) + 1 + t * 0.3;
    let ny = sin(angle) + 1;
    let n = noise(nx * 1.5, ny * 1.5);
    let fluidRadius = rBase + map(n, 0, 1, -deform, deform);
    
    let isInFluid = distToCenter < fluidRadius;
    
    if (isInFluid) {
      let dragForce = liquid.calculateDrag(mover);
      mover.applyForce(dragForce);
      
      if (distToCenter > fluidRadius * 0.8) {
        let toCenter = createVector(centerX - mover.position.x, centerY - mover.position.y);
        toCenter.normalize();
        toCenter.mult(0.2);
        mover.applyForce(toCenter);
      }
    }
    
    let gravity = createVector(0, 0.05 * mover.mass);
    mover.applyForce(gravity);
    
    mover.update();
    mover.show();
    mover.checkEdges(width, height, isInFluid);
  }
  
  t += 0.02;
}

function mousePressed() {
  movers.push(new Mover(mouseX, mouseY, random(1, 4)));
}

function keyPressed() {
  if (key === ' ') {
    resetMovers();
  } else if (key === 'm' || key === 'M') {
    if (mode === "ATTRACTION") {
      mode = "WAVES";
    } else {
      mode = "ATTRACTION";
    }
  }
}

function resetMovers() {
  movers = [];
  for (let i = 0; i < totalMovers; i++) {
    movers.push(new Mover(random(width), random(height), random(1, 3)));
  }
}

class Liquid {
  constructor(x, y, w, h, c) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.c = c;
  }

  calculateDrag(m) {
    let speed = m.velocity.mag();
    let dragMag = this.c * speed * speed;
    let drag = m.velocity.copy();
    drag.mult(-1).normalize().mult(dragMag);
    return drag;
  }
}

class Mover {
  constructor(x, y, mass) {
    this.mass = mass;
    this.radius = mass * 8;
    this.position = createVector(x, y);
    this.velocity = createVector(random(-0.5, 0.5), random(-0.5, 0.5));
    this.acceleration = createVector(0, 0);
    this.topspeed = 5;
    this.color = color(random(200, 255), random(150, 200), random(150, 200));
  }

  applyForce(force) {
    this.acceleration.add(p5.Vector.div(force, this.mass));
  }

  update() {
    this.velocity.add(this.acceleration);
    this.velocity.limit(this.topspeed);
    this.position.add(this.velocity);
    this.acceleration.mult(0);
  }

  show() {
    push();
    translate(this.position.x, this.position.y);
    
    if (mode === "WAVES") {
      fill(this.color);
      noStroke();
      circle(0, 0, this.radius * 1.5);
    } else {
      fill(this.color);
      noStroke();
      circle(0, 0, this.radius * 2);
    }
    
    pop();
  }

  checkEdges(w, h, isInFluid) {
    let bounce = isInFluid ? 0.3 : 0.7;
    
    if (this.position.x < this.radius) {
      this.position.x = this.radius;
      this.velocity.x *= -bounce;
    }
    if (this.position.x > w - this.radius) {
      this.position.x = w - this.radius;
      this.velocity.x *= -bounce;
    }
    if (this.position.y < this.radius) {
      this.position.y = this.radius;
      this.velocity.y *= -bounce;
    }
    if (this.position.y > h - this.radius) {
      this.position.y = h - this.radius;
      this.velocity.y *= -bounce;
    }
  }
}

class Oscillator {
  constructor(index) {
    this.index = index;
    this.angle = createVector(0, 0);
    this.angleVelocity = createVector(
      random(0.02, 0.05),
      random(0.02, 0.05)
    );
    this.amplitude = createVector(
      random(120, 220),
      random(60, 120)
    );
    this.offset = createVector(
      random(width),
      random(height)
    );
  }

  update() {
    this.angle.add(this.angleVelocity);
  }

  show() {
    let x = this.offset.x + sin(this.angle.x) * this.amplitude.x;
    let y = this.offset.y + cos(this.angle.y) * this.amplitude.y;
    
    if (mode === "WAVES" || dist(x, y, mouseX, mouseY) < 200) {
      push();
      stroke(80, 60, 120, 150);
      strokeWeight(2);
      noFill();
      
      circle(x, y, 30);
      line(this.offset.x, this.offset.y, x, y);
      
      fill(60, 40, 100, 180);
      noStroke();
      circle(x, y, 10);
      pop();
    }
  }
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}


```

> 
> 
> 
> Mi obra generativa parte de la idea de un pequeño sistema donde varias partículas se mueven dentro de un entorno que cambia. Primero definí que las partículas reaccionaran a fuerzas simples, como la atracción hacia el mouse y una pequeña gravedad. Entonces, aunque las reglas son básicas, el movimiento empieza a volverse más complejo cuando muchas partículas interactúan al mismo tiempo.
> 
> Además, en el centro hay una especie de fluido que funciona como una zona que modifica el comportamiento de las partículas. Cuando entran ahí se mueven más lento y tienden a quedarse dentro, lo que crea una dinámica diferente entre el interior y el exterior.
> 
> Por otro lado, los osciladores generan ondas que aportan movimiento visual al espacio. Finalmente, el usuario puede agregar nuevas partículas con el mouse, lo que cambia constantemente el equilibrio del sistema.
>
> 




