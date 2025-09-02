<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bitcoin Clicker</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; background: #111; color: #f0f0f0; }
    button { padding: 10px 20px; margin: 5px; cursor: pointer; }
    .upgrade { margin: 10px 0; }
    #achievementsScreen { display: none; background: #222; padding: 20px; margin-top: 20px; border-radius: 10px; }
  </style>
</head>
<body>

  <h1>Bitcoin Clicker 💰</h1>
  <p><strong>BitCoins:</strong> <span id="bitcoin">0</span></p>
  <p><strong>Total Clicks:</strong> <span id="clicks">0</span></p>
  <button onclick="clickBitcoin()">💸 Click for Bitcoin</button>

  <h2>Upgrades</h2>
  <div id="upgrades"></div>

  <h2>Achievements</h2>
  <button onclick="toggleAchievements()">View Achievements</button>
  <div id="achievementsScreen">
    <ul id="achievementsList"></ul>
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
      { name: "BITCOIN GALAXY", cost: 1000000, click: 0, sec: 2500 }
    ];

    const achievements = [
      { name: "1Mil", condition: () => bitcoin >= 1_000_000 },
      { name: "1Bil", condition: () => bitcoin >= 1_000_000_000 },
      { name: "1Tri", condition: () => bitcoin >= 1_000_000_000_000 },
      { name: "You Got The Bit Coin Galaxy", condition: () => ownedUpgrades.includes("BITCOIN GALAXY") },
      { name: "You Beat The Game", condition: () => bitcoin >= 1_000_000_000_000_000 }
    ];

    const ownedUpgrades = [];
    const earnedAchievements = [];

    function clickBitcoin() {
      bitcoin += perClick;
      totalClicks++;
      updateUI();
      saveGame();
    }

    function buyUpgrade(upgrade) {
      if (bitcoin >= upgrade.cost && !ownedUpgrades.includes(upgrade.name)) {
        bitcoin -= upgrade.cost;
        perClick += upgrade.click;
        perSecond += upgrade.sec;
        ownedUpgrades.push(upgrade.name);
        updateUI();
        saveGame();
      }
    }

    function updateUI() {
      document.getElementById("bitcoin").innerText = Math.floor(bitcoin);
      document.getElementById("clicks").innerText = totalClicks;

      const upgradesDiv = document.getElementById("upgrades");
      upgradesDiv.innerHTML = "";
      upgrades.forEach(upg => {
        if (!ownedUpgrades.includes(upg.name)) {
          const btn = document.createElement("button");
          btn.innerText = `${upg.name} - Cost: ${upg.cost}`;
          btn.onclick = () => buyUpgrade(upg);
          upgradesDiv.appendChild(btn);
        }
      });

      checkAchievements();
    }

    function toggleAchievements() {
      const screen = document.getElementById("achievementsScreen");
      screen.style.display = screen.style.display === "none" ? "block" : "none";
    }

    function checkAchievements() {
      achievements.forEach(ach => {
        if (ach.condition() && !earnedAchievements.includes(ach.name)) {
          earnedAchievements.push(ach.name);
          const li = document.createElement("li");
          li.innerText = `Achievement Unlocked: ${ach.name}`;
          document.getElementById("achievementsList").appendChild(li);
          saveGame();
        }
      });
    }

    setInterval(() => {
      bitcoin += perSecond;
      updateUI();
    }, 1000);

    function saveGame() {
      const save = {
        bitcoin, perClick, perSecond, totalClicks, ownedUpgrades, earnedAchievements
      };
      localStorage.setItem("btcClickerSave", JSON.stringify(save));
    }

    function loadGame() {
      const save = JSON.parse(localStorage.getItem("btcClickerSave"));
      if (save) {
        bitcoin = save.bitcoin;
        perClick = save.perClick;
        perSecond = save.perSecond;
        totalClicks = save.totalClicks;
        save.ownedUpgrades.forEach(name => {
          ownedUpgrades.push(name);
        });
        save.earnedAchievements.forEach(name => {
          earnedAchievements.push(name);
          const li = document.createElement("li");
          li.innerText = `Achievement Unlocked: ${name}`;
          document.getElementById("achievementsList").appendChild(li);
        });
        updateUI();
      }
    }

    loadGame();
    updateUI();
  </script>
</body>
</html>
