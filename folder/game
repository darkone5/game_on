// According to Gemini Auto at 10:56:08 PM, it says to copy all code onwards from line 155 in my index.html file to line 638, before the <html>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Audio Setup
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

function playSound(freq, duration) {
  if (audioCtx.state === 'suspended') audioCtx.resume();
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.frequency.value = freq;
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  
  gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
  osc.start();
  gain.gain.exponentialRampToValueAtTime(0.00001, audioCtx.currentTime + duration);
  osc.stop(audioCtx.currentTime + duration);
}

// Input state
const keys = {};
window.addEventListener('keydown', e => {
  keys[e.key] = true;
  if (e.key === 'Escape') {
    togglePauseMenu();
  }
  
  if (e.key === 'Enter') {
    const overlay = document.getElementById('gameOverlay');
    if (!overlay.classList.contains('hidden')) {
      const primaryBtn = overlay.querySelector('.overlay-btn');
      if (primaryBtn) {
        primaryBtn.click();
      }
    }
  }
});
window.addEventListener('keyup', e => keys[e.key] = false);

// Game state variables
let currentLevel = 1;
let treasures = [];
let traps = [];
let staticObstacles = [];
let isPaused = false;
let isManualPause = false;
let pendingAction = null;

// Overlay Management
function showOverlay(title, message, buttonsConfig) {
  isPaused = true;
  document.getElementById('overlayTitle').innerText = title;
  document.getElementById('overlayMessage').innerText = message;

  const btnContainer = document.getElementById('overlayBtnContainer');
  btnContainer.innerHTML = ''; 

  buttonsConfig.forEach(btnInfo => {
    const btn = document.createElement('button');
    btn.innerText = btnInfo.text;
    btn.className = `overlay-btn ${btnInfo.class || ''}`;
    btn.onclick = btnInfo.onClick;
    btnContainer.appendChild(btn);
  });

  document.getElementById('gameOverlay').classList.remove('hidden');
}

function closeOverlay() {
  document.getElementById('gameOverlay').classList.add('hidden');
  isPaused = false;
  isManualPause = false;
  if (pendingAction) {
    const action = pendingAction;
    pendingAction = null;
    action();
  }
}

function togglePauseMenu() {
  const overlay = document.getElementById('gameOverlay');
  if (!overlay.classList.contains('hidden') && !isManualPause) {
    return; 
  }

  if (isPaused && isManualPause) {
    closeOverlay();
  } else if (!isPaused) {
    isPaused = true;
    isManualPause = true;
    showOverlay(
      "⏸ Game Paused",
      `Current Level: ${currentLevel}`,
      [
        { text: "Resume", class: "", onClick: () => closeOverlay() },
        { 
          text: "Restart from Level 1", 
          class: "secondary", 
          onClick: () => {
            closeOverlay();
            player.fullReset();
            initLevel(1);
          } 
        }
      ]
    );
  }
}

// Player class
class Player {
  constructor() {
    this.size = 32;
    this.startX = (canvas.width / 2) - (this.size / 2);
    this.startY = (canvas.height / 2) - (this.size / 2);
    this.x = this.startX;
    this.y = this.startY;
    this.speed = 4.5;
    this.score = 0;
    this.hp = 100;
    this.lives = 3;
    this.isProtected = true; 
  }

  resetPos() {
    this.x = this.startX;
    this.y = this.startY;
    this.isProtected = true; 
  }

  fullReset() {
    this.score = 0;
    this.hp = 100;
    this.lives = 3;
    this.resetPos();
  }

  update() {
    if (isPaused) return;

    let dx = 0;
    let dy = 0;
    let isMoving = false;

    if (keys['w'] || keys['W'] || keys['ArrowUp']) { dy -= this.speed; isMoving = true; }
    if (keys['s'] || keys['S'] || keys['ArrowDown']) { dy += this.speed; isMoving = true; }
    if (keys['a'] || keys['A'] || keys['ArrowLeft']) { dx -= this.speed; isMoving = true; }
    if (keys['d'] || keys['D'] || keys['ArrowRight']) { dx += this.speed; isMoving = true; }

    if (isMoving) {
      this.isProtected = false;
    }

    if (dx !== 0 || dy !== 0) {
      this.x += dx;
      if (this.x < 0 || this.x > canvas.width - this.size || checkShapeCollision(this.x, this.y, this.size, this.size)) {
        this.x -= dx;
      }

      this.y += dy;
      if (this.y < 0 || this.y > canvas.height - this.size || checkShapeCollision(this.x, this.y, this.size, this.size)) {
        this.y -= dy;
      }
    }
  }

  draw() {
    ctx.save();
    if (this.isProtected && Math.floor(Date.now() / 150) % 2 === 0) {
      ctx.globalAlpha = 0.5;
    }

    ctx.font = '28px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    
    ctx.fillText('👨‍✈️', this.x + this.size/2, this.y + this.size/2);

    if (this.isProtected) {
      ctx.strokeStyle = '#00ffff';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(this.x + this.size/2, this.y + this.size/2, this.size/1.5, 0, Math.PI * 2);
      ctx.stroke();
    }

    ctx.restore();
  }
}

const player = new Player();

function checkShapeCollision(x, y, w, h) {
  for (let obs of staticObstacles) {
    if (x < obs.x + obs.size && x + w > obs.x &&
        y < obs.y + obs.size && y + h > obs.y) {
      return true;
    }
  }
  return false;
}

function initLevel(lvl) {
  currentLevel = lvl;
  document.getElementById('level-display').innerText = currentLevel;
  player.resetPos();

  if (lvl === 1) {
    canvas.style.backgroundColor = '#5c4033'; 
    canvas.style.border = '4px solid #2e8b57'; 
  } else if (lvl === 2) {
    canvas.style.backgroundColor = '#d4ac0d'; 
    canvas.style.border = '4px solid #ffeb3b'; 
  } else if (lvl === 3) {
    canvas.style.backgroundColor = '#b3e5fc'; 
    canvas.style.border = '4px solid #0288d1'; 
  }

  staticObstacles = [];
  let obstacleCount = 5;
  if (lvl === 2) obstacleCount = 15;
  if (lvl === 3) obstacleCount = 25;

  for (let i = 0; i < obstacleCount; i++) {
    let obsX, obsY, obsSize;
    let attempts = 0;
    let valid = false;

    do {
      obsSize = 32; 
      obsX = Math.random() * (canvas.width - 100) + 50;
      obsY = Math.random() * (canvas.height - 100) + 50;

      const centerBuffer = (obsX < player.startX + 50 && obsX + obsSize > player.startX - 50 && 
                            obsY < player.startY + 50 && obsY + obsSize > player.startY - 50);

      if (!centerBuffer) {
        valid = true;
      }
      attempts++;
      if (attempts > 100) break;
    } while (!valid);

    staticObstacles.push({
      x: obsX,
      y: obsY,
      size: obsSize
    });
  }

  treasures = [];
  const treasureCount = 10 + (lvl - 1) * 10; 

  for (let i = 0; i < treasureCount; i++) {
    let tX, tY, overlapping;
    let attempts = 0;

    const rand = Math.random();
    let tColor, tPoints, tRadius;

    if (lvl === 1) {
      if (rand < 0.60) {
        tColor = '#cd7f32'; tPoints = 10; tRadius = 8;  
      } else {
        tColor = '#c0c0c0'; tPoints = 20; tRadius = 9;  
      }
    } else if (lvl === 2) {
      if (rand < 0.50) {
        tColor = '#cd7f32'; tPoints = 10; tRadius = 8;  
      } else if (rand < 0.80) {
        tColor = '#c0c0c0'; tPoints = 20; tRadius = 9;  
      } else {
        tColor = '#ffd700'; tPoints = 35; tRadius = 10; 
      }
    } else {
      if (rand < 0.50) {
        tColor = '#cd7f32'; tPoints = 10; tRadius = 8;  
      } else if (rand < 0.75) {
        tColor = '#c0c0c0'; tPoints = 20; tRadius = 9;  
      } else if (rand < 0.90) {
        tColor = '#ffd700'; tPoints = 35; tRadius = 10; 
      } else {
        tColor = '#b9f2ff'; tPoints = 50; tRadius = 12; 
      }
    }

    do {
      tX = Math.random() * (canvas.width - 60) + 30;
      tY = Math.random() * (canvas.height - 60) + 30;
      overlapping = checkShapeCollision(tX - tRadius, tY - tRadius, tRadius * 2, tRadius * 2);
      attempts++;
      if (attempts > 500) break; 
    } while (overlapping);

    treasures.push({
      x: tX,
      y: tY,
      radius: tRadius,
      points: tPoints,
      color: tColor
    });
  }

  traps = [];
  if (lvl === 2) {
    for (let i = 0; i < 4; i++) {
      traps.push({
        x: Math.random() * (canvas.width - 80) + 40,
        y: Math.random() * (canvas.height - 80) + 40,
        vx: (Math.random() - 0.5) * 3,
        vy: (Math.random() - 0.5) * 3,
        size: 20,
        type: 'red',
        damage: 0.55 
      });
    }
  } else if (lvl === 3) {
    for (let i = 0; i < 8; i++) {
      const isBlue = i % 2 === 0;
      traps.push({
        x: Math.random() * (canvas.width - 80) + 40,
        y: Math.random() * (canvas.height - 80) + 40,
        vx: (Math.random() - 0.5) * (isBlue ? 5 : 4.5),
        vy: (Math.random() - 0.5) * (isBlue ? 5 : 4.5),
        size: isBlue ? 22 : 20,
        type: isBlue ? 'blue' : 'red',
        damage: isBlue ? 0.833 : 0.55 
      });
    }
  }
}

function checkCollision(circle1, circle2) {
  const dx = (circle1.x + (circle1.size ? circle1.size/2 : 0)) - circle2.x;
  const dy = (circle1.y + (circle1.size ? circle1.size/2 : 0)) - circle2.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  const r1 = circle1.size ? circle1.size/2 : circle1.radius;
  const r2 = circle2.size ? circle2.size/2 : circle2.radius;
  return distance < (r1 + r2);
}

function handlePlayerDeath() {
  player.lives--;
  playSound(100, 0.1);
  if (player.lives > 0) {
    showOverlay(
      `⚠️ You Lost a Life! ⚠️`, 
      `Remaining Lives: ${player.lives}`, 
      [
        { text: "Continue", onClick: () => { player.hp = 100; player.resetPos(); closeOverlay(); } }
      ]
    );
  } else {
    showOverlay(
      `❌ Game Over! ❌`, 
      "All 3 lives lost. Restarting game back to Level 1.", 
      [
        { text: "Try Again", onClick: () => { player.fullReset(); initLevel(1); closeOverlay(); } }
      ]
    );
  }
}

function update() {
  if (isPaused) return;

  player.update();

  traps.forEach(trap => {
    trap.x += trap.vx;
    trap.y += trap.vy;
    if (trap.x < 0 || trap.x > canvas.width - trap.size) trap.vx *= -1;
    if (trap.y < 0 || trap.y > canvas.height - trap.size) trap.vy *= -1;

    if (!player.isProtected && checkCollision(player, {x: trap.x + trap.size/2, y: trap.y + trap.size/2, radius: trap.size/2})) {
      player.hp -= trap.damage;
      playSound(trap.type === 'blue' ? 160 : 130, 0.04);
      if (player.hp <= 0) handlePlayerDeath();
    }
  });

  treasures.forEach((t, index) => {
    if (checkCollision(player, t)) {
      player.score += t.points;
      playSound(500 + t.points * 4, 0.08);
      treasures.splice(index, 1);
    }
  });

  if (treasures.length === 0) {
    if (currentLevel < 3) {
      showOverlay(
        `🏆 Level ${currentLevel} Cleared! 🏆`, 
        `Advancing to Level ${currentLevel + 1}...`, 
        [
          { text: "Next Level", onClick: () => { initLevel(currentLevel + 1); closeOverlay(); } }
        ]
      );
    } else {
      showOverlay(
        `🎉 VICTORY! 🎉`, 
        `Game Complete!\n\nFinal Score: ${player.score}`, 
        [
          { text: "Play Again", onClick: () => { player.fullReset(); initLevel(1); closeOverlay(); } }
        ]
      );
    }
  }

  document.getElementById('p-score').innerText = player.score;
  document.getElementById('p-hp').innerText = Math.max(0, Math.floor(player.hp));
  document.getElementById('p-lives').innerText = player.lives;
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  ctx.font = '28px sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';

  staticObstacles.forEach(obs => {
    let sprite = '🌲'; 
    if (currentLevel === 2) sprite = '🌵'; 
    if (currentLevel === 3) sprite = '🧊'; 

    if (currentLevel === 2) {
      ctx.fillStyle = '#f1c40f'; 
    } else {
      ctx.fillStyle = '#ffffff';
    }

    const displaySprite = (currentLevel === 2) ? '🌵' : sprite;
    ctx.fillText(displaySprite, obs.x + obs.size/2, obs.y + obs.size/2);
  });

  treasures.forEach(t => {
    ctx.fillStyle = t.color;
    ctx.beginPath();
    ctx.arc(t.x, t.y, t.radius, 0, Math.PI * 2);
    ctx.fill();
  });

  traps.forEach(t => {
    if (t.type === 'blue') {
      ctx.fillStyle = '#0055ff';
      ctx.strokeStyle = '#0022aa';
    } else {
      ctx.fillStyle = '#ff3333';
      ctx.strokeStyle = '#990000';
    }
    ctx.fillRect(t.x, t.y, t.size, t.size);
    ctx.strokeRect(t.x, t.y, t.size, t.size);
  });

  player.draw();
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

initLevel(1);
gameLoop();
