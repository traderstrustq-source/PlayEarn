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
.game{background:linear-gradient(135deg,#1a2a4a,#0f1e38);border-radius:18px;padding:14px;text-align:center;border:1px solid #2a4a7a;cursor:pointer;transition:.2s}
.game:active{transform:scale(.96)}
.btn{width:100%;background:var(--accent);color:#000;font-weight:800;border:none;padding:12px;border-radius:12px;margin-top:10px}
.btn-gold{background:var(--gold)}
.bet-row{display:flex;gap:8px;margin:10px 0}
.bet-row button{flex:1;background:#0f243e;color:#fff;border:1px solid #2a5a8a;padding:10px;border-radius:10px}
.timer{height:6px;background:#0f243e;border-radius:99px;overflow:hidden;margin:8px 0}
.timer i{display:block;height:100%;background:linear-gradient(90deg,#ff4444,#ffcc00);width:100%;transition:width 1s linear}
.spoiler{background:#000;color:transparent;border-radius:8px;padding:6px 12px}
/* Animations */
@keyframes mine{0%{transform:translateY(0) rotate(0)}50%{transform:translateY(-4px) rotate(-5deg)}100%{transform:translateY(0)}}
.mine-anim{animation:mine .3s infinite}
@keyframes spin{from{transform:rotate(0)}to{transform:rotate(360deg)}}
.wheel{width:140px;height:140px;border-radius:50%;background:conic-gradient(#ff4444 0 60deg,#ffcc00 60deg 120deg,#00d4ff 120deg 180deg,#7b61ff 180deg 240deg,#00ff88 240deg 300deg,#ff6b9d 300deg 360deg);margin:12px auto;border:4px solid #fff;position:relative}
@keyframes flip{0%{transform:rotateY(0)}100%{transform:rotateY(720deg)}}
.coin{width:80px;height:80px;border-radius:50%;background:radial-gradient(circle,#ffcc00,#ff9900);margin:12px auto;display:grid;place-items:center;font-size:32px;border:3px solid #fff}
.coin.flip{animation:flip .9s ease}
.snake-canv{width:100%;height:160px;background:#001122;border-radius:12px}
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

<!-- HOME -->
<div id="home" class="screen active">
  <div class="card" style="background:linear-gradient(135deg,#00d4ff,#7b61ff)">
    <h3>💎 1$ = 100 coins</h3><p style="font-size:13px">50⭐ = 1$ → 1⭐ = 2 coins (Telegram rate)</p>
    <p style="font-size:11px;opacity:.8">Deposit $ or Stars → coins. Play 12 games with coins!</p>
    <button class="btn btn-gold" onclick="show('wallet')">💳 Deposit Now</button>
  </div>
  <div class="card">👥 Refer & Earn: <b id="refLink" style="font-size:11px;word-break:break-all"></b><br><button class="btn" onclick="copyRef()">📤 Share</button></div>
  <div class="card">🏆 Leaderboard <div id="lbMini" style="font-size:13px;margin-top:8px"></div></div>
</div>

<!-- GAMES -->
<div id="games" class="screen">
  <div class="card"><b>🎮 12 BET GAMES • 2s/5s/10s Timer • Fair Spoiler</b><br><span style="font-size:12px;opacity:.7">Pick bet → answer fast or lose. Admin rates hidden, looks fair.</span></div>
  <div class="games" id="gamesGrid"></div>
</div>

<!-- WALLET -->
<div id="wallet" class="screen">
  <div class="card"><h3>💰 *** COINS Wallet</h3><div id="walletInfo" style="margin:8px 0;font-size:14px"></div>
    <div class="timer"><i id="rateBar" style="width:70%"></i></div>
    <p style="font-size:11px">💱 1$ = <b id="rCpd">100</b> coins • 1⭐ = <b id="rCps">2</b> coins</p>
  </div>
  <div class="card">
    <b>💳 Deposit</b>
    <button class="btn" onclick="depositStars(50)">⭐ 50⭐ → 100 coins (Instant)</button>
    <button class="btn" style="background:#1a3a5a;color:#fff" onclick="showDepositManual('UPI/Bank')">💳 Manual UPI/Bank ($→coins)</button>
    <button class="btn" style="background:#1a3a5a;color:#fff" onclick="showDepositManual('TRC20')">🪙 Manual Crypto TRC20</button>
    <div id="manualBox" style="margin-top:10px;background:#0a1a2e;padding:10px;border-radius:10px;font-size:12px;display:none"></div>
  </div>
  <div class="card"><b>💸 Withdraw (coins → $)</b><br><input id="wdAmt" placeholder="Coins e.g. 500" style="width:100%;padding:10px;border-radius:10px;border:none;margin:8px 0"><input id="wdInfo" placeholder="UPI / TRC20 address" style="width:100%;padding:10px;border-radius:10px;border:none"><button class="btn btn-gold" onclick="withdraw()">Request Withdraw</button></div>
</div>

<!-- GAME PLAY MODAL -->
<div id="play" class="screen">
  <div class="card" id="playCard"></div>
  <button class="btn" style="background:#333;color:#fff" onclick="show('games')">⬅️ Back to Games</button>
</div>

<!-- PROFILE / LEADERBOARD / SUPPORT -->
<div id="profile" class="screen">
  <div class="card" id="profileCard"></div>
  <div class="card"><b>💬 Support</b><br><input id="supMsg" placeholder="Message admin..." style="width:100%;padding:10px;border-radius:10px;border:none;margin:8px 0"><button class="btn" onclick="sendSupport()">Send</button><div id="supHist" style="font-size:12px;margin-top:8px;opacity:.8"></div></div>
  <div class="card" id="adminCard" style="display:none;background:#1a0a0a;border-color:#ff4444"><b>🔐 Admin Panel</b><div id="adminBox" style="margin-top:8px"></div></div>
</div>

<script>
let tg = window.Telegram?.WebApp; if(tg){tg.ready();tg.expand();}
let uid = tg?.initDataUnsafe?.user?.id || 123456;
let firstName = tg?.initDataUnsafe?.user?.first_name || "Player";
let balance = parseFloat(localStorage.getItem('bal')||'20');
let cpd = parseFloat(localStorage.getItem('cpd')||'100'); // coins per dollar
let spd = parseFloat(localStorage.getItem('spd')||'50'); // stars per dollar
let cps = cpd/spd;
let wins={}, played={};
let ref = localStorage.getItem('ref_'+uid)||'0';
document.getElementById('refLink').innerText = `https://t.me/Playearn1millionbot?start=${uid}`;
document.getElementById('rCpd').innerText=cpd; document.getElementById('rCps').innerText=cps.toFixed(2);

function saveBal(){localStorage.setItem('bal',balance);updateBal();}
function updateBal(){
  let usd=(balance/cpd).toFixed(2), stars=(balance/cps).toFixed(1);
  document.getElementById('bal').innerHTML = `${balance.toFixed(0)} coins<br><span style="font-size:10px">$${usd} | ${stars}⭐</span>`;
  document.getElementById('walletInfo').innerHTML = `🪙 <b>${balance.toFixed(0)} coins</b> ≈ $${usd}<br>📥 Earned: ${(parseFloat(localStorage.getItem('earned')||0)).toFixed(0)}`;
  document.getElementById('profileCard').innerHTML = `<b>👤 ${firstName}</b> ID:<code>${uid}</code><br>🪙 ${balance.toFixed(0)} coins ≈ $${usd}<br>1$=${cpd} coins<br>Games: ${Object.entries(played).map(([k,v])=>`${k}:${v}`).join(' ')||'None'}`;
  // admin visible if id in list (set your 4 ids)
  let admins=[uid]; // replace with your 4 ids: [123,456,789,101]
  if(admins.includes(uid)){document.getElementById('adminCard').style.display='block'; document.getElementById('adminBox').innerHTML=`💱 Rates: 1$=${cpd} | 1⭐=${cps.toFixed(2)}<br><button class="btn" onclick="editRates()">✏️ Edit Rates</button><hr><button class="btn" onclick="addFunds()">💰 Add 100 coins (demo)</button>`;}
  document.getElementById('lbMini').innerHTML = `🥇 You — ${balance.toFixed(0)} coins<br>🥈 Demo 420<br>🥉 Demo 310`;
}
function show(id){
  document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
  document.querySelectorAll('.nav button').forEach(b=>b.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  let n=document.getElementById('n-'+id); if(n) n.classList.add('active');
  if(id==='home')updateBal();
}
let gamesList=[
  {id:'miner',name:'⛏️ Miner',emoji:'⛏️',mult:1.8},{id:'wheel',name:'🎡 Wheel',emoji:'🎰',mult:2},{id:'gem',name:'💎 Gem',emoji:'💎',mult:2.5},{id:'number',name:'🔢 Number',emoji:'🧩',mult:2.2},{id:'snake',name:'🐍 Snake',emoji:'🐍',mult:2.8},{id:'quiz',name:'🧠 Quiz',emoji:'🏆',mult:2},{id:'treasure',name:'🗺️ Treasure',emoji:'🗝️',mult:3},{id:'coin',name:'🪙 Flip',emoji:'🪙',mult:2},{id:'dice',name:'🎲 Dice 5x',emoji:'🎲',mult:5},{id:'crash',name:'🚀 Crash',emoji:'🚀',mult:1.9},{id:'ludo',name:'🎲 Ludo',emoji:'🎲',mult:5},{id:'tap',name:'👆 Tap',emoji:'💨',mult:2},
];
let grid=document.getElementById('gamesGrid');
gamesList.forEach(g=>{
  let d=document.createElement('div'); d.className='game'; d.innerHTML=`<div style="font-size:28px">${g.emoji}</div><b>${g.name}</b><br><span style="font-size:11px">${g.mult}x • Bet</span>`;
  d.onclick=()=>openGame(g.id); grid.appendChild(d);
});

let currentBet=0, deadline=0, hidden=0, correct=0;
function openGame(id){
  let g=gamesList.find(x=>x.id===id);
  currentBet=0;
  document.getElementById('playCard').innerHTML = `
    <h3>${g.name} • ${g.mult}x</h3>
    <p style="font-size:12px;opacity:.7">💰 Balance: ${balance.toFixed(0)} coins • Choose bet (2s/5s/10s timer!)</p>
    <div class="bet-row"><button onclick="setBet(5)">5</button><button onclick="setBet(10)">10</button><button onclick="setBet(25)">25</button><button onclick="setBet(50)">50</button></div>
    <input id="customBet" placeholder="Custom coins" style="width:100%;padding:10px;border-radius:10px;border:none"><button class="btn" onclick="setBet(parseInt(document.getElementById('customBet').value)||0)">Lock Bet</button>
    <div id="gameArea" style="margin-top:12px"></div>
    <div class="timer"><i id="tBar"></i></div><span id="tText" style="font-size:12px"></span>`;
  show('play');
  window.activeGame=id;
}
function setBet(v){
  if(v<1||v>balance){alert('Invalid bet / balance');return;}
  currentBet=v;
  cps=cpd/spd;
  let timeout=[2,5,10][Math.floor(Math.random()*3)];
  deadline=Date.now()+timeout*1000;
  hidden=Math.floor(Math.random()*3);
  correct=Math.floor(Math.random()*4);
  // start timer
  let bar=document.getElementById('tBar'), txt=document.getElementById('tText');
  let iv=setInterval(()=>{let left=Math.max(0,deadline-Date.now()); let pct=left/(timeout*1000)*100; if(bar) bar.style.width=pct+'%'; if(txt) txt.innerText=`⏳ ${Math.ceil(left/1000)}s left - answer fast!`; if(left<=0){clearInterval(iv); lose('⏰ Time out!')}} ,200);
  renderGame();
  // send to bot via WebApp
  if(tg) try{tg.sendData(JSON.stringify({action:'bet',game:activeGame,bet:v}))}catch(e){}
}
function renderGame(){
  let area=document.getElementById('gameArea');
  if(!area) return;
  let g=activeGame;
  if(g==='miner') area.innerHTML=`<div class="mine-anim" style="font-size:40px;text-align:center">⛏️💎</div><p style="text-align:center">Mining coins...</p><button class="btn" onclick="play()">⛏️ MINE NOW!</button>`;
  else if(g==='wheel') area.innerHTML=`<div class="wheel" id="wheel" style="transition:transform 2s cubic-bezier(.2,.8,.2,1)"></div><button class="btn" onclick="play()">🎡 SPIN NOW!</button>`;
  else if(g==='gem') area.innerHTML=`<p>3 boxes <span class="spoiler">gem in ${hidden+1}</span> tap to reveal!</p><div style="display:flex;gap:8px"><button class="btn" onclick="play(0)">🎁 Box 1</button><button class="btn" onclick="play(1)">🎁 Box 2</button><button class="btn" onclick="play(2)">🎁 Box 3</button></div><div style="display:flex;gap:6px;margin-top:8px;justify-content:center"><span style="animation:spin 1s linear infinite">💎</span><span style="animation:spin 1.2s linear infinite">🔷</span><span style="animation:spin 1.4s linear infinite">🔶</span> shuffling</div>`;
  else if(g==='snake') area.innerHTML=`<canvas class="snake-canv" id="snakeCanvas"></canvas><div style="display:flex;gap:6px;margin-top:8px"><button class="btn" onclick="play(0)">⬆️</button><button class="btn" onclick="play(1)">➡️</button><button class="btn" onclick="play(2)">⬇️</button></div>`;
  else if(g==='coin') area.innerHTML=`<div class="coin" id="coin">🪙</div><button class="btn" onclick="play('head')">🪙 HEAD</button><button class="btn" style="background:#222;color:#fff" onclick="play('tail')">🥈 TAIL</button>`;
  else if(g==='dice') area.innerHTML=`<div style="font-size:50px;text-align:center">🎲</div><div style="display:grid;grid-template-columns:repeat(3,1fr);gap:6px">${[1,2,3,4,5,6].map(n=>`<button class="btn" onclick="play(${n})">${n}</button>`).join('')}</div>`;
  else if(g==='ludo') area.innerHTML=`<div style="font-size:60px;text-align:center">🎲</div><button class="btn" onclick="play()">🎲 ROLL DICE!</button>`;
  else if(g==='treasure') area.innerHTML=`<div style="display:flex;gap:8px"><button class="btn" onclick="play(0)">📦 1</button><button class="btn" onclick="play(1)">📦 2</button><button class="btn" onclick="play(2)">📦 3</button></div>`;
  else if(g==='crash') area.innerHTML=`<div style="text-align:center;font-size:40px">🚀</div><button class="btn" onclick="play()">🚀 LAUNCH!</button>`;
  else if(g==='number') {
    let a=Math.floor(Math.random()*80)+12,b=Math.floor(Math.random()*20)+7; correct=(a*b)%100; area.innerHTML=`<p><b>${a} × ${b%9+2} mod 100 = ?</b></p><div style="display:grid;grid-template-columns:repeat(2,1fr);gap:6px">${[correct,correct+7,correct-5,correct+11].sort(()=>Math.random()-.5).map(n=>`<button class="btn" onclick="play(${n},${correct})">${n}</button>`).join('')}</div>`;
  }
  else if(g==='quiz') {
    let qs=["3^x=81 x?","Derivative x³?","sin90°?","log2 1024?"]; let q=qs[Math.floor(Math.random()*qs.length)]; area.innerHTML=`<p><b>${q}</b></p><div style="display:grid;grid-template-columns:repeat(2,1fr);gap:6px"><button class="btn" onclick="play(0)">A</button><button class="btn" onclick="play(1)">B</button><button class="btn" onclick="play(0)">C</button><button class="btn" onclick="play(1)">D</button></div>`;
  }
  else if(g==='tap') area.innerHTML=`<div style="text-align:center"><div style="width:120px;height:120px;border-radius:50%;background:var(--accent);margin:12px auto;display:grid;place-items:center;font-size:30px" onclick="play()">👆 TAP</div><p>Tap 10 times in 30s!</p></div>`;
  // snake animation
  if(g==='snake'){ setTimeout(()=>{let c=document.getElementById('snakeCanvas'); if(!c)return; let ctx=c.getContext('2d'); let x=0; setInterval(()=>{ctx.clearRect(0,0,c.width,c.height); ctx.fillStyle='#00ff88'; ctx.fillRect(x,70,30,10); ctx.fillStyle='#ff0040'; ctx.beginPath(); ctx.arc(150,75,8,0,6.28);ctx.fill(); x=(x+4)%300;},30);},100);}
}
function play(pick, ans){
  if(Date.now()>deadline) return lose('⏰ Too slow!');
  // animations
  if(activeGame==='wheel') document.getElementById('wheel').style.transform='rotate('+ (720+Math.random()*720) +'deg)';
  if(activeGame==='coin'){ let c=document.getElementById('coin'); c.classList.add('flip'); setTimeout(()=>c.classList.remove('flip'),900);}
  if(activeGame==='miner') { let e=document.querySelector('.mine-anim'); if(e) e.style.transform='scale(1.2)'; }
  // win logic - 40% base, admin winrate hidden
  let win = Math.random()<0.4;
  if(activeGame==='gem' || activeGame==='treasure' || activeGame==='snake') win = (pick===hidden) && Math.random()<0.5;
  if(activeGame==='dice') win = (pick===Math.floor(Math.random()*6)+1);
  if(activeGame==='ludo') win = Math.floor(Math.random()*6)+1===6;
  let g=gamesList.find(x=>x.id===activeGame); let mult=g.mult;
  played[g.id]=(played[g.id]||0)+1;
  if(win){ let profit=currentBet*mult; balance+=profit; wins[g.id]=(wins[g.id]||0)+1; saveBal(); setTimeout(()=>{alert(`🎉 WIN +${profit.toFixed(0)} coins!`); show('games');},600);} else { balance-=currentBet; saveBal(); setTimeout(()=>{alert(`😢 Lose -${currentBet} coins`); show('games');},600);}
  localStorage.setItem('earned', (parseFloat(localStorage.getItem('earned')||0)+(win?currentBet*mult:0)));
  if(tg) try{tg.sendData(JSON.stringify({action:'result',game:activeGame,bet:currentBet,win}))}catch(e){}
}
function lose(msg){ balance-=currentBet; saveBal(); alert(msg+' -'+currentBet); show('games');}
function depositStars(stars){ let coins=Math.floor(stars*cps); if(tg) tg.sendData(JSON.stringify({action:'deposit_stars',stars,coins})); else {balance+=coins; saveBal(); alert(`⭐ ${stars} → ${coins} coins (demo)`);} }
function showDepositManual(type){ let box=document.getElementById('manualBox'); box.style.display='block'; box.innerHTML= type==='UPI/Bank' ? `💳 UPI: demo@upi<br>Bank: HDFC 123... IFSC HDFC001<br>Pay $ → ${cpd} coins per $` : `🪙 TRC20: TX... demo address<br>Network: TRON<br>Min $10 → ${10*cpd} coins`; }
function withdraw(){ let amt=parseFloat(document.getElementById('wdAmt').value); let info=document.getElementById('wdInfo').value; if(!amt||amt>balance){alert('Invalid');return;} balance-=amt; saveBal(); if(tg) tg.sendData(JSON.stringify({action:'withdraw',amt,info})); alert('Withdraw requested '+amt+' coins');}
function sendSupport(){ let m=document.getElementById('supMsg').value; if(!m) return; if(tg) tg.sendData(JSON.stringify({action:'support',msg:m})); document.getElementById('supHist').innerHTML+=`<div>You: ${m}</div>`; document.getElementById('supMsg').value='';}
function copyRef(){ navigator.clipboard.writeText(document.getElementById('refLink').innerText); alert('Copied!');}
function editRates(){ let n=prompt('1$ = ? coins',cpd); if(n){cpd=parseFloat(n); localStorage.setItem('cpd',cpd); cps=cpd/spd; document.getElementById('rCpd').innerText=cpd; updateBal();}}
function addFunds(){balance+=100; saveBal();}
updateBal();
</script>
</body>
</html>
