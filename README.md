<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Bitcoin Clicker 💰</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    /* Same modern dark-themed CSS as before, trimmed for brevity */
    body {
      margin: 0;
      font-family: 'Orbitron', monospace;
      background-color: #111;
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
    }
    h1 {
      color: #f7931a;
      text-shadow: 0 0 10px #f7931a;
    }
    #clickButton {
      font-size: 24px;
      padding: 20px 40px;
      border-radius: 10px;
      border: none;
      background: linear-gradient(45deg, #f7931a, #ffcc00);
      color: #111;
      font-weight: bold;
      cursor: pointer;
      box-shadow: 0 4px 12px #f7931a80;
      margin: 20px;
    }
    #clickButton:active {
      transform: scale(0.98);
      box-shadow: 0 2px 6px #f7931a60;
    }
    .container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 20px;
      width: 100%;
      max-width: 1000px;
    }
    .panel {
      background: #222;
      padding: 20px;
      border-radius: 12px;
      box-shadow: inset 0 0 10px #000;
    }
    button {
      background: #f7931a;
      border: none;
      border-radius: 8px;
      padding: 10px 20px;
      color: #111;
      font-weight: bold;
      cursor: pointer;
      margin-top: 5px;
    }
    button:disabled {
      background: #666;
      cursor: not-allowed;
    }
    .achievement {
      margin: 5px 0;
      padding: 8px;
      border-radius: 6px;
      background-color: #333;
      color: #ccc;
    }
    .achievement.unlocked {
      background: #f7931a;
      color: #111;
    }
  </style>
</head>
<body>

<h1>Bitcoin Clicker 💰</h1>
<p><strong>BitCoins:</strong> <span id="bitcoin">0</span></p>
<p><strong>Per Click:</strong> <span id="perClick">1</span> | <strong>Per Second:</strong> <span id="perSecond">0</span></p>
<p><strong>Total Clicks:</strong> <span id="totalClicks">0</span></p>

<!-- 👇 CLICK SOUND HERE -->
<audio id="clickSound" src="https://cdn.pixabay.com/audio/2021/08/04/audio_62b1e3a37e.mp3" preload="auto"></audio>

<button id="clickButton">💸 Click to earn BitCoins</button>

<div class="container">
  <div class="panel">
    <h2>Upgrades</h2>
    <div id="upgrades"></div>
  </div>
  <div class="panel">
    <h2>Achievements</h2>
    <button onclick="toggleAchievements()">Toggle Achievements</button>
    <div id="achievementsList" style="margin-top: 10px;"></div>
  </div>
</div>

<script>
  let bitcoin = 0;
  let perClick = 1;
  let perSecond = 0;
  let totalClicks = 0;

  const upgrades = [
    { name: "BitCoin Is Raising In net Worth", cost: 50, click: 1, sec: 0 },
    { name: "Going Up In Prize", cost: 100, click: 0, sec: 1 },
    { name: "More BitCoins", cost: 500, click: 5, sec: 0 },
    { name: "BitCoin Farm", cost: 1500, click: 0, sec: 2.5 },
    { name: "Better Called BitCoin", cost: 2500, click: 10, sec: 0 },
    { name: "Better BitCoin Farms", cost: 5000, click: 0, sec: 15 },
    { name: "Hmm I Think There Is Enough", cost: 10250, click: 25, sec: 0 },
    { name: "BitCoin Factory", cost: 15000, click: 0, sec: 25 },
    { name: "Never Enough BitCoin", cost: 30000, click: 100, sec: 0 },
    { name: "BitCoin Is To Good Of A Coin", cost: 50000, click: 0, sec: 100 },
    { name: "BitCoin Planet", cost: 100000, click: 0, sec: 500 },
    { name: "BITCOIN GALAXY", cost: 1000000, click: 0, sec: 2500 },
  ];

  const achievements = [
    { name: "1Mil", condition: () => bitcoin >= 1_000_000 },
    { name: "1Bil", condition: () => bitcoin >= 1_000_000_000 },
    { name: "1Tri", condition: () => bitcoin >= 1_000_000_000_000 },
    { name: "You Got The Bit Coin Galaxy", condition: () => getUpgrade("BITCOIN GALAXY").count > 0 },
    { name: "You Beat The Game", condition: () => bitcoin >= 1_000_000_000_000_000 },
    { name: "Click Master", condition: () => totalClicks >= 10000 },
  ];

  let ownedUpgrades = {};
  let unlockedAchievements = {};

  const bitcoinDisplay = document.getElementById("bitcoin");
  const perClickDisplay = document.getElementById("perClick");
  const perSecondDisplay = document.getElementById("perSecond");
  const totalClicksDisplay = document.getElementById("totalClicks");
  const upgradesDiv = document.getElementById("upgrades");
  const achievementsList = document.getElementById("achievementsList");

  const clickSound = document.getElementById("clickSound");

  function updateUI() {
    bitcoinDisplay.textContent = Math.floor(bitcoin);
    perClickDisplay.textContent = perClick;
    perSecondDisplay.textContent = perSecond;
    totalClicksDisplay.textContent = totalClicks;
    drawUpgrades();
    drawAchievements();
  }

  function clickBitcoin() {
    clickSound.currentTime = 0;
    clickSound.play();
    bitcoin += perClick;
    totalClicks++;
    updateUI();
    saveGame();
  }

  function drawUpgrades() {
    upgradesDiv.innerHTML = "";
    upgrades.forEach((upg) => {
      if (!ownedUpgrades[upg.name]) ownedUpgrades[upg.name] = 0;
      const btn = document.createElement("button");
      btn.innerText = `${upg.name} (+${upg.click || upg.sec} ${upg.click ? "CPC" : "CPS"}) | ${upg.cost} BTC | Owned: ${ownedUpgrades[upg.name]}`;
      btn.onclick = () => buyUpgrade(upg);
      if (bitcoin < upg.cost) btn.disabled = true;
      upgradesDiv.appendChild(btn);
    });
  }

  function buyUpgrade(upg) {
    if (bitcoin >= upg.cost) {
      bitcoin -= upg.cost;
      perClick += upg.click;
      perSecond += upg.sec;
      ownedUpgrades[upg.name]++;
      updateUI();
      saveGame();
    }
  }

  function getUpgrade(name) {
    return { count: ownedUpgrades[name] || 0 };
  }

  function drawAchievements() {
    achievementsList.innerHTML = "";
    achievements.forEach((ach) => {
      if (!unlockedAchievements[ach.name] && ach.condition()) {
        unlockedAchievements[ach.name] = true;
        alert("Achievement Unlocked: " + ach.name);
      }
      const div = document.createElement("div");
      div.className = "achievement";
      if (unlockedAchievements[ach.name]) div.classList.add("unlocked");
      div.textContent = ach.name;
      achievementsList.appendChild(div);
    });
  }

  function toggleAchievements() {
    achievementsList.style.display = achievementsList.style.display === "none" ? "block" : "none";
  }

  document.getElementById("clickButton").onclick = clickBitcoin;

  setInterval(() => {
    bitcoin += perSecond;
    updateUI();
  }, 
