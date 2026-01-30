# Unidad 1

## Bitácora de proceso de aprendizaje  

## Actividad 2

**Código modificado**

```jsx
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(600, 600);
  walker = new Walker();
  background(0);
}

function draw() {
  walker.step();
  walker.show();
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  show() {
    stroke(0,0,255);
    circle(this.x,this.y,9)
    fill(0)
    point(this.x,this.y,50);
  }

  step() {
    const choice = floor(random(7));
    if (choice == 0) {
      this.x++;
    } else if (choice == 1) {
      this.x--;
    } else if (choice == 2) {
      this.y++;
    } else if (choice == 3) {
      this.y--;
    } else if (choice == 4) {
      this.x = this.x + 5;
    } else if (choice == 5) {
      this.y = this.y + 5;
    } else if (choice == 6) {
      this.x = this.x - 5;
    } else if (choice == 7) {
      this.y = this.y - 5;
    }
  }
}

```

**¿Qué esperas que suceda?**

- Espero que cambie el color, la forma y ponerle relleno al walker (el objeto), el color del fondo también. Espero que el comportamiento cambie a que de zancadas mas grandes con 4 choices que le puse. En este le puse, por ejemplo `this.y = this.y-5`  y le puse su contraparte que sería `this.y = this.y+5` para que este se mantuviera igualitario en su comportamiento

**¿Qué sucedió realmente? ¿Por qué crees que es así?**

- Había mal entendido el funcionamiento del random, puesto que me estaba basando en el numero de la Choice y no en la cantidad de choices, por ende el objeto tendía a bajar. Luego modifiqué el código para que cumpliera con la condición, subiéndole a 8 en el random del Choice, y tuvo un comportamiento normal. En cuanto a la apariencia, este cumplió con lo que pensé que iba a hacer.

## Actividad 3

**Diferencia entre distribución uniforme y no uniforme de números aleatorios**

- La distribución uniforme sería que en las variables posibles, cada una tenga una contra parte y cada una tiene la misma probabilidad de que ocurra. En un caso contrario, sería que o cambie la probabilidad o la “Contraparte” no sea igual, por ejemplo +5 y -5 sería +5 y -9.

**Código modificado para que tienda a ir a la derecha**

```jsx
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(600, 600);
  walker = new Walker();
  background(0);
}

function draw() {
  walker.step();
  walker.show();
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  show() {
    stroke(0,0,255);
    circle(this.x,this.y,9)
    fill(0)
    point(this.x,this.y,50);
  }

  step() {
    const choice = floor(random(8));
    if (choice == 0) {
      this.x++;
    } else if (choice == 1) {
      this.x--;
    } else if (choice == 2) {
      this.y++;
    } else if (choice == 3) {
      this.y--;
    } else if (choice == 4) {
      this.x = this.x + 10;
    } else if (choice == 5) {
      this.y = this.y + 5;
    } else if (choice == 6) {
      this.x = this.x - 5;
    } else if (choice == 7) {
      this.y = this.y - 5;
    }
  }
}
```

## Actividad 4

```jsx
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(600, 600);
  walker = new Walker();
  background(255);
}

function drawHeart(x, y, size) {
  beginShape();
  vertex(x, y);
  bezierVertex(x - size / 2, y - size / 2,
               x - size, y + size / 3,
               x, y + size);
  bezierVertex(x + size, y + size / 3,
               x + size / 2, y - size / 2,
               x, y);
  endShape(CLOSE);
}

function draw() {
  let x = randomGaussian(300, 50);
  let y =randomGaussian(300,50)
  noStroke();
  fill(251, 186, 190, 100);
  drawHeart(x, y, 16);
  
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }
}
```
<img width="723" height="763" alt="image" src="https://github.com/user-attachments/assets/19e57123-66d4-4128-92a7-c553baaefe06" />

## Actividad 5

```jsx
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(600, 600);
  walker = new Walker();
  background(0);
}

function draw() {
  walker.step();
  walker.show();
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  show() {
    stroke(0,0,255);
    circle(this.x,this.y,9)
    fill(0)
    point(this.x,this.y,50);
  }

  step() {
  let r = random(1);

  if (r < 0.01) {
    // 1% de probabilidad: salto grande
    this.x += random(-100, 100);
    this.y += random(-100, 100);
  } else {
    // 99%: caminata normal tipo Nature of Code
    const choice = floor(random(4));
    if (choice == 0) {
      this.x++;
    } else if (choice == 1) {
      this.x--;
    } else if (choice == 2) {
      this.y++;
    } else {
      this.y--;
    }
  }
  }

}
```

</aside>

## Bitácora de aplicación 
### Actividad 7

```jsx
let t = 0;
let heartsLayer;
let walker;

function setup() {
  createCanvas(700, 700);
  noiseDetail(3, 0.4);

  heartsLayer = createGraphics(width, height);
  heartsLayer.background(128, 155, 206);

  walker = new Walker();
}

function draw() {

  heartsLayer.noStroke();
  heartsLayer.fill(0, 80);

  let hx = randomGaussian(walker.x, 20);
  let hy = randomGaussian(walker.y, 20);
  drawHeart(heartsLayer, hx, hy, 16);

  image(heartsLayer, 0, 0);

  push();
  translate(width / 2, height / 2);

  //fill(234, 196, 213);
  noFill();
  stroke(186, 225, 211);
  strokeWeight(3);

  let rBase = 120;
  let deform = 7 + map(mouseX, 0, width, 0, 60);

  beginShape();
  for (let a = 0; a < TWO_PI; a += 0.06) {
    let nx = cos(a) + 1 + t * 0.5;
    let ny = sin(a) + 1;
    let n = noise(nx * 1.5, ny * 1.5);
    let r = rBase + map(n, 0, 1, -deform, deform);
    vertex(r * cos(a), r * sin(a));
  }
  endShape(CLOSE);
  pop();

  walker.step(rBase, deform);
  walker.show();

  t += 0.05;
}


class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  show() {
    noStroke();
    fill(255);
    circle(this.x, this.y, 8);
  }

  step(rBase, deform) {
    let px = this.x;
    let py = this.y;

    this.x += random(-3, 3);
    this.y += random(-3, 3);

    // posición relativa al centro
    let dx = this.x - width / 2;
    let dy = this.y - height / 2;

    let angle = atan2(dy, dx);

    // mismo ruido que el círculo
    let nx = cos(angle) + 1 + t * 0.5;
    let ny = sin(angle) + 1;
    let n = noise(nx * 1.5, ny * 1.5);

    let maxR = rBase + map(n, 0, 1, -deform, deform);

    // si se sale, vuelve atrás
    if (sqrt(dx * dx + dy * dy) > maxR) {
      this.x = px;
      this.y = py;
    }
  }
}


function drawHeart(pg, x, y, size) {
  pg.beginShape();
  pg.vertex(x, y);
  pg.bezierVertex(
    x - size / 2, y - size / 2,
    x - size, y + size / 3,
    x, y + size
  );
  pg.bezierVertex(
    x + size, y + size / 3,
    x + size / 2, y - size / 2,
    x, y
  );
  pg.endShape(CLOSE);
}

```

> El codigo genera un circulo el cual se mueve utilizando Ruido de perlin, pensé en que sea como esos circulos que salen en los videos de los videos de dubstepgutter en Youtube. Pero obviamente no era a base de sonido, sino a base de el Ruido de Perlin, este aumenta la magnitud del ruido cuando se mueve el mouse a la derecha, y disminuye hacia la izquierda. Adicional a eso decidí utilizar el Walker de la segunda actividad, este no puede moverse cuando está fuera de los limites del que llamaré “circulo ruidoso de Perlin”, también tiene en cuenta los altibajos ocasionados por la magnitud del ruido. Y para aplicar el tercer concepto de los dados en clase, utilicé el de distribución normal, con un numero de dispersión no tan alto, este se va a seguir al Walker, es decir, como en distribución normal las formas tienen a reunirse en el centro, las formas en mi código tienden a reunirse en el Walker y cuando se mueve el Walker este punto cambia a donde el va.
> 

[]()
<img width="833" height="847" alt="image" src="https://github.com/user-attachments/assets/11f5ec66-b1ce-48e8-bb44-5ef362acd5d3" />
<img width="824" height="845" alt="image" src="https://github.com/user-attachments/assets/8907688e-9906-4d5d-a402-aa2fdc6ea2d4" />


<aside>
🔗

link: https://editor.p5js.org/Luden-32/sketches/v7a-kt8UX

</aside>




## Bitácora de reflexión
1. **Diferencia entre `random()`  y `noise()`**

La diferencia fundamental es que `random()`  genera valores completamente independientes entre si, mientras que `noise()` produce valores que están relacionados de forma suave y continua.

Con `random()`  no hay memoria: cada número no tiene relación con el anterior. En cambio, el Ruido de Perlin da una sensación de aleatoriedad, pero en realidad sigue un patron continuo.

Usaría `random()` cuando quiero caos, saltos brusco o comportamientos impredecibles, como partículas dispersándose o dediciones completamente al azar y usaría `noise()` cuando busco algo más orgánico y fluido como deformaciones suaves, movimiento natural o formas que evolucionan con el tiempo.

1. **Distribución de probabilidad**

Una distribución de probabilidad describe qué tan probable es que ocurra un valor dentro de un rango de posibilidades. No todos los valores tienen por qué tener la misma probabilidad de aparecer.

Visualmente, una caminata aleatoria con distribución uniforme se ve más errática y dispersa, ya que todos los movimientos son igual de probables.

En cambio, una caminata con distribución normal tiende a concentrarse alrededor de un punto central, con movimientos pequeños más frecuentes y saltos grandes más raros, lo que genera un movimiento más natural y controlado.

1. **Papel de la aleatoriedad en el arte generativo**

La aleatoriedad cumple varios roles en el arte generativo:

1. Introduce variación, evitando que las obras se vean repetitivas o mecánicas.
2. Permite la emergencia de resultados inesperados, donde el artista no controla cada detalle, sino que diseña un sistema que produce resultados únicos.
3. Funciona como una herramienta expresiva, ya que distintos tipos de aleatoriedad transmiten sensaciones diferentes, como caos, fluidez o equilibrio.
1. **Aleatoriedad en mi obra final (Actividad 07)**

En mi obra final utilicé Ruido Perlin para deformar una forma central y limitar el movimiento de elementos secundarios dentro de esa forma. Esta elección fue adecuada porque buscaba un efecto orgánico y coherente, donde las deformaciones no fueran bruscas ni arbitrarias.

El uso de `noise()` permitió que la forma y el movimiento evolucionaran de manera continua, generando una sensación de unidad visual y evitando el caos que habría producido un uso exclusivo de `random()`.

1. **Caminata (walk) y Lévy flight**

Una caminata en simulación es un proceso en el que un objeto se desplaza paso a paso, tomando decisiones de movimiento basadas en reglas probabilísticas. Cada nuevo paso depende del anterior, lo que genera trayectorias emergentes.

La caminata tipo *Lévy flight* se caracteriza por combinar muchos pasos pequeños con saltos grandes ocasionales. Esto produce trayectorias más exploratorias, donde el agente puede permanecer en una zona por un tiempo y luego desplazarse abruptamente a otra, a diferencia de una caminata aleatoria clásica más uniforme.
