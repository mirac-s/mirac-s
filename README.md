V1:
```
(function stableDinoBotV1() {
    const jump = () => {
        if (Runner.instance_.tRex.jumping || Runner.instance_.crashed) return;
        Runner.instance_.tRex.startJump(Runner.instance_.currentSpeed);
    };

    const checkAndJump = () => {
        const obstacles = Runner.instance_.horizon.obstacles;
        if (obstacles.length > 0) {
            const obs = obstacles[0];
            const dist = obs.xPos;
            const speed = Runner.instance_.currentSpeed;

            // Engel mesafesi ve tipi kontrolü
            if (dist < 100 && obs.yPos > 70) {
                jump();
            }
        }
        requestAnimationFrame(checkAndJump);
    };

    console.clear();
    console.log('%c[Bot V1 Başladı] Dino otomatik oynatılıyor.', 'color: green; font-weight: bold;');
    checkAndJump();
})();
```

-

V2:

```
(function safeDinoBotV2() {
    const interval = 10; // ms
    const botLoop = setInterval(() => {
        const runner = Runner.instance_;
        if (!runner || runner.crashed || !runner.runningTime) return;

        const tRex = runner.tRex;
        const obstacles = runner.horizon.obstacles;

        if (obstacles.length > 0) {
            const obs = obstacles[0];
            const dist = obs.xPos;
            const speed = runner.currentSpeed;

            if (dist > 0 && dist < 100 && obs.yPos > 70 && !tRex.jumping) {
                tRex.startJump(speed);
            }
        }
    }, interval);

    console.clear();
    console.log('%c[Bot V2 Başladı] Dino güvenli modda oynatılıyor.', 'color: blue; font-weight: bold;');
})();
```
-

V3:
```
(function() {
    'use strict';

    const BOT_INTERVAL_MS = 30;
    const REACTION_X_THRESHOLD_BASE = 90;
    const REACTION_X_SPEED_FACTOR = 6;
    const HIGH_SPEED_LOOKAHEAD_FACTOR = 4;
    const PTERODACTYL_DUCK_YPOS = [50, 75];
    const ENABLE_ACTION_LOGS = false;

    let dinoGameInstance;
    let lastLoggedObstacleId = -1;

    function logAction(message) {
        if (ENABLE_ACTION_LOGS) {
            console.log(`[DinoBot] Eylem: ${message}`);
        }
    }

    function logInfo(message) {
        console.log(`[DinoBot] Bilgi: ${message}`);
    }

    function dinoBotMainLogic() {
        if (typeof Runner === 'undefined' || Runner.instance_ === null) {
            return;
        }
        dinoGameInstance = Runner.instance_;

        if (!dinoGameInstance.playing || (dinoGameInstance.tRex && dinoGameInstance.tRex.crashed)) {
            if (dinoGameInstance.tRex && dinoGameInstance.tRex.crashed) {
                logInfo("Oyun bitti. Yeniden başlatılıyor...");
                dinoGameInstance.restart();
                setTimeout(() => {
                    if (dinoGameInstance.paused) dinoGameInstance.play();
                }, 100);
            } else if (dinoGameInstance.paused) {
                logInfo("Oyun duraklatılmış. Başlatılıyor...");
                dinoGameInstance.play();
            }
            if (dinoGameInstance.tRex && dinoGameInstance.tRex.ducking) {
                dinoGameInstance.tRex.setDuck(false);
            }
            return;
        }

        const tRex = dinoGameInstance.tRex;
        const obstacles = dinoGameInstance.horizon.obstacles;
        const currentSpeed = dinoGameInstance.currentSpeed;

        if (obstacles.length > 0) {
            const firstObstacle = obstacles[0];
            const obstacleXPos = firstObstacle.xPos;
            const obstacleYPos = firstObstacle.yPos;
            const obstacleWidth = firstObstacle.width;
            const obstacleType = firstObstacle.typeConfig ? firstObstacle.typeConfig.type : 'CACTUS';
            const obstacleId = firstObstacle.id;

            let reactionXThreshold = REACTION_X_THRESHOLD_BASE + (currentSpeed * REACTION_X_SPEED_FACTOR);
            if (currentSpeed > 10) {
                reactionXThreshold += (currentSpeed - 10) * HIGH_SPEED_LOOKAHEAD_FACTOR;
            }

            if (ENABLE_ACTION_LOGS && obstacleId !== lastLoggedObstacleId) {
                 console.log(`[DinoBot] Engel: ${obstacleType}, X: ${obstacleXPos.toFixed(0)}, Y: ${obstacleYPos}, Hız: ${currentSpeed.toFixed(1)}, EşikX: ${reactionXThreshold.toFixed(0)}`);
                 lastLoggedObstacleId = obstacleId;
            }

            if (obstacleXPos < reactionXThreshold && obstacleXPos > (tRex.config.START_X_POS - obstacleWidth / 2)) {
                if (tRex.jumping) {
                    return;
                }

                if (obstacleType === 'PTERODACTYL') {
                    if (PTERODACTYL_DUCK_YPOS.includes(obstacleYPos)) {
                        if (!tRex.ducking) {
                            logAction(`PTERODAKTİL için EĞİL (Y=${obstacleYPos})`);
                            tRex.setDuck(true);
                        }
                        return;
                    } else {
                        if (!tRex.jumping) {
                            logAction(`PTERODAKTİL için ZIPLA (Y=${obstacleYPos})`);
                            if (tRex.ducking) tRex.setDuck(false);
                            tRex.startJump(currentSpeed);
                        }
                    }
                } else {
                    if (!tRex.jumping) {
                        logAction("KAKTÜS için ZIPLA");
                        if (tRex.ducking) tRex.setDuck(false);
                        tRex.startJump(currentSpeed);
                    }
                }
            } else if (tRex.ducking && (obstacles.length === 0 || obstacleXPos > reactionXThreshold + obstacleWidth + 20 || obstacleXPos < (tRex.config.START_X_POS - obstacleWidth))) {
                logAction("EĞİLMEYİ BIRAK (tehdit geçti veya yok)");
                tRex.setDuck(false);
            }
        } else if (tRex.ducking) {
            logAction("EĞİLMEYİ BIRAK (engel yok)");
            tRex.setDuck(false);
        }
    }

    if (typeof window.dinoBotIntervalId !== 'undefined' && window.dinoBotIntervalId !== null) {
        clearInterval(window.dinoBotIntervalId);
        logInfo("Önceki Dino Bot interval'ı temizlendi.");
    }

    function attemptToStartBot() {
        if (typeof Runner !== 'undefined' && Runner.instance_) {
            dinoGameInstance = Runner.instance_;
            if (dinoGameInstance.paused) {
                dinoGameInstance.play();
            }
            window.dinoBotIntervalId = setInterval(dinoBotMainLogic, BOT_INTERVAL_MS);
            logInfo(`Chrome Dino Bot Aktif! Oyun otomatik oynanacak. Durdurmak için sayfayı yenileyin veya konsola şunu yazın: clearInterval(window.dinoBotIntervalId);`);
        } else {
            logInfo("Dino oyunu tam yüklenmemiş. Bot aktivasyonu 1 saniye sonra tekrar denenecek...");
            setTimeout(attemptToStartBot, 1000);
        }
    }

    attemptToStartBot();

})();
```

-

V4:
```
<script>
  (function() {
    let botInterval;
    const JUMP_THRESHOLD = 200;
    const CHECK_INTERVAL = 10;

    function startBot() {
      if (typeof Runner === 'undefined' || !Runner.instance_) {
        console.error('%c[Dino Bot] %cChrome Dino oyunu bulunamadı. Lütfen oyunu başlatın.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
        return;
      }
      if (botInterval) {
        console.warn('%c[Dino Bot] %cBot zaten çalışıyor.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
        return;
      }
      console.log('%c[Dino Bot] %cBot başlatılıyor...', 'color: #4CAF50; font-weight: bold;', 'color: #333;');

      botInterval = setInterval(() => {
        try {
          const tRex = Runner.instance_.tRex;
          const horizon = Runner.instance_.horizon;

          if (!tRex || !horizon) {
            console.warn('%c[Dino Bot] %cOyun nesneleri bulunamadı, bot duraklatıldı.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
            stopBot();
            return;
          }

          if (Runner.instance_.playing) {
            const obstacles = horizon.obstacles;
            if (obstacles.length > 0) {
              const firstObstacle = obstacles[0];
              const obstacleX = firstObstacle.xPos;
              const tRexX = tRex.xPos;

              if (obstacleX - tRexX < JUMP_THRESHOLD + firstObstacle.width && obstacleX > 0) {
                if (!tRex.jumping && !tRex.ducking) {
                  tRex.jump();
                }
              }
            }
          }
        } catch (e) {
          console.error(`%c[Dino Bot] %cBir hata oluştu: ${e.message}. Bot durduruluyor.`, 'color: #4CAF50; font-weight: bold;', 'color: #333;');
          stopBot();
        }
      }, CHECK_INTERVAL);

      console.log('%c[Dino Bot] %cBot başarıyla başlatıldı! İyi eğlenceler.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
      console.log('%c[Dino Bot] %cDurdurmak için `stopDinoBot()` yazın.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
    }

    function stopBot() {
      if (botInterval) {
        clearInterval(botInterval);
        botInterval = null;
        console.log('%c[Dino Bot] %cBot durduruldu.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
      } else {
        console.warn('%c[Dino Bot] %cBot zaten çalışmıyor.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
      }
    }

    window.startDinoBot = startBot;
    window.stopDinoBot = stopBot;

    console.log('%c[Dino Bot] %cDino Bot yüklendi. Başlatmak için `startDinoBot()` yazın.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
    console.log('%c[Dino Bot] %cDurdurmak için `stopDinoBot()` yazın.', 'color: #4CAF50; font-weight: bold;', 'color: #333;');
  })();
</script>
```

-

Uçuş Modu:
```
Runner.instance_.tRex.groundYPos = 0;
```

-

Skor Hack:
```
Runner.instance_.distanceMeter.maxScore = 999999999;
Runner.instance_.highestScore = 0;
var distanceMeter = Runner.instance_.distanceMeter;
var originalUpdate = distanceMeter.update;
distanceMeter.update = function(deltaTime) {
    originalUpdate.apply(this, arguments);
    if (this.digits[0] >= 9 && this.digits[1] >= 9 && this.digits[2] >= 9 &&
        this.digits[3] >= 9 && this.digits[4] >= 9) {}
};
```
