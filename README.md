[<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>ゆうちゃんRPG</title>
<script src="https://cdn.jsdelivr.net/npm/phaser@3/dist/phaser.js"></script>
<style>
body { margin:0; background:black; }
</style>
</head>
<body>

<script>
const config = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    scene: {
        preload: preload,
        create: create,
        update: update
    }
};

let player;
let cursors;
let hp = 100;
let mp = 50;
let attack = 10;
let defense = 5;
let critRate = 0.2;
let hpText;
let mpText;

const game = new Phaser.Game(config);

function preload() {}

function create() {
    this.add.text(300, 20, "ゆうちゃんRPG", { fontSize: "28px", fill: "#ffffff" });

    player = this.add.rectangle(400, 300, 40, 40, 0x00ffcc);

    cursors = this.input.keyboard.createCursorKeys();

    hpText = this.add.text(20, 520, "HP: " + hp, { fontSize: "20px", fill: "#00ff00" });
    mpText = this.add.text(20, 550, "MP: " + mp, { fontSize: "20px", fill: "#00ffff" });

    this.input.keyboard.on("keydown-SPACE", () => {
        attackEnemy.call(this);
    });
}

function update() {
    const speed = 4;

    if (cursors.left.isDown) player.x -= speed;
    if (cursors.right.isDown) player.x += speed;
    if (cursors.up.isDown) player.y -= speed;
    if (cursors.down.isDown) player.y += speed;
}

function attackEnemy() {
    let damage = attack;

    if (Math.random() < critRate) {
        damage *= 2;
        showCritEffect.call(this);
    }

    this.add.text(500, 300, "-" + damage, {
        fontSize: "28px",
        fill: "#ff0000"
    }).setOrigin(0.5);

    console.log("ダメージ:", damage);
}

function showCritEffect() {
    const crit = this.add.text(400, 250, "CRITICAL!", {
        fontSize: "40px",
        fill: "#ffff00"
    }).setOrigin(0.5);

    this.tweens.add({
        targets: crit,
        y: 200,
        alpha: 0,
        duration: 800,
        onComplete: () => crit.destroy()
    });
}
</script>

</body>
</html>
](https://lingjingliudou-ux.github.io/yu-rpg-8392xkq7/)
