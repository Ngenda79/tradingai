<!DOCTYPE html>
<html>
<head>
<title>Jesus is King Trading AI</title>

<style>
body{background:#020617;color:white;font-family:Arial;text-align:center;}
h2{color:#38bdf8;}
.card{background:#0f172a;border:1px solid #1e293b;padding:12px;margin:10px auto;width:95%;max-width:950px;border-radius:12px;}
button{padding:10px;margin:5px;border:none;border-radius:6px;font-weight:bold;cursor:pointer;}
.buy{background:#22c55e;}
.sell{background:#ef4444;}
.exec{background:#f59e0b;}
.win{background:#3b82f6;}
.loss{background:#ef4444;}
.reset{background:#64748b;}
.print{background:#06b6d4;}
select,input{padding:6px;margin:5px;}
table{width:100%;border-collapse:collapse;font-size:12px;}
th,td{border:1px solid #1e293b;padding:6px;}
.green{color:#22c55e;}
.red{color:#ef4444;}
</style>
</head>

<body>

<h2>Jesus is King Trading AI</h2>

<div class="card">
<h3>Balance: $<span id="balance">50</span></h3>

Daily Goal: $
<input id="goalInput" value="62" onchange="setGoal()">

<p>Progress: <span id="progress">0</span>%</p>
<p id="goalStatus">Not reached</p>

<p>Session Time (Kampala): <span id="time"></span></p>
</div>

<!-- TRADE SETUP -->
<div class="card">
<h3>Quick Trade Setup</h3>

<select id="pairInput">
<option value="">Select Pair</option>

<!-- NORMAL -->
<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>USD/CHF</option>
<option>AUD/USD</option>
<option>USD/CAD</option>
<option>EUR/GBP</option>

<option>EUR/JPY</option>
<option>GBP/JPY</option>
<option>AUD/CAD</option>
<option>AUD/CHF</option>
<option>CAD/JPY</option>
<option>CHF/JPY</option>
<option>EUR/AUD</option>
<option>EUR/CAD</option>
<option>EUR/CHF</option>
<option>GBP/AUD</option>
<option>GBP/CAD</option>
<option>GBP/CHF</option>
<option>CAD/CHF</option>
<option>AUD/JPY</option>

<!-- OTC -->
<option>EUR/USD OTC</option>
<option>GBP/USD OTC</option>
<option>USD/JPY OTC</option>
<option>USD/CHF OTC</option>
<option>AUD/USD OTC</option>
<option>USD/CAD OTC</option>
<option>EUR/GBP OTC</option>

<option>AUD/CAD OTC</option>
<option>AUD/CHF OTC</option>
<option>AUD/NZD OTC</option>
<option>CAD/CHF OTC</option>
<option>CAD/JPY OTC</option>
<option>CHF/JPY OTC</option>
<option>EUR/JPY OTC</option>
<option>EUR/NZD OTC</option>
<option>LBP/USD OTC</option>
<option>GBP/JPY OTC</option>

</select>

<br>

Payout %:
<input id="payoutInput" type="number" placeholder="e.g 82">

<br>

<button class="buy" onclick="setSignal('BUY')">BUY</button>
<button class="sell" onclick="setSignal('SELL')">SELL</button>

<p id="quickStatus">No signal selected</p>
</div>

<div class="card">
<p>Pair: <span id="pair">---</span></p>
<p>Signal: <span id="signal">---</span></p>
<p>Confidence: <span id="score">--</span>%</p>
<p>Payout: <span id="payout">--</span>%</p>
<p>Next Stake: $<span id="stake">0</span></p>

<p id="status">READY</p>
<p id="lock"></p>
</div>

<div class="card">
<button class="exec" id="executeBtn" onclick="executeTrade()" disabled>EXECUTE</button>
<button class="win" onclick="setResult('win')">WIN</button>
<button class="loss" onclick="setResult('loss')">LOSS</button>
<br>
<button class="reset" onclick="resetSession()">RESET SESSION</button>
<button class="print" onclick="window.print()">EXPORT / PRINT</button>
</div>

<div class="card">
<p>Trades: <span id="trades">0</span>/10</p>
<p>Wins: <span id="wins">0</span></p>
<p>Losses: <span id="losses">0</span></p>
<p>Loss Streak: <span id="streak">0</span></p>
</div>

<div class="card">
<h3>Trade History</h3>
<table>
<thead>
<tr>
<th>Date</th><th>Pair</th><th>Signal</th>
<th>Payout %</th><th>Result</th><th>Start $</th>
<th>End $</th><th>Profit $</th><th>Risk %</th><th>Next Stake</th>
</tr>
</thead>
<tbody id="history"></tbody>
</table>
</div>

<script>

// CORE
let balance=50, baseBalance=50, goal=62;
let riskPercent=2;
let recoveryMode=false, recoveryLoss=0;
let trades=0,wins=0,losses=0,lossStreak=0;
let tradeReady=false;

let current={pair:null,signal:null,score:0,payout:0,executed:false};

const MIN_STAKE=1;

// TIME
setInterval(()=>{
document.getElementById("time").innerText=
new Date().toLocaleString("en-UG",{timeZone:"Africa/Kampala"});
},1000);

// GOAL
function setGoal(){
goal=parseFloat(document.getElementById("goalInput").value);
}

// SET SIGNAL
function setSignal(type){

let pair=document.getElementById("pairInput").value;
let payout=parseFloat(document.getElementById("payoutInput").value);

if(!pair){alert("Select pair");return;}
if(!payout || payout<=0){alert("Enter payout %");return;}

// dynamic confidence
let confidence=80;
if(recoveryMode) confidence=70;
if(wins>=2) confidence=85;

current={
pair:pair,
signal:type,
score:confidence,
payout:payout,
executed:false
};

tradeReady=true;
document.getElementById("executeBtn").disabled=false;

// UI
document.getElementById("pair").innerText=pair;
document.getElementById("signal").innerText=type;
document.getElementById("score").innerText=confidence;
document.getElementById("payout").innerText=payout;

document.getElementById("quickStatus").innerText="Ready: "+type+" on "+pair;

status("Trade Ready → "+type);
}

// EXECUTE
function executeTrade(){
if(!tradeReady){alert("No trade");return;}
if(current.executed){alert("Already executed");return;}

current.executed=true;
tradeReady=false;
document.getElementById("executeBtn").disabled=true;

status("Trade placed ✔");
}

// RESULT
function setResult(res){

if(!current.executed){alert("No executed trade");return;}

let start=balance;

let stake=balance*(riskPercent/100);
if(stake<MIN_STAKE) stake=MIN_STAKE;
if(stake>balance*0.05) stake=balance*0.05;

let profit=0;
trades++;

if(res==="win"){
profit=stake*(current.payout/100);
balance+=profit;
wins++;lossStreak=0;
}else{
profit=-stake;
balance+=profit;
losses++;lossStreak++;
}

// recovery
if(res==="loss"){
recoveryMode=true;
recoveryLoss+=stake;
riskPercent=1;
}

if(recoveryMode && res==="win"){
recoveryLoss-=stake;
if(recoveryLoss<=0){
recoveryMode=false;
riskPercent=2;
recoveryLoss=0;
}
}

// stop rules
if(lossStreak>=3){
document.getElementById("lock").innerHTML="<span class='red'>STOPPED (3 LOSSES)</span>";
}

if(trades>=10){
document.getElementById("status").innerHTML="<span class='green'>SESSION COMPLETE</span>";
}

// history
addHistory(start,profit);

current.executed=false;
updateUI();
}

// HISTORY
function addHistory(start,profit){

let now=new Date();
let date=now.getDate()+"/"+(now.getMonth()+1)+"/"+now.getFullYear();
let time=now.toLocaleTimeString();

let nextStake=balance*(riskPercent/100);
if(nextStake<MIN_STAKE) nextStake=MIN_STAKE;

let row=`<tr>
<td>${date}<br>${time}</td>
<td>${current.pair}</td>
<td>${current.signal}</td>
<td>${current.payout}%</td>
<td class="${profit>=0?'green':'red'}">${profit>=0?'win':'loss'}</td>
<td>${start.toFixed(2)}</td>
<td>${balance.toFixed(2)}</td>
<td class="${profit>=0?'green':'red'}">${profit.toFixed(2)}</td>
<td>${riskPercent}%</td>
<td>${nextStake.toFixed(2)}</td>
</tr>`;

document.getElementById("history").innerHTML+=row;
}

// RESET
function resetSession(){
trades=0;wins=0;losses=0;lossStreak=0;
riskPercent=2;recoveryMode=false;recoveryLoss=0;
tradeReady=false;
document.getElementById("executeBtn").disabled=true;
status("Session reset");
updateUI();
}

// UI
function updateUI(){

document.getElementById("balance").innerText=balance.toFixed(2);

let stake=balance*(riskPercent/100);
if(stake<MIN_STAKE) stake=MIN_STAKE;

document.getElementById("stake").innerText=stake.toFixed(2);

document.getElementById("trades").innerText=trades;
document.getElementById("wins").innerText=wins;
document.getElementById("losses").innerText=losses;
document.getElementById("streak").innerText=lossStreak;

let progress=((balance-baseBalance)/(goal-baseBalance))*100;
progress=Math.max(0,Math.min(100,progress));

document.getElementById("progress").innerText=progress.toFixed(1);

document.getElementById("goalStatus").innerHTML=
balance>=goal?"<span class='green'>Goal reached</span>":"Not reached";

document.getElementById("lock").innerHTML=
recoveryMode?"<span class='red'>RECOVERY MODE ACTIVE (1%)</span>":"";
}

// STATUS
function status(msg){
document.getElementById("status").innerText=msg;
}

</script>

</body>
</html>
