<!DOCTYPE html>
<html lang="es">
<head>
    <link href="https://fonts.googleapis.com/css2?family=Cooper+Hewitt:wght@400;700&display=swap" rel="stylesheet">

    <meta charset="utf8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kit de herramientas</title>
    <link rel="stylesheet" href="styles.css">
    <style>
        body {
            background: #333;
            color: #ddd;
            font-family: Arial, sans-serif;
            display: flex; /* Para acomodar botones a la izquierda */
        }
        #left-panel { 
            width: 200px; 
            padding: 20px; 
        }
        #buttonContainer { 
            display: flex; 
            flex-direction: column; /* Botones uno debajo del otro */ 
        }
        #puzzleContainer {
            position: relative;
            margin: 20px; 
            width: 800px; /* 200% más grande */
            height: 600px; /* 200% más grande */
            border: 1px solid #ddd;
            background: #111;
        }
        .puzzle-piece {
            position: absolute;
            box-sizing: border-box;
            border: 1px solid #ddd;
            background-color: white;
            cursor: pointer;
        }
        .puzzle-piece span { 
            display: none; 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white; 
            font-weight: bold;
            text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black; /* Sombra de texto */
            padding: 5px 10px; 
            border-radius: 5px;
            background-color: rgba(0, 0, 0, 0.5); /* Fondo semi-transparente */
        }
        .show span {
            display: block;
        }
        #controls {
            text-align: center;
            margin-top: 20px;
        }
        .button {
            margin: 5px 0; /* Margen superior e inferior */ 
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="left-panel"> 
        <div id="floatingTitle">¿Qué hacer después de la crisis? Kit de herramientas</div>
        <div id="buttonContainer"></div>
    </div>
    <div id="game-area"> 
        <div id="puzzleContainer"></div>
        <div id="controls">
            <button id="resetButton" class="button" style="background-color:rgb(107, 17, 7);">Empezar de nuevo</button>
            <div>Movimientos: <span id="moves">0</span></div>
            <div>Progreso: <span id="progress">0</span></div> 
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.5/gsap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/howler/2.2.3/howler.min.js"></script>
    <script src="puzzle.js"></script> 
</body>
</html>


----------------------

body {
    background: #333;
    color: #ddd;
    font-family: 'Cooper Hewitt', sans-serif;
    padding-top: 100px; }

#puzzleContainer {
    position: relative;
    margin: 150px auto 20px; /* Ajustar el margen superior y mantener el inferior */
    width: 400px;
    height: 300px;
    border: 1px solid #ddd;
    background: #111;
    transform: translateY(100px);
}

.puzzle-piece {
    position: absolute;
    box-sizing: border-box;
    border: 1px solid #ddd;
    background-color: white;
    cursor: pointer;
}

.puzzle-piece span {
    display: none;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: black;
    font-weight: bold;
}

.show span {
    display: block;
}

#controls {
    text-align: center;
    margin-top: 20px;
}

.button {
    margin: 5px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
}
.fullscreen {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    object-fit: cover; /* Asegura que la imagen cubra todo el contenedor */
    z-index: 10;
}

#finalMessage {
    position: fixed;
    bottom: 80px; /* Ajusta la posición del mensaje */
    left: 50%;
    transform: translate(-50%, 0);
    font-size: 3em;
    color: white;
    text-shadow: 2px 2px 4px #000000;
    opacity: 0;
    z-index: 11;
}
#floatingTitle {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 3em;
    color: white;
    text-shadow: 2px 2px 4px #000000;
    z-index: 12;
    animation: float 3s ease-in-out infinite;
}

@keyframes float {
    0%, 100% {
        transform: translateX(-50%) translateY(0);
    }
    50% {
        transform: translateX(-50%) translateY(-10px);
    }
}




-----------------------------



const PUZZLE_DIFFICULTY = 12; 
const PIECE_SIZE = 200; // 200% más grande 
const TAB_RATIO = 0.2; 
const pieceOrder = [
    { index: 0, message: "Terapia Psicológica" },
    { index: 10, message: "Gestión Emocional" },
    { index: 5, message: "Diario Emocional" },
    { index: 6, message: "Gratitud" },
    { index: 7, message: "Meditación" },
    { index: 4, message: "Tres Bendiciones" },
    { index: 1, message: "Oración" },
    { index: 9, message: "Voluntariado" },
    { index: 2, message: "Arteterapia" },
    { index: 11, message: "Perdón" },
    { index: 3, message: "Lidérate" },
    { index: 8, message: "Planeación Estratégica Personal" }
];

let img;
let pieces = [];
let puzzleContainer;
let buttonContainer;
let resetButton;
let progressElement;
let movesElement;
let progress = 0;
let moves = 0;
let isPuzzleSolved = false; 

// --- Sonido --- 
const sound = {
    correct: new Howl({ src: ['audio/correct.wav'] }),
    move: new Howl({ src: ['audio/move.wav'] }),
    win: new Howl({ src: ['audio/win.wav'] }) 
};

const messages = [
    "Terapia Psicológica", "Oración", "Arteterapia", "Lidérate",
    "Tres Bendiciones", "Diario Emocional", "Gratitud", "Meditación",
    "Planeación Estratégica Personal", "Voluntariado", "Gestión Emocional", "Perdón"
];

function init() {
    img = new Image();
    img.addEventListener('load', onImageLoad, false);
    img.src = "images/your_image.jpg"; // Reemplaza con la ruta de tu imagen 
}

function onImageLoad() {
    puzzleContainer = document.getElementById('puzzleContainer');
    buttonContainer = document.getElementById('buttonContainer');
    resetButton = document.getElementById('resetButton');
    progressElement = document.getElementById('progress');
    movesElement = document.getElementById('moves');
    generatePuzzle();
    createButtons();
    shufflePieces();
    resetButton.addEventListener('click', resetPuzzle);
}

function generatePuzzle() {
    const rows = 3;
    const cols = 4;
    pieces = [];

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let tabs = [0, 0, 0, 0];

            if (c < cols - 1) tabs[1] = sample([1, -1]);
            if (r < rows - 1) tabs[2] = sample([1, -1]);
            if (r > 0) tabs[0] = -pieces[(r - 1) * cols + c].tabs[2];
            if (c > 0) tabs[3] = -pieces[r * cols + (c - 1)].tabs[1]; 

            const piece = createPuzzlePiece(r, c, tabs);
            pieces.push(piece);
            puzzleContainer.appendChild(piece.element); 
        }
    }
}

function createPuzzlePiece(row, col, tabs) {
    const pieceSize = PIECE_SIZE; 
    const tabSize = TAB_RATIO * pieceSize; 

    const piece = document.createElement('div');
    piece.className = 'puzzle-piece';
    piece.style.width = `${pieceSize}px`; 
    piece.style.height = `${pieceSize}px`; 
    piece.style.position = 'absolute';
    piece.style.backgroundImage = `url(${img.src})`;
    piece.style.backgroundSize = `${pieceSize * 4}px ${pieceSize * 3}px`; 
    piece.style.backgroundPosition = `-${col * pieceSize}px -${row * pieceSize}px`;

    piece.dataset.index = row * 4 + col;
    piece.dataset.row = row;
    piece.dataset.col = col; 

    const span = document.createElement('span');
    span.textContent = messages[row * 4 + col];
    piece.appendChild(span); 

    piece.style.clipPath = generateClipPath(pieceSize, tabs);

    return { element: piece, tabs: tabs }; 
}

function generateClipPath(size, tabs) {
    const tabSize = TAB_RATIO * size; 
    let path = ''; 

    const directions = [
        [1, 0], [0, 1], [-1, 0], [0, -1]
    ];

    for (let i = 0; i < 4; i++) {
        const [dx, dy] = directions[i];
        const startX = i % 2 === 0 ? (dx === 1 ? 100 : 0) : (dy === 1 ? 100 : 0);
        const startY = i % 2 === 0 ? (dy === 1 ? 100 : 0) : (dx === 1 ? 100 : 0); 

        path += `${startX}% ${startY}% `; 

        if (tabs[i] !== 0) {
            const midX = startX + (dx * 50) + (tabs[i] * dx * 10); 
            const midY = startY + (dy * 50) + (tabs[i] * dy * 10); 
            path += `, ${midX}% ${midY}% `; 
            path += `${startX + (dx * 100)}% ${startY + (dy * 100)}% `; 
        } else {
            path += `${startX + (dx * 100)}% ${startY + (dy * 100)}% `; 
        }
    }

    return `polygon(${path})`;
}

function createButtons() {
    buttonContainer.innerHTML = ''; // Limpiar el contenedor de botones
    for (let i = 0; i < pieceOrder.length; i++) {
        const button = document.createElement('button');
        button.className = 'button';

        // Cambiar el texto del botón aquí:
        button.textContent = `HERRAMIENTA ${i + 1}`; // Mostrar "HERRAMIENTA 1", "HERRAMIENTA 2", etc.

        const pieceIndex = pieceOrder[i].index;
        button.addEventListener('click', () => movePiece(pieceIndex)); 
        buttonContainer.appendChild(button);
    }
}

function shufflePieces() {
    pieces.forEach(piece => {
        const puzzleContainerRect = puzzleContainer.getBoundingClientRect();
        const pieceWidth = piece.element.offsetWidth;
        const pieceHeight = piece.element.offsetHeight;

        let x, y;

        // Generar una posición aleatoria fuera del contenedor del rompecabezas
        do {
            x = Math.random() * window.innerWidth;
            y = Math.random() * window.innerHeight;
        } while (
            // Asegurarse de que las piezas no estén dentro del contenedor del rompecabezas
            (x > puzzleContainerRect.left - pieceWidth && x < puzzleContainerRect.right) &&
            (y > puzzleContainerRect.top - pieceHeight && y < puzzleContainerRect.bottom) ||
            // Asegurarse de que las piezas no se superpongan con los botones en el lado izquierdo
            (x < 200) // Ancho del panel de botones
        );

        piece.element.style.transform = `translate(${x}px, ${y}px)`;
        piece.element.classList.remove('show');
        piece.element.firstChild.style.display = 'none';
    });

    isPuzzleSolved = false;
}
function resetPuzzle() {
    isPuzzleSolved = false; // Resetear la variable de estado

    // 1. Devolver las piezas a su posición original (esto sucede instantáneamente)
    pieces.forEach((piece, index) => {
        const col = index % 4;
        const row = Math.floor(index / 4); 
        gsap.to(piece.element, { 
            duration: 0, // Sin duración, la pieza se mueve a la posición inmediatamente
            x: col * PIECE_SIZE, 
            y: row * PIECE_SIZE 
        });
    });

    // 2. Aplicar el efecto de giro de reinicio
    gsap.fromTo(puzzleContainer, { rotation: 360 }, { 
        rotation: 0, 
        duration: 1, // 1 segundo de duración para el giro
        ease: "power2.out", 
        onComplete: () => {  // 3. Después de girar, barajar las piezas
            shufflePieces(); 
            moves = 0;
            progress = 0;
            movesElement.textContent = moves;
            progressElement.textContent = progress;
        } 
    });
}

function movePiece(index) { 
    if (isPuzzleSolved) return;

    const piece = pieces[index].element; 
    const cols = 4; 
    const row = Math.floor(index / cols); 
    const col = index % cols;

    gsap.to(piece, {
        duration: 0.5,
        ease: "power2.out",
        x: col * PIECE_SIZE, 
        y: row * PIECE_SIZE, 
        onComplete: () => {
            piece.classList.add('show');
            piece.firstChild.style.display = 'block'; 
            checkWin(); 
        }
    });

    sound.move.play(); 

    moves++; 
    movesElement.textContent = moves;
}

function checkWin() {
    let win = true;
    pieces.forEach((piece, index) => {
        const cols = 4; 
        const row = Math.floor(index / cols); 
        const col = index % cols; 
        const transform = piece.element.style.transform.match(/translate\((.*?)px, (.*?)px\)/);
        if (transform) {
            const [_, x, y] = transform; 
            if (parseInt(x) !== col * PIECE_SIZE || parseInt(y) !== row * PIECE_SIZE) { 
                win = false;
            }
        }
    }); 
    if (win) {
        sound.win.play(); 
        document.getElementById('floatingTitle').style.display = 'none';

        gsap.to(puzzleContainer, { 
            duration: 1, 
            rotation: 360, 
            ease: "power2.inOut",
            onComplete: () => {
                isPuzzleSolved = true;
                gsap.to(puzzleContainer, {
                    duration: 2,
                    scale: 2,
                    ease: "power2.inOut",
                    onComplete: () => {
                        pieces.forEach(piece => {
                            piece.element.style.border = 'none';
                            piece.element.firstChild.style.display = 'none';
                        });
                        showFinalMessage();
                    }
                });
                document.getElementById('buttonContainer').style.display = 'none'; // Ocultar botones de herramienta
            }
        });
    }
}

function showFinalMessage() {
    const finalMessage = document.createElement('div');
    finalMessage.textContent = "si puedes regresar a ti!";
    finalMessage.style.position = 'absolute';
    finalMessage.style.top = '80%';
    finalMessage.style.left = '50%';
    finalMessage.style.transform = 'translate(-50%, -50%)';
    finalMessage.style.fontSize = '3em';
    finalMessage.style.color = 'white';
    finalMessage.style.textShadow = '2px 2px 4px #000000';
    finalMessage.style.opacity = 0;
    document.body.appendChild(finalMessage);
    
    gsap.to(finalMessage, {
        duration: 2,
        opacity: 1,
        scale: 1.5,
        ease: "elastic.out"
    });
}


function sample(values) {
    return values[Math.floor(Math.random() * values.length)]; 
}





init();