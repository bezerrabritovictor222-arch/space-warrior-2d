# space-warrior-2d
Jogo espacial 2D — SPACE WARRIOR

<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>SPACE WARRIOR 2D</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#030516;color:white;font-family:Arial;overflow:hidden}
canvas{position:absolute;width:100%;height:100%;inset:0}
#hud{position:absolute;top:10px;left:10px;right:10px;display:none;pointer-events:none}
#hud button{pointer-events:auto}
.panel{position:absolute;inset:0;background:linear-gradient(#050923dd,#080319ee);display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:20px}
h1{font-size:clamp(28px,7vw,48px);color:#49eaff;text-shadow:0 0 20px #008cff}
p{color:#b9d9ff}
button{background:linear-gradient(135deg,#087bff,#7b31ff);border:1px solid #61eaff;border-radius:12px;padding:13px 20px;color:white;font-weight:bold;font-size:16px;margin:6px;cursor:pointer}
button:active{transform:scale(.96)}
.small{font-size:13px;color:#9db5e9}
#controls{position:absolute;bottom:18px;left:12px;right:12px;display:none;justify-content:space-between;align-items:end}
.pad{display:grid;grid-template-columns:48px 48px 48px;grid-template-rows:48px 48px 48px;gap:3px}
.pad button,.action{padding:0;width:48px;height:48px;background:#142b66bb;font-size:20px}
.pad .up{grid-column:2}.pad .left{grid-column:1;grid-row:2}.pad .down{grid-column:2;grid-row:2}.pad .right{grid-column:3;grid-row:2}
.actions{display:flex;flex-direction:column;align-items:center}
#fire{width:76px;height:76px;border-radius:50%;background:#e72d67bb}
#special{background:#087bffbb}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="hud">
<div id="stats">⭐ 0 | 🪙 0 | ❤️ 100</div>
<div id="progress">NÍVEL 1</div>
<button onclick="pauseGame()">Ⅱ PAUSA</button>
</div>
<div id="menu" class="panel">
<h1>🚀 SPACE WARRIOR</h1>
<p>DEFENDA A GALÁXIA!</p>
<button onclick="startGame()">▶ JOGAR</button>
<button onclick="hangar()">🛠 MELHORAR NAVE</button>
<button onclick="tutorial()">📖 TUTORIAL</button>
<button onclick="settings()">⚙ CONFIGURAÇÕES</button>
<p class="small">Seu recorde: <span id="record">0</span></p>
</div>
<div id="controls">
<div class="pad">
<button class="up" data-key="up">▲</button>
<button class="left" data-key="left">◀</button>
<button class="down" data-key="down">▼</button>
<button class="right" data-key="right">▶</button>
</div>
<div class="actions">
<button id="special" onclick="special()">⚡ ESPECIAL</button>
<button id="fire">🔥 ATIRAR</button>
</div>
</div>
<script>
const c=document.getElementById('game'),ctx=c.getContext('2d');
let W,H,stars=[],bullets=[],enemies=[],particles=[],coins=[],keys={};
let running=false,paused=false,score=0,money=0,life=100,level=1;
let ship={x:0,y:0,speed:260},last=0,spawn=0,shot=0,upgrade=0;
let difficulty=1,quality=true,autoFire=true,specialReady=true;
let record=Number(localStorage.getItem('swRecord')||0);
document.getElementById('record').textContent=record;
function resize(){W=c.width=innerWidth*devicePixelRatio;H=c.height=innerHeight*devicePixelRatio;ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);W=innerWidth;H=innerHeight;ship.x=W/2;ship.y=H*.78}
addEventListener('resize',resize);resize();
function rand(a,b){return Math.random()*(b-a)+a}
function starsInit(){stars=Array.from({length:100},()=>({x:rand(0,W),y:rand(0,H),s:rand(1,3)}))}
starsInit();
function show(id,on){document.getElementById(id).style.display=on?'flex':'none'}
function startGame(){
score=0;life=100;level=1;bullets=[];enemies=[];particles=[];coins=[];
ship.x=W/2;ship.y=H*.78;running=true;paused=false;
show('menu',false);show('controls',true);show('hud',true);last=performance.now();requestAnimationFrame(loop)
}
function pauseGame(){if(!running)return;paused=!paused;if(paused){show('menu',true);document.querySelector('#menu h1').textContent='JOGO PAUSADO';document.querySelector('#menu p').textContent='Sua aventura continua!';document.querySelector('#menu button').textContent='▶ CONTINUAR';document.querySelector('#menu button').onclick=()=>{show('menu',false);paused=false;last=performance.now();requestAnimationFrame(loop)}}}
function endGame(){
running=false;show('controls',false);show('hud',false);show('menu',true);
document.querySelector('#menu h1').textContent='MISSÃO ENCERRADA';
document.querySelector('#menu p').textContent=`Pontos: ${score} | Moedas: ${money}`;
document.querySelector('#menu button').textContent='▶ JOGAR NOVAMENTE';
document.querySelector('#menu button').onclick=()=>{document.querySelector('#menu h1').textContent='🚀 SPACE WARRIOR';document.querySelector('#menu button').onclick=startGame;startGame()};
if(score>record){record=score;localStorage.setItem('swRecord',record)}
document.getElementById('record').textContent=record;
}
function hangar(){show('menu',true);document.querySelector('#menu h1').textContent='🛠 HANGAR';document.querySelector('#menu p').textContent=`Moedas: ${money}. Melhoria de potência: nível ${upgrade+1}. Custo: ${(upgrade+1)*30} moedas.`;document.querySelector('#menu button').textContent='COMPRAR MELHORIA';document.querySelector('#menu button').onclick=()=>{if(money>=(upgrade+1)*30){money-=(upgrade+1)*30;upgrade++;document.querySelector('#menu p').textContent='Melhoria comprada! Potência aumentada.'}else document.querySelector('#menu p').textContent='Moedas insuficientes. Jogue para ganhar mais.'};}
function tutorial(){alert('Mova a nave usando as setas. Destrua os inimigos, desvie dos asteroides e ganhe moedas. A cada 500 pontos, o nível e o cenário mudam. O botão especial limpa os inimigos próximos.');}
function settings(){let d=prompt('Dificuldade: 1 fácil, 2 normal, 3 difícil',difficulty);if(d!==null&&[1,2,3].includes(Number(d)))difficulty=Number(d);autoFire=confirm('Ativar disparo automático?');quality=confirm('Usar efeitos gráficos avançados?');}
function fire(){if(!running||paused)return;bullets.push({x:ship.x,y:ship.y-22,v:520})}
let firing=false;
document.getElementById('fire').onpointerdown=e=>{firing=true;fire()};
document.getElementById('fire').onpointerup=()=>firing=false;
document.getElementById('fire').onpointerleave=()=>firing=false;
document.querySelectorAll('[data-key]').forEach(b=>{
b.onpointerdown=e=>{e.preventDefault();keys[b.dataset.key]=true};
b.onpointerup=b.onpointerleave=b.onpointercancel=()=>keys[b.dataset.key]=false;
});
addEventListener('keydown',e=>{keys[e.key.toLowerCase()]=true;if(e.code==='Space'){e.preventDefault();fire()}if(e.key==='Escape')pauseGame()});
addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);
function special(){if(!running||!specialReady)return;specialReady=false;enemies.forEach(e=>{score+=10;money+=2;burst(e.x,e.y)});enemies=[];setTimeout(()=>specialReady=true,5000)}
function burst(x,y){for(let i=0;i<(quality?12:5);i++)particles.push({x,y,vx:rand(-130,130),vy:rand(-130,130),life:rand(.2,.7)})}
function levelColor(){return ['#050923','#09264a','#321052','#54172e','#16433d','#241449'][Math.min(level-1,5)]}
function loop(t){
if(!running||paused)return;
let dt=Math.min((t-last)/1000,.035);last=t;
ctx.fillStyle=levelColor();ctx.fillRect(0,0,W,H);
stars.forEach(s=>{s.y+=s.s*level*dt*2;if(s.y>H){s.y=0;s.x=rand(0,W)}ctx.fillStyle='#bceaff';ctx.fillRect(s.x,s.y,s.s,s.s)});
if(keys.left||keys.a||keys.arrowleft)ship.x-=ship.speed*dt;
if(keys.right||keys.d||keys.arrowright)ship.x+=ship.speed*dt;
if(keys.up||keys.w||keys.arrowup)ship.y-=ship.speed*dt;
if(keys.down||keys.s||keys.arrowdown)ship.y+=ship.speed*dt;
ship.x=Math.max(18,Math.min(W-18,ship.x));ship.y=Math.max(40,Math.min(H-35,ship.y));
if(autoFire||firing){shot-=dt;if(shot<=0){fire();shot=.20/(1+upgrade*.12)}}
spawn-=dt;if(spawn<=0){enemies.push({x:rand(25,W-25),y:-25,r:rand(12,22),v:rand(65,110)*difficulty+level*7,hp:1});spawn=rand(.4,1.1)/difficulty}
bullets=bullets.filter(b=>b.y>-20);bullets.forEach(b=>{b.y-=b.v*dt;ctx.fillStyle='#56faff';ctx.fillRect(b.x-2,b.y,4,14)});
enemies.forEach(e=>{
e.y+=e.v*dt;
ctx.fillStyle=e.r>17?'#ff426d':'#ffae4a';ctx.beginPath();ctx.moveTo(e.x,e.y+e.r);ctx.lineTo(e.x-e.r,e.y-e.r);ctx.lineTo(e.x+e.r,e.y-e.r);ctx.closePath();ctx.fill();
bullets.forEach(b=>{if(!b.hit&&Math.hypot(b.x-e.x,b.y-e.y)<e.r+5){b.hit=true;e.hp--;burst(e.x,e.y);if(e.hp<=0){e.dead=true;score+=10;money+=1+Math.floor(level/2)}}});
if(Math.hypot(ship.x-e.x,ship.y-e.y)<e.r+14){e.dead=true;life-=15;burst(e.x,e.y)}
});
enemies=enemies.filter(e=>!e.dead&&e.y<H+30);bullets=bullets.filter(b=>!b.hit);
particles.forEach(p=>{p.x+=p.vx*dt;p.y+=p.vy*dt;p.life-=dt;ctx.fillStyle='#ffdf70';ctx.fillRect(p.x,p.y,3,3)});
particles=particles.filter(p=>p.life>0);
ctx.save();ctx.translate(ship.x,ship.y);
ctx.fillStyle='#48eaff';ctx.beginPath();ctx.moveTo(0,-24);ctx.lineTo(-17,17);ctx.lineTo(0,9);ctx.lineTo(17,17);ctx.closePath();ctx.fill();
ctx.fillStyle='#fff';ctx.fillRect(-4,3,8,12);ctx.restore();
level=1+Math.floor(score/500);
document.getElementById('stats').textContent=`⭐ ${score} | 🪙 ${money} | ❤️ ${Math.max(0,life)}`;
document.getElementById('progress').textContent=`NÍVEL ${level} · ${score%500}/500 PARA O PRÓXIMO`;
if(life<=0){endGame();return}
requestAnimationFrame(loop)
}
</script>
</body>
</html>
