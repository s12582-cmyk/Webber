text
index.htm1
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>李文博</title>
<style>
  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; margin: 0; padding: 0; }
  body {
    background: #0a0a14;
    color: #eee;
    font-family: 'Courier New', monospace;
    min-height: 100vh;
    padding: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    user-select: none;
  }
  h1 {
    color: #22d3ee;
    font-size: 22px;
    letter-spacing: 5px;
    margin-bottom: 4px;
    text-shadow: 0 0 20px #22d3ee;
  }
  .subtitle { color: #555; font-size: 11px; margin-bottom: 16px; letter-spacing: 2px; }
  .panel {
    width: 100%;
    max-width: 480px;
    background: #11111c;
    border: 2px solid #222;
    border-radius: 12px;
    padding: 16px;
    margin-bottom: 12px;
  }
  .resource { text-align: center; padding: 20px 0; }
  .resource .label { color: #888; font-size: 12px; letter-spacing: 2px; }
  .resource .value {
    color: #fbbf24;
    font-size: 32px;
    font-weight: bold;
    margin: 6px 0;
    text-shadow: 0 0 20px rgba(251,191,36,0.5);
  }
  .resource .rate { color: #4ade80; font-size: 13px; }
  .tapBtn {
    width: 100%;
    padding: 20px;
    background: linear-gradient(135deg, #22d3ee, #0891b2);
    border: none;
    border-radius: 10px;
    color: #000;
    font-family: inherit;
    font-size: 18px;
    font-weight: bold;
    letter-spacing: 2px;
    cursor: pointer;
    margin-bottom: 12px;
    transition: transform 0.08s;
    box-shadow: 0 4px 0 #0e7490;
  }
  .tapBtn:active { transform: translateY(3px); box-shadow: 0 1px 0 #0e7490; }
  .upgrade {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px;
    background: #15151f;
    border: 2px solid #2a2a3a;
    border-radius: 8px;
    margin-bottom: 8px;
    transition: all 0.15s;
    cursor: pointer;
  }
  .upgrade:hover { border-color: #22d3ee; }
  .upgrade.locked { opacity: 0.4; cursor: not-allowed; }
  .upgrade.affordable { border-color: #4ade80; }
  .upgrade .info { flex: 1; }
  .upgrade .name { color: #22d3ee; font-size: 14px; font-weight: bold; }
  .upgrade .desc { color: #888; font-size: 11px; margin-top: 3px; }
  .upgrade .lvl  { color: #fbbf24; font-size: 11px; margin-top: 2px; }
  .upgrade .cost {
    color: #fbbf24;
    font-size: 13px;
    font-weight: bold;
    text-align: right;
    margin-left: 10px;
  }
  .resetBtn {
    width: 100%;
    padding: 14px;
    background: #2a0a0a;
    border: 2px solid #ef4444;
    border-radius: 10px;
    color: #ef4444;
    font-family: inherit;
    font-size: 14px;
    font-weight: bold;
    letter-spacing: 2px;
    cursor: pointer;
    margin-top: 8px;
    transition: all 0.15s;
  }
  .resetBtn:hover { background: #ef4444; color: #000; }
  .resetBtn:disabled { opacity: 0.3; cursor: not-allowed; }
  .prestigeInfo { text-align: center; color: #a855f7; font-size: 12px; margin-top: 6px; }
  .stats {
    text-align: center;
    color: #666;
    font-size: 11px;
    margin-top: 8px;
    line-height: 1.6;
  }
  .stats span { color: #22d3ee; }
</style>
</head>
<body>

<h1>李文博</h1>
<div class="subtitle">IDLE ABYSS</div>

<div class="panel">
  <div class="resource">
    <div class="label">深淵能量</div>
    <div class="value" id="gold">0</div>
    <div class="rate" id="rate">+0 / 秒</div>
  </div>
  <button class="tapBtn" id="tapBtn">👆 點擊 +1</button>
</div>

<div class="panel" id="upgradePanel"></div>

<div class="panel">
  <button class="resetBtn" id="resetBtn" disabled>🔄 重置深淵（Prestige）</button>
  <div class="prestigeInfo" id="prestigeInfo">需要 1,000 能量才能重置</div>
</div>

<div class="stats">
  已遊玩 <span id="playTime">0</span> 秒 ｜ 總點擊 <span id="totalClicks">0</span> 次<br>
  歷史最高能量 <span id="maxGold">0</span>
</div>

<script>
(function(){
'use strict';

var game = {
  gold: 0,
  goldPerClick: 1,
  goldPerSec: 0,
  totalEarned: 0,
  totalClicks: 0,
  playTime: 0,
  maxGold: 0,
  prestigePoints: 0,
  prestigeMultiplier: 1,
  prestigeBonus: 1,
  upgrades: {},
  lastSave: Date.now()
};

var UPGRADES = [
  { id:'click', name:'強化點擊', desc:'每次點擊 +1 能量', baseCost:10, costMult:1.5, max:100,
    effect:function(lv){ return { goldPerClick: lv*1 }; } },
  { id:'auto1', name:'深淵汲取器', desc:'每秒 +0.5 能量', baseCost:50, costMult:1.6, max:100,
    effect:function(lv){ return { goldPerSec: lv*0.5 }; } },
  { id:'auto2', name:'能量收集陣', desc:'每秒 +3 能量', baseCost:500, costMult:1.7, max:100,
    effect:function(lv){ return { goldPerSec: lv*3 }; }, requires:'auto1' },
  { id:'auto3', name:'虛空引擎', desc:'每秒 +25 能量', baseCost:5000, costMult:1.8, max:100,
    effect:function(lv){ return { goldPerSec: lv*25 }; }, requires:'auto2' },
  { id:'boost', name:'全面增幅', desc:'所有收入 ×1.2', baseCost:2000, costMult:2.5, max:20,
    effect:function(lv){ return { globalMult: Math.pow(1.2, lv) }; }, requires:'auto1' },
  { id:'prestigeBoost', name:'深淵精華', desc:'重置點數 +50%', baseCost:50000, costMult:3, max:10,
    effect:function(lv){ return { prestigeBonus: 1 + lv*0.5 }; }, requires:'boost' }
];

function getUpgradeLevel(id) { return game.upgrades[id] || 0; }
function getUpgradeCost(def) {
  var lv = getUpgradeLevel(def.id);
  return Math.floor(def.baseCost * Math.pow(def.costMult, lv));
}
function isUpgradeAvailable(def) {
  if (def.requires) return getUpgradeLevel(def.requires) > 0;
  return true;
}
function recalcStats() {
  var baseClick = 1, baseSec = 0, globalMult = 1, prestigeBonus = 1;
  UPGRADES.forEach(function(def){
    var lv = getUpgradeLevel(def.id);
    if (lv === 0) return;
    var eff = def.effect(lv);
    if (eff.goldPerClick) baseClick += eff.goldPerClick;
    if (eff.goldPerSec) baseSec += eff.goldPerSec;
    if (eff.globalMult) globalMult *= eff.globalMult;
    if (eff.prestigeBonus) prestigeBonus = eff.prestigeBonus;
  });
  game.goldPerClick = baseClick * globalMult * game.prestigeMultiplier;
  game.goldPerSec = baseSec * globalMult * game.prestigeMultiplier;
  game.prestigeBonus = prestigeBonus;
}
function formatNum(n) {
  if (n < 1000) return Math.floor(n).toString();
  if (n < 1e6) return (n/1000).toFixed(2) + 'K';
  if (n < 1e9) return (n/1e6).toFixed(2) + 'M';
  if (n < 1e12) return (n/1e9).toFixed(2) + 'B';
  if (n < 1e15) return (n/1e12).toFixed(2) + 'T';
  if (n < 1e18) return (n/1e15).toFixed(2) + 'Qa';
  return (n/1e18).toFixed(2) + 'Qi';
}

var goldEl = document.getElementById('gold');
var rateEl = document.getElementById('rate');
var upgradePanel = document.getElementById('upgradePanel');
var resetBtn = document.getElementById('resetBtn');
var prestigeInfo = document.getElementById('prestigeInfo');
var playTimeEl = document.getElementById('playTime');
var totalClicksEl = document.getElementById('totalClicks');
var maxGoldEl = document.getElementById('maxGold');

function updateUI() {
  goldEl.textContent = formatNum(game.gold);
  rateEl.textContent = '+' + formatNum(game.goldPerSec) + ' / 秒';
  var prestigeGain = getPrestigeGain();
  if (game.totalEarned >= 1000) {
    resetBtn.disabled = false;
    resetBtn.textContent = '🔄 重置深淵（獲得 ' + formatNum(prestigeGain) + ' 精華）';
    prestigeInfo.textContent = '目前精華：' + game.prestigePoints + ' ｜ 加成 ×' + game.prestigeMultiplier.toFixed(2);
  } else {
    resetBtn.disabled = true;
    resetBtn.textContent = '🔄 重置深淵';
    prestigeInfo.textContent = '需要累積 1,000 能量才能重置';
  }
  playTimeEl.textContent = Math.floor(game.playTime);
  totalClicksEl.textContent = game.totalClicks;
  maxGoldEl.textContent = formatNum(game.maxGold);
  updateUpgradeCards();
}

function updateUpgradeCards() {
  UPGRADES.forEach(function(def){
    var card = document.getElementById('upg-' + def.id);
    if (!card) return;
    var lv = getUpgradeLevel(def.id);
    var cost = getUpgradeCost(def);
    var available = isUpgradeAvailable(def);
    var canAfford = game.gold >= cost;
    var maxed = lv >= def.max;
    card.classList.toggle('locked', !available || maxed);
    card.classList.toggle('affordable', available && canAfford && !maxed);
    var costEl = card.querySelector('.cost');
    var lvlEl = card.querySelector('.lvl');
    if (!available) {
      costEl.textContent = '🔒 未解鎖';
      lvlEl.textContent = '需要：' + UPGRADES.find(function(u){return u.id === def.requires;}).name;
    } else if (maxed) {
      costEl.textContent = 'MAX';
      lvlEl.textContent = '等級 ' + lv + ' / ' + def.max;
    } else {
      costEl.textContent = '💰 ' + formatNum(cost);
      lvlEl.textContent = '等級 ' + lv + ' / ' + def.max;
    }
  });
}

function buildUpgradeCards() {
  upgradePanel.innerHTML = '';
  UPGRADES.forEach(function(def){
    var card = document.createElement('div');
    card.className = 'upgrade';
    card.id = 'upg-' + def.id;
    card.innerHTML =
      '<div class="info">' +
        '<div class="name">' + def.name + '</div>' +
        '<div class="desc">' + def.desc + '</div>' +
        '<div class="lvl"></div>' +
      '</div>' +
      '<div class="cost"></div>';
    card.addEventListener('click', function(){ buyUpgrade(def); });
    upgradePanel.appendChild(card);
  });
}

function buyUpgrade(def) {
  if (!isUpgradeAvailable(def)) return;
  var lv = getUpgradeLevel(def.id);
  if (lv >= def.max) return;
  var cost = getUpgradeCost(def);
  if (game.gold < cost) return;
  game.gold -= cost;
  game.upgrades[def.id] = lv + 1;
  recalcStats();
  updateUI();
  playSound('buy');
}

document.getElementById('tapBtn').addEventListener('click', function(){
  game.gold += game.goldPerClick;
  game.totalEarned += game.goldPerClick;
  game.totalClicks++;
  if (game.gold > game.maxGold) game.maxGold = game.gold;
  updateUI();
  playSound('click');
});

function getPrestigeGain() {
  if (game.totalEarned < 1000) return 0;
  var base = Math.floor(Math.sqrt(game.totalEarned / 1000));
  return Math.floor(base * (game.prestigeBonus || 1));
}

document.getElementById('resetBtn').addEventListener('click', function(){
  var gain = getPrestigeGain();
  if (gain <= 0) return;
  if (!confirm('重置將失去所有升級與能量，但獲得 ' + gain + ' 深淵精華（永久加成）。\n確定要重置嗎？')) return;
  game.prestigePoints += gain;
  game.prestigeMultiplier = 1 + game.prestigePoints * 0.1;
  game.gold = 0;
  game.upgrades = {};
  game.totalEarned = 0;
  game.goldPerClick = 1;
  game.goldPerSec = 0;
  recalcStats();
  updateUI();
  playSound('prestige');
});

var lastTick = performance.now();
function gameLoop(now) {
  var dt = (now - lastTick) / 1000;
  lastTick = now;
  var gain = game.goldPerSec * dt;
  game.gold += gain;
  game.totalEarned += gain;
  if (game.gold > game.maxGold) game.maxGold = game.gold;
  game.playTime += dt;
  if (Math.floor(now / 100) !== Math.floor((now - dt*1000) / 100)) updateUI();
  requestAnimationFrame(gameLoop);
}

var SAVE_KEY = 'liwenboGameSave';
function save() {
  try { game.lastSave = Date.now(); localStorage.setItem(SAVE_KEY, JSON.stringify(game)); } catch(e) {}
}
function load() {
  try {
    var data = localStorage.getItem(SAVE_KEY);
    if (!data) return;
    var parsed = JSON.parse(data);
    Object.keys(parsed).forEach(function(k){ game[k] = parsed[k]; });
    if (game.lastSave) {
      var offlineSec = Math.min((Date.now() - game.lastSave) / 1000, 8 * 3600);
      recalcStats();
      var offlineGain = game.goldPerSec * offlineSec;
      if (offlineGain > 0) {
        game.gold += offlineGain;
        game.totalEarned += offlineGain;
        setTimeout(function(){
          alert('離線 ' + Math.floor(offlineSec) + ' 秒，獲得 ' + formatNum(offlineGain) + ' 能量！');
        }, 300);
      }
    }
    recalcStats();
  } catch(e) {}
}
setInterval(save, 5000);
window.addEventListener('beforeunload', save);
document.addEventListener('visibilitychange', function(){ if (document.hidden) save(); });

var audioCtx = null;
function playSound(type) {
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    var o = audioCtx.createOscillator();
    var g = audioCtx.createGain();
    var freq = 600, dur = 0.05, vol = 0.03, wave = 'square';
    if (type === 'buy')     { freq = 500; dur = 0.1; wave = 'sine'; vol = 0.05; }
    if (type === 'prestige'){ freq = 300; dur = 0.5; wave = 'sawtooth'; vol = 0.08; }
    o.type = wave;
    o.frequency.value = freq;
    g.gain.value = vol;
    g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + dur);
    o.connect(g); g.connect(audioCtx.destination);
    o.start(); o.stop(audioCtx.currentTime + dur);
  } catch(e) {}
}

buildUpgradeCards();
load();
recalcStats();
updateUI();
requestAnimationFrame(gameLoop);

})();
</script>
</body>
</html>
