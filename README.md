<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Music Space Invaders</title>

<style>

body{
    margin:0;
    overflow:hidden;
    background:black;
    font-family:Arial, sans-serif;
    text-align:center;
    color:white;
}

h1{
    margin-top:10px;
}

canvas{
    background:#111;
    border:3px solid cyan;
    display:block;
    margin:10px auto;
}

#scoreBox{
    font-size:24px;
    margin-bottom:10px;
}

#restartBtn{
    display:none;
    padding:12px 25px;
    font-size:22px;
    border:none;
    border-radius:10px;
    background:cyan;
    cursor:pointer;
}

#restartBtn:hover{
    background:#00bbbb;
}

</style>
</head>
<body>

<h1>🎵 MUSIC SPACE INVADERS 🎵</h1>

<div id="scoreBox">
    Punteggio: <span id="score">0</span>
</div>

<canvas id="gameCanvas" width="900" height="600"></canvas>

<button id="restartBtn">🔄 Ricomincia</button>

<!-- AUDIO BACKGROUND -->
<audio id="bgMusic" loop>
    <source src="background.wav" type="audio/wav">
</audio>

<script>

const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const scoreEl = document.getElementById("score");
const restartBtn = document.getElementById("restartBtn");

const bgMusic = document.getElementById("bgMusic");

// SUONO SPARO
const shootSound = new Audio(
"https://actions.google.com/sounds/v1/cartoon/pop.ogg"
);

let player;
let bullets;
let enemies;
let enemyDirection;
let score;
let gameOver;
let keys = {};

// AVVIO GIOCO
function initGame(){

    player = {
        x: canvas.width / 2 - 30,
        y: canvas.height - 80,
        width: 60,
        height: 60,
        speed: 8
    };

    bullets = [];
    enemies = [];
    enemyDirection = 1;

    score = 0;
    gameOver = false;

    scoreEl.textContent = score;

    restartBtn.style.display = "none";

    // CREA NEMICI MUSICALI
    for(let row = 0; row < 4; row++){

        for(let col = 0; col < 8; col++){

            const symbols = ["🎼","𝄞","𝄢"];

            enemies.push({
                x: 80 + col * 90,
                y: 60 + row * 70,
                width: 50,
                height: 50,
                alive: true,
                symbol: symbols[Math.floor(Math.random()*symbols.length)]
            });

        }
    }

    // RIPARTE MUSICA
    bgMusic.currentTime = 0;
    bgMusic.play();

    gameLoop();
}

// CONTROLLI
document.addEventListener("keydown",(e)=>{

    keys[e.key] = true;

    // SPARO NOTE MUSICALI
    if(e.key === " " && !gameOver){

        bullets.push({
            x: player.x + 25,
            y: player.y,
            speed: 10,
            symbol:"♪"
        });

        shootSound.currentTime = 0;
        shootSound.play();
    }
});

document.addEventListener("keyup",(e)=>{
    keys[e.key] = false;
});

// MOVIMENTO
function movePlayer(){

    if(keys["ArrowLeft"] && player.x > 0){
        player.x -= player.speed;
    }

    if(keys["ArrowRight"] &&
       player.x + player.width < canvas.width){

        player.x += player.speed;
    }
}

// DISEGNA BAMBINO
function drawPlayer(){

    // TESTA
    ctx.fillStyle = "#FFD39B";
    ctx.beginPath();
    ctx.arc(player.x + 30, player.y + 15, 15, 0, Math.PI*2);
    ctx.fill();

    // CORPO
    ctx.fillStyle = "#00AAFF";
    ctx.fillRect(player.x + 18, player.y + 30, 24, 30);

    // BRACCIA
    ctx.fillRect(player.x + 5, player.y + 32, 15, 5);
    ctx.fillRect(player.x + 40, player.y + 32, 15, 5);

    // GAMBE
    ctx.fillRect(player.x + 18, player.y + 60, 7, 12);
    ctx.fillRect(player.x + 35, player.y + 60, 7, 12);
}

// DISEGNA COLPI A FORMA DI NOTE
function drawBullets(){

    ctx.fillStyle = "yellow";
    ctx.font = "28px Arial";

    bullets.forEach((bullet,index)=>{

        bullet.y -= bullet.speed;

        ctx.fillText(
            bullet.symbol,
            bullet.x,
            bullet.y
        );

        if(bullet.y < 0){
            bullets.splice(index,1);
        }
    });
}

// DISEGNA NEMICI MUSICALI
function drawEnemies(){

    let moveDown = false;

    ctx.font = "40px Arial";

    enemies.forEach(enemy=>{

        if(!enemy.alive) return;

        enemy.x += enemyDirection;

        if(enemy.x + enemy.width > canvas.width ||
           enemy.x < 0){

            moveDown = true;
        }

        ctx.fillStyle = "red";

        ctx.fillText(
            enemy.symbol,
            enemy.x,
            enemy.y
        );
    });

    if(moveDown){

        enemyDirection *= -1;

        enemies.forEach(enemy=>{
            enemy.y += 20;
        });
    }
}

// COLLISIONI
function checkCollisions(){

    bullets.forEach((bullet,bulletIndex)=>{

        enemies.forEach(enemy=>{

            if(
                enemy.alive &&
                bullet.x < enemy.x + enemy.width &&
                bullet.x + 20 > enemy.x &&
                bullet.y < enemy.y + enemy.height &&
                bullet.y + 20 > enemy.y
            ){

                enemy.alive = false;

                bullets.splice(bulletIndex,1);

                score += 10;

                scoreEl.textContent = score;
            }

        });

    });

}

// GAME OVER
function checkGameOver(){

    enemies.forEach(enemy=>{

        if(enemy.alive &&
           enemy.y + enemy.height >= player.y){

            gameOver = true;
        }

    });

}

// SCHERMATA GAME OVER
function drawGameOver(){

    ctx.fillStyle = "white";
    ctx.font = "60px Arial";

    ctx.fillText(
        "GAME OVER",
        250,
        250
    );

    ctx.font = "28px Arial";

    ctx.fillText(
        "Premi il pulsante per ricominciare",
        210,
        320
    );

    restartBtn.style.display = "inline-block";

    // STOP MUSICA
    bgMusic.pause();
}

// LOOP PRINCIPALE
function gameLoop(){

    ctx.clearRect(0,0,canvas.width,canvas.height);

    if(gameOver){

        drawGameOver();
        return;
    }

    movePlayer();

    drawPlayer();

    drawBullets();

    drawEnemies();

    checkCollisions();

    checkGameOver();

    requestAnimationFrame(gameLoop);
}

// RICOMINCIA
restartBtn.addEventListener("click",()=>{

    initGame();

});

// AVVIO
initGame();

</script>

</body>
</html>
