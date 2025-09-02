<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Bitcoin Clicker</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Roboto+Mono&display=swap');
  body {
    margin: 0;
    font-family: 'Roboto Mono', monospace;
    background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
    color: #eee;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    user-select: none;
  }
  header {
    margin: 20px;
    text-align: center;
  }
  h1 {
    font-size: 3rem;
    letter-spacing: 2px;
    text-shadow: 0 0 10px #f7931a;
  }
  #bitcoin-btn {
    background: none;
    border: none;
    cursor: pointer;
    margin: 20px;
    width: 150px;
    height: 150px;
    filter: drop-shadow(0 0 5px #f7931a);
    transition: transform 0.1s ease;
  }
  #bitcoin-btn:active {
    transform: scale(0.95);
  }
  #bitcoin-icon {
    width: 100%;
    height: 100%;
  }
  #stats {
    margin-bottom: 20px;
    font-size: 1.2rem;
  }
  #total-clicks {
    margin-top: 10px;
    font-size: 1rem;
    color: #bbb;
  }
  #upgrades {
    max-width: 600px;
    width: 90vw;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 15px;
  }
  .upgrade {
    background: rgba(255 255 255 / 0.1);
    border: 1px solid #f7931a;
    border-radius: 10px;
    padding: 15px;
    width: 180px;
    text-align: center;
    cursor: pointer;
    transition: background 0.3s;
  }
  .upgrade:hover:not(.disabled) {
    background: rgba(255 255 255 / 0.2);
  }
  .upgrade.disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  .upgrade h3 {
    margin: 0 0 8px 0;
    font-size: 1.1rem;
    color: #f7931a;
  }
  .upgrade p {
    margin: 0 0 8px 0;
    font-size: 0.9rem;
  }
  .upgrade .cost {
    font-weight: bold;
    color: #f7931a;
  }
  #achievements-btn {
    margin: 25px;
    padding: 10px 25px;
    font-size: 1.1rem;
    background: #f7931a;
    border: none;
    border-radius: 25px;
    color: #111;
    cursor: pointer;
    box-shadow: 0 0 10px #f7931a;
    transition: background 0.3s;
  }
  #achievements-btn:hover {
    background: #d67c00;
  }
  #achievements-screen {
    position: fixed;
    top: 0; left: 0;
    width: 100vw; height: 100vh;
    background: rgba(20, 20, 20, 0.95);
    color: #f7931a;
    display: none;
    flex-direction: column;
    align-items: center;
    padding: 40px 20px;
    overflow-y: auto;
    z-index: 9999;
  }
  #achievements-screen h2 {
    margin-bottom: 20px;
  }
  #achievements-list {
    max-width: 600px;
    width: 90vw;
  }
  .achievement {
    border-bottom: 1px solid #f7931a;
    padding: 10px 0;
    font-size: 1.1rem;
    color: #bbb;
  }
  .achievement.unlocked {
    color: #f7b25c;
    font-weight: bold;
  }
  #close-achievements {
    margin-top: 30px;
    background: #f7931a;
    border: none;
    border-radius: 25px;
    padding: 10px 25px;
    font-size: 1.1rem;
    cursor: pointer;
    color: #111;
    box-shadow: 0 0 10px #f7931a;
  }
  footer {
    margin: 20px;
    font-size: 0.8rem;
    color: #555;
  }
  /* Scrollbar for achievements */
  #achievements-screen::-webkit-scrollbar {
    width: 8px;
  }
  #achievements-screen::-webkit-scrollbar-thumb {
    background-color: #f7931a;
    border-radius: 4px;
  }
</style>
</head>
<body>

<header>
  <h1>Bitcoin Clicker</h1>
</header>

<button id="bitcoin-btn" title="Click the Bitcoin!">
  <img id="bitcoin-icon" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Bitcoin.svg/1200px-Bitcoin.svg.png" alt="Bitcoin" />
</button>

<div id="stats">
  <div><strong>Bitcoin:</strong> <span id="bitcoin-count">0</span></div>
  <div><strong>Bitcoins Per Click:</strong> <span id="per-click">1</span></div>
  <div><strong>Bitcoins Per Second:</strong> <span id="per-second">0</span></div>
  <div id="total-clicks">Total Clicks: 0</div>
</div>

<section id="upgrades"></section>

<button id="achievements-btn">Show Achievements</button>

<div id="achievements-screen">
  <h2>Achievements</h2>
  <div id="achievements-list"></div>
  <button id="close-achievements">Close</button>
</div>

<audio id="bg-music" loop src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_b226d4d57f.mp3?filename=chill-loop-1108.mp3" preload="auto"></audio>
<audio id="click-sound" src="https://freesound.org/data/previews/256/256113_3263906-lq.mp3" preload="auto"></audio>

<footer>Made with ❤️ by ChatGPT</footer>

<script>
(() => {
  const bitcoinCountEl = document.getElementById('bitcoin-count');
  const perClickEl = document.getElementById('per-click');
  const perSecondEl = document.getElementById('per-second');
  const totalClicksEl = document.getElementById('total-clicks');
  const bitcoinBtn = document.getElementById('bitcoin-btn');
  const upgradesContainer = document.getElementById('upgrades');
  const achievementsBtn = document.getElementById('achievements-btn');
  const achievementsScreen = document.getElementById('achievements-screen');
  const achievementsList = document.getElementById('achievements-list');
  const closeAchievementsBtn = document.getElementById('close-achievements');
  const bgMusic = document.getElementById('bg-music');
  const clickSound = document.getElementById('click-sound');

  // Game state
  let bitcoin = 0;
  let totalClicks = 0;
  let bitcoinsPerClick = 1;
  let bitcoinsPerSecond = 0;

  // Upgrades data
  const upgrades = [
    {
      id: 'bitcoin_rising',
      name: 'BitCoin is Raising in Net Worth',
      cost: 50,
      perClick: 1,
      perSecond: 0,
      owned: 0
    },
    {
      id: 'going_up',
      name: 'Going Up in Prize',
      cost: 100,
      perClick: 0,
      perSecond: 1,
      owned: 0
    },
    {
      id: 'more_bitcoins',
      name: 'More BitCoins',
      cost: 500,
      perClick: 5,
      perSecond: 0,
      owned: 0
    },
    {
      id: 'bitcoin_farm',
      name: 'BitCoin Farm',
      cost: 1500,
      perClick: 0,
      perSecond: 2.5,
      owned: 0
    },
    {
      id: 'better_bitcoin',
      name: 'Better BitCoin',
      cost: 2500,
      perClick: 10,
      perSecond: 0,
      owned: 0
    },
    {
      id: 'better_farms',
      name: 'Better BitCoin Farms',
      cost: 5000,
      perClick: 0,
      perSecond: 15,
      owned: 0
    },
    {
      id: 'hmm_enough',
      name: 'Hmm I Think There Is Enough',
      cost: 10250,
      perClick: 25,
      perSecond: 0,
      owned: 0
    },
    {
      id: 'bitcoin_factory',
      name: 'BitCoin Factory',
      cost: 15000,
      perClick: 0,
      perSecond: 25,
      owned: 0
    },
    {
      id: 'never_enough',
      name: 'Never Enough BitCoin',
      cost: 30000,
      perClick: 100,
      perSecond: 0,
      owned: 0
    },
    {
      id: 'too_good',
      name: 'BitCoin Is To Good Of A Coin',
      cost: 50000,
      perClick: 0,
      perSecond: 100,
      owned: 0
    },
    {
      id: 'bitcoin_planet',
      name: 'BitCoin Planet',
      cost: 100000,
      perClick: 0,
      perSecond: 500,
      owned: 0
    },
    {
      id: 'bitcoin_galaxy',
      name: 'BITCOIN GALAXY',
      cost: 1000000,
      perClick: 0,
      perSecond: 2500,
      owned: 0
    }
  ];

  // Achievements data
  const achievements = [
    {
      id: '1mil',
      name: '1Mil',
      desc: 'Get 1,000,000 Bitcoin',
      unlocked: false,
      condition: () => bitcoin >= 1_000_000
    },
    {
      id: '1bil',
      name: '1Bil',
      desc: 'Get 1,000,000,000 Bitcoin',
      unlocked: false,
      condition: () => bitcoin >= 1_000_000_000
    },
    {
      id: '1tri',
      name: '1Tri',
      desc: 'Get 1,000,000,000,000 Bitcoin',
      unlocked: false,
      condition: () => bitcoin >= 1_000_000_000_000
    },
    {
      id: 'bitcoin_galaxy',
      name: 'You Got The Bit Coin Galaxy',
      desc: 'You need the BITCOIN GALAXY upgrade to unlock',
      unlocked: false,
      condition: () => upgrades.find(u => u.id === 'bitcoin_galaxy').owned > 0
    },
    {
      id: 'beat_game',
      name: 'You Beat The Game',
      desc: 'Have 1,000,000,000,000,000 Bitcoin',
      unlocked: false,
      condition: () => bitcoin >= 1_000_000_000_000_000
    }
  ];

  // Format large numbers for display
  function formatNumber(num) {
    if (num < 1_000) return num.toFixed(0);
    const units = ['K', 'M', 'B', 'T', 'Qa', 'Qi'];
    let unitIndex = -1;
    let scaled = num;
    while (scaled >= 1000 && unitIndex < units.length - 1) {
      scaled /= 1000;
      unitIndex++;
    }
    return scaled.toFixed(2) + units[unitIndex];
  }

  // Save/load game
  function saveGame() {
    const saveData = {
      bitcoin,
      totalClicks,
      bitcoinsPerClick,
      bitcoinsPerSecond,
      upgradesOwned: upgrades.map(u => u.owned),
      achievementsUnlocked: achievements.map(a => a.unlocked),
    };
    localStorage.setItem('bitcoin_clicker_save', JSON.stringify(saveData));
  }

  function loadGame() {
    const saveData = JSON.parse(localStorage.getItem('bitcoin_clicker_save'));
    if (!saveData) return;

    bitcoin = saveData.bitcoin || 0;
    totalClicks = saveData.totalClicks || 0;
    bitcoinsPerClick = saveData.bitcoinsPerClick || 1;
    bitcoinsPerSecond = saveData.bitcoinsPerSecond || 0;

    if (saveData.upgradesOwned && saveData.upgradesOwned.length === upgrades.length) {
      upgrades.forEach((u, i) => {
        u.owned = saveData.upgradesOwned[i];
      });
    }

    if (saveData.achievementsUnlocked && saveData.achievementsUnlocked.length === achievements.length) {
      achievements.forEach((a, i) => {
        a.unlocked = saveData.achievementsUnlocked[i];
      });
    }

    recalcStats();
  }

  // Update UI
  function updateUI() {
    bitcoinCountEl.textContent = formatNumber(bitcoin);
    perClickEl.textContent = formatNumber(bitcoinsPerClick);
    perSecondEl.textContent = formatNumber(bitcoinsPerSecond);
    totalClicksEl.textContent = 'Total Clicks: ' + totalClicks;

    upgradesContainer.innerHTML = '';
    upgrades.forEach(upg => {
      const div = document.createElement('div');
      div.className = 'upgrade';
      if (bitcoin < upg.cost) div.classList.add('disabled');
      div.innerHTML = `
        <h3>${upg.name}</h3>
        <p>Cost: <span class="cost">${formatNumber(upg.cost)}</span></p>
        <p>+${upg.perClick} per click, +${upg.perSecond} per second</p>
        <p>Owned: ${upg.owned}</p>
      `;
      div.addEventListener('click', () => buyUpgrade(upg.id));
      upgradesContainer.appendChild(div);
    });

    updateAchievementsUI();
  }

  // Buy upgrade
  function buyUpgrade(id) {
    const upg = upgrades.find(u => u.id === id);
    if (!upg) return;
    if (bitcoin < upg.cost) return;

    bitcoin -= upg.cost;
    upg.owned++;
    upg.cost = Math.floor(upg.cost * 1.15); // increase cost by 15% each purchase

    recalcStats();
    updateUI();
    saveGame();
  }

  // Recalculate stats
  function recalcStats() {
    bitcoinsPerClick = 1; // base
    bitcoinsPerSecond = 0;
    upgrades.forEach(u => {
      bitcoinsPerClick += u.perClick * u.owned;
      bitcoinsPerSecond += u.perSecond * u.owned;
    });
  }

  // Add bitcoins per second continuously
  setInterval(() => {
    bitcoin += bitcoinsPerSecond / 10;
    checkAchievements();
    updateUI();
    saveGame();
  }, 100);

  // Click handler
  bitcoinBtn.addEventListener('click', () => {
    bitcoin += bitcoinsPerClick;
    totalClicks++;
    checkAchievements();
    updateUI();
    saveGame();
    playClickSound();
  });

  // Achievements
  function checkAchievements() {
    achievements.forEach(a => {
      if (!a.unlocked && a.condition()) {
        a.unlocked = true;
        showAchievementToast(a.name);
        saveGame();
      }
    });
  }

  // Show achievement UI update
  function updateAchievementsUI() {
    achievementsList.innerHTML = '';
    achievements.forEach(a => {
      const div = document.createElement('div');
      div.className = 'achievement ' + (a.unlocked ? 'unlocked' : '');
      div.textContent = `${a.name} - ${a.desc}`;
      achievementsList.appendChild(div);
    });
  }

  // Achievement toast notification
  function showAchievementToast(name) {
    const toast = document.createElement('div');
    toast.textContent = `Achievement Unlocked: ${name}! 🎉`;
    toast.style.position = 'fixed';
    toast.style.bottom = '20px';
    toast.style.left = '50%';
    toast.style.transform = 'translateX(-50%)';
    toast.style.background = '#f7931a';
    toast.style.color = '#111';
    toast.style.padding = '10px 25px';
    toast.style.borderRadius = '25px';
    toast.style.fontWeight = 'bold';
    toast.style.boxShadow = '0 0 10px #f7931a';
    toast.style.zIndex = '10000';
    document.body.appendChild(toast);
    setTimeout(() => {
      toast.style.transition = 'opacity 1s ease';
      toast.style.opacity = '0';
      setTimeout(() => document.body.removeChild(toast), 1000);
    }, 3000);
  }

  // Show/hide achievements screen
  achievementsBtn.addEventListener('click', () => {
    updateAchievementsUI();
    achievementsScreen.style.display = 'flex';
  });
  closeAchievementsBtn.addEventListener('click', () => {
    achievementsScreen.style.display = 'none';
  });

  // Background music play/pause toggle
  let musicPlaying = false;
  bgMusic.volume = 0.3;
  // Auto play on first click
  bitcoinBtn.addEventListener('click', () => {
    if (!musicPlaying) {
      bgMusic.play().catch(() => {});
      musicPlaying = true;
    }
