# Unidad 6
## Actividad
#### link: https://editor.p5js.org/Luden-32/full/-NUu18Xnw
```jsx
let musica;
let fft;
let particulasPiano = [];
let particulasViolin = [];
let imgViolin;
let cargada = false;

// Variables de control de transiciones
let divinidad = 0; 
let nivelImagen = 0; 
let stepViolin = 9; 
let velocidadFade = 1.0625; 

// Colores para las transiciones
let colorPianoInicial, colorPianoFinal;
let colorFondoInicial, colorFondoFinal;

function preload() {
  musica = loadSound('De Blanca Mantilla - Carlos Viola.mp3', 
    () => { cargada = true; }, 
    (err) => { console.error("Error de audio"); }
  );
  
  imgViolin = loadImage('violin.png', 
    () => { setupViolinParticles(); },
    (err) => { console.error("Error de imagen"); }
  );
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  fft = new p5.FFT(0.85, 1024);
  frameRate(60);
  noSmooth(); 

  // --- PALETA DE COLORES ---
  colorPianoInicial = color(220, 20, 20);      // Rojo Intenso
  colorPianoFinal = color(255, 245, 180);      // Dorado Pastel (suave y luminoso)
  
  colorFondoInicial = color(0, 0, 0);          // Negro Absoluto
  colorFondoFinal = color(60, 100, 160);       // Azul Claro
}

function draw() {
  let tiempo = musica.currentTime();

  // --- 1. LÓGICA DE TIEMPOS ---
  let debeMostrar = (tiempo >= 66 && tiempo <= 99) ||      
                     (tiempo >= 137 && tiempo <= 202) ||    
                     (tiempo >= 210 && tiempo <= 366);      

  if (debeMostrar) {
    nivelImagen = min(nivelImagen + velocidadFade, 255); 
  } else {
    nivelImagen = max(nivelImagen - velocidadFade, 0); 
  }

  // --- 2. FONDO DINÁMICO (Negro a Azul) ---
  let factorTransicion = nivelImagen / 255; 
  
  let currentBgR = lerp(red(colorFondoInicial), red(colorFondoFinal), factorTransicion);
  let currentBgG = lerp(green(colorFondoInicial), green(colorFondoFinal), factorTransicion);
  let currentBgB = lerp(blue(colorFondoInicial), blue(colorFondoFinal), factorTransicion);
  
  background(currentBgR, currentBgG, currentBgB, 60);

  if (!cargada) return;

  // --- 3. ANÁLISIS DE AUDIO ---
  fft.analyze();
  let energiaMedios = fft.getEnergy("mid"); 
  let pulsoViolin = map(energiaMedios, 0, 255, 1, 1.8);
  let notasAltas = fft.getEnergy("treble");

  if (notasAltas > 135) { 
    divinidad = 255; 
  } else {
    divinidad = max(divinidad - 4, 0); 
  }

  // --- 4. DIBUJAR VIOLÍN ---
  if (nivelImagen > 0) {
    push();
    translate(width / 2, height / 2);
    for (let p of particulasViolin) {
      p.show(nivelImagen, pulsoViolin); 
    }
    pop();
  }

  // --- 5. PARTÍCULAS DEL PIANO ---
  let spectrum = fft.analyze();
  for (let i = 0; i < 80; i++) {
    let vol = spectrum[i];
    if (vol > 110) { 
      let x = map(i, 0, 80, 0, width);
      if (random(1) > 0.85) {
        particulasPiano.push(new ParticulaPiano(x, vol));
      }
    }
  }

  for (let i = particulasPiano.length - 1; i >= 0; i--) {
    // Actualizamos el color en cada frame para que la transición afecte a las que ya están en pantalla
    particulasPiano[i].updateColor(nivelImagen);
    particulasPiano[i].update();
    particulasPiano[i].show();
    if (particulasPiano[i].alpha < 0) particulasPiano.splice(i, 1);
  }
}

function setupViolinParticles() {
  particulasViolin = [];
  let alto = min(height * 0.7, 500);
  imgViolin.resize(0, alto);
  imgViolin.loadPixels();
  
  let halfW = imgViolin.width / 2;
  let halfH = imgViolin.height / 2;

  for (let y = 0; y < imgViolin.height; y += stepViolin) {
    for (let x = 0; x < imgViolin.width; x += stepViolin) {
      let index = (x + y * imgViolin.width) * 4;
      if (imgViolin.pixels[index + 3] > 127) {
        particulasViolin.push(new ParticulaViolin(x - halfW, y - halfH));
      }
    }
  }
}

function keyPressed() {
  if (key === 'f' || key === 'F') {
    let fs = fullscreen();
    fullscreen(!fs);
  }
}

class ParticulaViolin {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.baseSize = random(1.5, 3);
  }
  show(op, pulso) {
    noStroke();
    fill(255, 255, 230, op);
    ellipse(this.x, this.y, this.baseSize * pulso);
  }
}

class ParticulaPiano {
  constructor(x, amp) {
    this.x = x + random(-10, 10);
    this.y = height;
    this.vy = map(amp, 110, 255, -2, -9);
    this.alpha = 255;
    this.tam = random(5, 10);
    this.currentColor = color(220, 20, 20); 
  }

  updateColor(opGlobalImagen) {
    let factorTransicion = opGlobalImagen / 255; 
    let r = lerp(red(colorPianoInicial), red(colorPianoFinal), factorTransicion);
    let g = lerp(green(colorPianoInicial), green(colorPianoFinal), factorTransicion);
    let b = lerp(blue(colorPianoInicial), blue(colorPianoFinal), factorTransicion);
    this.currentColor = color(r, g, b, this.alpha);
  }

  update() {
    this.y += this.vy;
    this.alpha -= 3;
  }

  show() {
    noStroke();
    if (divinidad > 150) {
      fill(red(this.currentColor), green(this.currentColor), blue(this.currentColor), this.alpha / 4);
      ellipse(this.x, this.y, this.tam * 2.5);
    }
    fill(this.currentColor);
    ellipse(this.x, this.y, this.tam);
  }
}

function mousePressed() {
  if (cargada) {
    userStartAudio();
    if (musica.isPlaying()) musica.pause();
    else musica.play();
  }
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
  if (imgViolin) setupViolinParticles();
}
```

```jsx
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Visualizador - De Blanca Mantilla</title>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/addons/p5.sound.min.js"></script>
    
    <style>
        /* Reset de estilos para asegurar que el canvas sea el protagonista */
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden; /* Evita que aparezcan barras de scroll */
            background-color: #050a19; /* Fondo azul noche profundo */
        }

        canvas {
            display: block; /* Elimina espacios vacíos residuales */
        }

        /* Estilo para el mensaje de carga por defecto de p5.js */
        #p5_loading {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #ffffff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            font-size: 1.2rem;
            letter-spacing: 2px;
            background-color: #050a19;
            z-index: 10;
        }
    </style>
</head>
<body>
    <script src="sketch.js"></script>
</body>
</html>
```

### Ref
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/0c2be2cd-55ec-4ad5-a30f-cbf0f0431a22" />
<img width="825" height="464" alt="image" src="https://github.com/user-attachments/assets/076cdef7-6101-47b3-8019-46f127d43350" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9bb1224e-e693-4cd3-ae98-d3c413af809b" />

### Concepto
Quise representar una canción que me gusta mucho del OST de Blasphemous 2, De Blanca Mantilla, asociada a uno de los jefes que más me costó: Benedicta. Tanto la música como el juego tienen una carga muy marcada de lo divino y lo religioso, y eso fue justamente lo que intenté trasladar al visual, usando luces, atmósferas y transiciones que evocaran esa sensación.

El violín juega un papel clave en la composición, así que decidí darle protagonismo dentro de la pieza, usándolo como elemento central para reforzar ese tono solemne y casi sagrado que tiene la canción.

Para el piano, me inspiré en una escena de JoJo's Bizarre Adventure, específicamente cuando Enrico Pucci realiza una especie de concierto. Esa referencia me ayudó a pensar el piano no solo como acompañamiento, sino como algo más expresivo y visual dentro de la composición.


