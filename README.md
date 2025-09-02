<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bitcoin Clicker 💰</title>
  <style>
    * {
      box-sizing: border-box;
    }
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #0f0f0f, #1e1e1e);
      color: #f9f9f9;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 30px;
    }

    h1 {
      color: #f7931a;
      font-size: 3em;
      margin-bottom: 10px;
      text-shadow: 0 0 10px #f7931a;
    }

    .stats {
      margin: 10px;
      font-size: 1.2em;
    }

    .btn-click {
      font-size: 1.5em;
      padding: 20px 40px;
      margin: 20px;
      background: #f7931a;
      border: none;
      border-radius: 10px;
      color: #111;
      cursor: pointer;
      box-shadow: 0 0 15px #f7931a80;
      transition: all 0.2s ease-in-out;
    }

    .btn-click:hover {
      transform: scale(1.05);
      box-shadow: 0 0 25px #f7931a;
    }

    .section {
      background: #2b2b2b;
      border-radius: 10px;
      padding: 20px;
      width: 90%;
      max-width: 800px;
      margin-top: 20px;
      box-shadow: 0 0 20px #000;
    }

    .upgrade {
      margin: 10px 0;
      padding: 10px;
      background-color: #3a3a3a;
      border-radius: 8px;
    }

    .upgrade button {
      padding: 10px 20px;
      background: #f7931a;
      color: #111;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }

    .achievement {
      padding: 10px;
      margin: 5px;
      background-color: #444;
      border-radius: 5px;
      color: #ccc;
    }

    .achievement.unlocked {
      background-color: #f7931a;
      color: #111;
      font-weight: bold;
    }

    audio {
      display: none;
    }
  </style>
</head>
<body>

  <h1>Bitcoin Clicker 💰</h1>

  <div class="stats">
    <p>💰 BitCoins: <span id="bitcoin">0</span></p>
    <p>🖱️ Per Click: <span id="perClick">1</span> | ⏱️ Per Second: <span id="perSecond">0</span></p>
    <p>📈 Total Clicks: <span id="totalClicks">0</span></p>
  </div>

  <button class="btn-click" id="clickButton">💸 Click Me</button>

  <div class="section">
    <h2>🔧 Upgrades</h2>
    <div id="upgradeContainer"></div>
  </div>

  <div class="section">
    <h2>🏆 Achievements</h2>
    <div id="achievementContainer"></div>
  </div>

  <!-- Click Sound -->
  <audio id="clickSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_d9c2c6e7fb.mp3" preload="auto"></audio>

  <!-- Background Music -->
  <audio id="bgMusic" src="https://cdn.pixabay.com/audio/2023/01/09/audio_b3b12a30f6.mp3" autoplay loop></audio>

  <script>
    let bitcoin = 0;
    let perClick = 1;
    let perSecond = 0;
    let totalClicks = 0;

    const upgrades = [
      { name: "BitCoin Is Raising", cost: 50, click: 1, sec: 0 },
      { name: "Going Up In Prize", cost: 100, click: 0, sec: 1 },
      { name: "More BitCoins", cost: 500, click: 5, sec: 0 },
      { name: "BitCoin Farm", cost: 1500, click: 0, sec: 2.5 },
      { name: "Better BitCoin", cost: 2500, click: 10, sec: 0 },
      { name: "BitCoin Super Farm", cost: 5000, click: 0, sec: 15 },
      { name: "Hmm, Enough?", cost: 10250, click: 25, sec: 0 },
      { name: "BitCoin Factory", cost: 15000, click: 0, sec: 25 },
      { name: "Never Enough", cost: 30000, click: 100, sec: 0 },
      { name: "Too Good Of A Coin", cost: 50000, click: 0, sec: 100 },
      { name: "BitCoin Planet", cost: 100000, click: 0, sec: 500 },
      { name: "BITCOIN GALAXY", cost: 1000000, click: 0, sec: 2500 }
    ];

    const achievements = [
      { name: "1Mil", description: "Reach 1,000,000 BTC", condition: () => bitcoin >= 1_000_000 },
      { name: "1Bil", description: "Reach 1,000,000,000 BTC", condition: () => bitcoin >= 1_000_000_000 },
      { name: "1Tri", description: "Reach 1,000,000,000,000 BTC", condition: () => bitcoin >= 1_000_000_000_000 },
      { name: "You Got The Bit Coin Galaxy", description: "Own BITCOIN GALAXY", condition: () => upgradeCounts["BITCOIN GALAXY"] > 0 },
      { name: "You Beat The Game", description: "Reach 1 Quadrillion BTC", condition: () => bitcoin >= 1_000_000_000_000_000 },
      { name: "Click Master", description: "Reach 10,000 Clicks", condition: () => totalClicks >= 10000 }
    ];

    const upgradeCounts = {};
    const unlockedAchievements = {};

    const bitcoinEl = document.getElementById("bitcoin");
    const perClickEl = document.getElementById("perClick");
    const perSecondEl = document.getElementById("perSecond");
    const totalClicksEl = document.getElementById("totalClicks");
    const upgradeContainer = document.getElementById("upgradeContainer");
    const achievementContainer = document.getElementById("achievementContainer");
    const clickSound = document.getElementById("clickSound");

    // Click Button Logic
    document.getElementById("clickButton").addEventListener("click", () => {
      bitcoin += perClick;
      totalClicks++;
      clickSound.currentTime = 0;
      clickSound.play();
      updateDisplay();
      checkAchievements();
    });

    function updateDisplay() {
      bitcoinEl.textContent = Math.floor(bitcoin);
      perClickEl.textContent = perClick;
      perSecondEl.textContent = perSecond;
      totalClicksEl.textContent = totalClicks;
    }

    function createUpgrades() {
      upgrades.forEach(upg => {
        upgradeCounts[upg.name] = 0;
        const div = document.createElement("div");
        div.className = "upgrade";

        const btn = document.createElement("button");
        btn.textContent = `${upg.name} (+${upg.click} CPC, +${upg.sec} CPS) — ${upg.cost} BTC`;
        btn.addEventListener("click", () => {
          if (bitcoin >= upg.cost) {
            bitcoin -= upg.cost;
            perClick += upg.click;
            perSecond += upg.sec;
            upgradeCounts[upg.name]++;
            updateDisplay();
            checkAchievements();
          }
        });

        div.appendChild(btn);
        upgradeContainer.appendChild(div);
      });
    }

    function createAchievements() {
      achievements.forEach(ach => {
        const div = document.createElement("div");
        div.className = "achievement";
        div.id = `ach-${ach.name}`;
        div.textContent = `${ach.name} - Locked`;
        achievementContainer.append
