
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA Admin Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css">
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

/* HEADER */
header{
background:#111827;
padding:16px;
text-align:center;
font-size:24px;
font-weight:700;
box-shadow:0 4px 15px rgba(0,0,0,.25);
}

/* FILTER */
.topbar{
padding:14px;
display:flex;
gap:10px;
flex-wrap:wrap;
background:#1e293b;
}

select,input{
padding:11px;
border:none;
border-radius:12px;
font-size:14px;
}

/* SUMMARY */
.summary{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(180px,1fr));
gap:12px;
padding:14px;
}

.sum-card{
background:#111827;
padding:15px;
border-radius:14px;
box-shadow:0 4px 12px rgba(0,0,0,.25);
}

.sum-title{
font-size:13px;
color:#94a3b8;
margin-bottom:8px;
}

.sum-value{
font-size:26px;
font-weight:700;
}

/* 🔥 LARGE MAP */
#map{
height:78vh;
min-height:650px;
width:100%;
}

/* MOBILE */
@media(max-width:768px){
#map{
height:62vh;
min-height:500px;
}
}

/* SECTION */
.section{
padding:14px;
}

.card{
background:#111827;
padding:14px;
border-radius:14px;
margin-bottom:14px;
box-shadow:0 4px 12px rgba(0,0,0,.25);
}

canvas{
background:#fff;
border-radius:10px;
padding:10px;
}

/* PIN */
.pin-box{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:#fff;
border:1px solid #334155;
}

/* POPUP */
.leaflet-popup-content-wrapper{
background:#0f172a;
color:#fff;
border-radius:16px;
}

.leaflet-popup-tip{
background:#0f172a;
}

.leaflet-popup-content{
margin:10px;
width:280px !important;
max-height:320px;
overflow-y:auto;
}

.popup-card{
background:#111827;
padding:12px;
border-radius:14px;
margin-bottom:10px;
}

.popup-top{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:10px;
}

.emp{
font-weight:700;
font-size:15px;
}

.tm{
font-size:11px;
color:#94a3b8;
margin-top:4px;
}

.tag{
padding:5px 9px;
border-radius:999px;
font-size:11px;
font-weight:700;
}

.in{background:#14532d;color:#86efac;}
.out{background:#7f1d1d;color:#fca5a5;}

.purpose{
padding:9px;
background:#0f172a;
border-left:4px solid #38bdf8;
border-radius:10px;
font-size:12px;
}

.footer{
margin-top:10px;
font-size:11px;
color:#94a3b8;
}
</style>
<style>

header{
background:#111827;
padding:14px;
display:flex;
align-items:center;
justify-content:center;
gap:12px;
font-size:22px;
font-weight:700;
color:#fff;
}

.logo{
width:80px;
height:80px;
object-fit:contain;
border-radius:10px;
background:#fff;
padding:2px;
box-shadow:0 4px 10px rgba(0,0,0,.25);
}

.title-wrap{
display:flex;
flex-direction:column;
line-height:1.1;
}

.subtxt{
font-size:11px;
font-weight:400;
color:#94a3b8;
}

</style>

<!-- =========================================
STEP 3:
PALITAN ANG HEADER NG QA
========================================= -->

<header>

<img
src="https://i.postimg.cc/bvJLR3N1/ostrom-climate-solutions-inc-logo.jpg"
class="logo">

<div class="title-wrap">
<div>QA DASHBOARD</div>
<div class="subtxt">Company Monitoring System</div>
</div>

</header>

<div class="topbar">
<select id="employeeFilter">
<option value="ALL">All Employees</option>
</select>

<input type="date" id="dateFilter">
</div>

<div class="summary">

<div class="sum-card">
<div class="sum-title">Employees Today</div>
<div class="sum-value" id="empTotal">0</div>
</div>

<div class="sum-card">
<div class="sum-title">TIME IN</div>
<div class="sum-value" id="inTotal">0</div>
</div>

<div class="sum-card">
<div class="sum-title">TIME OUT</div>
<div class="sum-value" id="outTotal">0</div>
</div>

<div class="sum-card">
<div class="sum-title">Missing OUT</div>
<div class="sum-value" id="missingTotal">0</div>
</div>

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

// FIREBASE CONFIG
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

let allLogs=[];
let markers=[];
let chart1=null;
let chart2=null;

// LOAD REALTIME DATA
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
.onchange=renderAll;

document.getElementById("dateFilter")
.onchange=renderAll;

// EMPLOYEE FILTER
function loadEmployeeFilter(){

const select =
document.getElementById("employeeFilter");

let cur = select.value;

let names = [
...new Set(allLogs.map(x=>x.name))
];

select.innerHTML =
'<option value="ALL">All Employees</option>';

names.forEach(n=>{
select.innerHTML +=
`<option value="${n}">${n}</option>`;
});

if(names.includes(cur)){
select.value = cur;
}

}

// FILTER DATA
function getFiltered(){

const emp =
document.getElementById("employeeFilter").value;

const date =
document.getElementById("dateFilter").value;

return allLogs.filter(x=>{

let empOk =
(emp==="ALL" || x.name===emp);

let dateOk = true;

if(date){

let d =
new Date(x.timestamp)
.toISOString()
.split("T")[0];

dateOk = d===date;

}

return empOk && dateOk;

});

}

// MAIN
function renderAll(){

let data = getFiltered();

renderSummary(data);
renderMap(data);
renderCharts(data);

}

// SUMMARY
function renderSummary(data){

let names=[
...new Set(data.map(x=>x.name))
];

let ins =
data.filter(x=>x.type==="IN").length;

let outs =
data.filter(x=>x.type==="OUT").length;

let grouped={};

data.forEach(x=>{
if(!grouped[x.name]) grouped[x.name]=[];
grouped[x.name].push(x.type);
});

let missing=0;

Object.keys(grouped).forEach(n=>{

if(grouped[n].includes("IN") &&
!grouped[n].includes("OUT")){
missing++;
}

});

document.getElementById("empTotal").innerText=names.length;
document.getElementById("inTotal").innerText=ins;
document.getElementById("outTotal").innerText=outs;
document.getElementById("missingTotal").innerText=missing;

}

// MAP
function renderMap(data){

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped={};

data.forEach(d=>{

let key =
d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];

grouped[key].push(d);

});

Object.keys(grouped).forEach(key=>{

let logs = grouped[key];

let lat = logs[0].lat;
let lon = logs[0].lon;

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html="";

logs.forEach(l=>{

html += `
<div class="popup-card">

<div class="popup-top">

<div>
<div class="emp">${l.name}</div>
<div class="tm">${l.time}</div>
</div>

<div class="tag ${l.type==='IN'?'in':'out'}">
${l.type}
</div>

</div>

<div class="purpose">
📌 ${l.purpose || 'No purpose'}
</div>

<div class="footer">
📍 ${Number(l.lat).toFixed(5)},
${Number(l.lon).toFixed(5)}
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

let inC=0;
let outC=0;
let purpose={};

data.forEach(x=>{

if(x.type==="IN") inC++;
if(x.type==="OUT") outC++;

let p = x.purpose || "None";

if(!purpose[p]) purpose[p]=0;

purpose[p]++;

});

// BAR
if(chart1) chart1.destroy();

chart1 = new Chart(
document.getElementById("typeChart"),
{
type:"bar",
data:{
labels:["IN","OUT"],
datasets:[{
label:"Logs",
data:[inC,outC]
}]
}
});

// PIE
if(chart2) chart2.destroy();

chart2 = new Chart(
document.getElementById("purposeChart"),
{
type:"pie",
data:{
labels:Object.keys(purpose),
datasets:[{
data:Object.values(purpose)
}]
}
});

}

</script>
</body>
</html>
