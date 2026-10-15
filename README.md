<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Play & Earn Hub - Mini App</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
:root{--bg:#0a0f1e;--card:#141e35;--accent:#00d4ff;--gold:#ffcc00;--text:#fff}
*{margin:0;padding:0;box-sizing:border-box;font-family:'Inter',system-ui}
body{background:var(--bg);color:var(--text);overflow-x:hidden}
.header{position:sticky;top:0;z-index:10;background:rgba(10,15,30,.9);backdrop-filter:blur(12px);padding:12px 16px;display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid #1e3a5f}
.bal{font-weight:800;color:var(--gold)} .bal span{font-size:11px;opacity:.7}
.nav{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;padding:12px}
.nav button{background:var(--card);border:1px solid #1e3a5f;color:#fff;padding:10px 6px;border-radius:14px;font-size:12px}
.nav button.active{background:var(--accent);color:#000;font-weight:800}
.screen{padding:16px;display:none} .screen.active{display:block}
.card{background:var(--card);border-radius:20px;padding:16px;margin-bottom:12px;border:1px solid #1e3a5f}
.games{display:grid;grid-template-columns:repeat(2,1fr);gap:10px}
.game{background:linear-gradient(135deg,#1a2a4a,#0f1e38);border-radius:18px;padding:14px;text-align:center;border:1px solid #2a4a7a;cursor:pointer}
.game:active{transform:scale(.97)}
.btn{width:100%;background:var(--accent);color:#000;font-weight:800;border:none;padding:12px;border-radius:12px;margin-top:10px}
.btn-gold{background:var(--gold)}
.bet-row{display:flex;gap:8px;margin:10px 0}
.bet-row button{flex:1;background:#0f243e;color:#fff;border:1px solid #2a5a8a;padding:10px;border-radius:10px}
.timer{height:6px;background:#0f243e;border-radius:99px;overflow:hidden;margin:8px 0}
.timer i{display:block;height:100%;background:linear-gradient(90deg,#ff4444,#ffcc00);width:100%;transition:width 1s linear}
</style>
</head>
<body>
<div class="header">
  <div><b>🎮 PLAY & EARN</b> <span style="font-size:10px;opacity:.6">*** Coins</span></div>
  <div class="bal" id="bal">0 coins<br><span id="balUsd">$0 | 0⭐</span></div>
</div>
<div class="nav">
  <button onclick="show('home')" id="n-home" class="active">🏠 Home</button>
  <button onclick="show('games')" id="n-games">🎮 Games</button>
  <button onclick="show('wallet')" id="n-wallet">💰 Wallet</button>
  <button onclick="show('profile')" id="n-profile">👤 Profile</button>
</div>

<div id="home" class="screen active">
  <div class="card" style="background:linear-gradient(135deg,#00d4ff,#7b61ff)">
    <h3>💎 1$ = 100 coins</h3><p style="font-size:13px">50⭐ = 1$ → 1⭐ = 2 coins (Telegram rate)</p>
    <button class="btn btn-gold" onclick="show('wallet')">💳 Deposit Now</button>
  </div>
  <div class="card">👥 Refer: <b id="refLink" style="font-size:11px;word-break:break-all"></b><br><button class="btn" onclick="copyRef()">📤 Share</button></div>
  <div class="card">🏆 Leaderboard <div id="lbMini" style="font-size:13px;margin-top:8px"></div></div>
</div>

<div id="games" class="screen">
  <div class="card"><b>🎮 12 BET GAMES • 2s/5s/10s Timer • Fair</b><br><span style="font-size:12px;opacity:.7">Pick bet → answer fast or lose</span></div>
  <div class="games" id="gamesGrid"></div>
</div>

<div id="wallet" class="screen">
  <div class="card"><h3>💰 Wallet *** Coins</h3><div id="walletInfo"></div><p style="font-size:11px">💱 1$ = <b id="rCpd">100</b> coins • 1⭐ = <b id="rCps">2</b> coins</p></div>
  <div class="card"><b>💳 Deposit</b><button class="btn" onclick="depositStars(50)">⭐ 50⭐ → 100 coins (Instant)</button><button class="btn" style="background:#1a3a5a;color:#fff" onclick="showDepositManual('UPI')">💳 Manual UPI ($→coins)</button><button class="btn" style="background:#1a3a5a;color:#fff" onclick="showDepositManual('TRC20')">🪙 Crypto TRC20</button><div id="manualBox" style="margin-top:10px;background:#0a1a2e;padding:10px;border-radius:10px;font-size:12px;display:none"></div></div>
  <div class="card"><b>💸 Withdraw (coins→$)</b><input id="wdAmt" placeholder="Coins e.g. 500" style="width:100%;padding:10px;border-radius:10px;border:none;margin:8px 0"><input id="wdInfo" placeholder="UPI / TRC20" style="width:100%;padding:10px;border-radius:10px;border:none"><button class="btn btn-gold" onclick="withdraw()">Request Withdraw</button></div>
</div>

<div id="play" class="screen"><div class="card" id="playCard"></div><button class="btn" style="background:#333;color:#fff" onclick="show('games')">⬅️ Back to Games</button></div>

<div id="profile" class="screen"><div class="card" id="profileCard"></div><div class="card"><b>💬 Support</b><input id="supMsg" placeholder="Message admin..." style="width:100%;padding:10px;border-radius:10px;border:none;margin:8px 0"><button class="btn" onclick="sendSupport()">Send</button><div id="supHist" style="font-size:12px;margin-top:8px"></div></div></div>

<script>
let tg = window.Telegram?.WebApp; if(tg){tg.ready();tg.expand();}
let uid = tg?.initDataUnsafe?.user?.id || 123456;
let firstName = tg?.initDataUnsafe?.user?.first_name || "Player";
const WEBHOOK = "https://api.telebotcreator.com/new-webhook?data=TBCSECRET_secret_4bc286da";
let balance = parseFloat(localStorage.getItem('bal')||'20');
let cpd = 100, spd = 50, cps = 2;

function updateBal(){
  let usd=(balance/cpd).toFixed(2), stars=(balance/cps).toFixed(1);
  document.getElementById('bal').innerHTML = balance.toFixed(0)+' coins<br><span style="font-size:10px">$'+usd+' | '+stars+'⭐</span>';
  document.getElementById('walletInfo').innerHTML = '🪙 <b>'+balance.toFixed(0)+' coins</b> ≈ $'+usd;
  document.getElementById('profileCard').innerHTML = '<b>👤 '+firstName+'</b> ID:<code>'+uid+'</code><br>🪙 '+balance.toFixed(0)+' coins ≈ $'+usd+'<br>1$='+cpd+' coins';
  document.getElementById('rCpd').innerText=cpd; document.getElementById('rCps').innerText=cps.toFixed(2);
  document.getElementById('lbMini').innerHTML = '🥇 You — '+balance.toFixed(0)+' coins<br>🥈 Demo 420<br>🥉 Demo 310';
  document.getElementById('refLink').innerText = 'https://t.me/Playearn1millionbot?start='+uid;
}
function show(id){
  document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
  document.querySelectorAll('.nav button').forEach(b=>b.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  let n=document.getElementById('n-'+id); if(n) n.classList.add('active');
}
let gamesList=[
  {id:'miner',name:'⛏️ Miner',emoji:'⛏️',mult:1.8},{id:'wheel',name:'🎡 Wheel',emoji:'🎰',mult:2},{id:'gem',name:'💎 Gem',emoji:'💎',mult:2.5},{id:'number',name:'🔢 Number',emoji:'🧩',mult:2.2},{id:'snake',name:'🐍 Snake',emoji:'🐍',mult:2.8},{id:'quiz',name:'🧠 Quiz',emoji:'🏆',mult:2},{id:'treasure',name:'🗺️ Treasure',emoji:'🗝️',mult:3},{id:'coin',name:'🪙 Flip',emoji:'🪙',mult:2},{id:'dice',name:'🎲 Dice 5x',emoji:'🎲',mult:5},{id:'crash',name:'🚀 Crash',emoji:'🚀',mult:1.9},{id:'ludo',name:'🎲 Ludo',emoji:'🎲',mult:5},{id:'tap',name:'👆 Tap',emoji:'💨',mult:2}
];
let grid=document.getElementById('gamesGrid');
gamesList.forEach(g=>{
  let d=document.createElement('div'); d.className='game';
  d.innerHTML='<div style="font-size:28px">'+g.emoji+'</div><b>'+g.name+'</b><br><span style="font-size:11px">'+g.mult+'x</span>';
  d.onclick=()=>openGame(g.id); grid.appendChild(d);
});
let currentBet=0, deadline=0, hidden=0, activeGame='';
function openGame(id){
  let g=gamesList.find(x=>x.id===id); activeGame=id;
  document.getElementById('playCard').innerHTML='<h3>'+g.name+' • '+g.mult+'x</h3><p style="font-size:12px;opacity:.7">Balance: '+balance.toFixed(0)+' coins</p><div class="bet-row"><button onclick="setBet(5)">5</button><button onclick="setBet(10)">10</button><button onclick="setBet(25)">25</button><button onclick="setBet(50)">50</button></div><input id="customBet" placeholder="Custom coins" style="width:100%;padding:10px;border-radius:10px;border:none"><button class="btn" onclick="setBet(parseInt(document.getElementById(\'customBet\').value)||0)">Lock Bet</button><div id="gameArea" style="margin-top:12px"></div><div class="timer"><i id="tBar"></i></div><span id="tText" style="font-size:12px"></span>';
  show('play');
}
function setBet(v){
  if(v<1||v>balance){alert('Invalid bet');return;}
  currentBet=v; hidden=Math.floor(Math.random()*3);
  let timeout=[2,5,10][Math.floor(Math.random()*3)]; deadline=Date.now()+timeout*1000;
  let bar=document.getElementById('tBar'), txt=document.getElementById('tText');
  let iv=setInterval(()=>{let left=Math.max(0,deadline-Date.now()); if(bar) bar.style.width=(left/(timeout*1000)*100)+'%'; if(txt) txt.innerText='⏳ '+Math.ceil(left/1000)+'s'; if(left<=0){clearInterval(iv);}},200);
  renderGame();
}
function renderGame(){
  let a=document.getElementById('gameArea'); if(!a) return;
  if(activeGame==='miner') a.innerHTML='<div style="font-size:40px;text-align:center">⛏️💎</div><button class="btn" onclick="play()">⛏️ MINE!</button>';
  else if(activeGame==='wheel') a.innerHTML='<div style="width:140px;height:140px;border-radius:50%;background:conic-gradient(#ff4444 0 60deg,#ffcc00 60deg 120deg,#00d4ff 120deg 180deg,#7b61ff 180deg 240deg,#00ff88 240deg 300deg,#ff6b9d 300deg 360deg);margin:12px auto;border:4px solid #fff"></div><button class="btn" onclick="play()">🎡 SPIN!</button>';
  else if(activeGame==='gem') a.innerHTML='<p>3 boxes tap!</p><div style="display:flex;gap:8px"><button class="btn" onclick="play(0)">Box 1</button><button class="btn" onclick="play(1)">Box 2</button><button class="btn" onclick="play(2)">Box 3</button></div>';
  else if(activeGame==='snake') a.innerHTML='<div style="width:100%;height:100px;background:#001122;border-radius:12px;display:grid;place-items:center">🐍💨</div><div style="display:flex;gap:6px;margin-top:8px"><button class="btn" onclick="play(0)">⬆️</button><button class="btn" onclick="play(1)">➡️</button><button class="btn" onclick="play(2)">⬇️</button></div>';
  else if(activeGame==='coin') a.innerHTML='<div style="font-size:60px;text-align:center">🪙</div><button class="btn" onclick="play(\'head\')">HEAD</button><button class="btn" style="background:#222;color:#fff" onclick="play(\'tail\')">TAIL</button>';
  else if(activeGame==='dice') a.innerHTML='<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:6px">'+[1,2,3,4,5,6].map(n=>'<button class="btn" onclick="play('+n+')">'+n+'</button>').join('')+'</div>';
  else if(activeGame==='ludo') a.innerHTML='<div style="font-size:60px;text-align:center">🎲</div><button class="btn" onclick="play()">ROLL!</button>';
  else if(activeGame==='treasure') a.innerHTML='<div style="display:flex;gap:8px"><button class="btn" onclick="play(0)">📦1</button><button class="btn" onclick="play(1)">📦2</button><button class="btn" onclick="play(2)">📦3</button></div>';
  else if(activeGame==='crash') a.innerHTML='<div style="font-size:40px;text-align:center">🚀</div><button class="btn" onclick="play()">LAUNCH!</button>';
  else a.innerHTML='<button class="btn" onclick="play(0)">Play</button>';
}
function play(pick){
  if(Date.now()>deadline){balance-=currentBet; localStorage.setItem('bal',balance); updateBal(); alert('⏰ Time out!'); show('games'); return;}
  let win=Math.random()<0.4; if(['gem','treasure','snake'].includes(activeGame)) win=(pick===hidden)&&Math.random()<0.5;
  let g=gamesList.find(x=>x.id===activeGame);
  if(win){ balance+=currentBet*g.mult; alert('🎉 WIN +'+(currentBet*g.mult).toFixed(0)+' coins'); } else { balance-=currentBet; alert('😢 Lose -'+currentBet); }
  localStorage.setItem('bal',balance); updateBal();
  fetch(WEBHOOK+'&user_id='+uid, {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({action:'bet', game:activeGame, bet:currentBet, pick:pick, win:win})}).catch(()=>{});
  show('games');
}
function depositStars(s){
  fetch(WEBHOOK+'&user_id='+uid, {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({action:'deposit_stars', stars:s})}).catch(()=>{});
  alert('⭐ Invoice sent in bot chat for '+s+' ⭐');
}
function withdraw(){
  let a=parseFloat(document.getElementById('wdAmt').value), i=document.getElementById('wdInfo').value;
  if(!a||a>balance){alert('Invalid');return;}
  fetch(WEBHOOK+'&user_id='+uid, {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({action:'withdraw', amount:a, info:i})}).catch(()=>{});
  alert('Withdraw requested '+a+' coins');
}
function sendSupport(){
  let m=document.getElementById('supMsg').value; if(!m) return;
  fetch(WEBHOOK+'&user_id='+uid, {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({action:'support', msg:m})}).catch(()=>{});
  document.getElementById('supHist').innerHTML+='<div>You: '+m+'</div>'; document.getElementById('supMsg').value='';
}
function showDepositManual(t){ let b=document.getElementById('manualBox'); b.style.display='block'; b.innerHTML=t==='UPI'?'UPI: demo@upi<br>Pay $ → '+cpd+' coins per $':'TRC20: TX... demo address'; }
function copyRef(){ navigator.clipboard.writeText(document.getElementById('refLink').innerText); alert('Copied'); }
updateBal();
</script>
</body>
</html>
