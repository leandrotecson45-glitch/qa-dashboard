
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<style>
*{box-sizing:border-box}
body{
margin:0;
font-family:Arial,Helvetica,sans-serif;
background:#0b1220;
color:#fff;
}

/* HEADER */
.header{
padding:14px;
text-align:center;
font-size:22px;
font-weight:bold;
background:#111827;
}

/* MAP */
#map{
height:58vh;
width:100%;
}

/* STATS */
.stats{
display:grid;
grid-template-columns:repeat(4,1fr);
gap:10px;
padding:10px;
}

.card{
background:#1f2937;
padding:12px;
border-radius:14px;
text-align:center;
}

.num{
font-size:22px;
font-weight:bold;
margin-bottom:4px;
}

/* CHARTS */
.bottom{
padding:10px;
display:grid;
grid-template-columns:1fr 1fr;
gap:10px;
}

.panel{
background:#1f2937;
padding:12px;
border-radius:14px;
}

.panel h3{
margin:0 0 10px;
font-size:15px;
}

/* PIN */
.pin{
background:#111827;
padding:6px 10px;
border-radius:18px;
font-size:12px;
font-weight:bold;
border:1px solid #334155;
white-space:nowrap;
}

/* POPUP */
.popup-box{
max-height:260px;
overflow-y:auto;
}

.item{
background:#111827;
padding:10px;
border-radius:10px;
margin-bottom:8px;
font-size:13px;
}

.tag{
display:inline-block;
padding:4px 8px;
border-radius:8px;
font-size:11px;
font-weight:bold;
margin-top:5px;
}

.in{background:#22c55e;}
.out{background:#ef4444;}

.purpose{
margin-top:6px;
padding:7px;
background:#1e293b;
border-left:3px solid #3b82f6;
border-radius:8px;
font-size:12px;
}

@media(max-width:700px){
.stats{grid-template-columns:repeat(2,1fr)}
.bottom{grid-template-columns:1fr}
#map{height:52vh}
}
</style>
</head>
<body>

<div class="header">QA Dashboard</div>

<div class="stats">
<div class="card"><div class="num" id="total">0</div>Total</div>
<div class="card"><div class="num" id="inCount">0</div>IN</div>
<div class="card"><div class="num" id="outCount">0</div>OUT</div>
<div class="card"><div class="num" id="locationCount">0</div>Locations</div>
</div>

<div id="map"></div>

<div class="bottom">
<div class="panel">
<h3>Activity Chart</h3>
<canvas id="barChart"></canvas>
</div>

<div class="panel">
<h3>Purpose Chart</h3>
<canvas id="pieChart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>
// =====================
// FIREBASE CONFIG
// =====================
const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// =====================
// MAP
// =====================
const map = L.map("map").setView([15.5,120.9],12);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",{
maxZoom:19
}).addTo(map);

let markers = [];
let chart1 = null;
let chart2 = null;

// =====================
// REALTIME DATA
// =====================
db.collection("attendance").orderBy("timestamp","desc")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let rows = [];
snapshot.forEach(doc=>{
rows.push(doc.data());
});

// STATS
let total = rows.length;
let inCount = rows.filter(x=>x.type==="IN").length;
let outCount = rows.filter(x=>x.type==="OUT").length;

document.getElementById("total").textContent = total;
document.getElementById("inCount").textContent = inCount;
document.getElementById("outCount").textContent = outCount;

// GROUP SAME LOCATION
let grouped = {};

rows.forEach(d=>{

if(!d.lat || !d.lon) return;

let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);

});

document.getElementById("locationCount").textContent =
Object.keys(grouped).length;

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

let html = `<div class="popup-box"><b>📍 ${logs.length} Record(s)</b><br><br>`;

logs.forEach(l=>{

let tag = l.type==="IN" ? "in":"out";

html += `
<div class="item">
<b>${l.name || "-"}</b><br>
🕒 ${l.time || "-"}<br>
<span class="tag ${tag}">${l.type}</span>
<div class="purpose">📌 ${l.purpose || "No purpose"}</div>
</div>
`;

});

html += `</div>`;

let icon = L.divIcon({
className:"",
html:`<div class="pin">📍 ${logs.length}</div>`,
iconSize:[60,30]
});

let marker = L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

// =====================
// CHART 1
// =====================
if(chart1) chart1.destroy();

chart1 = new Chart(
document.getElementById("barChart"),
{
type:"bar",
data:{
labels:["IN","OUT"],
datasets:[{
data:[inCount,outCount]
}]
},
options:{
responsive:true,
plugins:{legend:{display:false}}
}
}
);

// =====================
// CHART 2
// =====================
let purposeMap = {};

rows.forEach(r=>{
let p = r.purpose || "None";
purposeMap[p] = (purposeMap[p]||0)+1;
});

if(chart2) chart2.destroy();

chart2 = new Chart(
document.getElementById("pieChart"),
{
type:"pie",
data:{
labels:Object.keys(purposeMap),
datasets:[{
data:Object.values(purposeMap)
}]
},
options:{
responsive:true
}
}
);

});
</script>

</body>
</html>
