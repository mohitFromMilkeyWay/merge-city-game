cat > game.js << 'EOF'
// ===== Merge City - Full Prototype (merge + spawn + sell + idle income + save/load) =====

const GRID_SIZE = 4;
const SAVE_KEY = "mergeCitySave";

let grid = [];
let coins = 0;
let lastSavedTime = Date.now();
let touchSourceIndex = null;
let draggedIndex = null;

const BUILDING_ICONS = {
  1: "🏠", 2: "🏡", 3: "🏘️", 4: "🏢", 5: "🏬", 6: "🏛️", 7: "🌆"
};

const MAX_LEVEL = 7;

const gridEl = document.getElementById("grid");
const coinCountEl = document.getElementById("coin-count");
const incomeRateEl = document.getElementById("income-rate");
const spawnBtn = document.getElementById("spawn-btn");
const resetBtn = document.getElementById("reset-btn");
const toastEl = document.getElementById("toast");

let toastTimeout = null;
function showToast(message) {
  toastEl.textContent = message;
  toastEl.classList.add("show");
  clearTimeout(toastTimeout);
  toastTimeout = setTimeout(() => toastEl.classList.remove("show"), 1500);
}

function incomePerSecond() {
  return grid.reduce((total, level) => level === 0 ? total : total + level * 0.5, 0);
}

function updateIncomeDisplay() {
  incomeRateEl.textContent = incomePerSecond().toFixed(1);
}

function saveGame() {
  const data = { grid: grid, coins: coins, lastSavedTime: Date.now() };
  localStorage.setItem(SAVE_KEY, JSON.stringify(data));
}

function loadGame() {
  const raw = localStorage.getItem(SAVE_KEY);
  if (!raw) { initEmptyGrid(); return false; }
  try {
    const data = JSON.parse(raw);
    grid = data.grid || [];
    coins = data.coins || 0;
    lastSavedTime = data.lastSavedTime || Date.now();
    if (grid.length !== GRID_SIZE * GRID_SIZE) initEmptyGrid();
    return true;
  } catch (e) {
    initEmptyGrid();
    return false;
  }
}

function applyOfflineEarnings() {
  const now = Date.now();
  const secondsAway = Math.floor((now - lastSavedTime) / 1000);
  const MAX_OFFLINE_SECONDS = 8 * 60 * 60;
  const cappedSeconds = Math.min(secondsAway, MAX_OFFLINE_SECONDS);
  if (cappedSeconds > 5) {
    const earned = Math.floor(incomePerSecond() * cappedSeconds);
    if (earned > 0) {
      coins += earned;
      showToast(`Welcome back! +${earned} coins earned while away`);
    }
  }
}

function initEmptyGrid() {
  grid = new Array(GRID_SIZE * GRID_SIZE).fill(0);
}

function renderGrid() {
  gridEl.innerHTML = "";
  grid.forEach((level, index) => {
    const cell = document.createElement("div");
    cell.classList.add("cell");
    cell.dataset.index = index;

    if (level > 0) {
      cell.classList.add("filled");
      cell.textContent = BUILDING_ICONS[level] || "❓";
      cell.draggable = true;
      const badge = document.createElement("span");
      badge.classList.add("level-badge");
      badge.textContent = "Lv" + level;
      cell.appendChild(badge);
    }

    if (index === touchSourceIndex) cell.classList.add("selected");

    cell.addEventListener("dragstart", onDragStart);
    cell.addEventListener("dragover", onDragOver);
    cell.addEventListener("drop", onDrop);
    cell.addEventListener("dragend", onDragEnd);
    cell.addEventListener("click", onCellClick);

    let pressTimer = null;
    const startPress = () => {
      if (level === 0) return;
      pressTimer = setTimeout(() => trySellBuilding(index), 600);
    };
    const cancelPress = () => clearTimeout(pressTimer);

    cell.addEventListener("touchstart", startPress);
    cell.addEventListener("touchend", cancelPress);
    cell.addEventListener("touchmove", cancelPress);
    cell.addEventListener("mousedown", startPress);
    cell.addEventListener("mouseup", cancelPress);
    cell.addEventListener("mouseleave", cancelPress);

    gridEl.appendChild(cell);
  });
  updateIncomeDisplay();
}

function onDragStart(e) {
  draggedIndex = parseInt(e.target.dataset.index);
  e.target.classList.add("dragging");
}
function onDragOver(e) { e.preventDefault(); }
function onDrop(e) {
  e.preventDefault();
  const targetIndex = parseInt(e.target.closest(".cell").dataset.index);
  tryMerge(draggedIndex, targetIndex);
}
function onDragEnd(e) { e.target.classList.remove("dragging"); }

function onCellClick(e) {
  const index = parseInt(e.currentTarget.dataset.index);
  if (touchSourceIndex === null) {
    if (grid[index] > 0) {
      touchSourceIndex = index;
      renderGrid();
    }
  } else {
    const source = touchSourceIndex;
    touchSourceIndex = null;
    tryMerge(source, index);
  }
}

function trySellBuilding(index) {
  const level = grid[index];
  if (level === 0) return;
  const sellValue = level * 25;
  const confirmed = confirm(`Sell this Lv${level} building for ${sellValue} coins?`);
  if (confirmed) {
    grid[index] = 0;
    touchSourceIndex = null;
    addCoins(sellValue);
    showToast(`Sold for +${sellValue} coins`);
    renderGrid();
    saveGame();
  }
}

function tryMerge(fromIndex, toIndex) {
  if (fromIndex === null || toIndex === null) return;
  if (fromIndex === toIndex) { renderGrid(); return; }

  const fromLevel = grid[fromIndex];
  const toLevel = grid[toIndex];

  if (fromLevel > 0 && toLevel === 0) {
    grid[toIndex] = fromLevel;
    grid[fromIndex] = 0;
    renderGrid();
    saveGame();
    return;
  }

  if (fromLevel > 0 && fromLevel === toLevel) {
    if (fromLevel >= MAX_LEVEL) {
      addCoins(100);
      showToast("Max level! +100 bonus coins");
      renderGrid();
      saveGame();
      return;
    }
    const newLevel = fromLevel + 1;
    grid[toIndex] = newLevel;
    grid[fromIndex] = 0;
    const reward = newLevel * 10;
    addCoins(reward);
    showToast(`Merged into Lv${newLevel}! +${reward} coins`);
    renderGrid();
    requestAnimationFrame(() => {
      const cellEl = gridEl.querySelector(`.cell[data-index="${toIndex}"]`);
      if (cellEl) {
        cellEl.classList.add("merged");
        setTimeout(() => cellEl.classList.remove("merged"), 300);
      }
    });
    saveGame();
    return;
  }
  renderGrid();
}

function addCoins(amount) {
  coins += amount;
  coinCountEl.textContent = Math.floor(coins);
}

function spawnBuilding() {
  const emptyIndexes = grid.map((val, idx) => val === 0 ? idx : null).filter(idx => idx !== null);
  if (emptyIndexes.length === 0) {
    showToast("Grid is full! Sell or merge buildings first.");
    return;
  }
  const randomIndex = emptyIndexes[Math.floor(Math.random() * emptyIndexes.length)];
  grid[randomIndex] = 1;
  renderGrid();
  saveGame();
}

function resetGame() {
  const confirmed = confirm("Reset all progress? This cannot be undone.");
  if (!confirmed) return;
  localStorage.removeItem(SAVE_KEY);
  coins = 0;
  initEmptyGrid();
  addCoins(0);
  renderGrid();
  spawnBuilding();
  saveGame();
}

function startIdleLoop() {
  setInterval(() => {
    const perSecond = incomePerSecond();
    if (perSecond > 0) {
      coins += perSecond;
      coinCountEl.textContent = Math.floor(coins);
    }
  }, 1000);
  setInterval(saveGame, 10000);
}

spawnBtn.addEventListener("click", spawnBuilding);
resetBtn.addEventListener("click", resetGame);
window.addEventListener("beforeunload", saveGame);

const hadSave = loadGame();
if (hadSave) {
  applyOfflineEarnings();
} else {
  spawnBuilding();
}
addCoins(0);
renderGrid();
startIdleLoop();
saveGame();
EOF
