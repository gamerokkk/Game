<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Time Rewind Mobile Game</title>
<style>
  body { margin:0; background:#222; color:#fff; font-family:sans-serif; overflow:hidden; }
  canvas { display:block; margin:0 auto; background: linear-gradient(#444,#111); touch-action:none; }
  #scoreboard { position:absolute; top:10px; left:10px; font-size:24px; }
  #rewind { position:absolute; top:50px; left:10px; font-size:18px; padding:6px 12px; background:#fff; color:#000; border-radius:6px; cursor:pointer; user-select:none; }
</style>
</head>
<body>

<div id="scoreboard">Очки: <span id="score">0</span></div>
<div id="rewind">⏪ Время назад</div>
<canvas id="canvas"></canvas>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const rewindBtn = document.getElementById('rewind');

// Подстройка под экран
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// Игровые объекты
let score = 0;
let ball = {x:canvas.width/2, y:canvas.height/2, r:30, color:'#0f0'};
let stars = [];
let history = [];

// Управление пальцем
let touchDir = {x:0, y:0};

// Генерация падающих звезд
function addStar() {
    stars.push({x: Math.random()*(canvas.width-20)+10, y:-20, r:15, color:'#ff0'});
}

// Движение шарика
function moveBall() {
    ball.x += touchDir.x*6;
    ball.y += touchDir.y*6;

    // Ограничение по экрану
    if(ball.x < ball.r) ball.x = ball.r;
    if(ball.x > canvas.width - ball.r) ball.x = canvas.width - ball.r;
    if(ball.y < ball.r) ball.y = ball.r;
    if(ball.y > canvas.height - ball.r) ball.y = canvas.height - ball.r;
}

// Проверка столкновений
function checkCollisions() {
    for(let i=stars.length-1;i>=0;i--){
        const s = stars[i];
        const dx = s.x - ball.x;
        const dy = s.y - ball.y;
        const dist = Math.sqrt(dx*dx + dy*dy);
        if(dist < s.r + ball.r){
            score++;
            scoreEl.textContent = score;
            stars.splice(i,1);
        }
    }
}

// Сохраняем историю для перемотки
function saveHistory() {
    history.push({x: ball.x, y: ball.y, stars: JSON.parse(JSON.stringify(stars))});
    if(history.length > 180) history.shift();
}

// Перемотка времени
rewindBtn.addEventListener('click', ()=>{
    if(history.length>0){
        const state = history.shift();
        ball.x = state.x;
        ball.y = state.y;
        stars = state.stars;
    }
});

// Сенсорное управление
canvas.addEventListener('touchstart', handleTouch);
canvas.addEventListener('touchmove', handleTouch);
canvas.addEventListener('touchend', ()=>{ touchDir={x:0,y:0}; });

function handleTouch(e){
    const touch = e.touches[0];
    touchDir.x = touch.clientX < ball.x ? -1 : 1;
    touchDir.y = touch.clientY < ball.y ? -1 : 1;
}

// Основной цикл
function gameLoop(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    moveBall();
    checkCollisions();
    saveHistory();

    // Рисуем шарик
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.r,0,Math.PI*2);
    ctx.fillStyle = ball.color;
    ctx.fill();

    // Рисуем звезды
    stars.forEach(s=>{
        s.y += 4;
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.r,0,Math.PI*2);
        ctx.fillStyle = s.color;
        ctx.fill();
    });

    // Генерация звезд
    if(Math.random()<0.03) addStar();

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
