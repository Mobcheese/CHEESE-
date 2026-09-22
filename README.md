const game=document.querySelector('#game');
const tokenLayer=document.querySelector('#tokenLayer');
const enemyLayer=document.querySelector('#enemyLayer');
const player=document.querySelector('#player');
const scoreEl=document.querySelector('#score');
const timerEl=document.querySelector('#timer');
const rankEl=document.querySelector('#rank');
const districtLabel=document.querySelector('#districtLabel');
const progress=document.querySelector('#progress');
const feed=document.querySelector('#feed');
const healthBar=document.querySelector('#healthBar');
const missionTitle=document.querySelector('#missionTitle');
const missionGoal=document.querySelector('#missionGoal');
const startButton=document.querySelector('#startButton');
const overlay=document.querySelector('#gameOverlay');

const districts = {
  docks: { label: 'DOCKS', target: 10, multiplier: 1 },
  casino: { label: 'CASINO', target: 14, multiplier: 1.25 },
  uptown: { label: 'UPTOWN', target: 18, multiplier: 1.5 }
};

const state = {
  running: false,
  timer: 60,
  score: 0,
  missionProgress: 0,
  missionTarget: 10,
  district: 'docks',
  health: 100,
  lastTime: 0,
  player: { x: 50, y: 50 },
  keys: { up: false, down: false, left: false, right: false },
  tokens: [],
  enemies: [],
  districtIndex: 0
};

function addFeed(title, body) {
  const p = document.createElement('p');
  p.innerHTML = `<b>${title}</b><br>${body}`;
  feed.prepend(p);
}

function clamp(value, min, max) {
  return Math.min(max, Math.max(min, value));
}

function setMission() {
  const district = districts[state.district];
  missionTitle.textContent = `Clean the ${district.label}`;
  missionGoal.textContent = `Collect ${district.target} cheese tokens`;
  districtLabel.textContent = district.label;
}

function spawnTokens(count = 10) {
  tokenLayer.innerHTML = '';
  state.tokens = [];
  for (let i = 0; i < count; i++) {
    const token = document.createElement('div');
    token.className = 'token';
    token.textContent = '₡';
    const x = 12 + Math.random() * 72;
    const y = 18 + Math.random() * 62;
    token.style.left = `${x}%`;
    token.style.top = `${y}%`;
    tokenLayer.appendChild(token);
    state.tokens.push({ x, y, el: token });
  }
}

function spawnEnemies() {
  enemyLayer.innerHTML = '';
  state.enemies = [];
  const count = state.district === 'docks' ? 3 : state.district === 'casino' ? 5 : 6;

  for (let i = 0; i < count; i++) {
    const enemy = document.createElement('div');
    enemy.className = 'enemy';
    const x = 12 + Math.random() * 76;
    const y = 15 + Math.random() * 72;
    enemy.style.left = `${x}%`;
    enemy.style.top = `${y}%`;
    enemyLayer.appendChild(enemy);
    state.enemies.push({ x, y, velX: (Math.random() - 0.5) * 0.35, velY: (Math.random() - 0.5) * 0.35, el: enemy });
  }
}

function updateHUD() {
  scoreEl.textContent = state.score;
  timerEl.textContent = state.timer;
  healthBar.style.width = `${state.health}%`;
  const rank = state.score >= 80 ? 'CAPO' : state.score >= 50 ? 'ENFORCER' : state.score >= 20 ? 'THIEF' : 'ROOKIE';
  rankEl.textContent = rank;
  progress.style.width = `${Math.min((state.missionProgress / state.missionTarget) * 100, 100)}%`;
}

function resetPlayer() {
  state.player.x = 50;
  state.player.y = 50;
  player.style.left = `${state.player.x}%`;
  player.style.top = `${state.player.y}%`;
}

function setDistrict(name) {
  state.district = name;
  state.missionTarget = districts[name].target;
  state.missionProgress = 0;
  setMission();
  spawnTokens(state.missionTarget);
  spawnEnemies();
  resetPlayer();
  updateHUD();
}

function collectTokens() {
  for (const token of state.tokens) {
    const dx = token.x - state.player.x;
    const dy = token.y - state.player.y;
    if (Math.abs(dx) < 5 && Math.abs(dy) < 5) {
      token.el.remove();
      state.missionProgress += 1;
      state.score += 10;
      addFeed('CHEESE PICKUP', 'You stole a fresh haul.');
      const index = state.tokens.indexOf(token);
      if (index !== -1) state.tokens.splice(index, 1);
      if (state.missionProgress >= state.missionTarget) {
        advanceDistrict();
        return;
      }
    }
  }
  updateHUD();
}

function advanceDistrict() {
  const order = ['docks', 'casino', 'uptown'];
  const currentIndex = order.indexOf(state.district);
  const nextIndex = currentIndex + 1;

  if (nextIndex < order.length) {
    state.district = order[nextIndex];
    addFeed('DISTRICT CLEARED', `The crew moves into ${districts[state.district].label}.`);
    setDistrict(state.district);
  } else {
    state.running = false;
    overlay.classList.remove('hidden');
    overlay.querySelector('h2').textContent = 'City Boss';
    overlay.querySelector('p').textContent = `You cleared all districts with ${state.score} cheese.`;
    startButton.textContent = 'Play Again';
    addFeed('VICTORY', 'The syndicate owes you a lifetime of cheese.');
  }
}

function updateEnemies() {
  if (!state.running) return;

  for (const enemy of state.enemies) {
    const dx = state.player.x - enemy.x;
    const dy = state.player.y - enemy.y;
    const dist = Math.hypot(dx, dy) || 1;
    enemy.x += (dx / dist) * 0.25;
    enemy.y += (dy / dist) * 0.25;

    if (enemy.x < 5) enemy.x = 5;
    if (enemy.x > 95) enemy.x = 95;
    if (enemy.y < 12) enemy.y = 12;
    if (enemy.y > 86) enemy.y = 86;

    enemy.el.style.left = `${enemy.x}%`;
    enemy.el.style.top = `${enemy.y}%`;

    if (Math.abs(state.player.x - enemy.x) < 4 && Math.abs(state.player.y - enemy.y) < 4) {
      state.health = clamp(state.health - 8, 0, 100);
      healthBar.style.width = `${state.health}%`;
      addFeed('ATTACK', 'A patrol caught your trail.');
      if (state.health <= 0) {
        state.running = false;
        overlay.classList.remove('hidden');
        overlay.querySelector('h2').textContent = 'Busted';
        overlay.querySelector('p').textContent = 'The patrol got you. Try another run.';
        startButton.textContent = 'Restart';
        addFeed('DEFEAT', 'The night is over. Reset and try again.');
      }
    }
  }
}

function updatePlayer() {
  const step = 0.8;
  if (state.keys.left) state.player.x -= step;
  if (state.keys.right) state.player.x += step;
  if (state.keys.up) state.player.y -= step;
  if (state.keys.down) state.player.y += step;

  state.player.x = clamp(state.player.x, 5, 95);
  state.player.y = clamp(state.player.y, 12, 88);
  player.style.left = `${state.player.x}%`;
  player.style.top = `${state.player.y}%`;
}

function tick(timestamp) {
  const delta = timestamp - state.lastTime || 16;
  state.lastTime = timestamp;

  if (state.running) {
    updatePlayer();
    updateEnemies();
    collectTokens();

    if (state.timer > 0) {
      state.timer = Math.max(0, state.timer - delta / 1000);
      timerEl.textContent = Math.ceil(state.timer);
    }

    if (state.timer <= 0) {
      state.running = false;
      overlay.classList.remove('hidden');
      overlay.querySelector('h2').textContent = 'Time Up';
      overlay.querySelector('p').textContent = 'The crew lost the window. Hit restart and go again.';
      startButton.textContent = 'Retry';
      addFeed('TIMEOUT', 'The city closed before the heist was done.');
    }
  }

  requestAnimationFrame(tick);
}

function startGame() {
  state.running = true;
  state.score = 0;
  state.timer = 60;
  state.health = 100;
  state.missionProgress = 0;
  state.district = 'docks';
  state.districtIndex = 0;
  feed.innerHTML = '<p><b>DON VERMICELLI</b><br>Welcome to the operation, soldier.</p><p><b>TIP</b><br>Keep moving. The patrols hate cheese thieves.</p>';
  setDistrict('docks');
  overlay.classList.add('hidden');
  updateHUD();
}

window.addEventListener('keydown', (event) => {
  const key = event.key.toLowerCase();
  if (key === 'w' || key === 'arrowup') state.keys.up = true;
  if (key === 's' || key === 'arrowdown') state.keys.down = true;
  if (key === 'a' || key === 'arrowleft') state.keys.left = true;
  if (key === 'd' || key === 'arrowright') state.keys.right = true;
});

window.addEventListener('keyup', (event) => {
  const key = event.key.toLowerCase();
  if (key === 'w' || key === 'arrowup') state.keys.up = false;
  if (key === 's' || key === 'arrowdown') state.keys.down = false;
  if (key === 'a' || key === 'arrowleft') state.keys.left = false;
  if (key === 'd' || key === 'arrowright') state.keys.right = false;
});

startButton.addEventListener('click', startGame);

document.querySelectorAll('.district').forEach((button) => {
  button.addEventListener('click', () => {
    if (!state.running) return;
    document.querySelector('.district.active').classList.remove('active');
    button.classList.add('active');
    const nextDistrict = button.dataset.district;
    setDistrict(nextDistrict);
  });
});

document.querySelector('#walletButton').addEventListener('click', () => {
  const walletButton = document.querySelector('#walletButton');
  const walletStatus = document.querySelector('#walletStatus');
  walletButton.textContent = 'Wallet connected';
  walletStatus.textContent = 'Demo wallet: 7xK...MOB';
});

setDistrict('docks');
updateHUD();
requestAnimationFrame(tick);

startGame();
