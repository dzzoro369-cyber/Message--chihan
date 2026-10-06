<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🏎️ سباق السيارات</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #111;
  font-family: Arial, sans-serif;
  overflow: hidden;
  touch-action: none;
}

#game {
  position: relative;
  width: 100%;
  max-width: 430px;
  height: 100vh;
  margin: auto;
  overflow: hidden;
  background: #333;
}

/* الطريق */
#road {
  position: absolute;
  inset: 0;
  background: #333;
}

#road::before,
#road::after {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  width: 8px;
  background: white;
}

#road::before {
  left: 7%;
}

#road::after {
  right: 7%;
}

/* خطوط الطريق */
.line {
  position: absolute;
  width: 8px;
  height: 100px;
  background: white;
  top: -100px;
  animation: roadMove 0.6s linear infinite;
}

.line1 {
  left: 34%;
}

.line2 {
  left: 66%;
}

@keyframes roadMove {
  from {
    transform: translateY(0);
  }

  to {
    transform: translateY(900px);
  }
}

/* المعلومات */
#hud {
  position: absolute;
  top: 15px;
  left: 15px;
  right: 15px;
  z-index: 10;

  display: flex;
  justify-content: space-between;

  color: white;
  font-size: 18px;
  font-weight: bold;
}

.info {
  background: rgba(0,0,0,0.7);
  padding: 10px 14px;
  border-radius: 12px;
}

/* سيارة اللاعب */
#player {
  position: absolute;

  width: 58px;
  height: 100px;

  bottom: 40px;
  left: calc(50% - 29px);

  background: linear-gradient(
    90deg,
    #990000,
    red,
    #990000
  );

  border: 3px solid #ff8888;
  border-radius: 12px;

  z-index: 5;
}

/* زجاج السيارة */
#player::before,
.enemy::before {
  content: "";

  position: absolute;

  top: 15px;
  left: 10px;
  right: 10px;

  height: 28px;

  background: #9ee7ff;

  border-radius: 7px;
  border: 2px solid #222;
}

/* السيارات المنافسة */
.enemy {
  position: absolute;

  width: 58px;
  height: 100px;

  top: -120px;

  background: linear-gradient(
    90deg,
    #003b99,
    #1683ff,
    #003b99
  );

  border: 3px solid #8dc4ff;

  border-radius: 12px;

  z-index: 4;
}

.enemy.yellow {
  background: linear-gradient(
    90deg,
    #b87500,
    #ffd21a,
    #b87500
  );
}

/* شاشة البداية والنهاية */
.screen {
  position: absolute;

  inset: 0;

  background: rgba(0,0,0,0.85);

  z-index: 20;

  display: flex;

  flex-direction: column;

  justify-content: center;

  align-items: center;

  text-align: center;

  color: white;
}

.screen h1 {
  font-size: 40px;
}

.screen p {
  font-size: 18px;
}

button {
  border: none;

  background: red;

  color: white;

  font-size: 20px;

  font-weight: bold;

  padding: 15px 30px;

  border-radius: 15px;

  cursor: pointer;
}

/* أزرار الهاتف */
.controls {
  position: absolute;

  bottom: 15px;

  left: 0;
  right: 0;

  z-index: 15;

  display: flex;

  justify-content: space-between;

  padding: 0 20px;
}

.control {
  width: 75px;
  height: 75px;

  border-radius: 50%;

  background: rgba(255,255,255,0.25);

  border: 2px solid white;

  color: white;

  display: flex;

  align-items: center;
  justify-content: center;

  font-size: 32px;

  user-select: none;
}
</style>
</head>

<body>

<div id="game">

  <!-- الطريق -->
  <div id="road">

    <div class="line line1"></div>
    <div class="line line2"></div>

  </div>

  <!-- النقاط -->
  <div id="hud">

    <div class="info">
      🏆 النقاط:
      <span id="score">0</span>
    </div>

    <div class="info">
      ⚡ السرعة:
      <span id="speed">1</span>
    </div>

  </div>

  <!-- السيارة -->
  <div id="player"></div>

  <!-- أزرار التحكم -->
  <div class="controls">

    <div class="control" id="left">
      ⬅️
    </div>

    <div class="control" id="right">
      ➡️
    </div>

  </div>

  <!-- البداية -->
  <div class="screen" id="startScreen">

    <h1>🏎️ سباق السيارات</h1>

    <p>
      تفادى السيارات واجمع أكبر عدد من النقاط!
    </p>

    <p>
      استعمل الأسهم أو أزرار الشاشة
    </p>

    <button onclick="startGame()">
      ابدأ اللعبة
    </button>

  </div>

  <!-- النهاية -->
  <div class="screen" id="gameOver"
       style="display:none">

    <h1>💥 انتهت اللعبة</h1>

    <p>
      نتيجتك:
      <span id="finalScore">0</span>
    </p>

    <button onclick="startGame()">
      🔄 العب من جديد
    </button>

  </div>

</div>


<script>

const game = document.getElementById("game");

const player = document.getElementById("player");

const scoreText = document.getElementById("score");

const speedText = document.getElementById("speed");

const startScreen =
  document.getElementById("startScreen");

const gameOver =
  document.getElementById("gameOver");

const finalScore =
  document.getElementById("finalScore");


let playerX;

let enemies = [];

let score = 0;

let speed = 1;

let running = false;

let lastTime = 0;

let spawnTimer = 0;


let keys = {
  left: false,
  right: false
};


/* بداية اللعبة */

function startGame() {

  enemies.forEach(enemy => {
    enemy.element.remove();
  });

  enemies = [];

  score = 0;

  speed = 1;

  spawnTimer = 0;

  running = true;

  scoreText.textContent = 0;

  speedText.textContent = 1;

  startScreen.style.display = "none";

  gameOver.style.display = "none";

  playerX =
    (game.clientWidth - player.offsetWidth) / 2;

  player.style.left =
    playerX + "px";

  lastTime = performance.now();

  requestAnimationFrame(gameLoop);
}


/* نهاية اللعبة */

function endGame() {

  running = false;

  finalScore.textContent =
    Math.floor(score);

  gameOver.style.display = "flex";
}


/* إنشاء سيارة منافسة */

function createEnemy() {

  const enemy =
    document.createElement("div");

  enemy.className = "enemy";

  if (Math.random() > 0.5) {
    enemy.classList.add("yellow");
  }

  const lanes = [
    0.19,
    0.50,
    0.81
  ];

  const lane =
    lanes[Math.floor(Math.random() * lanes.length)];

  const x =
    game.clientWidth * lane -
    enemy.offsetWidth / 2;

  enemy.style.left =
    x + "px";

  enemy.style.top =
    "-120px";

  game.appendChild(enemy);

  enemies.push({

    element: enemy,

    x: x,

    y: -120,

    speed: 220 + speed * 20

  });
}


/* كشف الاصطدام */

function collision(a,b) {

  const A =
    a.getBoundingClientRect();

  const B =
    b.getBoundingClientRect();

  return (

    A.left < B.right - 8 &&

    A.right > B.left + 8 &&

    A.top < B.bottom - 8 &&

    A.bottom > B.top + 8

  );
}


/* حلقة اللعبة */

function gameLoop(time) {

  if (!running) return;

  const delta =
    Math.min(
      (time - lastTime) / 1000,
      0.04
    );

  lastTime = time;


  /* حركة اللاعب */

  const movement =
    300 * delta;


  if (keys.left) {

    playerX -= movement;

  }


  if (keys.right) {

    playerX += movement;

  }


  /* منع السيارة من الخروج */

  playerX = Math.max(

    10,

    Math.min(

      game.clientWidth -
      player.offsetWidth -
      10,

      playerX

    )

  );


  player.style.left =
    playerX + "px";


  /* إنشاء السيارات */

  spawnTimer += delta;

  const spawnTime =
    Math.max(
      0.4,
      1 - speed * 0.05
    );


  if (spawnTimer > spawnTime) {

    createEnemy();

    spawnTimer = 0;

  }


  /* تحريك السيارات */

  enemies =
    enemies.filter(enemy => {

      enemy.y +=
        enemy.speed * delta;

      enemy.element.style.top =
        enemy.y + "px";


      /* اصطدام */

      if (
        collision(
          player,
          enemy.element
        )
      ) {

        endGame();

        return false;

      }


      /* خرجت من الشاشة */

      if (
        enemy.y >
        game.clientHeight + 50
      ) {

        enemy.element.remove();

        return false;

      }


      return true;

    });


  /* النقاط */

  score += delta * 10;

  speed =
    1 + Math.floor(score / 100);


  scoreText.textContent =
    Math.floor(score);

  speedText.textContent =
    speed;


  requestAnimationFrame(gameLoop);
}


/* أزرار الهاتف */

function controlButton(button,key) {

  button.addEventListener(
    "pointerdown",
    function(e) {

      e.preventDefault();

      keys[key] = true;

    }
  );


  button.addEventListener(
    "pointerup",
    function() {

      keys[key] = false;

    }
  );


  button.addEventListener(
    "pointercancel",
    function() {

      keys[key] = false;

    }
  );


  button.addEventListener(
    "pointerleave",
    function() {

      keys[key] = false;

    }
  );

}


controlButton(
  document.getElementById("left"),
  "left"
);


controlButton(
  document.getElementById("right"),
  "right"
);


/* لوحة المفاتيح */

document.addEventListener(
  "keydown",
  function(e) {

    if (e.key === "ArrowLeft") {

      keys.left = true;

    }

    if (e.key === "ArrowRight") {

      keys.right = true;

    }

  }
);


document.addEventListener(
  "keyup",
  function(e) {

    if (e.key === "ArrowLeft") {

      keys.left = false;

    }

    if (e.key === "ArrowRight") {

      keys.right = false;

    }

  }
);

</script>

</body>
</html>
