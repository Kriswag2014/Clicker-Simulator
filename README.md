<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bitcoin Clicker</title>
  <style>
    body {
      background-color: #1e1e1e;
      color: #f1f1f1;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
    }
    h1 {
      color: #f7931a;
    }
    button {
      background-color: #f7931a;
      border: none;
      color: #1e1e1e;
      font-size: 18px;
      padding: 15px 30px;
      margin: 10px;
      border-radius: 10px;
      cursor: pointer;
    }
    button:disabled {
      background-color: #555;
      cursor: not-allowed;
    }
    .section {
      margin: 20px auto;
      max-width: 800px;
      background-color: #2b2b2b;
      border-radius: 10px;
      padding: 20px;
    }
    .upgrade {
      margin: 10px;
      padding: 10px;
      background-color: #3a3a3a;
      border-radius: 8px;
    }
    .achievement {
      padding: 10px;
      margin: 5px;
      background-color: #444;
      border-radius: 5px;
    }
    .achievement.unlocked {
      background-color: #f7931a;
      color: #1e1e1e;
      font-weight: bold;
    }
  </style>
</head>
<body>

  <h1>💰 Bitcoin Clicker 💰</h1>

  <p><strong>BitCoins:</strong> <span id="bitcoin">0</span></p>
  <p><strong>Per Click:</strong> <span id="perClick">1</span> |
     <strong>Per Second:</strong> <span id="perSecond">0</span></p>
  <p><strong>Total Clicks:</strong> <span id="totalClicks">0</span></p>

  <button id="clickButton">Click for BitCoins</button>

  <div class="section">
    <h2>🛠️ Upgrades</h2>
    <div id="upgradeContainer"></div>
  </div>

  <div class="section">
    <h2>🏆 Achievements</h2>
    <div id="achievementContainer"></div>
  </div>

  <!-- 🔊 Click sound -->
  <audio id="clickSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_d9c2c6e7fb.mp3"></audio>

  <!-- 🎵 Background music (chill lofi) -->
  <audio id="bgMusic" src="https://cdn.pixabay.com/audio/2023/01/09/audio_b3b12a30f6.mp3" autoplay loop></audio>

  <script>
    // Core variables
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
      { name: "BITCOIN GALAXY", cost: 1000000, click: 0, sec: 2500 }
    ];

    const achievements = [
      { name: "1Mil", description: "Reach 1,000,000 BTC", condition: () => bitcoin >= 1_000_000 },
      { name: "1Bil", description: "Reach 1,000,000,000 BTC", condition: () => bitcoin >= 1_000_000_000 },
      { name: "1Tri", description: "Reach 1,000,000,000,000 BTC", condition: () => bitcoin >= 1_000_000_000_000 },
      { name: "You Got The Bit Coin Galaxy", description: "Buy BITCOIN GALAXY", condition: () => upgradeCounts["BITCOIN GALAXY"] > 0 },
      { name: "You Beat The Game", description: "Reach 1,000,000,000,000,000 BTC", condition: () => bitcoin >= 1_000_000_000_000_000 },
      { name: "Click Master", description: "Reach 10,000 Total Clicks", condition: () => totalClicks >= 10000 }
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

    document.getElementById("clickButton").addEventListener("click", () => {
      clickSound.currentTime = 0;
      clickSound.play();
      bitcoin += perClick;
      totalClicks++;
      updateDisplay();
      checkAchievements();
      saveGame();
    });

    function createUpgrades() {
      upgrades.forEach(upg => {
        upgradeCounts[upg.name] = 0;

        const div = document.createElement("div");
        div.className = "upgrade";
        const btn = document.createElement("button");
        btn.textContent = `Buy "${upg.name}" - ${upg.cost} BTC`;
        btn.onclick = () => {
          if (bitcoin >= upg.cost) {
            bitcoin -= upg.cost;
            perClick += upg.click;
            perSecond += upg.sec;
            upgradeCounts[upg.name]++;
            updateDisplay();
            checkAchievements();
            saveGame();
          }
        };
        div.appendChild(btn);
        upgradeContainer.appendChild(div);
      });
    }

    function createAchievements() {
      achievements.forEach(ach => {
        unlockedAchievements[ach.name] = false;
        const div = document.createElement("div");
        div.className = "achievement";
        div.id = `ach-${ach.name}`;
        div.textContent = `${ach.name} - Locked`;
        achievementContainer.appendChild(div);
      });
    }

    function checkAchievements() {
      achievements.forEach(ach => {
        if (!unlockedAchievements[ach.name] && ach.condition()) {
          unlockedAchievements[ach.name] = true;
          const el = document.getElementById(`ach-${ach.name}`);
          el.classList.add("unlocked");
          el.textContent = `${ach.name} - ${ach.description}`;
          alert("Achievement unlocked: " + ach.name);
        }
      });
    }

    function updateDisplay() {
      bitcoinEl.textContent = Math.floor(bitcoin);
      perClickEl.textContent = perClick;
      perSecondEl.textContent = perSecond;
      totalClicksEl.textContent = totalClicks;
    }

    // Passive income every second
    setInterval(() => {
      bitcoin += perSecond;
      updateDisplay();
      checkAchievements();
      saveGame();
    }, 1000);

    // Save/load
    function saveGame() {
      const save = {
        bitcoin, perClick, perSecond, totalClicks, upgradeCounts, unlockedAchievements
      };
      localStorage.setItem("btcClickerSave", JSON.stringify(save));
    }

    function loadGame() {
      const save = JSON.parse(localStorage.getItem("btcClickerSave"));
      if (save) {
        bitcoin = save.bitcoin || 0;
        perClick = save.perClick || 1;
        perSecond = save.perSecond || 0;
       
