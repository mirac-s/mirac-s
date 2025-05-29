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

```(function safeDinoBotV2() {
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
})();```
