# -<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Geometry Dash</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body {
    width: 100%; height: 100%; overflow: hidden;
    background: #000; font-family: Arial, sans-serif;
    touch-action: none; user-select: none;
  }
  #game { display: block; width: 100%; height: 100%; }
  #ui {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    pointer-events: none; color: #fff;
  }
  #hud {
    position: absolute; top: 15px; left: 15px; right: 15px;
    display: flex; justify-content: space-between;
    font-size: 20px; font-weight: bold; text-shadow: 2px 2px 4px #000;
  }

  /* ====== ЭКРАН ЗАГРУЗКИ ====== */
  #loading {
    position: fixed; inset: 0; z-index: 10000;
    background: linear-gradient(180deg, #0a0015, #1a0033);
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    color: #fff;
  }
  #loadingTitle {
    font-family: 'Arial Black', Arial, sans-serif;
    font-size: clamp(28px, 6vw, 48px);
    color: #7ed957;
    text-shadow: 3px 3px 0 #1a3a0a, 0 0 20px rgba(120,220,90,0.5);
    margin-bottom: 30px;
    letter-spacing: 3px;
  }
  #loadingBarWrap {
    width: min(500px, 80vw); height: 26px;
    background: rgba(0,0,0,0.6);
    border: 3px solid #00e5ff;
    border-radius: 15px; overflow: hidden;
    box-shadow: 0 0 25px rgba(0,229,255,0.5);
  }
  #loadingBar {
    height: 100%; width: 0%;
    background: linear-gradient(90deg, #00ff88, #00e5ff, #ff00e5);
    transition: width 0.6s ease;
    border-radius: 12px;
  }
  #loadingPercent {
    font-size: 32px; font-weight: bold; margin-top: 20px;
    color: #00e5ff; text-shadow: 0 0 15px rgba(0,229,255,0.7);
    font-family: 'Arial Black', Arial, sans-serif;
  }
  #loadingHint {
    margin-top: 20px; font-size: 15px; opacity: 0.7;
    letter-spacing: 1px;
  }

  /* ====== ЭКРАН ВВОДА НИКА ====== */
  #nicknameScreen {
    position: fixed; inset: 0; z-index: 9999;
    background: linear-gradient(180deg, #0a0015, #1a0033);
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    color: #fff; padding: 20px;
  }
  #nicknameScreen h2 {
    font-size: clamp(22px, 5vw, 34px);
    color: #7ed957;
    text-shadow: 3px 3px 0 #1a3a0a;
    margin-bottom: 25px;
    text-align: center;
    font-family: 'Arial Black', Arial, sans-serif;
  }
  #nicknameInput {
    width: min(400px, 85vw);
    padding: 15px 20px;
    font-size: 20px;
    border-radius: 12px;
    border: 3px solid #00e5ff;
    background: rgba(0,0,0,0.6);
    color: #fff;
    outline: none;
    text-align: center;
    box-shadow: 0 0 20px rgba(0,229,255,0.4);
  }
  #nicknameInput:focus { border-color: #ff00e5; box-shadow: 0 0 25px rgba(255,0,229,0.6); }
  #nicknameBtn {
    margin-top: 25px;
    padding: 14px 50px; font-size: 20px; font-weight: bold;
    border: none; border-radius: 12px; cursor: pointer;
    background: linear-gradient(135deg, #00ff88, #00e5ff);
    color: #000; box-shadow: 0 6px 20px rgba(0,200,255,0.5);
    transition: transform 0.1s;
    font-family: 'Arial Black', Arial, sans-serif;
  }
  #nicknameBtn:active { transform: scale(0.95); }

  /* ====== Главное меню ====== */
  #menu {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    background: linear-gradient(180deg, #f0a020 0%, #d08010 100%);
    pointer-events: auto; display: flex; flex-direction: column;
    overflow: hidden;
  }
  #menu::before {
    content: '';
    position: absolute; inset: 0;
    background-image:
      repeating-linear-gradient(0deg, transparent 0 58px, rgba(0,0,0,0.12) 58px 62px),
      repeating-linear-gradient(90deg, transparent 0 118px, rgba(0,0,0,0.12) 118px 122px);
    pointer-events: none;
    z-index: 0;
  }
  #menuInner {
    position: relative; z-index: 1;
    width: 100%; height: 100%;
    display: flex; flex-direction: column;
  }

  .menuTitle {
    text-align: center;
    margin-top: 4vh;
    font-size: clamp(28px, 7vw, 64px);
    font-weight: 900;
    letter-spacing: 4px;
    color: #7ed957;
    text-shadow:
      3px 3px 0 #1a3a0a,
      -1px -1px 0 #a8e060,
      0 0 20px rgba(120,220,90,0.5);
    font-family: 'Arial Black', Arial, sans-serif;
    -webkit-text-stroke: 2px #1a3a0a;
    line-height: 1;
  }
  .menuTitle.hidden { display: none; }

  .menuCenter {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: clamp(15px, 4vw, 50px);
    padding: 0 5%;
  }

  .bigIcon {
    width: clamp(70px, 14vw, 130px);
    height: clamp(70px, 14vw, 130px);
    border-radius: 20px;
    border: 5px solid #4a7a20;
    background: linear-gradient(145deg, #6ec93a, #3e8a15);
    box-shadow:
      0 8px 0 #2a5a10,
      0 12px 20px rgba(0,0,0,0.4),
      inset 0 4px 0 rgba(255,255,255,0.3);
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; transition: transform 0.1s, box-shadow 0.1s;
    position: relative;
    pointer-events: auto;
  }
  .bigIcon:active {
    transform: translateY(5px);
    box-shadow: 0 3px 0 #2a5a10, 0 6px 10px rgba(0,0,0,0.4);
  }
  .bigIcon .iconEmoji {
    font-size: clamp(35px, 7vw, 65px);
    filter: drop-shadow(2px 2px 2px rgba(0,0,0,0.4));
  }
  .bigIcon.play {
    background: linear-gradient(145deg, #ffe066, #f0a020);
    border-color: #b07010;
    box-shadow:
      0 8px 0 #8a5010,
      0 12px 25px rgba(255,200,0,0.5),
      inset 0 4px 0 rgba(255,255,255,0.4);
    width: clamp(90px, 17vw, 160px);
    height: clamp(90px, 17vw, 160px);
  }
  .bigIcon.play:active {
    box-shadow: 0 3px 0 #8a5010, 0 6px 12px rgba(0,0,0,0.4);
  }
  .bigIcon.play .iconEmoji {
    font-size: clamp(48px, 9vw, 85px);
    color: #fff;
  }

  .menuBottom {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 15px 20px;
    background: linear-gradient(180deg, #a06010, #7a4510);
    border-top: 4px solid #ffd060;
    gap: 10px;
    flex-wrap: wrap;
  }
  .menuBottomLeft {
    display: flex; gap: 12px; align-items: center;
  }
  .socialBtn {
    width: 42px; height: 42px;
    border-radius: 10px;
    border: 3px solid #4a2010;
    display: flex; align-items: center; justify-content: center;
    font-size: 20px; cursor: pointer;
    box-shadow: 0 3px 0 rgba(0,0,0,0.4);
    transition: transform 0.1s;
    pointer-events: auto;
  }
  .socialBtn:active { transform: translateY(3px); box-shadow: none; }
  .socialBtn.fb { background: #3b5998; }
  .socialBtn.tw { background: #1da1f2; }
  .socialBtn.yt { background: #ff0000; }
  .socialBtn.dc { background: #5865f2; }
  .socialBtn.tg { background: #0088cc; }

  .menuBottomCenter {
    display: flex; gap: clamp(10px, 2vw, 20px);
    align-items: center;
    justify-content: center;
    flex: 1;
    flex-wrap: wrap;
  }
  .circleBtn {
    width: clamp(48px, 9vw, 68px);
    height: clamp(48px, 9vw, 68px);
    border-radius: 50%;
    border: 4px solid #fff;
    background: linear-gradient(145deg, #6ec93a, #3e8a15);
    display: flex; align-items: center; justify-content: center;
    font-size: clamp(22px, 4vw, 32px);
    cursor: pointer;
    box-shadow:
      0 5px 0 #2a5a10,
      0 8px 15px rgba(0,0,0,0.4),
      inset 0 3px 0 rgba(255,255,255,0.3);
    transition: transform 0.1s;
    pointer-events: auto;
    position: relative;
  }
  .circleBtn:active {
    transform: translateY(4px);
    box-shadow: 0 1px 0 #2a5a10, 0 4px 8px rgba(0,0,0,0.4);
  }
  .circleBtn.gold {
    background: linear-gradient(145deg, #ffe066, #f0a020);
    border-color: #fff8d0;
    box-shadow:
      0 5px 0 #8a5010,
      0 8px 15px rgba(255,200,0,0.5),
      inset 0 3px 0 rgba(255,255,255,0.4);
  }
  .circleBtn .iconEmoji {
    filter: drop-shadow(1px 1px 2px rgba(0,0,0,0.5));
  }

  .menuBottomRight {
    display: flex; gap: 12px; align-items: center;
  }
  .moreGames {
    padding: 8px 16px;
    background: linear-gradient(145deg, #6ec93a, #3e8a15);
    border: 3px solid #fff;
    border-radius: 12px;
    font-weight: 900;
    font-size: 14px;
    letter-spacing: 1px;
    text-align: center;
    cursor: pointer;
    box-shadow: 0 4px 0 #2a5a10;
    color: #fff;
    text-shadow: 1px 1px 2px rgba(0,0,0,0.4);
    pointer-events: auto;
  }
  .moreGames:active { transform: translateY(3px); box-shadow: none; }

  .topRightInfo {
    position: absolute; top: 10px; right: 12px;
    display: flex; flex-direction: column; gap: 6px;
    align-items: flex-end; z-index: 5;
  }
  .infoPill {
    padding: 6px 14px;
    background: rgba(0,0,0,0.5);
    border: 2px solid #ffd060;
    border-radius: 20px;
    font-weight: bold;
    font-size: 14px;
    color: #ffe066;
    text-shadow: 1px 1px 2px #000;
    display: flex; align-items: center; gap: 6px;
  }
  .infoPill.nick {
    border-color: #00e5ff; color: #00e5ff;
    max-width: 160px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  #gameover, #skins, #shop, #settings, #levelSelect, #stats {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    display: flex; flex-direction: column; justify-content: flex-start; align-items: center;
    background: rgba(0,0,0,0.88); pointer-events: auto; padding: 20px;
    overflow-y: auto;
  }
  #gameover, #skins, #shop, #settings, #levelSelect, #stats { justify-content: center; }
  #skins, #shop, #stats { justify-content: flex-start; padding-top: 30px; }

  #gameover h1, #skins h1, #shop h1, #settings h1, #levelSelect h1, #stats h1 {
    font-size: clamp(28px, 5vw, 40px); margin-bottom: 15px;
    background: linear-gradient(90deg, #00e5ff, #ff00e5);
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
    background-clip: text;
    text-align: center;
  }
  #gameover p { font-size: 18px; margin: 6px; opacity: 0.9; }
  .btn-row { display: flex; gap: 15px; flex-wrap: wrap; justify-content: center; margin-top: 20px; }
  button {
    padding: 14px 35px; font-size: 20px; font-weight: bold;
    border: none; border-radius: 12px; cursor: pointer;
    background: linear-gradient(135deg, #00e5ff, #0088ff);
    color: #fff; box-shadow: 0 6px 20px rgba(0,200,255,0.5);
    transition: transform 0.1s;
  }
  button:active { transform: scale(0.95); }
  button.secondary {
    background: linear-gradient(135deg, #ff00e5, #8a00ff);
    box-shadow: 0 6px 20px rgba(255,0,229,0.5);
  }
  button.gold {
    background: linear-gradient(135deg, #ffcc00, #ff8800);
    box-shadow: 0 6px 20px rgba(255,180,0,0.5);
    color: #000;
  }
  button.back {
    background: linear-gradient(135deg, #555, #333);
    box-shadow: 0 6px 20px rgba(0,0,0,0.5);
  }
  .hidden { display: none !important; }
  #progress {
    position: absolute; top: 50px; left: 15px; right: 15px;
    height: 12px; background: rgba(255,255,255,0.2);
    border-radius: 6px; overflow: hidden;
  }
  #progressBar {
    height: 100%; width: 0%;
    background: linear-gradient(90deg, #00ff88, #00e5ff);
    border-radius: 6px; transition: width 0.1s linear;
  }

  #fpsDisplay {
    position: absolute; top: 70px; right: 15px;
    font-size: 14px; font-weight: bold;
    color: #00ff88; text-shadow: 0 0 5px #000;
    pointer-events: none; z-index: 101;
  }

  .section {
    width: 100%; max-width: 600px; margin-bottom: 20px;
  }
  .section h3 {
    font-size: 18px; margin-bottom: 10px; color: #00e5ff;
    text-align: center; text-transform: uppercase; letter-spacing: 2px;
  }
  .cube-grid {
    display: grid; grid-template-columns: repeat(auto-fill, minmax(70px, 1fr));
    gap: 12px; justify-items: center;
  }
  .cube-item {
    width: 70px; height: 70px; border-radius: 12px;
    border: 3px solid rgba(255,255,255,0.2);
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; transition: all 0.2s;
    background: rgba(0,0,0,0.4);
    position: relative;
  }
  .cube-item.selected {
    border-color: #00e5ff;
    box-shadow: 0 0 20px #00e5ff, inset 0 0 15px rgba(0,229,255,0.3);
    transform: scale(1.08);
  }
  .cube-item:active { transform: scale(0.95); }
  .cube-item.locked {
    filter: grayscale(0.7) brightness(0.5);
    cursor: pointer;
  }
  .cube-item.locked::after {
    content: '🔒';
    position: absolute;
    font-size: 22px;
    color: #fff;
    text-shadow: 0 0 5px #000;
    z-index: 2;
  }
  .cube-item .cube-price {
    position: absolute;
    bottom: -8px; left: 50%;
    transform: translateX(-50%);
    background: #ffcc00;
    color: #000;
    font-size: 11px; font-weight: bold;
    padding: 2px 8px;
    border-radius: 10px;
    white-space: nowrap;
    box-shadow: 0 2px 6px rgba(0,0,0,0.5);
    z-index: 3;
  }
  .cube-preview {
    width: 44px; height: 44px; border-radius: 6px;
    box-shadow: 0 0 10px rgba(255,255,255,0.3);
  }
  .color-grid {
    display: flex; flex-wrap: wrap; gap: 12px; justify-content: center;
  }
  .color-item {
    width: 55px; height: 55px; border-radius: 50%;
    border: 3px solid rgba(255,255,255,0.2);
    cursor: pointer; transition: all 0.2s;
    position: relative;
    display: flex; align-items: center; justify-content: center;
  }
  .color-item.selected {
    border-color: #fff;
    box-shadow: 0 0 20px currentColor;
    transform: scale(1.12);
  }
  .color-item:active { transform: scale(0.95); }
  .color-item.locked {
    filter: grayscale(0.7) brightness(0.5);
    cursor: not-allowed;
  }
  .color-item.locked::after {
    content: '🔒';
    position: absolute;
    font-size: 22px;
    color: #fff;
    text-shadow: 0 0 5px #000;
  }
  .color-item .price {
    position: absolute;
    bottom: -8px; left: 50%;
    transform: translateX(-50%);
    background: #ffcc00;
    color: #000;
    font-size: 11px; font-weight: bold;
    padding: 2px 8px;
    border-radius: 10px;
    white-space: nowrap;
    box-shadow: 0 2px 6px rgba(0,0,0,0.5);
  }

  #previewBox {
    width: 120px; height: 120px; margin: 0 auto 20px;
    display: flex; align-items: center; justify-content: center;
    background: radial-gradient(circle, rgba(0,229,255,0.15), transparent 70%);
    border-radius: 50%;
  }

  #shopCoins {
    font-size: 28px;
    margin-bottom: 20px;
    color: #ffcc00;
    font-weight: bold;
    text-shadow: 0 0 15px rgba(255,204,0,0.8);
  }
  .shop-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    max-width: 400px;
    padding: 12px 18px;
    margin: 8px 0;
    background: rgba(255,255,255,0.05);
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 12px;
    transition: all 0.2s;
  }
  .shop-item:hover { background: rgba(255,255,255,0.1); }
  .shop-item.owned {
    border-color: #00ff88;
    background: rgba(0,255,136,0.08);
  }
  .shop-item-info { display: flex; align-items: center; gap: 12px; }
  .shop-item-swatch {
    width: 40px; height: 40px; border-radius: 8px;
    border: 2px solid rgba(255,255,255,0.4);
  }
  .shop-item-name { font-size: 16px; font-weight: bold; }
  .shop-item-price { font-size: 16px; font-weight: bold; color: #ffcc00; }
  .shop-item.owned .shop-item-price { color: #00ff88; }

  .setting-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    max-width: 400px;
    padding: 15px 20px;
    margin: 10px 0;
    background: rgba(255,255,255,0.05);
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 12px;
    flex-wrap: wrap; gap: 10px;
  }
  .setting-row label { font-size: 18px; font-weight: bold; }
  .setting-row input[type="range"] {
    width: 180px; accent-color: #00e5ff; cursor: pointer;
  }
  .setting-row input[type="checkbox"] {
    width: 24px; height: 24px; accent-color: #00e5ff; cursor: pointer;
  }
  .setting-value {
    min-width: 50px; text-align: right;
    color: #00e5ff; font-weight: bold;
  }

  .fps-grid {
    display: flex; gap: 8px; flex-wrap: wrap; justify-content: center;
  }
  .fps-option {
    padding: 8px 16px;
    border-radius: 8px;
    background: rgba(255,255,255,0.1);
    border: 2px solid rgba(255,255,255,0.2);
    cursor: pointer; font-weight: bold;
    transition: all 0.2s;
    color: #fff;
  }
  .fps-option.selected {
    background: #00e5ff; color: #000;
    border-color: #fff;
    box-shadow: 0 0 15px rgba(0,229,255,0.6);
  }

  .level-grid {
    display: flex; gap: 20px; flex-wrap: wrap;
    justify-content: center; margin: 20px 0;
  }
  .level-card {
    width: 200px; padding: 20px;
    background: rgba(255,255,255,0.05);
    border: 3px solid rgba(255,255,255,0.2);
    border-radius: 16px;
    cursor: pointer; transition: all 0.2s;
    text-align: center;
  }
  .level-card:hover {
    background: rgba(255,255,255,0.1);
    transform: translateY(-3px);
  }
  .level-card.selected {
    border-color: #00e5ff;
    box-shadow: 0 0 25px rgba(0,229,255,0.5);
  }
  .level-card .level-icon { font-size: 48px; margin-bottom: 10px; }
  .level-card .level-name { font-size: 20px; font-weight: bold; margin-bottom: 8px; }
  .level-card .level-desc { font-size: 14px; opacity: 0.8; }

  .stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 15px;
    max-width: 600px;
    width: 100%;
    margin: 20px 0;
  }
  .stat-card {
    padding: 18px 14px;
    background: linear-gradient(145deg, rgba(255,255,255,0.08), rgba(255,255,255,0.03));
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 14px;
    text-align: center;
  }
  .stat-card .stat-icon { font-size: 30px; margin-bottom: 8px; }
  .stat-card .stat-value {
    font-size: 26px; font-weight: bold;
    color: #00e5ff;
    text-shadow: 0 0 10px rgba(0,229,255,0.5);
  }
  .stat-card .stat-label {
    font-size: 12px; opacity: 0.8;
    margin-top: 4px;
    text-transform: uppercase;
    letter-spacing: 1px;
  }
  .stat-card.gold .stat-value { color: #ffcc00; text-shadow: 0 0 10px rgba(255,204,0,0.5); }
  .stat-card.green .stat-value { color: #00ff88; text-shadow: 0 0 10px rgba(0,255,136,0.5); }
  .stat-card.red .stat-value { color: #ff4466; text-shadow: 0 0 10px rgba(255,68,102,0.5); }
  .stat-card.purple .stat-value { color: #cc66ff; text-shadow: 0 0 10px rgba(204,102,255,0.5); }

  .coin-icon {
    display: inline-block;
    width: 22px; height: 22px;
    background: radial-gradient(circle at 30% 30%, #ffe066, #ffaa00);
    border-radius: 50%;
    border: 2px solid #cc8800;
    box-shadow: 0 0 8px rgba(255,200,0,0.7);
    vertical-align: middle;
    margin-right: 4px;
  }
</style>
</head>
<body>
<canvas id="game"></canvas>

<!-- ====== ЭКРАН ЗАГРУЗКИ ====== -->
<div id="loading">
  <div id="loadingTitle">ЗАГРУЗКА...</div>
  <div id="loadingBarWrap"><div id="loadingBar"></div></div>
  <div id="loadingPercent">0%</div>
  <div id="loadingHint">Подготовка игры...</div>
</div>

<!-- ====== ЭКРАН ВВОДА НИКА ====== -->
<div id="nicknameScreen" class="hidden">
  <h2>ВВЕДИ СВОЙ НИК</h2>
  <input type="text" id="nicknameInput" maxlength="16" placeholder="Твой ник" autocomplete="off">
  <button id="nicknameBtn">ИГРАТЬ</button>
</div>

<div id="ui">
  <div id="fpsDisplay" class="hidden">FPS: 60</div>

  <div id="hud" class="hidden">
    <div>💎 <span id="attempts">1</span></div>
    <div id="score">0%</div>
  </div>
  <div id="progress" class="hidden"><div id="progressBar"></div></div>

  <!-- ====== ГЛАВНОЕ МЕНЮ ====== -->
  <div id="menu" class="hidden">
    <div id="menuInner">
      <div class="topRightInfo">
        <div class="infoPill nick">👤 <span id="menuNick">Игрок</span></div>
        <div class="infoPill">🪙 <span id="menuCoinCount">0</span></div>
      </div>

      <div class="menuTitle hidden">GEOMETRY DASH</div>

      <div class="menuCenter">
        <div class="bigIcon" id="skinsBtnBig" title="Настроить куб">
          <span class="iconEmoji">🎨</span>
        </div>
        <div class="bigIcon play" id="startBtnBig" title="Играть">
          <span class="iconEmoji">▶</span>
        </div>
        <div class="bigIcon" id="shopBtnBig" title="Магазин">
          <span class="iconEmoji">🛠️</span>
        </div>
      </div>

      <div class="menuBottom">
        <div class="menuBottomLeft">
          <div class="socialBtn fb" title="Facebook">f</div>
          <div class="socialBtn tw" title="Twitter">🐦</div>
          <div class="socialBtn yt" title="YouTube">▶</div>
          <div class="socialBtn dc" title="Discord">💬</div>
          <div class="socialBtn tg" title="Telegram">✈</div>
        </div>

        <div class="menuBottomCenter">
          <div class="circleBtn" id="levelSelectBtnBig" title="Уровни">🏆</div>
          <div class="circleBtn" id="settingsBtnBig" title="Настройки">⚙️</div>
          <div class="circleBtn" id="statsBtnBig" title="Статистика">📊</div>
          <div class="circleBtn gold" id="soundBtnBig" title="Звук">🔊</div>
        </div>

        <div class="menuBottomRight">
          <div class="moreGames">MORE<br>GAMES</div>
        </div>
      </div>
    </div>
  </div>

  <!-- ====== ВЫБОР УРОВНЯ ====== -->
  <div id="levelSelect" class="hidden">
    <h1>📋 ВЫБОР УРОВНЯ</h1>
    <div class="level-grid">
      <div class="level-card selected" data-level="cube">
        <div class="level-icon">🟦</div>
        <div class="level-name">Куб</div>
        <div class="level-desc">Классический режим. Прыгай через шипы и блоки.</div>
      </div>
      <div class="level-card" data-level="plane">
        <div class="level-icon">✈️</div>
        <div class="level-name">Самолёт</div>
        <div class="level-desc">Лети вперёд! Управляй высотой и уклоняйся.</div>
      </div>
      <div class="level-card" data-level="hybrid">
        <div class="level-icon">🔀</div>
        <div class="level-name">Гибрид</div>
        <div class="level-desc">Куб → Портал → Самолёт. Два режима в одном!</div>
      </div>
    </div>
    <div class="btn-row">
      <button id="levelBackBtn" class="back">← НАЗАД</button>
    </div>
  </div>

  <!-- ====== СКИНЫ / КУБЫ ====== -->
  <div id="skins" class="hidden">
    <h1>МОЙ КУБ</h1>
    <div id="previewBox">
      <canvas id="previewCanvas" width="100" height="100"></canvas>
    </div>
    <div class="section">
      <h3>Форма куба</h3>
      <div class="cube-grid" id="cubeGrid"></div>
    </div>
    <div class="section">
      <h3>Основной цвет</h3>
      <div class="color-grid" id="colorGrid1"></div>
    </div>
    <div class="section">
      <h3>Цвет деталей</h3>
      <div class="color-grid" id="colorGrid2"></div>
    </div>
    <div class="btn-row">
      <button id="backBtn" class="back">← НАЗАД</button>
    </div>
  </div>

  <!-- ====== МАГАЗИН ====== -->
  <div id="shop" class="hidden">
    <h1>🛒 МАГАЗИН</h1>
    <div id="shopCoins">🪙 <span id="shopCoinCount">0</span></div>
    <div id="shopItems"></div>
    <div class="btn-row">
      <button id="shopBackBtn" class="back">← НАЗАД</button>
    </div>
  </div>

  <!-- ====== СТАТИСТИКА ====== -->
  <div id="stats" class="hidden">
    <h1>📊 СТАТИСТИКА</h1>
    <div class="stats-grid">
      <div class="stat-card gold">
        <div class="stat-icon">🪙</div>
        <div class="stat-value" id="statCoins">0</div>
        <div class="stat-label">Монет сейчас</div>
      </div>
      <div class="stat-card green">
        <div class="stat-icon">🏆</div>
        <div class="stat-value" id="statWins">0</div>
        <div class="stat-label">Побед</div>
      </div>
      <div class="stat-card red">
        <div class="stat-icon">💀</div>
        <div class="stat-value" id="statLosses">0</div>
        <div class="stat-label">Проигрышей</div>
      </div>
      <div class="stat-card purple">
        <div class="stat-icon">🛒</div>
        <div class="stat-value" id="statSpent">0</div>
        <div class="stat-label">Потрачено монет</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">🎮</div>
        <div class="stat-value" id="statAttempts">0</div>
        <div class="stat-label">Всего попыток</div>
      </div>
      <div class="stat-card gold">
        <div class="stat-icon">💰</div>
        <div class="stat-value" id="statEarned">0</div>
        <div class="stat-label">Заработано всего</div>
      </div>
    </div>
    <div class="btn-row">
      <button id="statsBackBtn" class="back">← НАЗАД</button>
    </div>
  </div>

  <!-- ====== НАСТРОЙКИ ====== -->
  <div id="settings" class="hidden">
    <h1>⚙️ НАСТРОЙКИ</h1>
    <div class="setting-row">
      <label>🎵 Громкость музыки</label>
      <input type="range" id="musicVolume" min="0" max="100" value="22">
      <span class="setting-value" id="musicVolumeValue">22%</span>
    </div>
    <div class="setting-row">
      <label>📊 Показывать FPS</label>
      <input type="checkbox" id="showFps">
    </div>
    <div class="setting-row" style="flex-direction:column; align-items:flex-start;">
      <label style="margin-bottom:10px;">🎯 Целевой FPS</label>
      <div class="fps-grid" id="fpsGrid">
        <div class="fps-option" data-fps="30">30</div>
        <div class="fps-option" data-fps="90">90</div>
        <div class="fps-option" data-fps="120">120</div>
        <div class="fps-option selected" data-fps="144">144</div>
      </div>
    </div>
    <div class="btn-row">
      <button id="settingsBackBtn" class="back">← НАЗАД</button>
    </div>
  </div>

  <!-- ====== GAME OVER ====== -->
  <div id="gameover" class="hidden">
    <h1>GAME OVER</h1>
    <p>Прогресс: <span id="finalScore">0%</span></p>
    <p>Попытка: <span id="finalAttempts">1</span></p>
    <p id="coinReward" class="hidden" style="color:#ffcc00; font-size:22px; margin-top:10px;">🪙 +1 монета!</p>
    <div class="btn-row">
      <button id="retryBtn">ЗАНОВО</button>
      <button id="menuBtn" class="back">МЕНЮ</button>
    </div>
  </div>
</div>

<script>
'use strict';
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d', { alpha: false });

let W, H, DPR;
function resize() {
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  W = window.innerWidth;
  H = window.innerHeight;
  canvas.width = W * DPR;
  canvas.height = H * DPR;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
}
resize();
window.addEventListener('resize', resize);

// ===== ПОЛНОЭКРАННЫЙ РЕЖИМ =====
function enterFullscreen() {
  const el = document.documentElement;
  if (el.requestFullscreen) el.requestFullscreen().catch(()=>{});
  else if (el.webkitRequestFullscreen) el.webkitRequestFullscreen();
  else if (el.msRequestFullscreen) el.msRequestFullscreen();
}

// =====================================================
// ============ МУЗЫКАЛЬНЫЙ ДВИЖОК =====================
// =====================================================
const Music = {
  ctx: null, masterGain: null, enabled: true, volume: 0.22,
  currentTrack: null, nextNoteTime: 0, currentStep: 0, timerID: null,
  tempo: 130, noiseBuffer: null,

  init() {
    if (this.ctx) return;
    try {
      this.ctx = new (window.AudioContext || window.webkitAudioContext)();
      this.masterGain = this.ctx.createGain();
      this.masterGain.gain.value = this.enabled ? this.volume : 0;
      this.masterGain.connect(this.ctx.destination);
      const bufferSize = this.ctx.sampleRate * 0.05;
      this.noiseBuffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
      const data = this.noiseBuffer.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
    } catch(e) { console.warn('Audio not supported'); }
  },
  resume() { if (this.ctx && this.ctx.state === 'suspended') this.ctx.resume(); },

  playNote(freq, time, duration, type = 'square', gainVal = 0.4, detune = 0) {
    if (!this.ctx || !this.enabled) return;
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.type = type; osc.frequency.value = freq; osc.detune.value = detune;
    gain.gain.setValueAtTime(0, time);
    gain.gain.linearRampToValueAtTime(gainVal, time + 0.01);
    gain.gain.exponentialRampToValueAtTime(0.001, time + duration);
    osc.connect(gain); gain.connect(this.masterGain);
    osc.start(time); osc.stop(time + duration + 0.05);
  },
  playKick(time) {
    if (!this.ctx || !this.enabled) return;
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    osc.frequency.setValueAtTime(150, time);
    osc.frequency.exponentialRampToValueAtTime(45, time + 0.12);
    gain.gain.setValueAtTime(0.7, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.18);
    osc.connect(gain); gain.connect(this.masterGain);
    osc.start(time); osc.stop(time + 0.2);
  },
  playHat(time) {
    if (!this.ctx || !this.enabled || !this.noiseBuffer) return;
    const noise = this.ctx.createBufferSource();
    noise.buffer = this.noiseBuffer;
    const gain = this.ctx.createGain();
    gain.gain.setValueAtTime(0.15, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.05);
    const filter = this.ctx.createBiquadFilter();
    filter.type = 'highpass'; filter.frequency.value = 7000;
    noise.connect(filter); filter.connect(gain); gain.connect(this.masterGain);
    noise.start(time); noise.stop(time + 0.06);
  },
  playBass(freq, time, duration) {
    if (!this.ctx || !this.enabled) return;
    const osc = this.ctx.createOscillator();
    const gain = this.ctx.createGain();
    const filter = this.ctx.createBiquadFilter();
    osc.type = 'sawtooth'; osc.frequency.value = freq;
    filter.type = 'lowpass'; filter.frequency.value = 600; filter.Q.value = 6;
    gain.gain.setValueAtTime(0, time);
    gain.gain.linearRampToValueAtTime(0.35, time + 0.01);
    gain.gain.exponentialRampToValueAtTime(0.001, time + duration);
    osc.connect(filter); filter.connect(gain); gain.connect(this.masterGain);
    osc.start(time); osc.stop(time + duration + 0.05);
  },
  noteToFreq(n) { return 261.63 * Math.pow(2, n / 12); },

  tracks: {
    menu: { tempo: 100,
      lead: [12,null,15,null,19,null,15,null,12,null,15,null,17,null,15,null,10,null,14,null,17,null,14,null,10,null,14,null,15,null,14,null],
      bass: [-12,null,null,null,-5,null,null,null,-12,null,null,null,-5,null,null,null,-14,null,null,null,-7,null,null,null,-14,null,null,null,-7,null,null,null],
      kick: [1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0],
      hat:  [0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1],
      leadType: 'triangle', leadGain: 0.18 },
    level: { tempo: 140,
      lead: [12,12,15,12,19,12,22,19,12,12,15,12,19,22,19,15,10,10,14,10,17,10,22,17,10,10,14,10,17,22,17,14],
      bass: [-12,-12,-12,-12,-5,-5,-5,-5,-12,-12,-12,-12,-5,-5,-5,-5,-14,-14,-14,-14,-7,-7,-7,-7,-14,-14,-14,-14,-7,-7,-7,-7],
      kick: [1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,1,0],
      hat:  [0,0,1,1,0,0,1,1,0,0,1,1,0,0,1,1,0,0,1,1,0,0,1,1,0,0,1,1,0,0,1,1],
      leadType: 'square', leadGain: 0.14 },
    plane: { tempo: 120,
      lead: [19,null,22,null,24,null,22,19,17,null,19,null,22,null,19,17,15,null,17,null,19,null,17,15,14,null,15,null,17,null,15,14],
      bass: [-5,null,-5,null,-7,null,-7,null,-8,null,-8,null,-7,null,-7,null,-10,null,-10,null,-8,null,-8,null,-12,null,-12,null,-10,null,-10,null],
      kick: [1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0,1,0,0,0,0,0,1,0],
      hat:  [0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1],
      leadType: 'triangle', leadGain: 0.16 },
    hybrid: { tempo: 130,
      lead: [12,15,19,22,19,15,12,15,10,14,17,22,17,14,10,14,12,15,19,24,19,15,12,15,10,14,17,22,19,17,14,10],
      bass: [-12,-12,-5,-5,-12,-12,-5,-5,-14,-14,-7,-7,-14,-14,-7,-7,-12,-12,-5,-5,-12,-12,-5,-5,-14,-14,-7,-7,-14,-14,-7,-7],
      kick: [1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,1,0,1,0],
      hat:  [0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1,0,0,1,0,1,0,0,1],
      leadType: 'square', leadGain: 0.15 }
  },

  playTrack(name) {
    this.init(); this.resume();
    if (this.currentTrack === name && this.timerID) return;
    this.stop();
    this.currentTrack = name;
    const track = this.tracks[name];
    if (!track) return;
    this.tempo = track.tempo;
    this.currentStep = 0;
    this.nextNoteTime = this.ctx.currentTime + 0.1;
    this.scheduler(track);
  },
  stop() {
    if (this.timerID) { clearTimeout(this.timerID); this.timerID = null; }
    this.currentTrack = null;
  },
  scheduler(track) {
    const stepDur = 60 / this.tempo / 4;
    while (this.nextNoteTime < this.ctx.currentTime + 0.15) {
      const step = this.currentStep % 32;
      const time = this.nextNoteTime;
      if (track.lead[step] !== null && track.lead[step] !== undefined) {
        this.playNote(this.noteToFreq(track.lead[step]), time, stepDur * 0.9, track.leadType, track.leadGain);
        this.playNote(this.noteToFreq(track.lead[step]), time + 0.005, stepDur * 0.85, track.leadType, track.leadGain * 0.5, 8);
      }
      if (track.bass[step] !== null && track.bass[step] !== undefined) {
        this.playBass(this.noteToFreq(track.bass[step]) * 0.5, time, stepDur * 0.85);
      }
      if (track.kick[step]) this.playKick(time);
      if (track.hat[step]) this.playHat(time);
      this.nextNoteTime += stepDur;
      this.currentStep = (this.currentStep + 1) % 32;
    }
    this.timerID = setTimeout(() => this.scheduler(track), 25);
  },
  setEnabled(on) { this.enabled = on; if (this.masterGain) this.masterGain.gain.value = on ? this.volume : 0; },
  setVolume(v) { this.volume = v; if (this.masterGain) this.masterGain.gain.value = this.enabled ? v : 0; }
};

// =====================================================
// ============== ЗВУКОВЫЕ ЭФФЕКТЫ =====================
// =====================================================
function sfxJump() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  const osc = Music.ctx.createOscillator(); const gain = Music.ctx.createGain();
  osc.type = 'square'; osc.frequency.setValueAtTime(400, t);
  osc.frequency.exponentialRampToValueAtTime(800, t + 0.08);
  gain.gain.setValueAtTime(0.18, t); gain.gain.exponentialRampToValueAtTime(0.001, t + 0.12);
  osc.connect(gain); gain.connect(Music.masterGain); osc.start(t); osc.stop(t + 0.14);
}
function sfxDeath() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  const osc = Music.ctx.createOscillator(); const gain = Music.ctx.createGain();
  osc.type = 'sawtooth'; osc.frequency.setValueAtTime(400, t);
  osc.frequency.exponentialRampToValueAtTime(50, t + 0.5);
  gain.gain.setValueAtTime(0.3, t); gain.gain.exponentialRampToValueAtTime(0.001, t + 0.55);
  osc.connect(gain); gain.connect(Music.masterGain); osc.start(t); osc.stop(t + 0.6);
}
function sfxWin() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  [0, 4, 7, 12].forEach((n, i) => Music.playNote(Music.noteToFreq(12 + n), t + i * 0.12, 0.25, 'square', 0.22));
}
function sfxCoin() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  Music.playNote(Music.noteToFreq(19), t, 0.1, 'square', 0.25);
  Music.playNote(Music.noteToFreq(24), t + 0.08, 0.2, 'square', 0.25);
}
function sfxBuy() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  Music.playNote(Music.noteToFreq(12), t, 0.1, 'triangle', 0.3);
  Music.playNote(Music.noteToFreq(16), t + 0.07, 0.1, 'triangle', 0.3);
  Music.playNote(Music.noteToFreq(19), t + 0.14, 0.2, 'triangle', 0.3);
}
function sfxPortal() {
  if (!Music.ctx || !Music.enabled) return;
  const t = Music.ctx.currentTime;
  Music.playNote(Music.noteToFreq(7), t, 0.15, 'sine', 0.3);
  Music.playNote(Music.noteToFreq(12), t + 0.1, 0.2, 'sine', 0.3);
  Music.playNote(Music.noteToFreq(19), t + 0.2, 0.3, 'sine', 0.3);
}

// =====================================================
// ==================== ДАННЫЕ =========================
// =====================================================
const CUBE_STYLES = [
  { id: 'classic', name: 'Классик', price: 0 },
  { id: 'round', name: 'Скругленный', price: 0 },
  { id: 'star', name: 'Звезда', price: 0 },
  { id: 'cross', name: 'Крест', price: 0 },
  { id: 'robot', name: 'Робот', price: 0 },
  { id: 'ninja', name: 'Ниндзя', price: 0 },
  { id: 'hollow', name: 'Пустой', price: 0 },
  { id: 'gradient', name: 'Градиент', price: 0 },
  { id: 'diamond', name: 'Алмаз', price: 0 },
  { id: 'hexagon', name: 'Гексагон', price: 0 },
  { id: 'skull', name: 'Череп', price: 0 },
  { id: 'alien', name: 'Пришелец', price: 0 },
  { id: 'crown', name: 'Корона', price: 0 },
  { id: 'heart', name: 'Сердце', price: 0 },
  { id: 'lightning', name: 'Молния', price: 0 },
  { id: 'target', name: 'Мишень', price: 0 },
  { id: 'phantom', name: 'Фантом', price: 3 },
  { id: 'cyber', name: 'Кибер', price: 3 },
  { id: 'magma', name: 'Магма', price: 3 },
  { id: 'frost', name: 'Лёд', price: 3 },
  { id: 'void', name: 'Пустота', price: 10 },
  { id: 'pixel', name: 'Пиксель', price: 1 },
  // НОВЫЙ КУБ С МАШИНОЙ
  { id: 'car', name: 'Машина', price: 7 },
  { id: 'winner', name: 'Победитель', price: 0, unlockByWin: true },
  { id: 'champion', name: 'Чемпион', price: 0, unlockByWin: true },
  { id: 'legend', name: 'Легенда', price: 0, unlockByWin: true },
  { id: 'hero', name: 'Герой', price: 0, unlockByWin: true }
];

const BASE_COLORS = [
  '#00e5ff', '#0088ff', '#8a00ff', '#ff00e5',
  '#ff0044', '#ff6600', '#ffcc00', '#00ff88'
];

// +10 НОВЫХ ЦВЕТОВ (всего 7 старых премиум + 10 новых = 17 платных)
const PREMIUM_COLORS = [
  { color: '#ff0000', name: 'Красный', price: 5 },
  { color: '#000000', name: 'Чёрный', price: 5 },
  { color: '#ffffff', name: 'Белый', price: 5 },
  { color: '#0033ff', name: 'Синий', price: 5 },
  { color: '#8000ff', name: 'Фиолетовый', price: 5 },
  { color: '#00ff00', name: 'Ядовитый', price: 2 },
  { color: '#ff69b4', name: 'Розовый', price: 8 },
  // 10 НОВЫХ ЦВЕТОВ
  { color: '#00ffff', name: 'Аква', price: 4 },
  { color: '#ffaa00', name: 'Оранж', price: 4 },
  { color: '#66ff00', name: 'Лайм', price: 4 },
  { color: '#ff0066', name: 'Малина', price: 6 },
  { color: '#6600ff', name: 'Индиго', price: 6 },
  { color: '#aaaaaa', name: 'Серый', price: 3 },
  { color: '#00ffcc', name: 'Бирюза', price: 5 },
  { color: '#ccff00', name: 'Кислота', price: 7 },
  { color: '#ff3300', name: 'Алый', price: 7 },
  { color: '#330066', name: 'Тёмный', price: 9 }
];

const COLORS = BASE_COLORS.concat(PREMIUM_COLORS.map(p => p.color));

let playerData = {
  coins: 0, ownedColors: [], ownedStyles: [],
  wins: 0, losses: 0, totalSpent: 0, totalEarned: 0,
  totalAttempts: 0, hasWonAny: false, nickname: ''
};

function loadPlayerData() {
  try {
    const saved = JSON.parse(localStorage.getItem('gd_player') || 'null');
    if (saved) playerData = { ...playerData, ...saved };
  } catch(e) {}
  if (!Array.isArray(playerData.ownedColors)) playerData.ownedColors = [];
  if (!Array.isArray(playerData.ownedStyles)) playerData.ownedStyles = [];
  if (typeof playerData.coins !== 'number') playerData.coins = 0;
  if (typeof playerData.wins !== 'number') playerData.wins = 0;
  if (typeof playerData.losses !== 'number') playerData.losses = 0;
  if (typeof playerData.totalSpent !== 'number') playerData.totalSpent = 0;
  if (typeof playerData.totalEarned !== 'number') playerData.totalEarned = 0;
  if (typeof playerData.totalAttempts !== 'number') playerData.totalAttempts = 0;
  if (typeof playerData.hasWonAny !== 'boolean') playerData.hasWonAny = false;
  if (typeof playerData.nickname !== 'string') playerData.nickname = '';
}
function savePlayerData() { localStorage.setItem('gd_player', JSON.stringify(playerData)); }
function isColorOwned(color) { if (BASE_COLORS.includes(color)) return true; return playerData.ownedColors.includes(color); }
function getColorInfo(color) { return PREMIUM_COLORS.find(p => p.color === color) || null; }
function isStyleOwned(styleId) {
  const style = CUBE_STYLES.find(s => s.id === styleId);
  if (!style) return false;
  if (style.unlockByWin) return playerData.hasWonAny;
  if (style.price === 0) return true;
  return playerData.ownedStyles.includes(styleId);
}
function isStyleLockedByWin(styleId) {
  const style = CUBE_STYLES.find(s => s.id === styleId);
  return style && style.unlockByWin && !playerData.hasWonAny;
}

let currentSkin = { style: 'classic', color1: '#00e5ff', color2: '#ff00e5' };
function loadSkin() {
  try {
    const saved = JSON.parse(localStorage.getItem('gd_skin') || 'null');
    if (saved) currentSkin = { ...currentSkin, ...saved };
  } catch(e) {}
  if (!isColorOwned(currentSkin.color1)) currentSkin.color1 = '#00e5ff';
  if (!isColorOwned(currentSkin.color2)) currentSkin.color2 = '#ff00e5';
  if (!isStyleOwned(currentSkin.style)) currentSkin.style = 'classic';
}
function saveSkin() { localStorage.setItem('gd_skin', JSON.stringify(currentSkin)); }
function updateCoinUI() {
  document.getElementById('menuCoinCount').textContent = playerData.coins;
  const shopCount = document.getElementById('shopCoinCount');
  if (shopCount) shopCount.textContent = playerData.coins;
}
function updateNickUI() {
  const el = document.getElementById('menuNick');
  if (el) el.textContent = playerData.nickname || 'Игрок';
}

// =====================================================
// ==================== НАСТРОЙКИ ======================
// =====================================================
let settings = { volume: 22, showFps: false, targetFps: 144 };
function loadSettings() {
  try {
    const saved = JSON.parse(localStorage.getItem('gd_settings') || 'null');
    if (saved) settings = { ...settings, ...saved };
  } catch(e) {}
  Music.volume = settings.volume / 100;
  document.getElementById('musicVolume').value = settings.volume;
  document.getElementById('musicVolumeValue').textContent = settings.volume + '%';
  document.getElementById('showFps').checked = settings.showFps;
  updateFpsVisibility(); updateFpsGridUI();
}
function saveSettings() { localStorage.setItem('gd_settings', JSON.stringify(settings)); }
function updateFpsVisibility() {
  const el = document.getElementById('fpsDisplay');
  if (settings.showFps) el.classList.remove('hidden'); else el.classList.add('hidden');
}
function updateFpsGridUI() {
  document.querySelectorAll('#fpsGrid .fps-option').forEach(el => {
    el.classList.toggle('selected', parseInt(el.dataset.fps) === settings.targetFps);
  });
}

document.getElementById('musicVolume').addEventListener('input', (e) => {
  const val = parseInt(e.target.value);
  document.getElementById('musicVolumeValue').textContent = val + '%';
  Music.setVolume(val / 100); settings.volume = val; saveSettings();
});
document.getElementById('showFps').addEventListener('change', (e) => {
  settings.showFps = e.target.checked; updateFpsVisibility(); saveSettings();
});
document.querySelectorAll('#fpsGrid .fps-option').forEach(el => {
  el.addEventListener('click', () => {
    settings.targetFps = parseInt(el.dataset.fps); updateFpsGridUI(); saveSettings();
  });
});

// =====================================================
// ==================== РИСОВАНИЕ КУБА =================
// =====================================================
function drawCube(g, x, y, size, skin, rotation = 0, glow = true) {
  g.save(); g.translate(x, y); g.rotate(rotation);
  const half = size / 2;
  if (glow) { g.shadowColor = skin.color1; g.shadowBlur = 18; }

  if (skin.style === 'round') {
    g.fillStyle = skin.color1; roundRect(g, -half, -half, size, size, size * 0.28); g.fill();
    g.shadowBlur = 0; g.fillStyle = skin.color2;
    roundRect(g, -half + 5, -half + 5, size - 10, size - 10, size * 0.22); g.fill();
  } else if (skin.style === 'star') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size);
    g.shadowBlur = 0; g.fillStyle = skin.color2;
    drawStar(g, 0, 0, 5, half * 0.75, half * 0.35); g.fill();
  } else if (skin.style === 'cross') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size);
    g.shadowBlur = 0; g.fillStyle = skin.color2;
    const t = size * 0.22;
    g.fillRect(-t/2, -half + 4, t, size - 8); g.fillRect(-half + 4, -t/2, size - 8, t);
  } else if (skin.style === 'robot') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, skin.color1); grad.addColorStop(1, skin.color2);
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#000';
    g.fillRect(-half + 6, -half + 8, size * 0.28, size * 0.22);
    g.fillRect(half - 6 - size * 0.28, -half + 8, size * 0.28, size * 0.22);
    g.fillRect(-size * 0.15, half * 0.15, size * 0.3, size * 0.12);
  } else if (skin.style === 'ninja') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = skin.color2; g.fillRect(-half, -half + size * 0.28, size, size * 0.32);
    g.fillStyle = '#fff';
    g.fillRect(-half + 6, -half + size * 0.34, size * 0.22, size * 0.14);
    g.fillRect(half - 6 - size * 0.22, -half + size * 0.34, size * 0.22, size * 0.14);
  } else if (skin.style === 'hollow') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#000'; g.fillRect(-half + size * 0.18, -half + size * 0.18, size * 0.64, size * 0.64);
    g.fillStyle = skin.color2; g.fillRect(-half + size * 0.3, -half + size * 0.3, size * 0.4, size * 0.4);
  } else if (skin.style === 'gradient') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, skin.color1); grad.addColorStop(1, skin.color2);
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.strokeStyle = '#fff'; g.lineWidth = 2; g.strokeRect(-half, -half, size, size);
  } else if (skin.style === 'diamond') {
    g.fillStyle = skin.color1;
    g.beginPath(); g.moveTo(0, -half); g.lineTo(half, 0); g.lineTo(0, half); g.lineTo(-half, 0); g.closePath(); g.fill();
    g.shadowBlur = 0; g.fillStyle = skin.color2;
    g.beginPath(); g.moveTo(0, -half * 0.55); g.lineTo(half * 0.55, 0); g.lineTo(0, half * 0.55); g.lineTo(-half * 0.55, 0); g.closePath(); g.fill();
  } else if (skin.style === 'hexagon') {
    g.fillStyle = skin.color1; drawPolygon(g, 0, 0, half, 6); g.fill();
    g.shadowBlur = 0; g.fillStyle = skin.color2; drawPolygon(g, 0, 0, half * 0.55, 6); g.fill();
  } else if (skin.style === 'skull') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#000';
    g.beginPath(); g.arc(-half * 0.4, -half * 0.1, half * 0.22, 0, Math.PI * 2); g.arc(half * 0.4, -half * 0.1, half * 0.22, 0, Math.PI * 2); g.fill();
    g.beginPath(); g.moveTo(0, half * 0.1); g.lineTo(-half * 0.12, half * 0.3); g.lineTo(half * 0.12, half * 0.3); g.closePath(); g.fill();
    g.fillStyle = skin.color2;
    for (let i = -2; i <= 2; i++) g.fillRect(i * size * 0.13 - size * 0.04, half * 0.55, size * 0.08, size * 0.15);
  } else if (skin.style === 'alien') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, skin.color1); grad.addColorStop(1, skin.color2);
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#000';
    g.beginPath(); g.ellipse(-half * 0.35, -half * 0.15, half * 0.28, half * 0.38, -0.3, 0, Math.PI * 2); g.fill();
    g.beginPath(); g.ellipse(half * 0.35, -half * 0.15, half * 0.28, half * 0.38, 0.3, 0, Math.PI * 2); g.fill();
    g.fillStyle = '#fff';
    g.beginPath(); g.arc(-half * 0.3, -half * 0.28, half * 0.08, 0, Math.PI * 2); g.arc(half * 0.4, -half * 0.28, half * 0.08, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'crown') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = skin.color2;
    g.beginPath();
    g.moveTo(-half * 0.75, -half * 0.15); g.lineTo(-half * 0.75, -half * 0.6); g.lineTo(-half * 0.4, -half * 0.3);
    g.lineTo(0, -half * 0.7); g.lineTo(half * 0.4, -half * 0.3); g.lineTo(half * 0.75, -half * 0.6);
    g.lineTo(half * 0.75, -half * 0.15); g.closePath(); g.fill();
    g.fillStyle = '#fff'; g.beginPath(); g.arc(0, -half * 0.3, half * 0.12, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'heart') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = skin.color2;
    g.beginPath();
    const s = size * 0.5;
    g.moveTo(0, s * 0.5);
    g.bezierCurveTo(-s * 1.2, -s * 0.2, -s * 0.5, -s * 0.9, 0, -s * 0.4);
    g.bezierCurveTo(s * 0.5, -s * 0.9, s * 1.2, -s * 0.2, 0, s * 0.5);
    g.fill();
  } else if (skin.style === 'lightning') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = skin.color2;
    g.beginPath();
    g.moveTo(half * 0.3, -half * 0.8); g.lineTo(-half * 0.3, half * 0.1); g.lineTo(0, half * 0.1);
    g.lineTo(-half * 0.2, half * 0.85); g.lineTo(half * 0.4, -half * 0.05); g.lineTo(half * 0.05, -half * 0.05);
    g.closePath(); g.fill();
  } else if (skin.style === 'target') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = skin.color2; g.beginPath(); g.arc(0, 0, half * 0.75, 0, Math.PI * 2); g.fill();
    g.fillStyle = skin.color1; g.beginPath(); g.arc(0, 0, half * 0.45, 0, Math.PI * 2); g.fill();
    g.fillStyle = skin.color2; g.beginPath(); g.arc(0, 0, half * 0.18, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'phantom') {
    g.globalAlpha = 0.75; g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size);
    g.globalAlpha = 1; g.shadowBlur = 0;
    g.fillStyle = skin.color2; g.fillRect(-half + 4, -half + 4, size - 8, size - 8);
    g.fillStyle = '#fff';
    g.beginPath(); g.arc(-half * 0.35, -half * 0.1, half * 0.18, 0, Math.PI * 2); g.arc(half * 0.35, -half * 0.1, half * 0.18, 0, Math.PI * 2); g.fill();
    g.fillStyle = '#000';
    g.beginPath(); g.arc(-half * 0.35, -half * 0.1, half * 0.09, 0, Math.PI * 2); g.arc(half * 0.35, -half * 0.1, half * 0.09, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'cyber') {
    g.fillStyle = '#0a0a1a'; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.strokeStyle = skin.color1; g.lineWidth = 3; g.shadowColor = skin.color1; g.shadowBlur = 12;
    g.strokeRect(-half + 3, -half + 3, size - 6, size - 6); g.shadowBlur = 0;
    g.fillStyle = skin.color2; g.fillRect(-half + 7, -half + 7, size - 14, size - 14);
    g.fillStyle = skin.color1;
    g.fillRect(-half + 11, -half + 11, size - 22, 4);
    g.fillRect(-half + 11, half - 15, size - 22, 4);
  } else if (skin.style === 'magma') {
    const grad = g.createRadialGradient(0, 0, 0, 0, 0, half);
    grad.addColorStop(0, '#ff6600'); grad.addColorStop(0.5, '#ff0044'); grad.addColorStop(1, '#330000');
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#ffcc00';
    g.beginPath(); g.arc(-half * 0.3, -half * 0.2, half * 0.15, 0, Math.PI * 2); g.arc(half * 0.35, half * 0.25, half * 0.1, 0, Math.PI * 2); g.arc(-half * 0.1, half * 0.4, half * 0.08, 0, Math.PI * 2); g.fill();
    g.fillStyle = skin.color1; g.beginPath(); g.arc(0, 0, half * 0.2, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'frost') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, '#aaddff'); grad.addColorStop(0.5, '#00e5ff'); grad.addColorStop(1, '#0088ff');
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = 'rgba(255,255,255,0.6)';
    g.beginPath(); g.moveTo(-half, -half); g.lineTo(0, -half); g.lineTo(-half * 0.3, 0); g.closePath(); g.fill();
    g.fillStyle = skin.color2;
    g.beginPath(); g.moveTo(half * 0.3, -half * 0.5); g.lineTo(half * 0.7, -half * 0.2); g.lineTo(half * 0.5, half * 0.3); g.closePath(); g.fill();
  } else if (skin.style === 'void') {
    g.fillStyle = '#000'; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.strokeStyle = '#8a00ff'; g.lineWidth = 3;
    g.shadowColor = '#8a00ff'; g.shadowBlur = 15;
    g.strokeRect(-half + 2, -half + 2, size - 4, size - 4);
    g.shadowBlur = 0;
    const rg = g.createRadialGradient(0, 0, 0, 0, 0, half * 0.7);
    rg.addColorStop(0, '#ffffff'); rg.addColorStop(0.4, '#cc66ff'); rg.addColorStop(1, 'rgba(138,0,255,0)');
    g.fillStyle = rg; g.beginPath(); g.arc(0, 0, half * 0.7, 0, Math.PI * 2); g.fill();
    g.fillStyle = '#000'; g.beginPath(); g.arc(0, 0, half * 0.28, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'pixel') {
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    const px = size / 8;
    g.fillStyle = skin.color2;
    g.fillRect(-half + px * 2, -half + px * 2, px * 2, px * 2);
    g.fillRect(half - px * 4, -half + px * 2, px * 2, px * 2);
    g.fillStyle = '#000';
    g.fillRect(-half + px * 2.5, -half + px * 2.5, px, px);
    g.fillRect(half - px * 3.5, -half + px * 2.5, px, px);
    g.fillStyle = '#000';
    g.fillRect(-half + px * 2, half - px * 3, px * 4, px);
  } else if (skin.style === 'car') {
    // НОВЫЙ КУБ С МАШИНОЙ
    g.fillStyle = skin.color1; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    // Дорога
    g.fillStyle = '#222'; g.fillRect(-half, half * 0.55, size, half * 0.45);
    g.fillStyle = '#ffcc00';
    for (let i = 0; i < 3; i++) g.fillRect(-half + i * size * 0.4 + size * 0.1, half * 0.75, size * 0.1, size * 0.05);
    // Кузов машины
    g.fillStyle = skin.color2;
    g.beginPath();
    g.moveTo(-half * 0.8, half * 0.5); g.lineTo(-half * 0.6, half * 0.05); g.lineTo(half * 0.5, half * 0.05);
    g.lineTo(half * 0.8, half * 0.5); g.closePath(); g.fill();
    // Крыша
    g.fillStyle = skin.color1;
    g.beginPath();
    g.moveTo(-half * 0.45, half * 0.05); g.lineTo(-half * 0.2, -half * 0.3); g.lineTo(half * 0.35, -half * 0.3);
    g.lineTo(half * 0.5, half * 0.05); g.closePath(); g.fill();
    // Стёкла
    g.fillStyle = '#88ddff';
    g.beginPath();
    g.moveTo(-half * 0.35, 0); g.lineTo(-half * 0.15, -half * 0.22); g.lineTo(half * 0.3, -half * 0.22);
    g.lineTo(half * 0.4, 0); g.closePath(); g.fill();
    // Колёса
    g.fillStyle = '#111';
    g.beginPath(); g.arc(-half * 0.45, half * 0.5, half * 0.2, 0, Math.PI * 2); g.fill();
    g.beginPath(); g.arc(half * 0.5, half * 0.5, half * 0.2, 0, Math.PI * 2); g.fill();
    g.fillStyle = '#888';
    g.beginPath(); g.arc(-half * 0.45, half * 0.5, half * 0.08, 0, Math.PI * 2); g.fill();
    g.beginPath(); g.arc(half * 0.5, half * 0.5, half * 0.08, 0, Math.PI * 2); g.fill();
    // Фара
    g.fillStyle = '#ffff88';
    g.fillRect(half * 0.65, half * 0.2, half * 0.15, half * 0.12);
  } else if (skin.style === 'winner') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, '#ffe066'); grad.addColorStop(1, '#ff8800');
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#fff';
    g.beginPath();
    g.moveTo(-half * 0.6, -half * 0.1); g.lineTo(-half * 0.6, -half * 0.55); g.lineTo(-half * 0.25, -half * 0.25);
    g.lineTo(0, -half * 0.65); g.lineTo(half * 0.25, -half * 0.25); g.lineTo(half * 0.6, -half * 0.55);
    g.lineTo(half * 0.6, -half * 0.1); g.closePath(); g.fill();
    g.fillStyle = '#ff0044';
    g.beginPath(); g.arc(0, half * 0.25, half * 0.15, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'champion') {
    g.fillStyle = '#cc0000'; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#ffcc00';
    g.fillRect(-half, -half, size, size * 0.15);
    g.fillRect(-half, half - size * 0.15, size, size * 0.15);
    g.fillStyle = '#fff';
    g.beginPath(); g.arc(-half * 0.35, 0, half * 0.15, 0, Math.PI * 2); g.arc(half * 0.35, 0, half * 0.15, 0, Math.PI * 2); g.fill();
  } else if (skin.style === 'legend') {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, '#ff00e5'); grad.addColorStop(1, '#00e5ff');
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#fff';
    for (let i = 0; i < 5; i++) {
      const ang = (i / 5) * Math.PI * 2;
      const sx = Math.cos(ang) * half * 0.5;
      const sy = Math.sin(ang) * half * 0.5;
      g.beginPath(); g.arc(sx, sy, half * 0.08, 0, Math.PI * 2); g.fill();
    }
    g.beginPath();
    drawStar(g, 0, 0, 5, half * 0.35, half * 0.15); g.fill();
  } else if (skin.style === 'hero') {
    g.fillStyle = '#0033ff'; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#ffcc00';
    g.beginPath();
    g.moveTo(half * 0.2, -half * 0.7); g.lineTo(-half * 0.2, half * 0.1); g.lineTo(half * 0.05, half * 0.1);
    g.lineTo(-half * 0.15, half * 0.75); g.lineTo(half * 0.3, -half * 0.05); g.lineTo(half * 0.05, -half * 0.05);
    g.closePath(); g.fill();
    g.fillStyle = '#ff0044';
    g.beginPath();
    g.moveTo(-half * 0.6, -half * 0.4); g.lineTo(-half * 0.2, -half * 0.4); g.lineTo(-half * 0.4, half * 0.3); g.closePath(); g.fill();
  } else {
    const grad = g.createLinearGradient(-half, -half, half, half);
    grad.addColorStop(0, skin.color1); grad.addColorStop(1, skin.color2);
    g.fillStyle = grad; g.fillRect(-half, -half, size, size); g.shadowBlur = 0;
    g.fillStyle = '#000';
    g.fillRect(-size * 0.22, -size * 0.22, size * 0.16, size * 0.16);
    g.fillRect(size * 0.06, -size * 0.22, size * 0.16, size * 0.16);
    g.fillRect(-size * 0.22, size * 0.08, size * 0.44, size * 0.12);
  }

  g.shadowBlur = 0;
  g.strokeStyle = 'rgba(255,255,255,0.85)'; g.lineWidth = 2;
  if (skin.style === 'round') { roundRect(g, -half, -half, size, size, size * 0.28); g.stroke(); }
  else if (skin.style === 'diamond') {
    g.beginPath(); g.moveTo(0, -half); g.lineTo(half, 0); g.lineTo(0, half); g.lineTo(-half, 0); g.closePath(); g.stroke();
  } else if (skin.style === 'hexagon') { drawPolygon(g, 0, 0, half, 6); g.stroke(); }
  else if (['heart','lightning','cyber','void','hero','car'].includes(skin.style)) { /* без обводки */ }
  else { g.strokeRect(-half, -half, size, size); }

  g.restore();
}

function roundRect(g, x, y, w, h, r) {
  g.beginPath();
  g.moveTo(x + r, y);
  g.arcTo(x + w, y, x + w, y + h, r);
  g.arcTo(x + w, y + h, x, y + h, r);
  g.arcTo(x, y + h, x, y, r);
  g.arcTo(x, y, x + w, y, r);
  g.closePath();
}
function drawStar(g, cx, cy, spikes, outerR, innerR) {
  let rot = Math.PI / 2 * 3;
  const step = Math.PI / spikes;
  g.beginPath(); g.moveTo(cx, cy - outerR);
  for (let i = 0; i < spikes; i++) {
    g.lineTo(cx + Math.cos(rot) * outerR, cy + Math.sin(rot) * outerR);
    rot += step;
    g.lineTo(cx + Math.cos(rot) * innerR, cy + Math.sin(rot) * innerR);
    rot += step;
  }
  g.closePath();
}
function drawPolygon(g, cx, cy, r, sides) {
  g.beginPath();
  for (let i = 0; i < sides; i++) {
    const angle = (Math.PI * 2 * i / sides) - Math.PI / 2;
    const x = cx + Math.cos(angle) * r;
    const y = cy + Math.sin(angle) * r;
    if (i === 0) g.moveTo(x, y); else g.lineTo(x, y);
  }
  g.closePath();
}

// =====================================================
// ==================== ИГРА ===========================
// =====================================================
const GROUND_Y_RATIO = 0.78;
const PLAYER_SIZE = 38;
const GRAVITY = 0.75;
const JUMP_VELOCITY = -14.5;
const SPEED = 7;
const LEVEL_LENGTH = 8000;
const PLANE_LEVEL_LENGTH = 6000;
const HYBRID_CUBE_LENGTH = 3500;
const HYBRID_TOTAL_LENGTH = 8000;

let player, obstacles, particles, state, distance, attempts, cameraShake, bgOffset, groundOffset;
let currentLevel = 'cube';
let planeStars = [];
let mode = 'cube'; // 'cube' | 'plane' — текущий режим в гибриде
let portals = [];
let bgStars = [];

function generateCubeLevel(startX, endX) {
  let x = startX || 800;
  const end = endX || LEVEL_LENGTH;
  while (x < end) {
    const type = Math.random();
    const groundY = H * GROUND_Y_RATIO;
    if (type < 0.5) {
      const count = Math.random() < 0.3 ? (Math.random() < 0.5 ? 2 : 3) : 1;
      for (let i = 0; i < count; i++) obstacles.push({ x: x + i * 30, y: groundY - 38, w: 30, h: 38, type: 'spike' });
      x += count * 30 + 260 + Math.random() * 200;
    } else if (type < 0.8) {
      const blocks = Math.floor(Math.random() * 2) + 1;
      for (let i = 0; i < blocks; i++) obstacles.push({ x: x, y: groundY - 45 - i * 45, w: 45, h: 45, type: 'block' });
      x += 45 + 300 + Math.random() * 200;
    } else {
      obstacles.push({ x: x, y: groundY - 130, w: 40, h: 40, type: 'block' });
      x += 40 + 280 + Math.random() * 150;
    }
  }
}
function generatePlaneLevel(startX, endX) {
  let x = startX || 1200;
  const end = endX || PLANE_LEVEL_LENGTH;
  const minY = H * 0.15;
  const maxY = H * 0.65;
  while (x < end) {
    const type = Math.random();
    if (type < 0.5) {
      const fromTop = Math.random() < 0.5;
      const h = 100 + Math.random() * 150;
      if (fromTop) obstacles.push({ x: x, y: 0, w: 55, h: h, type: 'wall' });
      else obstacles.push({ x: x, y: H - h, w: 55, h: h, type: 'wall' });
      x += 55 + 500 + Math.random() * 200;
    } else if (type < 0.8) {
      const gapCenter = minY + Math.random() * (maxY - minY);
      const gapSize = 220 + Math.random() * 100;
      const topH = gapCenter - gapSize / 2;
      const botH = H - gapCenter - gapSize / 2;
      if (topH > 30) obstacles.push({ x: x, y: 0, w: 55, h: topH, type: 'wall' });
      if (botH > 30) obstacles.push({ x: x, y: H - botH, w: 55, h: botH, type: 'wall' });
      x += 55 + 550 + Math.random() * 200;
    } else {
      const y = minY + Math.random() * (maxY - minY);
      obstacles.push({ x: x, y: y, w: 60, h: 60, type: 'floating' });
      x += 60 + 480 + Math.random() * 200;
    }
  }
}

function init() {
  const groundY = H * GROUND_Y_RATIO;
  player = { x: 150, y: groundY - PLAYER_SIZE, vy: 0, onGround: true, rotation: 0 };
  obstacles = [];
  portals = [];
  mode = 'cube';

  if (currentLevel === 'cube') {
    generateCubeLevel();
  } else if (currentLevel === 'plane') {
    generatePlaneLevel();
    mode = 'plane';
  } else if (currentLevel === 'hybrid') {
    generateCubeLevel(800, HYBRID_CUBE_LENGTH);
    // Портал в конце кубовой части
    portals.push({
      x: HYBRID_CUBE_LENGTH + 200,
      y: 0, w: 60, h: H, type: 'cube-to-plane',
      used: false
    });
    generatePlaneLevel(HYBRID_CUBE_LENGTH + 400, HYBRID_TOTAL_LENGTH);
  }

  if (currentLevel !== 'cube') {
    planeStars = [];
    for (let i = 0; i < 60; i++) {
      planeStars.push({ x: Math.random() * W, y: Math.random() * H, size: 1 + Math.random() * 2, speed: 0.5 + Math.random() * 1.5 });
    }
  }
  bgStars = [];
  for (let i = 0; i < 80; i++) {
    bgStars.push({ x: Math.random() * W, y: Math.random() * H * 0.7, size: 1 + Math.random() * 2, alpha: 0.2 + Math.random() * 0.6 });
  }

  particles = [];
  distance = 0;
  cameraShake = 0;
  bgOffset = 0;
  groundOffset = 0;
  attempts = parseInt(localStorage.getItem('gd_attempts') || '1');
  updateHUD();
}

function jump() {
  if (state !== 'playing') return;
  if (mode === 'cube') {
    if (player.onGround) {
      player.vy = JUMP_VELOCITY;
      player.onGround = false;
      sfxJump();
      for (let i = 0; i < 6; i++) {
        particles.push({
          x: player.x + PLAYER_SIZE/2, y: player.y + PLAYER_SIZE,
          vx: (Math.random() - 0.5) * 4, vy: Math.random() * 2,
          life: 1, color: currentSkin.color1
        });
      }
    }
  }
}

window.addEventListener('keydown', e => {
  if (e.code === 'Space' || e.code === 'ArrowUp') {
    e.preventDefault();
    if (state === 'menu') startGame();
    else if (state === 'gameover') startGame();
    else if (state === 'playing') jump();
  }
  if (e.code === 'Escape') {
    if (['skins','shop','settings','levelSelect','stats'].includes(state)) showMenu();
    else if (state === 'playing' || state === 'gameover') showMenu();
  }
});

canvas.addEventListener('touchstart', e => { e.preventDefault(); if (state === 'playing') jump(); }, { passive: false });
canvas.addEventListener('mousedown', () => { if (state === 'playing') jump(); });

function update(dt) {
  if (state !== 'playing') return;
  const groundY = H * GROUND_Y_RATIO;

  if (mode === 'cube') {
    player.vy += GRAVITY * dt;
    player.y += player.vy * dt;
    if (player.y >= groundY - PLAYER_SIZE) {
      player.y = groundY - PLAYER_SIZE; player.vy = 0; player.onGround = true;
    } else player.onGround = false;

    if (!player.onGround) player.rotation += 0.13 * dt;
    else {
      const targetRot = Math.round(player.rotation / (Math.PI/2)) * (Math.PI/2);
      player.rotation += (targetRot - player.rotation) * 0.25 * dt;
    }

    const px = player.x, py = player.y, ps = PLAYER_SIZE * 0.82;
    for (const o of obstacles) {
      const sx = o.x - distance + player.x;
      const sy = o.y;
      if (sx + o.w < px || sx > px + ps) continue;
      if (sy + o.h < py || sy > py + ps) continue;
      if (o.type === 'spike') {
        const cx = sx + o.w / 2, cy = sy + o.h;
        const dx = Math.abs(px + ps/2 - cx), dy = Math.abs(py + ps - cy);
        if (dx < o.w/2 * 0.65 && dy < o.h * 0.85) { gameOver(); return; }
      } else { gameOver(); return; }
    }
  } else {
    // Самолёт
    const isHolding = window._keysDown['Space'] || window._keysDown['ArrowUp'] ||
                      window._touchHolding || window._mouseHolding;
    if (isHolding) player.vy -= 0.7 * dt; else player.vy += 0.4 * dt;
    player.vy = Math.max(-6, Math.min(8, player.vy));
    player.y += player.vy * dt;
    const minY = 40;
    const maxY = groundY - PLAYER_SIZE - 10;
    if (player.y < minY) { player.y = minY; player.vy = Math.max(0, player.vy); }
    if (player.y > maxY) { player.y = maxY; player.vy = Math.min(0, player.vy); }
    player.rotation = Math.max(-0.5, Math.min(0.5, player.vy * 0.08));

    const px = player.x, py = player.y, ps = PLAYER_SIZE * 0.7;
    for (const o of obstacles) {
      const sx = o.x - distance + player.x;
      const sy = o.y;
      if (sx + o.w < px || sx > px + ps) continue;
      if (sy + o.h < py || sy > py + ps) continue;
      gameOver(); return;
    }
  }

  // Проверка порталов (гибрид)
  for (const p of portals) {
    if (p.used) continue;
    const sx = p.x - distance + player.x;
    if (player.x + PLAYER_SIZE > sx && player.x < sx + p.w) {
      if (p.type === 'cube-to-plane' && mode === 'cube') {
        p.used = true;
        mode = 'plane';
        player.vy = 0;
        sfxPortal();
        for (let i = 0; i < 25; i++) {
          particles.push({
            x: player.x + PLAYER_SIZE/2, y: player.y + PLAYER_SIZE/2,
            vx: (Math.random() - 0.5) * 12, vy: (Math.random() - 0.5) * 12,
            life: 1, color: Math.random() < 0.5 ? '#00e5ff' : '#ff00e5'
          });
        }
      }
    }
  }

  const levelLength = currentLevel === 'cube' ? LEVEL_LENGTH :
                      currentLevel === 'plane' ? PLANE_LEVEL_LENGTH : HYBRID_TOTAL_LENGTH;
  distance += SPEED * dt;
  bgOffset = (bgOffset + SPEED * 0.3 * dt) % W;
  groundOffset = (groundOffset + SPEED * dt) % 60;

  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.x += p.vx * dt; p.y += p.vy * dt;
    p.vy += 0.3 * dt; p.life -= 0.03 * dt;
    if (p.life <= 0) particles.splice(i, 1);
  }
  if (cameraShake > 0) cameraShake *= 0.9;

  for (const s of planeStars) {
    s.x -= s.speed * dt;
    if (s.x < -5) { s.x = W + 5; s.y = Math.random() * H; }
  }

  const progress = Math.min(100, Math.floor((distance / levelLength) * 100));
  document.getElementById('score').textContent = progress + '%';
  document.getElementById('progressBar').style.width = progress + '%';
  if (distance >= levelLength) win();
}

window._keysDown = {};
window._touchHolding = false;
window._mouseHolding = false;
window.addEventListener('keydown', e => { if (e.code === 'Space' || e.code === 'ArrowUp') window._keysDown[e.code] = true; });
window.addEventListener('keyup', e => { if (e.code === 'Space' || e.code === 'ArrowUp') window._keysDown[e.code] = false; });
canvas.addEventListener('touchstart', () => { window._touchHolding = true; });
canvas.addEventListener('touchend', () => { window._touchHolding = false; });
canvas.addEventListener('mousedown', () => { window._mouseHolding = true; });
canvas.addEventListener('mouseup', () => { window._mouseHolding = false; });
canvas.addEventListener('mouseleave', () => { window._mouseHolding = false; });

function draw() {
  const groundY = H * GROUND_Y_RATIO;
  const shakeX = cameraShake ? (Math.random() - 0.5) * cameraShake : 0;
  const shakeY = cameraShake ? (Math.random() - 0.5) * cameraShake : 0;
  ctx.save();
  ctx.translate(shakeX, shakeY);

  // Фон
  let bgGrad;
  if (mode === 'cube') {
    bgGrad = ctx.createLinearGradient(0, 0, 0, H);
    bgGrad.addColorStop(0, '#1a0033'); bgGrad.addColorStop(0.5, '#0d001a'); bgGrad.addColorStop(1, '#000');
  } else {
    bgGrad = ctx.createLinearGradient(0, 0, 0, H);
    bgGrad.addColorStop(0, '#000033'); bgGrad.addColorStop(0.4, '#001133'); bgGrad.addColorStop(1, '#000011');
  }
  ctx.fillStyle = bgGrad;
  ctx.fillRect(-10, -10, W + 20, H + 20);

  // Мерцающие звёзды на фоне (для красоты)
  for (const s of bgStars) {
    ctx.globalAlpha = s.alpha * (0.6 + 0.4 * Math.sin(distance * 0.01 + s.x));
    ctx.fillStyle = mode === 'cube' ? '#cc66ff' : '#88aaff';
    ctx.fillRect(s.x, s.y, s.size, s.size);
  }
  ctx.globalAlpha = 1;

  if (mode === 'cube') {
    // Красивая сетка
    ctx.strokeStyle = 'rgba(120, 0, 200, 0.2)';
    ctx.lineWidth = 1;
    const gridSize = 60;
    for (let x = -bgOffset % gridSize; x < W; x += gridSize) {
      ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, groundY); ctx.stroke();
    }
    for (let y = 0; y < groundY; y += gridSize) {
      ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
    }

    // Красивая земля с градиентом и свечением
    const groundGrad = ctx.createLinearGradient(0, groundY, 0, H);
    groundGrad.addColorStop(0, '#2a0055'); groundGrad.addColorStop(1, '#0a0015');
    ctx.fillStyle = groundGrad; ctx.fillRect(0, groundY, W, H - groundY);

    // Свечение линии земли
    ctx.strokeStyle = '#00e5ff'; ctx.lineWidth = 3;
    ctx.shadowColor = '#00e5ff'; ctx.shadowBlur = 20;
    ctx.beginPath(); ctx.moveTo(0, groundY); ctx.lineTo(W, groundY); ctx.stroke();
    ctx.shadowBlur = 0;

    // Пульсирующая линия
    ctx.strokeStyle = 'rgba(255, 0, 229, 0.4)';
    ctx.lineWidth = 2;
    ctx.beginPath();
    for (let x = 0; x < W; x += 20) {
      const y = groundY + 15 + Math.sin((x + distance) * 0.05) * 8;
      if (x === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
    }
    ctx.stroke();

    // Декоративные линии на земле
    ctx.strokeStyle = 'rgba(0, 229, 255, 0.4)'; ctx.lineWidth = 2;
    for (let x = -groundOffset; x < W; x += 60) {
      ctx.beginPath(); ctx.moveTo(x, groundY + 8); ctx.lineTo(x + 25, groundY + 8); ctx.stroke();
    }

    // Препятствия
    for (const o of obstacles) {
      const sx = o.x - distance + player.x;
      if (sx + o.w < -50 || sx > W + 50) continue;
      if (o.type === 'spike') {
        ctx.fillStyle = '#ff0044'; ctx.shadowColor = '#ff0044'; ctx.shadowBlur = 15;
        ctx.beginPath(); ctx.moveTo(sx, o.y + o.h); ctx.lineTo(sx + o.w / 2, o.y);
        ctx.lineTo(sx + o.w, o.y + o.h); ctx.closePath(); ctx.fill();
        ctx.shadowBlur = 0;
        // Блик
        ctx.fillStyle = 'rgba(255,255,255,0.3)';
        ctx.beginPath(); ctx.moveTo(sx + o.w*0.3, o.y + o.h); ctx.lineTo(sx + o.w/2, o.y + o.h*0.3);
        ctx.lineTo(sx + o.w*0.5, o.y + o.h); ctx.closePath(); ctx.fill();
      } else {
        // Красивый блок с градиентом
        const bg = ctx.createLinearGradient(sx, o.y, sx + o.w, o.y + o.h);
        bg.addColorStop(0, '#00e5ff'); bg.addColorStop(1, '#0088ff');
        ctx.fillStyle = bg; ctx.shadowColor = '#00e5ff'; ctx.shadowBlur = 15;
        ctx.fillRect(sx, o.y, o.w, o.h); ctx.shadowBlur = 0;
        // Блик
        ctx.fillStyle = 'rgba(255,255,255,0.4)';
        ctx.fillRect(sx + 4, o.y + 4, o.w - 8, 5);
        // Обводка
        ctx.strokeStyle = 'rgba(255,255,255,0.6)'; ctx.lineWidth = 2;
        ctx.strokeRect(sx, o.y, o.w, o.h);
      }
    }
  } else {
    // Самолётный режим
    for (const s of planeStars) {
      ctx.fillStyle = `rgba(255, 255, 255, ${0.3 + s.size * 0.15})`;
      ctx.fillRect(s.x, s.y, s.size, s.size);
    }
    const groundGrad = ctx.createLinearGradient(0, groundY, 0, H);
    groundGrad.addColorStop(0, '#001133'); groundGrad.addColorStop(1, '#000011');
    ctx.fillStyle = groundGrad; ctx.fillRect(0, groundY, W, H - groundY);
    ctx.strokeStyle = '#0088ff'; ctx.lineWidth = 3;
    ctx.shadowColor = '#0088ff'; ctx.shadowBlur = 20;
    ctx.beginPath(); ctx.moveTo(0, groundY); ctx.lineTo(W, groundY); ctx.stroke();
    ctx.shadowBlur = 0;

    for (const o of obstacles) {
      const sx = o.x - distance + player.x;
      if (sx + o.w < -50 || sx > W + 50) continue;
      if (o.type === 'wall') {
        const wg = ctx.createLinearGradient(sx, o.y, sx + o.w, o.y + o.h);
        wg.addColorStop(0, '#ff6600'); wg.addColorStop(1, '#cc3300');
        ctx.fillStyle = wg; ctx.shadowColor = '#ff6600'; ctx.shadowBlur = 12;
        ctx.fillRect(sx, o.y, o.w, o.h); ctx.shadowBlur = 0;
        ctx.fillStyle = 'rgba(255,255,255,0.3)';
        ctx.fillRect(sx + 3, o.y + 3, o.w - 6, 4);
      } else if (o.type === 'floating') {
        ctx.fillStyle = '#ff00e5'; ctx.shadowColor = '#ff00e5'; ctx.shadowBlur = 18;
        ctx.fillRect(sx, o.y, o.w, o.h); ctx.shadowBlur = 0;
        ctx.strokeStyle = 'rgba(255,255,255,0.6)'; ctx.lineWidth = 2;
        ctx.strokeRect(sx, o.y, o.w, o.h);
      }
    }
  }

  // Портал
  for (const p of portals) {
    const sx = p.x - distance + player.x;
    if (sx + p.w < -50 || sx > W + 50) continue;
    const cx = sx + p.w / 2;
    const cy = H / 2;
    const r = Math.min(H * 0.35, 200);
    // Внешнее свечение
    const grad = ctx.createRadialGradient(cx, cy, r * 0.3, cx, cy, r);
    grad.addColorStop(0, 'rgba(255, 0, 229, 0.8)');
    grad.addColorStop(0.5, 'rgba(138, 0, 255, 0.5)');
    grad.addColorStop(1, 'rgba(0, 229, 255, 0)');
    ctx.fillStyle = grad;
    ctx.beginPath(); ctx.ellipse(cx, cy, p.w * 1.5, r, 0, 0, Math.PI * 2); ctx.fill();
    // Ядро портала
    const core = ctx.createLinearGradient(cx - p.w/2, 0, cx + p.w/2, 0);
    core.addColorStop(0, '#00e5ff'); core.addColorStop(0.5, '#ffffff'); core.addColorStop(1, '#ff00e5');
    ctx.fillStyle = core;
    ctx.beginPath(); ctx.ellipse(cx, cy, p.w / 2, r, 0, 0, Math.PI * 2); ctx.fill();
  }

  // Частицы
  for (const p of particles) {
    ctx.globalAlpha = p.life;
    ctx.fillStyle = p.color;
    ctx.fillRect(p.x, p.y, 4, 4);
  }
  ctx.globalAlpha = 1;

  // Игрок
  if (mode === 'cube') {
    drawCube(ctx, player.x + PLAYER_SIZE/2, player.y + PLAYER_SIZE/2, PLAYER_SIZE, currentSkin, player.rotation);
  } else {
    drawPlane(ctx, player.x + PLAYER_SIZE/2, player.y + PLAYER_SIZE/2, PLAYER_SIZE, currentSkin, player.rotation);
  }

  ctx.restore();
}

function drawPlane(g, x, y, size, skin, rotation) {
  g.save(); g.translate(x, y); g.rotate(rotation);
  const half = size / 2;
  g.shadowColor = skin.color1; g.shadowBlur = 18;
  g.fillStyle = skin.color1;
  g.beginPath();
  g.moveTo(half, 0); g.lineTo(-half * 0.6, -half * 0.7); g.lineTo(-half, 0); g.lineTo(-half * 0.6, half * 0.7);
  g.closePath(); g.fill();
  g.shadowBlur = 0;
  g.fillStyle = skin.color2;
  g.beginPath(); g.ellipse(half * 0.1, -half * 0.1, half * 0.25, half * 0.2, 0, 0, Math.PI * 2); g.fill();
  g.fillStyle = skin.color2;
  g.beginPath(); g.moveTo(-half * 0.3, -half * 0.4); g.lineTo(-half * 0.7, -half * 0.9); g.lineTo(-half * 0.5, -half * 0.5); g.closePath(); g.fill();
  g.beginPath(); g.moveTo(-half * 0.3, half * 0.4); g.lineTo(-half * 0.7, half * 0.9); g.lineTo(-half * 0.5, half * 0.5); g.closePath(); g.fill();
  g.fillStyle = skin.color1;
  g.beginPath(); g.moveTo(-half * 0.8, -half * 0.2); g.lineTo(-half * 1.1, -half * 0.6); g.lineTo(-half * 0.9, 0); g.closePath(); g.fill();
  g.beginPath(); g.moveTo(-half * 0.8, half * 0.2); g.lineTo(-half * 1.1, half * 0.6); g.lineTo(-half * 0.9, 0); g.closePath(); g.fill();
  g.strokeStyle = 'rgba(255,255,255,0.8)'; g.lineWidth = 2;
  g.beginPath();
  g.moveTo(half, 0); g.lineTo(-half * 0.6, -half * 0.7); g.lineTo(-half, 0); g.lineTo(-half * 0.6, half * 0.7);
  g.closePath(); g.stroke();
  g.restore();
}

// =====================================================
// ==================== ИГРОВОЙ ЦИКЛ ===================
// =====================================================
let lastTime = 0;
let fpsFrames = 0;
let fpsLastTime = 0;
let fpsValue = 60;
const fpsDisplay = document.getElementById('fpsDisplay');

function loop(t) {
  const dt = Math.min((t - lastTime) / 16.67, 3);
  lastTime = t;
  fpsFrames++;
  if (t - fpsLastTime >= 500) {
    fpsValue = Math.round(fpsFrames * 1000 / (t - fpsLastTime));
    fpsFrames = 0; fpsLastTime = t;
    if (settings.showFps) fpsDisplay.textContent = 'FPS: ' + fpsValue;
  }
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

function updateHUD() { document.getElementById('attempts').textContent = attempts; }

function startGame() {
  Music.init(); Music.resume();
  init();
  state = 'playing';
  ['menu','gameover','skins','shop','settings','levelSelect','stats'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('hud').classList.remove('hidden');
  document.getElementById('progress').classList.remove('hidden');
  document.getElementById('coinReward').classList.add('hidden');
  Music.playTrack(currentLevel === 'hybrid' ? 'hybrid' : currentLevel);
}

function showMenu() {
  state = 'menu';
  document.getElementById('menu').classList.remove('hidden');
  ['gameover','skins','shop','settings','levelSelect','stats'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('hud').classList.add('hidden');
  document.getElementById('progress').classList.add('hidden');
  document.getElementById('gameover').querySelector('h1').textContent = 'GAME OVER';
  document.getElementById('coinReward').classList.add('hidden');
  updateCoinUI();
  updateNickUI();
  Music.playTrack('menu');
}

function showSkins() {
  state = 'skins';
  ['menu','gameover','shop','settings','levelSelect','stats','hud','progress'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('skins').classList.remove('hidden');
  updateSkinUI();
  Music.playTrack('menu');
}
function showShop() {
  state = 'shop';
  ['menu','gameover','skins','settings','levelSelect','stats','hud','progress'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('shop').classList.remove('hidden');
  updateCoinUI(); buildShopUI();
  Music.playTrack('menu');
}
function showSettings() {
  state = 'settings';
  ['menu','gameover','skins','shop','levelSelect','stats','hud','progress'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('settings').classList.remove('hidden');
  Music.playTrack('menu');
}
function showStats() {
  state = 'stats';
  ['menu','gameover','skins','shop','settings','levelSelect','hud','progress'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('stats').classList.remove('hidden');
  updateStatsUI();
  Music.playTrack('menu');
}
function updateStatsUI() {
  document.getElementById('statCoins').textContent = playerData.coins;
  document.getElementById('statWins').textContent = playerData.wins;
  document.getElementById('statLosses').textContent = playerData.losses;
  document.getElementById('statSpent').textContent = playerData.totalSpent;
  document.getElementById('statAttempts').textContent = playerData.totalAttempts;
  document.getElementById('statEarned').textContent = playerData.totalEarned;
}
function showLevelSelect() {
  state = 'levelSelect';
  ['menu','gameover','skins','shop','settings','stats','hud','progress'].forEach(id => {
    document.getElementById(id).classList.add('hidden');
  });
  document.getElementById('levelSelect').classList.remove('hidden');
  updateLevelSelectUI();
  Music.playTrack('menu');
}
function updateLevelSelectUI() {
  document.querySelectorAll('.level-card').forEach(el => {
    el.classList.toggle('selected', el.dataset.level === currentLevel);
  });
}

function gameOver() {
  state = 'gameover';
  cameraShake = 25;
  sfxDeath();
  attempts++;
  playerData.losses++;
  playerData.totalAttempts++;
  localStorage.setItem('gd_attempts', attempts);
  savePlayerData();
  for (let i = 0; i < 30; i++) {
    particles.push({
      x: player.x + PLAYER_SIZE/2, y: player.y + PLAYER_SIZE/2,
      vx: (Math.random() - 0.5) * 15, vy: (Math.random() - 0.5) * 15,
      life: 1, color: Math.random() < 0.5 ? currentSkin.color1 : currentSkin.color2
    });
  }
  setTimeout(() => {
    const levelLength = currentLevel === 'cube' ? LEVEL_LENGTH :
                        currentLevel === 'plane' ? PLANE_LEVEL_LENGTH : HYBRID_TOTAL_LENGTH;
    const progress = Math.min(100, Math.floor((distance / levelLength) * 100));
    document.getElementById('finalScore').textContent = progress + '%';
    document.getElementById('finalAttempts').textContent = attempts;
    document.getElementById('gameover').classList.remove('hidden');
    document.getElementById('hud').classList.add('hidden');
    document.getElementById('progress').classList.add('hidden');
    Music.playTrack('menu');
  }, 600);
}

function win() {
  state = 'gameover';
  sfxWin();
  attempts = 1;
  localStorage.setItem('gd_attempts', 1);
  playerData.coins += 1;
  playerData.totalEarned += 1;
  playerData.wins++;
  playerData.totalAttempts++;
  if (!playerData.hasWonAny) playerData.hasWonAny = true;
  savePlayerData(); updateCoinUI(); sfxCoin();
  setTimeout(() => {
    document.getElementById('finalScore').textContent = '100% 🏆';
    document.getElementById('finalAttempts').textContent = attempts;
    document.getElementById('gameover').querySelector('h1').textContent = 'ПОБЕДА!';
    document.getElementById('coinReward').classList.remove('hidden');
    document.getElementById('gameover').classList.remove('hidden');
    document.getElementById('hud').classList.add('hidden');
    document.getElementById('progress').classList.add('hidden');
    Music.playTrack('menu');
  }, 300);
}

// ===== UI СКИНОВ =====
function buildColorGrid(elId, onSelect) {
  const grid = document.getElementById(elId);
  grid.innerHTML = '';
  COLORS.forEach(c => {
    const item = document.createElement('div');
    item.className = 'color-item';
    item.style.background = c;
    item.style.color = c;
    item.dataset.color = c;
    const info = getColorInfo(c);
    const owned = isColorOwned(c);
    if (!owned) {
      item.classList.add('locked');
      const price = document.createElement('div');
      price.className = 'price';
      price.textContent = '🔒 ' + (info ? info.price : '?');
      item.appendChild(price);
    }
    item.onclick = () => {
      if (!isColorOwned(c)) { showShop(); return; }
      onSelect(c);
    };
    grid.appendChild(item);
  });
}

function buildSkinsUI() {
  const cubeGrid = document.getElementById('cubeGrid');
  cubeGrid.innerHTML = '';
  CUBE_STYLES.forEach(style => {
    const item = document.createElement('div');
    item.className = 'cube-item';
    item.dataset.style = style.id;
    item.title = style.name;
    const owned = isStyleOwned(style.id);
    if (!owned) {
      item.classList.add('locked');
      const price = document.createElement('div');
      price.className = 'cube-price';
      if (style.unlockByWin) {
        price.textContent = '🏆 Пройди уровень';
        price.style.background = '#00ff88';
        price.style.color = '#000';
        price.style.fontSize = '9px';
      } else {
        price.textContent = '🔒 ' + style.price;
      }
      item.appendChild(price);
    }
    const cv = document.createElement('canvas');
    cv.width = 60; cv.height = 60;
    cv.className = 'cube-preview';
    const g = cv.getContext('2d');
    drawCube(g, 30, 30, 40, { ...currentSkin, style: style.id }, 0, false);
    item.appendChild(cv);
    item.onclick = () => {
      if (isStyleLockedByWin(style.id)) { showLockedMessage('Пройди любой уровень, чтобы разблокировать!'); return; }
      if (!isStyleOwned(style.id)) { buyStyle(style); return; }
      currentSkin.style = style.id; saveSkin(); updateSkinUI();
    };
    cubeGrid.appendChild(item);
  });
  buildColorGrid('colorGrid1', (c) => { currentSkin.color1 = c; saveSkin(); updateSkinUI(); });
  buildColorGrid('colorGrid2', (c) => { currentSkin.color2 = c; saveSkin(); updateSkinUI(); });
}

function showLockedMessage(text) {
  const msg = document.createElement('div');
  msg.style.cssText = 'position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);' +
    'background:rgba(0,0,0,0.9);color:#00ff88;padding:20px 30px;border-radius:12px;' +
    'border:2px solid #00ff88;font-weight:bold;font-size:16px;z-index:9999;' +
    'box-shadow:0 0 30px rgba(0,255,136,0.5);text-align:center;max-width:80%;';
  msg.textContent = text;
  document.body.appendChild(msg);
  setTimeout(() => msg.remove(), 2000);
}

function updateSkinUI() {
  document.querySelectorAll('#cubeGrid .cube-item').forEach(el => {
    el.classList.toggle('selected', el.dataset.style === currentSkin.style);
    const owned = isStyleOwned(el.dataset.style);
    el.classList.toggle('locked', !owned);
  });
  document.querySelectorAll('#colorGrid1 .color-item').forEach(el => {
    el.classList.toggle('selected', el.dataset.color === currentSkin.color1);
  });
  document.querySelectorAll('#colorGrid2 .color-item').forEach(el => {
    el.classList.toggle('selected', el.dataset.color === currentSkin.color2);
  });
  document.querySelectorAll('#cubeGrid .cube-item').forEach(el => {
    const cv = el.querySelector('canvas');
    const g = cv.getContext('2d');
    g.clearRect(0, 0, cv.width, cv.height);
    drawCube(g, 30, 30, 40, { ...currentSkin, style: el.dataset.style }, 0, false);
  });
  const pv = document.getElementById('previewCanvas');
  const pg = pv.getContext('2d');
  pg.clearRect(0, 0, pv.width, pv.height);
  drawCube(pg, 50, 50, 70, currentSkin, 0, true);
}

// ===== МАГАЗИН =====
function buildShopUI() {
  const container = document.getElementById('shopItems');
  container.innerHTML = '';
  const colorHeader = document.createElement('h3');
  colorHeader.textContent = '🎨 Цвета';
  colorHeader.style.cssText = 'color:#00e5ff; margin:15px 0 5px; font-size:16px;';
  container.appendChild(colorHeader);
  PREMIUM_COLORS.forEach(info => {
    const owned = playerData.ownedColors.includes(info.color);
    const item = document.createElement('div');
    item.className = 'shop-item' + (owned ? ' owned' : '');
    const infoDiv = document.createElement('div');
    infoDiv.className = 'shop-item-info';
    const swatch = document.createElement('div');
    swatch.className = 'shop-item-swatch';
    swatch.style.background = info.color;
    if (info.color === '#ffffff') swatch.style.border = '2px solid #888';
    const name = document.createElement('div');
    name.className = 'shop-item-name';
    name.textContent = info.name;
    infoDiv.appendChild(swatch); infoDiv.appendChild(name);
    item.appendChild(infoDiv);
    if (owned) {
      const price = document.createElement('div');
      price.className = 'shop-item-price';
      price.textContent = '✓ Куплено';
      item.appendChild(price);
    } else {
      const price = document.createElement('div');
      price.className = 'shop-item-price';
      price.textContent = '🪙 ' + info.price;
      item.appendChild(price);
      item.style.cursor = 'pointer';
      item.onclick = () => buyColor(info);
    }
    container.appendChild(item);
  });
  const cubeHeader = document.createElement('h3');
  cubeHeader.textContent = '🧊 Кубы';
  cubeHeader.style.cssText = 'color:#00e5ff; margin:20px 0 5px; font-size:16px;';
  container.appendChild(cubeHeader);
  CUBE_STYLES.filter(s => s.price > 0 && !s.unlockByWin).forEach(style => {
    const owned = playerData.ownedStyles.includes(style.id);
    const item = document.createElement('div');
    item.className = 'shop-item' + (owned ? ' owned' : '');
    const infoDiv = document.createElement('div');
    infoDiv.className = 'shop-item-info';
    const swatch = document.createElement('div');
    swatch.className = 'shop-item-swatch';
    swatch.style.background = 'rgba(0,0,0,0.3)';
    swatch.style.display = 'flex'; swatch.style.alignItems = 'center'; swatch.style.justifyContent = 'center';
    const miniCanvas = document.createElement('canvas');
    miniCanvas.width = 40; miniCanvas.height = 40;
    miniCanvas.style.width = '36px'; miniCanvas.style.height = '36px';
    const mg = miniCanvas.getContext('2d');
    drawCube(mg, 20, 20, 30, { ...currentSkin, style: style.id }, 0, false);
    swatch.appendChild(miniCanvas);
    const name = document.createElement('div');
    name.className = 'shop-item-name';
    name.textContent = style.name;
    infoDiv.appendChild(swatch); infoDiv.appendChild(name);
    item.appendChild(infoDiv);
    if (owned) {
      const price = document.createElement('div');
      price.className = 'shop-item-price';
      price.textContent = '✓ Куплено';
      item.appendChild(price);
    } else {
      const price = document.createElement('div');
      price.className = 'shop-item-price';
      price.textContent = '🪙 ' + style.price;
      item.appendChild(price);
      item.style.cursor = 'pointer';
      item.onclick = () => buyStyle(style);
    }
    container.appendChild(item);
  });
}
function buyColor(info) {
  if (playerData.ownedColors.includes(info.color)) return;
  if (playerData.coins < info.price) { showInsufficientCoins(); return; }
  playerData.coins -= info.price;
  playerData.totalSpent += info.price;
  playerData.ownedColors.push(info.color);
  savePlayerData(); updateCoinUI(); sfxBuy();
  buildShopUI(); buildSkinsUI(); updateSkinUI();
}
function buyStyle(style) {
  if (playerData.ownedStyles.includes(style.id)) return;
  if (playerData.coins < style.price) { showInsufficientCoins(); return; }
  playerData.coins -= style.price;
  playerData.totalSpent += style.price;
  playerData.ownedStyles.push(style.id);
  savePlayerData(); updateCoinUI(); sfxBuy();
  buildShopUI(); buildSkinsUI(); updateSkinUI();
}
function showInsufficientCoins() {
  const el = document.getElementById('shopCoins');
  el.style.color = '#ff0044';
  el.textContent = '❌ Недостаточно монет!';
  setTimeout(() => { el.style.color = '#ffcc00'; updateCoinUI(); }, 1200);
}

// ===== КНОПКА ЗВУКА =====
const soundBtnBig = document.getElementById('soundBtnBig');
function updateSoundBtn() { soundBtnBig.textContent = Music.enabled ? '🔊' : '🔇'; }
soundBtnBig.addEventListener('click', (e) => {
  e.stopPropagation();
  Music.init(); Music.resume();
  Music.setEnabled(!Music.enabled);
  updateSoundBtn();
});

// ===== ПРИВЯЗКА КНОПОК =====
document.getElementById('startBtnBig').onclick = startGame;
document.getElementById('skinsBtnBig').onclick = showSkins;
document.getElementById('shopBtnBig').onclick = showShop;
document.getElementById('settingsBtnBig').onclick = showSettings;
document.getElementById('statsBtnBig').onclick = showStats;
document.getElementById('levelSelectBtnBig').onclick = showLevelSelect;
document.getElementById('retryBtn').onclick = startGame;
document.getElementById('backBtn').onclick = showMenu;
document.getElementById('menuBtn').onclick = showMenu;
document.getElementById('shopBackBtn').onclick = showMenu;
document.getElementById('settingsBackBtn').onclick = showMenu;
document.getElementById('statsBackBtn').onclick = showMenu;
document.getElementById('levelBackBtn').onclick = showMenu;

document.querySelectorAll('.level-card').forEach(card => {
  card.addEventListener('click', () => {
    currentLevel = card.dataset.level;
    updateLevelSelectUI();
    startGame();
  });
});

// ===== ЗАГРУЗКА / НИК / СТАРТ =====
loadPlayerData();
loadSkin();
loadSettings();
buildSkinsUI();
updateCoinUI();
updateNickUI();
state = 'menu';
init();

// Экран загрузки 5 секунд
const loadingSteps = [20, 45, 59, 79, 99];
let loadStepIndex = 0;
const loadingEl = document.getElementById('loading');
const loadingBar = document.getElementById('loadingBar');
const loadingPercent = document.getElementById('loadingPercent');
const nicknameScreen = document.getElementById('nicknameScreen');
const nicknameInput = document.getElementById('nicknameInput');
const nicknameBtn = document.getElementById('nicknameBtn');

function advanceLoading() {
  if (loadStepIndex < loadingSteps.length) {
    const pct = loadingSteps[loadStepIndex];
    loadingBar.style.width = pct + '%';
    loadingPercent.textContent = pct + '%';
    loadStepIndex++;
    if (loadStepIndex < loadingSteps.length) {
      setTimeout(advanceLoading, 1000);
    } else {
      setTimeout(() => {
        loadingEl.classList.add('hidden');
        if (playerData.nickname) {
          finishLoading();
        } else {
          nicknameScreen.classList.remove('hidden');
          nicknameInput.focus();
        }
      }, 1000);
    }
  }
}
setTimeout(advanceLoading, 100);

function finishLoading() {
  nicknameScreen.classList.add('hidden');
  document.getElementById('menu').classList.remove('hidden');
  enterFullscreen();
  Music.init(); Music.resume();
  Music.playTrack('menu');
  requestAnimationFrame(loop);
}

nicknameBtn.onclick = () => {
  const val = nicknameInput.value.trim().slice(0, 16) || 'Игрок';
  playerData.nickname = val;
  savePlayerData();
  updateNickUI();
  finishLoading();
};
nicknameInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') nicknameBtn.click();
});

// Аудио-анлок
function unlockAudio() {
  Music.init(); Music.resume();
  if (state === 'menu') Music.playTrack('menu');
  document.removeEventListener('touchstart', unlockAudio);
  document.removeEventListener('mousedown', unlockAudio);
  document.removeEventListener('keydown', unlockAudio);
}
document.addEventListener('touchstart', unlockAudio, { once: true });
document.addEventListener('mousedown', unlockAudio, { once: true });
document.addEventListener('keydown', unlockAudio, { once: true });
</script>
</body>
</html>
