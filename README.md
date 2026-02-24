<!DOCTYPE html>
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

// =====================
// 基本データ
// =====================

const glassesList = {
  normal: { name:"ふつうのメガネ", rarity:1, hp:0, mp:0, attack:0, defense:1, critRate:0.02 },
  power: { name:"パワーメガネ", rarity:2, hp:5, mp:0, attack:3, defense:0, critRate:0.03 },
  crit: { name:"かいしんメガネ", rarity:4, hp:5, mp:0, attack:2, defense:2, critRate:0.15 }
};

let playerData = {
  name:"ゆうちゃん",
  base:{ hp:30, mp:10, attack:8, defense:4 },
  equipment:{ accessory:"normal" },
  hp:30,
  mp:10
};

function calculateStats(){
  const g = glassesList[playerData.equipment.accessory];
  playerData.maxHP = playerData.base.hp + g.hp;
  playerData.maxMP = playerData.base.mp + g.mp;
  playerData.attack = playerData.base.attack + g.attack;
  playerData.defense = playerData.base.defense + g.defense;
  playerData.critRate = g.critRate;
}

function saveGame(){
  localStorage.setItem("yuRPG", JSON.stringify(playerData));
}

function loadGame(){
  const data = localStorage.getItem("yuRPG");
  if(data) playerData = JSON.parse(data);
}

loadGame();
calculateStats();

// =====================
// Phaser設定
// =====================

const config = {
  type: Phaser.AUTO,
  width: 360,
  height: 640,
  physics:{ default:"arcade" },
  scene:[MapScene, BattleScene]
};

const game = new Phaser.Game(config);

// =====================
// マップシーン
// =====================

function MapScene(){ Phaser.Scene.call(this,{key:"MapScene"}); }
MapScene.prototype = Object.create(Phaser.Scene.prototype);

MapScene.prototype.create = function(){

  this.add.text(100,50,"ゆうちゃんRPG",{color:"#fff"});

  this.player = this.physics.add.rectangle(180,320,32,32,0x00ff00);

  this.cursors = this.input.keyboard.createCursorKeys();

  this.cameras.main.startFollow(this.player);
  this.cameras.main.setZoom(1.5);
};

MapScene.prototype.update = function(){

  const speed = 150;
  this.player.body.setVelocity(0);

  if(this.cursors.left.isDown) this.player.body.setVelocityX(-speed);
  if(this.cursors.right.isDown) this.player.body.setVelocityX(speed);
  if(this.cursors.up.isDown) this.player.body.setVelocityY(-speed);
  if(this.cursors.down.isDown) this.player.body.setVelocityY(speed);

  // ランダムエンカウント
  if(Math.random() < 0.005){
    this.scene.start("BattleScene");
  }
};

// =====================
// 戦闘シーン
// =====================

function BattleScene(){ Phaser.Scene.call(this,{key:"BattleScene"}); }
BattleScene.prototype = Object.create(Phaser.Scene.prototype);

BattleScene.prototype.create = function(){

  calculateStats();

  this.enemy = {
    hp:20,
    attack:6,
    defense:2
  };

  this.add.rectangle(180,480,40,40,0x00ff00); // ゆうちゃん（背中）
  this.enemySprite = this.add.rectangle(180,200,60,60,0x0000ff); // 敵

  this.hpText = this.add.text(20,550,"HP:"+playerData.hp+"/"+playerData.maxHP,{color:"#fff"});
  this.mpText = this.add.text(20,580,"MP:"+playerData.mp+"/"+playerData.maxMP,{color:"#00f"});

  const attackBtn = this.add.text(250,550,"たたかう",{backgroundColor:"#444",padding:10})
  .setInteractive()
  .on("pointerdown", ()=> this.playerAttack());
};

BattleScene.prototype.playerAttack = function(){

  let damage = playerData.attack - this.enemy.defense;
  let isCrit = false;

  if(Math.random() < playerData.critRate){
    damage *= 2;
    isCrit = true;
  }

  damage = Math.max(1,damage);
  this.enemy.hp -= damage;

  // 会心演出
  if(isCrit){
    this.cameras.main.flash(150,255,255,255);
    this.cameras.main.shake(200,0.01);
    this.add.text(100,300,"かいしんのいちげき！",{fontSize:"24px",color:"#ffff00"});
  }

  this.add.text(180,250,"-"+damage,{fontSize:isCrit?"40px":"24px",color:"#ff0000"})
  .setOrigin(0.5);

  if(this.enemy.hp <= 0){
    saveGame();
    this.time.delayedCall(1000,()=>this.scene.start("MapScene"));
  }else{
    this.time.delayedCall(800,()=>this.enemyAttack());
  }
};

BattleScene.prototype.enemyAttack = function(){

  let damage = Math.max(1,this.enemy.attack - playerData.defense);
  playerData.hp -= damage;

  this.hpText.setText("HP:"+playerData.hp+"/"+playerData.maxHP);

  if(playerData.hp <= 0){
    alert("ゲームオーバー");
    playerData.hp = playerData.maxHP;
    saveGame();
    this.scene.start("MapScene");
  }
};

</script>

</body>
</html>
