<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Bitcoin Clicker - Modern GUI</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Orbitron&display=swap');

  * {
    box-sizing: border-box;
  }

  body {
    margin: 0;
    font-family: 'Orbitron', monospace, sans-serif;
    background: #121212;
    color: #eee;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    user-select: none;
  }

  h1 {
    margin: 20px 0 5px 0;
    font-weight: 900;
    color: #f7931a;
    text-shadow: 0 0 10px #f7931a99;
  }

  .container {
    max-width: 900px;
    width: 95%;
    background: #1e1e1e;
    border-radius: 12px;
    box-shadow: 0 0 15px #f7931a66;
    padding: 20px 30px 40px 30px;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 25px;
  }

  .stats-panel, .click-panel, .upgrades-panel, .achievements-panel {
    background: #222;
    border-radius: 10px;
    padding: 20px;
    box-shadow: inset 0 0 12px #333;
  }

  .stats-panel h2,
  .click-panel h2,
  .upgrades-panel h2,
  .achievements-panel h2 {
    margin-top: 0;
    color: #f7931a;
    text-shadow: 0 0 6px #f7931aaa;
  }

  .stat {
    font-size: 1.25rem;
    margin: 8px 0;
  }

  #bitcoinAmount {
    font-size: 2.5rem;
    font-weight: 900;
    color: #f0b90b;
    letter-spacing: 0.05em;
    text-shadow: 0 0 15px #f0b90bcc;
  }

  #clickButton {
    font-size: 1.8rem;
    padding: 20px 40px;
    border: none;
    border-radius: 15px;
    background: linear-gradient(45deg, #f7931a, #f0b90b);
    color: #222;
    font-weight: 900;
    box-shadow: 0 5px 15px #f7931a88;
    cursor: pointer;
    transition: all 0.2s ease-in-out;
    user-select: none;
  }
  #clickButton:active {
    transform: scale(0.95);
    box-shadow: 0 2px 7px #f7931acc;
  }

  /* Upgrades List */
  .upgrade {
    background: #2a2a2a;
    margin: 12px 0;
    padding: 12px 16px;
    border-radius: 10px;
    box-shadow: 0 0 8px #000 inset;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: background 0.3s ease;
  }
  .upgrade:hover {
    background: #3d2c07;
    box-shadow: 0 0 12px #f7931acc inset;
  }

  .upgrade-info {
    text-align: left;
  }
  .upgrade-name {
    font-weight: 700;
    font-size: 1.1rem;
    color: #f7a519;
  }
  .upgrade-desc {
    font-size: 0.9rem;
    color: #bbb;
  }
  .upgrade-cost {
    font-weight: 700;
    color: #f0b90b;
    min-width: 110px;
    text-align: right;
  }

  .upgrade-buy-btn {
    margin-left: 15px;
    padding: 8px 14px;
    border: none;
    background: #f7931a;
    color: #222;
    font-weight: 900;
    border-radius: 10px;
    cursor: pointer;
    transition: background 0.25s ease;
  }
  .upgrade-buy-btn:disabled {
    background: #555;
    cursor: not-allowed;
  }
  .upgrade-buy-btn:hover:not(:disabled) {
    background: #f0b90b;
  }

  /* Achievements Panel */
  #achievementsScreen {
    max-height: 300px;
    overflow-y: auto;
    padding-right: 8px;
  }
  .achievement {
    background: #292929;
    margin: 8px 0;
    padding: 10px 15px;
    border-radius: 10px;
    box-shadow: inset 0 0 6px #000;
    font-weight: 700;
    color: #bbb;
  }
  .achievement.unlocked {
    background: #f7931a;
    color: #222;
    box-shadow: 0 0 10px #f7931acc;
  }

  /* Achievements toggle button */
  #achievementsToggle {
    margin-top: 15px;
    padding: 10px 18px;
    background: transparent;
    border: 2px solid #f7931a;
    color: #f7931a;
    font-weight: 700;
    border-radius: 12px;
    cursor: pointer;
    user-select: none;
    transition: all 0.3s ease;
  }
  #achievementsToggle:hover {
    background: #f7931a;
    color: #222;
  }

  /* Responsive */
  @media (max-width: 700px) {
    .container {
      grid-template-columns: 1fr;
    }
  }
</style>
</head>
<body>

<h1>Bitcoin Clicker 💰</h1>

<div class="container">

  <!-- Stats Panel -->
  <div class="stats-panel">
    <h2>Stats</h2>
    <p class="stat">BitCoins: <span id="bitcoinAmount">0</span></p>
    <p class="stat">BitCoins per Click: <span id="perClickAmount">1</span></p>
    <p class="stat">BitCoins per Second: <span id="perSecondAmount">0</span></p>
    <p class="stat">Total Clicks: <span id="totalClicksAmount">0</span></p>
  </div>

  <!-- Click Panel -->
  <div class="click-panel" style="display:flex; flex-direction: column; align-items:center; justify-content:center;">
    <h2>Click</h2>
    <button id="clickButton" title="Click to earn BitCoins!">💸 Click for BitCoins</button>
  </div>

  <!-- Upgrades Panel -->
  <div class="upgrades-panel">
    <h2>Upgrades</h2>
    <div id="upgradesList">
      <!-- Dynamically filled -->
    </div>
  </div>

  <!-- Achievements Panel -->
  <div class="achievements-panel">
    <h2>Achievements</h2>
    <button id="achievementsToggle">Show Achievements</button>
    <div id="achievementsScreen" style="display:none;"></div>
  </div>

</div>

<script>
(() => {
  // Game State
  let bitcoin = 0;
  let perClick = 1;
  let perSecond = 0;
  let totalClicks = 0;

  // Upgrades - fixed costs, unlimited purchases
  const upgrades = [
    { name: "BitCoin Is Raising In net Worth", cost: 50, clickInc: 1, secInc: 0, owned: 0, desc: "+1 BTC per click" },
    { name: "Going Up In Prize", cost: 100, clickInc: 0, secInc: 1, owned: 0, desc: "+1 BTC per second" },
    { name: "More BitCoins", cost: 500, clickInc: 5, secInc: 0, owned: 0, desc: "+5 BTC per click" },
    { name: "BitCoin Farm", cost: 1500, clickInc: 0, secInc: 2.5, owned: 0, desc: "+2.5 BTC per second" },
    { name: "Better Called BitCoin", cost: 2500, clickInc: 10, secInc: 0, owned: 0, desc: "+10 BTC per click" },
    { name: "Better BitCoin Farms", cost: 5000, clickInc: 0, secInc: 15, owned: 0, desc: "+15 BTC per second" },
    { name: "Hmm I Think There Is Enough", cost: 10250, clickInc: 25, secInc: 0, owned: 0, desc: "+25 BTC per click" },
    { name: "BitCoin Factory", cost: 15000, clickInc: 0, secInc: 25, owned: 0, desc: "+25 BTC per second" },
    { name: "Never Enough BitCoin", cost: 30000, clickInc: 100, secInc: 0, owned: 0, desc: "+100 BTC per click" },
    { name: "BitCoin Is To Good Of A Coin", cost: 50000, clickInc: 0, secInc: 100, owned: 0, desc: "+100 BTC per second" },
    { name: "BitCoin Planet", cost: 100000, clickInc: 0, secInc: 500, owned: 0, desc: "+500 BTC per second" },
    { name: "BITCOIN GALAXY", cost: 1000000, clickInc: 0, secInc: 2500, owned: 0, desc: "+2500 BTC per second" }
  ];

  // Achievements list - total clicks and btc milestones
  const achievements = [
    { id: '1Mil', name: "1Mil", desc: "Reach 1,000,000 BitCoins", unlocked: false, condition: () => bitcoin >= 1_000_000 },
    { id: '1Bil', name: "1Bil", desc: "Reach 1,000,000,000 BitCoins", unlocked: false, condition: () => bitcoin >= 1_000_000_000 },
    { id: '1Tri', name: "1Tri", desc: "Reach 1,000,000,000,000 BitCoins", unlocked: false, condition: () => bitcoin >= 1_000_000_000_000 },
    { id: 'Galaxy', name: "You Got The Bit Coin Galaxy", desc: "Buy the BITCOIN GALAXY upgrade", unlocked: false, condition: () => upgrades.find(u => u.name === "BITCOIN GALAXY").owned > 0 },
    { id: 'BeatGame', name: "You Beat The Game", desc: "Reach 1,000,000,000,000,000 BitCoins", unlocked: false, condition: () => bitcoin >= 1_000_000_000_000_000 },
    { id: 'ClickMaster', name: "Click Master", desc: "Make 10,000 total clicks", unlocked: false, condition: () => totalClicks >= 10_000 }
  ];

  // DOM Elements
  const bitcoinAmountEl = document.getElementById('bitcoinAmount');
  const perClickAmountEl = document.getElementById('perClickAmount');
  const perSecondAmountEl = document.getElementById('perSecondAmount');
  const totalClicksAmountEl = document.getElementById('totalClicksAmount');
  const clickButton = document.getElementById('clickButton');
  const upgradesListEl = document.getElementById('upgradesList');
  const achievementsScreen = document.getElementById('achievementsScreen');
  const achievementsToggle = document.getElementById('achievementsToggle');

  // Update UI function
  function updateUI() {
    bitcoinAmountEl.textContent = formatNumber(bitcoin);
    perClickAmountEl.textContent = formatNumber(perClick);
    perSecondAmountEl.textContent = formatNumber(perSecond);
    totalClicksAmountEl.textContent = totalClicks;

    // Render upgrades with buy buttons and counts
    upgradesListEl.innerHTML = '';
    upgrades.forEach((upg, i) => {
      const div = document.createElement('div');
      div.classList.add('upgrade');

      const infoDiv = document.createElement('div');
      infoDiv.classList.add('upgrade-info');

      const name = document.createElement('div');
      name.classList.add('upgrade-name');
      name.textContent = upg.name;
      infoDiv.appendChild(name);

      const desc = document.createElement('div');
      desc.classList.add('upgrade-desc');
      desc.textContent = `${upg.desc} | Owned: ${upg.owned}`;
      infoDiv.appendChild(desc);

      const cost = document.createElement('div');
      cost.classList.add('upgrade-cost');
      cost.textContent = formatNumber(upg.cost);

      const btn = document.createElement('button');
      btn.classList.add('upgrade-buy-btn');
      btn.textContent = 'Buy';
      btn.disabled = bitcoin < upg.cost;
      btn.title = `Cost: ${formatNumber(upg.cost)} BTC`;
      btn.onclick = () => buyUpgrade(i);

      div.appendChild(infoDiv);
      div.appendChild(cost);
      div.appendChild(btn);

      upgradesListEl.appendChild(div);
    });

    updateAchievementsUI();
  }

  // Buy upgrade logic
  function buyUpgrade(index) {
    const upg = upgrades[index];
    if (bitcoin >= upg.cost) {
      bitcoin -= upg.cost;
      upg.owned++;
      perClick += upg.clickInc;
      perSecond += upg.secInc;
      totalClicks++; // Count each upgrade purchase as a click for fun
      updateUI();
      saveGame();
    }
  }

  // Format numbers nicely with suffixes
  function formatNumber(num) {
    if (num < 1000) return Math.floor(num).toString();
    const suffixes = ["K", "M", "B", "T", "Qa", "Qi"];
    let suffixIndex = -1;
    let n = num;
    while (n >= 1000 && suffixIndex < suffixes.length -1) {
      n /= 1000;
      suffixIndex++;
    }
    return n.toFixed(2) + suffixes[suffixIndex];
  }

  // Click button handler
  clickButton.addEventListener('click', () => {
    bitcoin += perClick;
    totalClicks++;
    updateUI();
    saveGame();
  });

  // Auto BTC per second interval
  setInterval(() => {
    if (perSecond > 0) {
      bitcoin += perSecond;
      updateUI();
      saveGame();
    }
  }, 1000);

  // Achievements toggle button
  achievementsToggle.addEventListener('click', () => {
    if (achievementsScreen.style.display === 'none') {
      achievementsScreen.style.display = 'block';
      achievementsToggle.textContent = 'Hide Achievements';
    } else {
      achievementsScreen.style.display = 'none';
      achievementsToggle.textContent = 'Show Achievements';
    }
  });

  // Update achievements UI
  function updateAchievementsUI() {
    achievements.forEach(ach => {
      if (!ach.unlocked && ach.condition()) {
        ach.unlocked = true;
        showAchievementNotification(ach.name);
      }
    });

    achievementsScreen.innerHTML = '';
    achievements.forEach(ach => {
      const achEl = document.createElement('div');
      achEl.classList.add('achievement');
      if (ach.unlocked) achEl.classList.add('unlocked');
      achEl.textContent = `${ach.name}: ${ach.desc}`;
      achievementsScreen.appendChild(achEl);
    });
  }

  // Achievement unlocked notification (console + alert fallback)
  function showAchievementNotification(name) {
    // Simple alert or you can improve with toast notification
    alert(`Achievement Unlocked: ${name}!`);
  }

  // Save game to localStorage
  function saveGame() {
    const save = {
      bitcoin,
      perClick,
      perSecond,
      totalClicks,
      upgradesOwned: upgrades.map(u => u.owned),
      achievementsUnlocked: achievements.map(a => a.unlocked)
    };
    localStorage.setItem('bitcoinClickerSave', JSON.stringify(save));
  }

  // Load game from localStorage
  function loadGame() {
    const save = JSON.parse(localStorage.getItem('bitcoinClickerSave'));
    if (save) {
      bitcoin = save.bitcoin || 0;
      perClick = save.perClick || 1;
      perSecond = save.perSecond || 0;
      totalClicks = save.totalClicks || 0;
      if (save.upgradesOwned) {
        upgrades.forEach((upg, i) => {
          upg.owned = save.upgradesOwned[i] || 0;
        });
      }
      if (save.achievementsUnlocked) {
        achievements.forEach((ach, i) => {
          ach.unlocked = save.achievementsUnlocked[i] || false;
        });
      }
    }
  }

  // Initialize game
  loadGame();
  updateUI();

})();
</script>

</body>
</html>
