# <!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pilgrims vs Turkeys</title>
<style>
    body {
        margin: 0;
        background: #7c9c5a;
        overflow: hidden;
        touch-action: none;
        user-select: none;
    }
    canvas {
        display: block;
        margin: 0 auto;
        background: #8bbf6b;
        border: 4px solid #333;
    }
    #controls {
        position: fixed;
        bottom: 10px;
        left: 50%;
        transform: translateX(-50%);
        width: 90%;
        display: flex;
        justify-content: space-between;
        gap: 10px;
        z-index: 5;
    }
    button {
        width: 30%;
        padding: 14px;
        font-size: 20px;
        border-radius: 10px;
        border: none;
        background: #333;
        color: white;
    }
</style>
</head>
<body>

<canvas id="game" width="600" height="600"></canvas>

<div id="controls">
    <button id="left">←</button>
    <button id="shoot">Shoot</button>
    <button id="right">→</button>
</div>

<script>
/* --------------------------
   BASIC SETUP
--------------------------- */
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// 30 × 30 grid but scaled visually to 600px
const GRID = 30;
const CELL = canvas.width / GRID;

let keys = { left: false, right: false, shoot: false };

/* --------------------------
   GAME OBJECTS
--------------------------- */
class Player {
    constructor(x, y, type) {
        this.x = x;
        this.y = y;
        this.type = type; // "pilgrim" or "turkey"
        this.vx = 0;
        this.vy = 0;
        this.canShoot = true;
        this.cooldown = 0;
    }

    update() {
        // Movement
        this.x += this.vx;
        this.y += this.vy;

        // Bounds
        this.x = Math.max(0, Math.min(GRID - 1, this.x));
        this.y = Math.max(0, Math.min(GRID - 1, this.y));

        if (this.cooldown > 0) this.cooldown--;
        if (this.cooldown === 0) this.canShoot = true;
    }

    draw() {
        if (this.type === "pilgrim") {
            ctx.fillStyle = "#442200";
        } else {
            ctx.fillStyle = "orange";
        }
        ctx.fillRect(this.x * CELL, this.y * CELL, CELL, CELL);
    }
}

class Bullet {
    constructor(x, y, dir) {
        this.x = x;
        this.y = y;
        this.dir = dir;
        this.active = true;
    }

    update() {
        this.x += this.dir;
        if (this.x < 0 || this.x >= GRID) this.active = false;
    }

    draw() {
        ctx.fillStyle = "yellow";
        ctx.fillRect(this.x * CELL, this.y * CELL + CELL / 3, CELL, CELL / 3);
    }
}

/* --------------------------
   GAME SETUP
--------------------------- */
let pilgrims = [
    new Player(5, 15, "pilgrim"),
    new Player(5, 5, "pilgrim"),
    new Player(5, 25, "pilgrim")
];

let turkeys = [
    new Player(25, 10, "turkey"),
    new Player(25, 20, "turkey"),
    new Player(25, 5, "turkey")
];

let bullets = [];

/* --------------------------
   SIMPLE AI FOR TURKEYS
--------------------------- */
function updateTurkeys() {
    turkeys.forEach(t => {
        // Random movement
        if (Math.random() < 0.2) {
            t.vx = Math.floor(Math.random() * 3) - 1;
            t.vy = Math.floor(Math.random() * 3) - 1;
        }

        // Jump / quick dodge
        if (Math.random() < 0.05) {
            t.vy = -1;
        }
        if (Math.random() < 0.05) {
            t.vy = 1;
        }

        t.update();
    });
}

/* --------------------------
   PLAYER CONTROLS (Pilgrim)
--------------------------- */
function updatePilgrimControls() {
    let p = pilgrims[0]; // player-controlled pilgrim

    // Movement left/right
    if (keys.left) {
        p.vx = -0.3;
    } else if (keys.right) {
        p.vx = 0.3;
    } else {
        p.vx = 0;
    }

    // Shooting
    if (keys.shoot && p.canShoot) {
        bullets.push(new Bullet(p.x + 1, p.y, 1));
        p.canShoot = false;
        p.cooldown = 20; // 20 frames
    }

    p.update();
}

/* --------------------------
   BULLET UPDATES
--------------------------- */
function updateBullets() {
    bullets.forEach(b => b.update());
    bullets = bullets.filter(b => b.active);

    // Collision with turkeys
    bullets.forEach(b => {
        turkeys.forEach(t => {
            if (
                Math.abs(b.x - t.x) < 0.5 &&
                Math.abs(b.y - t.y) < 0.5
            ) {
                t.x = -100; // remove
                t.y = -100;
                b.active = false;
            }
        });
    });
}

/* --------------------------
   GAME LOOP
--------------------------- */
function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw grid background
    ctx.fillStyle = "#89b46a";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    updatePilgrimControls();
    updateTurkeys();
    updateBullets();

    // Draw players
    pilgrims.forEach(p => p.draw());
    turkeys.forEach(t => t.draw());

    // Draw bullets
    bullets.forEach(b => b.draw());

    requestAnimationFrame(loop);
}
loop();

/* --------------------------
   MOBILE BUTTON CONTROLS
--------------------------- */
document.getElementById("left").ontouchstart = () => keys.left = true;
document.getElementById("left").ontouchend = () => keys.left = false;

document.getElementById("right").ontouchstart = () => keys.right = true;
document.getElementById("right").ontouchend = () => keys.right = false;

document.getElementById("shoot").ontouchstart = () => keys.shoot = true;
document.getElementById("shoot").ontouchend = () => keys.shoot = false;

</script>
</body>
</html>