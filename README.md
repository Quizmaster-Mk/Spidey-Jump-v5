<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Spider-Man Runner Comic Immersive</title>
<style>
  body {
    margin: 0;
    background: #f5f5dc;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    overflow: hidden;
  }
  canvas {
    border: 4px solid #000;
    background: linear-gradient(to top, #87ceeb, #ffffff);
  }
</style>
</head>
<body>
<canvas id="gameCanvas" width="600" height="200"></canvas>
<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Images Spider-Man et vilains (exemples)
const spiderImg = new Image();
spiderImg.src = "https://i.imgur.com/OdL0XPt.png"; // Remplacer par Spider-Man
const villainImgs = [
  "https://i.imgur.com/6XgQXQ2.png", 
  "https://i.imgur.com/6XgQXQ2.png", 
  "https://i.imgur.com/6XgQXQ2.png"  
].map(src => { let img = new Image(); img.src = src; return img; });

// Joueur
const player = { x: 50, y: 150, width: 40, height: 40, dy: 0, jumping: false, swing: 0 };
const gravity = 0.7;
const jumpPower = -14;

let obstacles = [];
let frame = 0;
let score = 0;
let gameOver = false;
let speed = 3;

// Créer un obstacle avec image aléatoire
function createObstacle() {
  const height = 40;
  const img = villainImgs[Math.floor(Math.random() * villainImgs.length)];
  obstacles.push({ x: canvas.width, y: 200 - height, width: 40, height: height, img });
}

// Collision
function collision(p, o) {
  return p.x < o.x + o.width && p.x + p.width > o.x &&
         p.y < o.y + o.height && p.y + p.height > o.y;
}

// Saut
function jump() {
  if(!player.jumping){
    player.dy = jumpPower;
    player.jumping = true;
    player.swing = -20; // balancement vers la gauche
    spawnStars();
    playSound("jump");
  }
}

// Étoiles du saut
let stars = [];
function spawnStars() {
  for(let i=0;i<5;i++){
    stars.push({x:player.x+player.width/2,y:player.y+player.height/2,dy:Math.random()*-2-1,life:30});
  }
}

// Dessiner étoiles
function drawStars(){
  ctx.fillStyle="yellow";
  for(let i=stars.length-1;i>=0;i--){
    const s=stars[i];
    ctx.fillRect(s.x,s.y,3,3);
    s.y += s.dy;
    s.life--;
    if(s.life<=0) stars.splice(i,1);
  }
}

// Onomatopée
function drawComicText(text,x,y,color="red"){
  ctx.font="30px Comic Sans MS";
  ctx.fillStyle=color;
  ctx.strokeStyle="black";
  ctx.lineWidth=2;
  ctx.strokeText(text,x,y);
  ctx.fillText(text,x,y);
}

// Sons simples
function playSound(type){
  if(type==="jump"){
    const audio = new Audio("https://www.soundjay.com/button/beep-07.mp3");
    audio.play();
  } else if(type==="hit"){
    const audio = new Audio("https://www.soundjay.com/button/beep-10.mp3");
    audio.play();
  }
}

// Événements
document.addEventListener("keydown", e=>{ if(e.code==="Space"||e.key===" ") jump(); });
canvas.addEventListener("click", jump);
canvas.addEventListener("touchstart", jump);

// Boucle de jeu
function update(){
  if(gameOver) return;

  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Gravité
  player.dy += gravity;
  player.y += player.dy;
  if(player.y + player.height >= 200){
    player.y = 200 - player.height;
    player.dy = 0;
    player.jumping = false;
  }

  // Balancement Spider-Man
  if(player.swing < 20) player.swing += 1;

  // Dessiner Spider-Man avec léger balancement
  ctx.save();
  ctx.translate(player.x + player.width/2, player.y + player.height/2);
  ctx.rotate(player.swing * Math.PI/180);
  ctx.drawImage(spiderImg, -player.width/2, -player.height/2, player.width, player.height);
  ctx.restore();

  // Créer obstacles
  if(frame % 100 === 0) createObstacle();

  // Score et vitesse progressive
  score = Math.floor(frame/10);
  speed = 3 + Math.floor(score/50);

  // Déplacer et dessiner obstacles
  for(let i=0;i<obstacles.length;i++){
    const o = obstacles[i];
    o.x -= speed;
    ctx.drawImage(o.img, o.x, o.y, o.width, o.height);

    if(collision(player,o)){
      gameOver = true;
      drawComicText("BAM!", player.x, player.y - 10);
      playSound("hit");
    }
  }

  drawStars();

  // Afficher score
  ctx.fillStyle="black";
  ctx.font="20px Comic Sans MS";
  ctx.fillText("Score: "+score,10,30);

  frame++;
  if(!gameOver) requestAnimationFrame(update);
  else drawComicText("POW! GAME OVER",150,100);
}

spiderImg.onload = () => update();
</script>
</body>
</html>
