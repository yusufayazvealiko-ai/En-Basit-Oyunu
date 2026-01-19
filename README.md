# En-Basit-Oyunu
Bu Dünya'nın en basit oyun. Bir şehirde yürüme oyunu. 3D bile değil 2D. Bu Dünyada ki en basit oyun olabilir
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<title>Şehir Oyunu</title>
<style>
  body {margin:0; overflow:hidden; background:#70a046;}
  canvas {display:block;}
  #ui {position:absolute; top:0; left:0; color:white; font-family:sans-serif;}
  #buttons button {width:60px; height:60px; margin:2px; font-size:14px;}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="ui">
  <div id="stats">Can: 100 | Para: 0</div>
  <div id="buttons">
    <button id="up">↑</button><br>
    <button id="left">←</button>
    <button id="down">↓</button>
    <button id="right">→</button>
    <button id="jump">Zıpla</button>
    <button id="shoot">Ateş</button>
  </div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// =================== HARİTA ===================
const MAP_W = 3000;
const MAP_H = 3000;

// =================== OYUNCU ===================
let player = {x:1500, y:1500, w:40, h:40, hp:100, money:0, jump:false, inv:false, hasGun:false};

// =================== TUŞLAR ===================
let keys = {up:false, down:false, left:false, right:false};
document.getElementById('up').addEventListener('mousedown',()=>keys.up=true);
document.getElementById('up').addEventListener('mouseup',()=>keys.up=false);
document.getElementById('down').addEventListener('mousedown',()=>keys.down=true);
document.getElementById('down').addEventListener('mouseup',()=>keys.down=false);
document.getElementById('left').addEventListener('mousedown',()=>keys.left=true);
document.getElementById('left').addEventListener('mouseup',()=>keys.left=false);
document.getElementById('right').addEventListener('mousedown',()=>keys.right=true);
document.getElementById('right').addEventListener('mouseup',()=>keys.right=false);

document.getElementById('jump').addEventListener('click',()=>{
  player.jump=true; player.inv=true;
  setTimeout(()=>{player.jump=false; player.inv=false},500);
});

document.getElementById('shoot').addEventListener('click',()=>{
  if(player.hasGun) bullets.push({x:player.x+player.w/2, y:player.y+player.h/2, dx:10, dy:0});
});

// =================== NESNELER ===================
let buildings=[], pools=[], trees=[], people=[], cats=[], police=[], thieves=[], bullets=[], cars=[];

// Binalar ve havuz
for(let i=0;i<12;i++){
  let bx=Math.random()*2600+200, by=Math.random()*2600+200;
  buildings.push({x:bx,y:by,w:260,h:220});
  if(Math.random()>0.5) pools.push({x:bx+60,y:by+140,w:80,h:40});
}

// Ağaçlar
for(let i=0;i<40;i++) trees.push({x:Math.random()*MAP_W,y:Math.random()*MAP_H});

// NPC
for(let i=0;i<30;i++) people.push({x:Math.random()*MAP_W,y:Math.random()*MAP_H,dx:Math.random()>0.5?1:-1,dy:Math.random()>0.5?1:-1,talk:false});

// Kediler
for(let i=0;i<15;i++) cats.push({x:Math.random()*MAP_W,y:Math.random()*MAP_H,dx:Math.random()>0.5?2:-2,dy:Math.random()>0.5?2:-2});

// Polis ve Hırsız
for(let i=0;i<4;i++) police.push({x:Math.random()*MAP_W,y:Math.random()*MAP_H});
thieves.push({x:1500,y:1500});

// Arabalar
for(let i=0;i<16;i++){
  cars.push({x:Math.random()*MAP_W,y:Math.random()*MAP_H,dx:Math.random()>0.5?3:-3,dy:0,c:`rgb(${Math.random()*195+60},${Math.random()*195+60},${Math.random()*195+60})`});
}

// Banka ve paralar
let bank={x:1800,y:1200,w:220,h:180};
let bank_money=[];
for(let i=0;i<5;i++) bank_money.push({x:bank.x+30+i*30,y:bank.y+80,w:20,h:20});

// Silah
let gun={x:buildings[0].x+100,y:buildings[0].y+260,w:30,h:15};

// Kamera dönüşümü
function worldToScreen(x,y){
  return {x:x-player.x+canvas.width/2, y:y-player.y+canvas.height/2};
}

// =================== OYUN DÖNGÜSÜ ===================
function update(){
  // hareket
  let speed=4;
  if(keys.up) player.y-=speed;
  if(keys.down) player.y+=speed;
  if(keys.left) player.x-=speed;
  if(keys.right) player.x+=speed;

  // ekran temizle
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // yollar
  for(let y=0;y<MAP_H;y+=200){
    let sy=worldToScreen(0,y).y;
    ctx.fillStyle="#5a5a5a";
    ctx.fillRect(0,sy,MAP_W,80);
    for(let x=0;x<MAP_W;x+=120){
      ctx.strokeStyle="#fff"; ctx.lineWidth=4;
      let sx1=worldToScreen(x,y+40).x;
      let sx2=worldToScreen(x+60,y+40).x;
      let sy1=worldToScreen(x,y+40).y;
      ctx.beginPath();
      ctx.moveTo(sx1,sy1);
      ctx.lineTo(sx2,sy1);
      ctx.stroke();
    }
  }

  // binalar
  ctx.fillStyle="#8c8c9f";
  buildings.forEach(b=>{let s=worldToScreen(b.x,b.y); ctx.fillRect(s.x,s.y,b.w,b.h);});

  // havuz
  ctx.fillStyle="#0078c8";
  pools.forEach(p=>{let s=worldToScreen(p.x,p.y); ctx.fillRect(s.x,s.y,p.w,p.h);});

  // banka
  ctx.fillStyle="#787878";
  let s=worldToScreen(bank.x,bank.y);
  ctx.fillRect(s.x,s.y,bank.w,bank.h);
  ctx.fillStyle="gold";
  bank_money.forEach(m=>{let sm=worldToScreen(m.x,m.y); ctx.fillRect(sm.x,sm.y,m.w,m.h);});

  // NPC
  people.forEach(p=>{
    if(Math.abs(p.x-player.x)<100 && Math.abs(p.y-player.y)<100){
      p.talk=true;
      let st=worldToScreen(p.x,p.y-20);
      ctx.fillStyle="white";
      ctx.fillText("Merhaba!",st.x,st.y);
    }else{
      p.talk=false;
      p.x+=p.dx; p.y+=p.dy;
      if(Math.random()<0.01){p.dx=Math.random()>0.5?1:-1; p.dy=Math.random()>0.5?1:-1;}
    }
    let sp=worldToScreen(p.x,p.y);
    ctx.fillStyle="green"; ctx.fillRect(sp.x,sp.y,32,32);
  });

  // Kediler
  cats.forEach(c=>{
    c.x+=c.dx; c.y+=c.dy;
    let sc=worldToScreen(c.x,c.y);
    ctx.fillStyle="red"; ctx.fillRect(sc.x,sc.y,18,18);
    // kulaklar
    ctx.beginPath();
    ctx.moveTo(sc.x+2,sc.y);
    ctx.lineTo(sc.x+6,sc.y-8);
    ctx.lineTo(sc.x+10,sc.y); ctx.fill();
    ctx.beginPath();
    ctx.moveTo(sc.x+8,sc.y);
    ctx.lineTo(sc.x+12,sc.y-8);
    ctx.lineTo(sc.x+16,sc.y); ctx.fill();
  });

  // karakter
  let pl=worldToScreen(player.x,player.y);
  ctx.fillStyle="red"; ctx.fillRect(pl.x,pl.y,player.w,player.h);

  requestAnimationFrame(update);
}
update();
</script>
</body>
</html>
