let duration = 2000; // Süre (ms cinsinden) - 2000 = 2 saniye
let interval = 1; // Tıklama aralığı (1ms = çok hızlı)
let endTime = Date.now() + duration;

let spamClick = setInterval(() => {
    let x = window.event.clientX;
    let y = window.event.clientY;
    let el = document.elementFromPoint(x, y);
    if (el) el.click();
    if (Date.now() >= endTime) clearInterval(spamClick);
}, interval);

Code Block:
```let duration = 2000; // Süre (ms cinsinden) - 2000 = 2 saniye
let interval = 1; // Tıklama aralığı (1ms = çok hızlı)
let endTime = Date.now() + duration;

let spamClick = setInterval(() => {
    let x = window.event.clientX;
    let y = window.event.clientY;
    let el = document.elementFromPoint(x, y);
    if (el) el.click();
    if (Date.now() >= endTime) clearInterval(spamClick);
}, interval);```

V2:
```setInterval(()=>document.querySelector('button').click(),1);```

Chrome Dino:
```(() => {
  if(window.dinoModActive) return;
  window.dinoModActive = true;

  // Stil ekle
  const style = document.createElement('style');
  style.innerHTML = `
    #dinoModMenu {
      position: fixed;
      top: 50px; left: 50px;
      background: #222;
      color: #eee;
      font-family: monospace;
      padding: 15px;
      border-radius: 8px;
      z-index: 99999;
      width: 250px;
      user-select: none;
      box-shadow: 0 0 15px #00ff00aa;
    }
    #dinoModMenu h2 {
      margin: 0 0 10px 0;
      font-size: 18px;
      color: #0f0;
    }
    #dinoModMenu button {
      background: #0a0;
      border: none;
      color: #eee;
      padding: 7px 10px;
      margin: 5px 0;
      width: 100%;
      cursor: pointer;
      font-weight: bold;
      border-radius: 5px;
      transition: background 0.3s;
    }
    #dinoModMenu button:hover {
      background: #0f0;
      color: #000;
    }
  `;
  document.head.appendChild(style);

  // Menü oluştur
  const menu = document.createElement('div');
  menu.id = 'dinoModMenu';
  menu.innerHTML = `
    <h2>Dino Mega Mod Menu</h2>
    <button id="godModeBtn">Tanrı Modu (Ölümsüzlük)</button>
    <button id="superJumpBtn">Süper Zıplama</button>
    <button id="speedUpBtn">Hız Arttır</button>
    <button id="slowDownBtn">Hızı Azalt</button>
    <button id="infiniteScoreBtn">Sonsuz Skor</button>
    <button id="resetScoreBtn">Skoru Sıfırla</button>
    <button id="toggleGravityBtn">Yerçekimini Aç/Kapa</button>
    <button id="invisibleDinoBtn">Dino’yu Görünmez Yap</button>
    <button id="resetGameBtn">Oyunu Sıfırla</button>
    <button id="closeMenuBtn" style="background:#900;">Menüyü Kapat</button>
  `;
  document.body.appendChild(menu);

  // Oyun objesine referans
  const runner = Runner.instance_;

  // Tanrı Modu: Ölümü engelle
  let godMode = false;
  document.getElementById('godModeBtn').onclick = () => {
    godMode = !godMode;
    runner.gameOver = () => {};
    alert(`Tanrı Modu ${godMode ? 'Aktif' : 'Pasif'}`);
  };

  // Süper Zıplama
  document.getElementById('superJumpBtn').onclick = () => {
    runner.tRex.setJumpVelocity(25);
    alert('Süper zıplama aktif!');
  };

  // Hız arttır
  document.getElementById('speedUpBtn').onclick = () => {
    runner.setSpeed(runner.currentSpeed + 5);
    alert('Hız arttırıldı!');
  };

  // Hızı azalt
  document.getElementById('slowDownBtn').onclick = () => {
    runner.setSpeed(Math.max(runner.currentSpeed - 5, 1));
    alert('Hız azaltıldı!');
  };

  // Sonsuz skor
  let infiniteScore = false;
  document.getElementById('infiniteScoreBtn').onclick = () => {
    infiniteScore = !infiniteScore;
    if(infiniteScore){
      runner.distanceRan = 999999;
      runner.getDistanceRan = () => 999999;
      alert('Sonsuz skor aktif!');
    } else {
      runner.distanceRan = 0;
      runner.getDistanceRan = Runner.prototype.getDistanceRan;
      alert('Sonsuz skor pasif!');
    }
  };

  // Skoru sıfırla
  document.getElementById('resetScoreBtn').onclick = () => {
    runner.distanceRan = 0;
    alert('Skor sıfırlandı!');
  };

  // Yerçekimini aç/kapa
  let gravityOff = false;
  document.getElementById('toggleGravityBtn').onclick = () => {
    gravityOff = !gravityOff;
    if(gravityOff) {
      runner.tRex.config.GRAVITY = 0;
      alert('Yerçekimi kapatıldı!');
    } else {
      runner.tRex.config.GRAVITY = 0.6;
      alert('Yerçekimi açıldı!');
    }
  };

  // Dino’yu görünmez yap
  let invisible = false;
  document.getElementById('invisibleDinoBtn').onclick = () => {
    invisible = !invisible;
    runner.tRex.draw = function(ctx){
      if(!invisible) Runner.TRex.prototype.draw.call(this, ctx);
    };
    alert(`Dino görünmezlik ${invisible ? 'aktif' : 'pasif'}`);
  };

  // Oyunu sıfırla
  document.getElementById('resetGameBtn').onclick = () => {
    runner.restart();
    alert('Oyun sıfırlandı!');
  };

  // Menüyü kapat
  document.getElementById('closeMenuBtn').onclick = () => {
    menu.remove();
    style.remove();
    window.dinoModActive = false;
  };
})();```
