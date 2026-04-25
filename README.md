
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
background:#0f172a;
color:#fff;
}

header{
background:#111827;
padding:15px;
text-align:center;
font-size:22px;
font-weight:bold;
}

.topbar{
padding:15px;
display:flex;
gap:10px;
flex-wrap:wrap;
background:#1e293b;
}

select,input{
padding:10px;
border:none;
border-radius:10px;
font-size:14px;
}

#map{
height:420px;
width:100%;
}

.section{
padding:15px;
}

.card{
background:#1e293b;
padding:15px;
border-radius:14px;
margin-bottom:15px;
box-shadow:0 4px 10px rgba(0,0,0,.25);
}

.pin-box{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:#fff;
border:1px solid #334155;
}

/* =========================
PREMIUM POPUP
========================= */

.leaflet-popup-content-wrapper{
background:#0f172a;
color:#fff;
border-radius:16px;
padding:0;
box-shadow:0 10px 30px rgba(0,0,0,.45);
}

.leaflet-popup-content{
margin:0;
width:280px !important;
max-height:320px;
overflow-y:auto;
}

.leaflet-popup-tip{
background:#0f172a;
}

.popup-card{
background:#111827;
border-radius:14px;
padding:12px;
margin:10px;
border:1px solid #1f2937;
box-shadow:0 4px 12px rgba(0,0,0,.25);
}

.popup-top{
display:flex;
justify-content:space-between;
align-items:center;
gap:10px;
margin-bottom:10px;
}

.emp-name{
font-size:15px;
font-weight:700;
color:#fff;
}

.emp-time{
font-size:11px;
color:#94a3b8;
margin-top:3px;
}

.status-tag{
padding:6px 10px;
border-radius:999px;
font-size:11px;
font-weight:700;
}

.status-in{
background:#14532d;
color:#86efac;
}

.status-out{
background:#7f1d1d;
color:#fca5a5;
}

.popup-purpose{
background:#0f172a;
border:1px solid #1e293b;
border-left:4px solid #38bdf8;
padding:10px;
border-radius:12px;
font-size:12px;
line-height:1.45;
color:#e2e8f0;
}

.popup-footer{
margin-top:10px;
padding-top:10px;
border-top:1px solid #1e293b;
font-size:11px;
color:#94a3b8;
}

/* CHARTS */
canvas{
background:#fff;
border-radius:10px;
padding:10px;
}

</style>
</head>
<body>

<header>📊 QA Dashboard</header>

<div class="topbar">

<select id="employeeFilter">
<option value="ALL">All Employees</option>
</select>

<input type="date" id="dateFilter">

</div>

<div id="map"></div>

<div class="section">

<div class="card">
<canvas id="typeChart"></canvas>
</div>

<div class="card">
<canvas id="purposeChart"></canvas>
</div>

</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// FIREBASE
const firebaseConfig = {
apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map("map").setView([15.486,120.967],13);

L.tileLayer(
"https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
{maxZoom:19}
).addTo(map);

let markers=[];
let allLogs=[];
let chart1=null;
let chart2=null;

// LOAD DATA
db.collection("attendance")
.orderBy("timestamp")
.onSnapshot(snapshot=>{

allLogs=[];

snapshot.forEach(doc=>{
allLogs.push(doc.data());
});

loadEmployeeFilter();
renderAll();

});

// FILTER EVENTS
document.getElementById("employeeFilter")
.addEventListener("change",renderAll);

document.getElementById("dateFilter")
.addEventListener("change",renderAll);

// EMPLOYEE FILTER
function loadEmployeeFilter(){

const select =
document.getElementById("employeeFilter");

let current = select.value;

let names =
[...new Set(allLogs.map(x=>x.name))];

select.innerHTML =
'<option value="ALL">All Employees</option>';

names.forEach(name=>{

select.innerHTML +=
`<option value="${name}">
${name}
</option>`;

});

if(names.includes(current)){
select.value=current;
}else{
select.value="ALL";
}

}

// GET FILTERED
function getFiltered(){

const emp =
document.getElementById("employeeFilter").value;

const date =
document.getElementById("dateFilter").value;

return allLogs.filter(item=>{

let empOk =
(emp==="ALL" || item.name===emp);

let dateOk=true;

if(date){

let itemDate =
new Date(item.timestamp)
.toISOString()
.split("T")[0];

dateOk = itemDate===date;

}

return empOk && dateOk;

});

}

// RENDER ALL
function renderAll(){

let data=getFiltered();

renderMap(data);
renderCharts(data);

}

// MAP
function renderMap(data){

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped={};

data.forEach(d=>{

let key =
d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]){
grouped[key]=[];
}

grouped[key].push(d);

});

Object.keys(grouped).forEach(key=>{

let logs=grouped[key];

let lat=logs[0].lat;
let lon=logs[0].lon;

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html="";

logs.forEach(l=>{

html += `
<div class="popup-card">

<div class="popup-top">

<div>
<div class="emp-name">${l.name}</div>
<div class="emp-time">🕒 ${l.time}</div>
</div>

<div class="status-tag ${l.type==='IN'?'status-in':'status-out'}">
${l.type}
</div>

</div>

<div class="popup-purpose">
📌 ${l.purpose || "No purpose"}
</div>

<div class="popup-footer">
📍 Lat: ${Number(l.lat).toFixed(5)} |
Lon: ${Number(l.lon).toFixed(5)}
</div>

</div>
`;

});

let icon=L.divIcon({
html:`<div class="pin-box">📍 ${logs.length}</div>`,
className:"",
iconSize:[60,30]
});

let marker=
L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

}

// CHARTS
function renderCharts(data){

let inCount=0;
let outCount=0;
let purposeCount={};

data.forEach(d=>{

if(d.type==="IN") inCount++;
if(d.type==="OUT") outCount++;

let p=d.purpose || "None";

if(!purposeCount[p]){
purposeCount[p]=0;
}

purposeCount[p]++;

});

// BAR
if(chart1) chart1.destroy();

chart1 =
new Chart(
document.getElementById("typeChart"),
{
type:"bar",
data:{
labels:["IN","OUT"],
datasets:[{
label:"Logs",
data:[inCount,outCount]
}]
}
});

// PIE
if(chart2) chart2.destroy();

chart2 =
new Chart(
document.getElementById("purposeChart"),
{
type:"pie",
data:{
labels:Object.keys(purposeCount),
datasets:[{
data:Object.values(purposeCount)
}]
}
});

}

</script>
</body>
</html>
