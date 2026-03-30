<!DOCTYPE html>
<html>
<head>
<title>Trading AI v6 Stable</title>

<style>
body{background:#020617;color:white;font-family:Arial;text-align:center;}
h2{color:#38bdf8;}

.card{
background:#0f172a;
border:1px solid #1e293b;
padding:12px;
margin:10px auto;
width:95%;
max-width:1000px;
border-radius:12px;
}

button{
padding:10px;
margin:5px;
border:none;
border-radius:6px;
font-weight:bold;
cursor:pointer;
}

table{
width:100%;
border-collapse:collapse;
font-size:12px;
}

th,td{
border:1px solid #1e293b;
padding:6px;
}

.lock{color:#ef4444;font-weight:bold;}
.green{color:#22c55e;}
</style>
</head>

<body>

<h2>Trading AI v6 Stable Engine</h2>

<div class="card">
Balance: $<span id="balance">50</span><br>
Trades: <span id="trades">0</span>/10 |
Streak: <span id="streak">0</span><br>
Status: <span id="status"></span>
</div>

<div class="card">
<select id="pair">
<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>EUR/USD OTC</option>
<option>GBP/JPY OTC</option>
<option>USD/CAD OTC</option>
<option>AUD/USD OTC</option>
<option>EUR/JPY OTC</option>
</select>

<input id="payout" placeholder="Payout %" type="number">

<br>

<button onclick="setSignal('BUY')">BUY</button>
<button onclick="setSignal('SELL')">SELL</button>
</div>

<div class="card">
Signal: <span id="signal">---</span>
</div>

<div class="card">
<button onclick="executeTrade()">EXECUTE</button>
<button onclick="setResult('win')">WIN</button>
<button onclick="setResult('loss')">LOSS</button>
</div>

<div class="card">
<h3>Session Table</h3>
<table>
<thead>
<tr>
<th>Date</th>
<th>Session</th>
<th>Profit</th>
<th>Best Pairs</th>
</tr>
</thead>
<tbody id="sessionTable"></tbody>
</table>
</div>

<script>

// ================= CORE STATE =================
let balance = 50;

let trades = 0;
let streak = 0;

let sessionProfit = 0;
let sessionCount = 1;
let sessionPairs = {};

let signal = null;
let executed = false;

let sessionLocked = false;

// Recovery system
let recoveryCount = 0; // max 3
let risk = 2;

// ================= SIGNAL =================
function setSignal(type){
if(sessionLocked) return;

signal = type;
document.getElementById("signal").innerText = type;
}

// ================= EXECUTE =================
function executeTrade(){
if(sessionLocked){
alert("Session locked");
return;
}
if(!signal){
alert("Select signal first");
return;
}
executed = true;
}

// ================= RESULT =================
function setResult(res){

if(sessionLocked) return;
if(!executed){
alert("Click EXECUTE first");
return;
}

let payout = parseFloat(document.getElementById("payout").value || 80);
let stake = Math.max(balance * (risk / 100), 1);

let profit = res === "win"
? stake * (payout / 100)
: -stake;

balance += profit;
sessionProfit += profit;

trades++;

if(res === "win"){
streak = 0;
} else {
streak++;
}

// ================= RECOVERY LIMIT (ONLY 3 TIMES) =================
if(res === "loss"){
recoveryCount++;
risk = 1;
}

if(res === "win" && recoveryCount > 0){
recoveryCount = 0;
risk = 2;
}

// STOP AFTER 3 RECOVERY USES
if(recoveryCount >= 3){
endSession("Recovery limit reached (3 uses)");
return;
}

// track pair
let pair = document.getElementById("pair").value;
if(!sessionPairs[pair]) sessionPairs[pair] = 0;
sessionPairs[pair] += profit;

// update UI
document.getElementById("balance").innerText = balance.toFixed(2);
document.getElementById("trades").innerText = trades;
document.getElementById("streak").innerText = streak;

// session stop conditions
if(trades >= 10 || streak >= 3){
endSession("Session completed");
}

executed = false;
}

// ================= END SESSION =================
function endSession(reason){

if(sessionLocked) return;

sessionLocked = true;

// SAVE BEFORE RESET
saveSession();

document.getElementById("status").innerHTML =
"<span class='lock'>SESSION LOCKED: " + reason + "</span>";
}

// ================= SAVE SESSION =================
function saveSession(){

let table = document.getElementById("sessionTable");

let now = new Date();
let date = now.getDate() + "/" + (now.getMonth()+1) + "/" + now.getFullYear();

let bestPairs = Object.keys(sessionPairs).join(", ") || "-";

let row = document.createElement("tr");

row.innerHTML =
"<td>" + date + "</td>" +
"<td>" + sessionCount + "</td>" +
"<td>" + sessionProfit.toFixed(2) + "</td>" +
"<td>" + bestPairs + "</td>";

table.appendChild(row);

sessionCount++;

console.log("SESSION SAVED");
}

// ================= RESET =================
function resetSession(){

trades = 0;
streak = 0;

sessionProfit = 0;
sessionPairs = {};

signal = null;
executed = false;

sessionLocked = false;

recoveryCount = 0;
risk = 2;

document.getElementById("signal").innerText = "---";
document.getElementById("status").innerText = "";
document.getElementById("trades").innerText = "0";
document.getElementById("streak").innerText = "0";
}

</script>

</body>
</html>
