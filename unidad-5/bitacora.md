# Unidad 5  
# Actividad  
## Codigo  
<aside>
🔗

**link:** [https://editor.p5js.org/Luden-32/full/3JEKdBPvu](https://editor.p5js.org/Luden-32/sketches/3JEKdBPvu)

</aside>

#### DustParticle

```jsx
class DustParticle extends Particle {
  constructor(x, y) {
    super(x, y);
    this.alpha = 0;
  }

  update(universe) {
    super.update();

    this.alpha = min(this.alpha + 2, 255);
    this.velocity.mult(0.97);

    if (universe.targets.length > 0) {
      let closest = null;
      let minDist = Infinity;

      for (let t of universe.targets) {
        let d = p5.Vector.dist(this.position, t);
        if (d < minDist) {
          minDist = d;
          closest = t;
        }
      }

      if (closest) {
        let dir = p5.Vector.sub(closest, this.position);
        let d = dir.mag();

        dir.normalize();

        let strength = 1 / (d * 0.05);
        strength = constrain(strength, 0.01, 0.5);

        dir.mult(strength);
        this.applyForce(dir);
      }
    }
  }

  show() {
    noStroke();
    fill(200, 200, 255, this.alpha);
    circle(this.position.x, this.position.y, 3);
  }
}
```

#### Particle

```jsx
class Particle {
  constructor(x, y) {
    this.position = createVector(x, y);
    this.velocity = p5.Vector.random2D().mult(random(0.5, 1.5));
    this.acceleration = createVector(0, 0);
    this.lifespan = 255;
  }

  applyForce(force) {
    this.acceleration.add(force);
  }

  update() {
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
    this.acceleration.mult(0);
    this.lifespan -= 1;
  }

  isDead() {
    return this.lifespan <= 0;
  }
}
```

#### Planet

```jsx
class Planet {
  constructor(x, y) {
    this.position = createVector(x, y);
    this.velocity = createVector(0, 0);
    this.acceleration = createVector(0, 0);

    this.size = 20;
    this.mass = 1;
  }

  applyForce(force) {
    let f = p5.Vector.div(force, this.mass);
    this.acceleration.add(f);
  }

  grow() {
    this.size += 0.4;
    this.mass += 0.05;

    this.mass = constrain(this.mass, 1, 12);
  }

  update() {
    this.velocity.add(this.acceleration);
    this.velocity.mult(0.99); // fricción leve
    this.position.add(this.velocity);
    this.acceleration.mult(0);
  }

  show() {
    noStroke();
    fill(100, 200, 255);
    circle(this.position.x, this.position.y, this.size);
  }
}
```

#### Universo

```jsx
class Universe {
  constructor() {
    this.particles = [];
    this.planets = [];
    this.targets = [];
  }

  run() {
    // 🌫️ generar polvo
    if (this.particles.length < 500 && random(1) < 0.3) {
      this.particles.push(new DustParticle(random(width), random(height)));
    }

    // 🌍 CREAR PLANETAS POR DENSIDAD
    for (let t of this.targets) {
      let count = 0;

      for (let p of this.particles) {
        let d = p5.Vector.dist(p.position, t);
        if (d < 40) count++;
      }

      let exists = false;
      for (let planet of this.planets) {
        let d = p5.Vector.dist(planet.position, t);
        if (d < 10) {
          exists = true;
          break;
        }
      }

      if (count > 20 && !exists) {
        this.planets.push(new Planet(t.x, t.y));
      }
    }

    // 🌍 GRAVEDAD ENTRE PLANETAS
    for (let i = 0; i < this.planets.length; i++) {
      let a = this.planets[i];

      for (let j = i + 1; j < this.planets.length; j++) {
        let b = this.planets[j];

        let dir = p5.Vector.sub(b.position, a.position);
        let d = dir.mag();

        d = constrain(d, 10, 200);

        dir.normalize();

        let strength = (a.mass * b.mass * 0.05) / (d * d);
        strength = constrain(strength, 0, 0.2);

        let force = dir.copy().mult(strength);

        a.applyForce(force);
        b.applyForce(force.copy().mult(-1));
      }
    }

    // 🔁 loop partículas
    for (let i = this.particles.length - 1; i >= 0; i--) {
      let p = this.particles[i];

      p.update(this);

      // 🌍 gravedad de planetas sobre partículas
      for (let planet of this.planets) {
        let dir = p5.Vector.sub(planet.position, p.position);
        let d = dir.mag();

        d = constrain(d, 5, 100);

        dir.normalize();

        let strength = (planet.mass * 0.2) / (d * d);
        strength = constrain(strength, 0, 0.05);

        dir.mult(strength);
        p.applyForce(dir);
      }

      p.show();

      let absorbed = false;

      // 🌍 absorción
      for (let planet of this.planets) {
        let d = dist(
          p.position.x,
          p.position.y,
          planet.position.x,
          planet.position.y
        );

        if (d < planet.size / 2) {
          planet.grow();
          this.particles.splice(i, 1);
          absorbed = true;
          break;
        }
      }

      if (absorbed) continue;

      if (p.isDead()) {
        this.particles.splice(i, 1);
      }
    }

    // 🌍 actualizar planetas
    for (let planet of this.planets) {
      planet.update();
      planet.show();
    }
  }
}
```

#### sketch

```jsx
let universe;

function setup() {
  createCanvas(800, 600);
  universe = new Universe();
}

function draw() {
  background(0);
  universe.run();
}

function mousePressed() {
  universe.targets.push(createVector(mouseX, mouseY));
}
```

#### index

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Dust to Planet</title>

    <!-- p5 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>

    <style>
      body {
        margin: 0;
        overflow: hidden;
        background: black;
      }
    </style>
  </head>

  <body>
    <!-- ORDEN IMPORTANTE -->

    <script src="Particle.js"></script>
    <script src="DustParticle.js"></script>
    <script src="Planet.js"></script>
    <script src="Universe.js"></script>

    <!-- siempre el último -->
    <script src="sketch.js"></script>
  </body>
</html>
```

## Concepto

> El sistema representa un ciclo de formación cósmica donde partículas de polvo dispersas se agrupan en torno a centros de atracción, dando origen a planetas que crecen y se influyen mutuamente. A medida que aumenta su masa, también lo hace su capacidad de atraer más materia y dominar el entorno.
> 
### Estados
<img width="946" height="872" alt="image" src="https://github.com/user-attachments/assets/0cd22a9f-eba7-471e-89c4-79871eb45210" />
<img width="944" height="869" alt="2" src="https://github.com/user-attachments/assets/9fe12dcb-e0e5-4e03-966e-afbbe3dcc408" />
<img width="945" height="867" alt="3" src="https://github.com/user-attachments/assets/5cdb1193-8492-4025-8663-46930888ac5f" />

### Bocetos
![BOCETO](https://github.com/user-attachments/assets/e1523405-93aa-44c6-b200-dc83acc1ff13)
### Mapa
<img width="1974" height="1735" alt="MAPA" src="https://github.com/user-attachments/assets/2edacd44-3667-4348-a9bc-364432107f24" />



# Reflect

### Parte 1

**1. Una partícula es una entidad con estado**

Una partícula no es solo un punto visual, sino un objeto que contiene información interna como posición, velocidad, aceleración y tiempo de vida. Esto le permite comportarse de manera autónoma y evolucionar en el tiempo según sus propias variables.

**2. Una partícula tiene ciclo de vida**

Cada partícula tiene una existencia limitada que incluye nacimiento, transformación y muerte. Este ciclo no es solo técnico, sino que define cómo cambia visual y dinámicamente dentro del sistema. En el proyecto, las partículas nacen como polvo, se desplazan y finalmente son absorbidas por un planeta.

**3. Un sistema de partículas gestiona colecciones dinámicas de elementos**

El sistema no trabaja con una sola partícula, sino con múltiples instancias que se crean y eliminan constantemente. Esto genera comportamientos complejos y emergentes que no pueden predecirse observando una sola partícula.

**4. La creación y eliminación de partículas es parte central del modelo**

La aparición y desaparición de partículas no es solo una optimización técnica, sino una decisión de diseño. En este caso, las partículas no desaparecen arbitrariamente, sino que son absorbidas por planetas, lo que refuerza la idea de transformación dentro del sistema.

**5. Separación entre lógica individual y lógica del sistema**

Cada partícula tiene su propia lógica de movimiento y comportamiento, mientras que el sistema se encarga de gestionar el conjunto: creación, eliminación y aplicación de reglas globales. Esta separación permite mantener el código organizado y escalable.

**6. El emisor o sistema de partículas como abstracción**

El emisor define cómo y dónde nacen las partículas. En este proyecto, el "universo" cumple ese rol, generando partículas constantemente y permitiendo que el usuario introduzca puntos de atracción que influyen en su comportamiento.

**7. Sistemas de sistemas**

El proyecto no se limita a un solo nivel: hay partículas que forman planetas, y planetas que interactúan dentro de un sistema mayor. Esto crea una jerarquía de comportamientos donde cada nivel tiene sus propias reglas.

**8. Heterogeneidad mediante herencia y polimorfismo**

Aunque el sistema parte de una clase base de partícula, se pueden definir distintos tipos con comportamientos específicos. Esto permite extender el sistema sin cambiar su estructura base, añadiendo complejidad de forma controlada.

**9. Respuesta a fuerzas globales y locales**

Las partículas responden a diferentes tipos de fuerzas: algunas afectan a todo el sistema (como la dispersión inicial) y otras dependen del contexto, como la atracción hacia un planeta cercano o hacia un punto definido por el usuario.

**10. Independencia entre lógica y representación visual**

El comportamiento del sistema es independiente de su apariencia. Es posible cambiar colores, tamaños o formas sin alterar la lógica subyacente, lo que permite reinterpretar el mismo sistema en diferentes contextos visuales.

### Parte 2

Si esta pieza se recreara en otra herramienta como Unity, TouchDesigner o Blender, gran parte de su lógica se mantendría intacta, mientras que la implementación técnica cambiaría.

**Elementos que se mantienen**

- La estructura conceptual del sistema (partículas, planetas, universo).
- El ciclo de vida de las partículas.
- Las reglas de interacción (atracción, absorción, crecimiento).
- La idea de comportamiento emergente a partir de elementos simples.

Estos aspectos son independientes de la herramienta, ya que pertenecen al diseño del sistema y no a su implementación específica.

**Elementos que cambiarían**

- El uso de librerías y estructuras propias de cada entorno (por ejemplo, `p5.Vector` en p5.js vs `Vector3` en Unity).
- La forma en que se actualizan los objetos en el tiempo (loop de p5.js vs `Update()` en Unity).
- La representación visual (shaders, sistemas de partículas nativos, nodos, etc.).
- La gestión de rendimiento y optimización.
