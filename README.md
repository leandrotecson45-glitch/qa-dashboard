
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA Live360 Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<style>
*{box-sizing:border-box}
body{
margin:0;
font-family:Arial,Helvetica,sans-serif;
background:#0b1220;
color:#fff;
}

.header{
padding:14px;
background:#111827;
font-size:20px;
font-weight:bold;
text-align:center;
}

.top-stats{
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
box-shadow:0 4px 10px rgba(0,0,0,.25);
}

.card .num{
font-size:22px;
font-weight:bold;
margin-bottom:4px;
}

#map{
height:58vh;
width:100%;
}

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
min-height:180px;
overflow:auto;
}

.panel h3{
margin:0 0 10px;
font-size:15px;
}

.list-item{
padding:10px;
background:#111827;
border-radius:10px;
margin-bottom:8px;
font-size:13px;
}

.route-btn{
margin-top:8px;
padding:8px 10px;
border:none;
border-radius:8px;
background:#2563eb;
color:#fff;
width:100%;
font-weight:bold;
cursor:pointer;
}

.pin{
background:#111827;
padding:6px 10px;
border-radius:18px;
font-size:12px;
font-weight:bold;
border:1px solid #334155;
color:#fff;
white-space:nowrap;
}

.green{color:#22c55e}
.orange{color:#f59e0b}
.red{color:#ef4444}

@media(max-width:700px){
.top-stats{grid-template-columns:repeat(2,1fr)}
.bottom{grid-template-columns:1fr}
#map{height:52vh}
}
</style>
</head>
<body>

<div class="header">QA Live360 Dashboard</div>

<div class="top-stats">
<div class="card"><div class="num" id="activeCount">0</div>Active</div>
<div class="card"><div class="num" id="movingCount">0</div>Moving</div>
<div class="card"><div class="num" id="idleCount">0</div>Idle</div>
<div class="card"><div class="num" id="offlineCount">0</div>Offline</div>
</div>

<div id="map"></div>

<div class="bottom">
<div class="panel">
<h3>Live Employees</h3>
<div id="employeeList"></div>
</div>

<div class="panel">
<h3>Route Details</h3>
<div id="routeInfo">Select employee pin to show route.</div>
</div>
</div>

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
const map = L.map('map').setView([15.5,120.9],12);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
maxZoom:19
}).addTo(map);

let markers = [];
let routeLine = null;

// =====================
// LOAD LIVE LOCATIONS
// expected collection: liveLocations
// fields:
// name, lat, lon, updatedAt, status, purpose
// =====================

db.collection("liveLocations").onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let active=0,moving=0,idle=0,offline=0;

document.getElementById("employeeList").innerHTML="";

snapshot.forEach(doc=>{

const d = doc.data();

if(!d.lat || !d.lon) return;

active++;

let status = d.status || "Idle";
let statusClass = "orange";

if(status==="Moving"){
moving++;
statusClass="green";
}else if(status==="Idle"){
idle++;
statusClass="orange";
}else{
offline++;
statusClass="red";
}

const pin = L.divIcon({
className:"",
html:`<div class="pin">📍 ${d.name}</div>`,
iconSize:[80,30]
});

const marker = L.marker([d.lat,d.lon],{icon:pin}).addTo(map);

marker.bindPopup(`
<b>${d.name}</b><br>
<span class="${statusClass}">${status}</span><br>
Purpose: ${d.purpose || '-'}<br>
Last Update: ${formatTime(d.updatedAt)}
<br><br>
<button class="route-btn" onclick="showRoute('${doc.id}','${d.name}')">
Show Route
</button>
`);

markers.push(marker);

// list
document.getElementById("employeeList").innerHTML += `
<div class="list-item">
<b>${d.name}</b><br>
<span class="${statusClass}">${status}</span><br>
${d.purpose || '-'}
</div>
`;

});

document.getElementById("activeCount").textContent = active;
document.getElementById("movingCount").textContent = moving;
document.getElementById("idleCount").textContent = idle;
document.getElementById("offlineCount").textContent = offline;

});

// =====================
// SHOW ROUTE
// collection: routeLogs
// fields:
// employeeId, lat, lon, time
// =====================
window.showRoute = async function(employeeId,name){

if(routeLine){
map.removeLayer(routeLine);
routeLine=null;
}

const snap = await db.collection("routeLogs")
.where("employeeId","==",employeeId)
.orderBy("time")
.get();

let pts = [];
let html = `<b>${name}</b><br><br>`;

snap.forEach(doc=>{
const d = doc.data();

pts.push([d.lat,d.lon]);

html += `
<div class="list-item">
📍 ${d.lat.toFixed(5)}, ${d.lon.toFixed(5)}<br>
🕒 ${formatTime(d.time)}
</div>
`;
});

if(pts.length){
routeLine = L.polyline(pts,{weight:5}).addTo(map);
map.fitBounds(routeLine.getBounds(),{padding:[30,30]});
}

document.getElementById("routeInfo").innerHTML = html || "No route today.";

}

// =====================
// TIME FORMAT
// =====================
function formatTime(ts){
if(!ts) return "-";

let date;

if(ts.seconds){
date = new Date(ts.seconds*1000);
}else{
date = new Date(ts);
}

return date.toLocaleString();
}
</script>

</body>
</html>
