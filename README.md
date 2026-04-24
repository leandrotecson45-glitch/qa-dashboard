<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA KM Route Dashboard</title>

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
text-align:center;
font-size:22px;
font-weight:bold;
background:#111827;
}

.top{
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
margin-bottom:5px;
}

#map{
height:56vh;
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
min-height:220px;
overflow:auto;
}

.panel h3{
margin:0 0 10px;
font-size:15px;
}

.pin{
background:#111827;
padding:6px 10px;
border-radius:18px;
font-size:12px;
font-weight:bold;
border:1px solid #334155;
white-space:nowrap;
}

.item{
background:#111827;
padding:10px;
border-radius:10px;
margin-bottom:8px;
font-size:13px;
}

.btn{
width:100%;
margin-top:8px;
padding:8px;
border:none;
border-radius:10px;
background:#2563eb;
color:#fff;
font-weight:bold;
cursor:pointer;
}

.green{color:#22c55e}

@media(max-width:700px){
.top{grid-template-columns:repeat(2,1fr)}
.bottom{grid-template-columns:1fr}
#map{height:52vh}
}
</style>
</head>
<body>

<div class="header">QA KM Route Dashboard</div>

<div class="top">
<div class="card"><div class="num" id="empCount">0</div>Employees</div>
<div class="card"><div class="num" id="todayKm">0</div>Total KM</div>
<div class="card"><div class="num" id="selectedKm">0</div>Selected KM</div>
<div class="card"><div class="num" id="selectedPay">0</div>Pay</div>
</div>

<div id="map"></div>

<div class="bottom">
<div class="panel">
<h3>Employees Today</h3>
<div id="list"></div>
</div>

<div class="panel">
<h3>Route Details</h3>
<div id="details">Select employee to compute route.</div>
</div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>
// ======================
// FIREBASE CONFIG
// ======================
const firebaseConfig = {
  apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// ======================
// SETTINGS
// ======================
const RATE_PER_KM = 15; // palitan mo kung magkano per km

// ======================
// MAP
// ======================
const map = L.map("map").setView([15.5,120.9],12);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",{
maxZoom:19
}).addTo(map);

let markers = [];
let routeLine = null;
let routePins = [];

// ======================
// LOAD ATTENDANCE
// expects attendance:
// name, lat, lon, timestamp, purpose
// ======================
db.collection("attendance").orderBy("timestamp","desc")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let rows = [];
snapshot.forEach(doc=>{
rows.push(doc.data());
});

let groupedEmp = {};
let latestPerEmp = {};

rows.forEach(r=>{

if(!r.name || !r.lat || !r.lon) return;

if(!groupedEmp[r.name]) groupedEmp[r.name]=[];
groupedEmp[r.name].push(r);

if(!latestPerEmp[r.name]){
latestPerEmp[r.name]=r;
}

});

// stats
document.getElementById("empCount").textContent =
Object.keys(groupedEmp).length;

// employee list
document.getElementById("list").innerHTML="";

let totalKmAll = 0;

Object.keys(groupedEmp).forEach(name=>{

let logs = groupedEmp[name].sort((a,b)=>a.timestamp-b.timestamp);

let km = totalDistance(logs);
totalKmAll += km;

document.getElementById("list").innerHTML += `
<div class="item">
<b>${name}</b><br>
Today KM: <span class="green">${km.toFixed(2)}</span><br>
<button class="btn" onclick="showRoute('${name}')">Show Route</button>
</div>
`;

let latest = latestPerEmp[name];

let icon = L.divIcon({
className:"",
html:`<div class="pin">📍 ${name}</div>`,
iconSize:[90,30]
});

let marker = L.marker([latest.lat,latest.lon],{icon})
.addTo(map)
.bindPopup(`
<b>${name}</b><br>
Latest Position<br>
KM Today: ${km.toFixed(2)}
`);

markers.push(marker);

});

document.getElementById("todayKm").textContent =
totalKmAll.toFixed(2);

});

// ======================
// SHOW ROUTE
// ======================
window.showRoute = async function(name){

clearRoute();

const snap = await db.collection("attendance")
.where("name","==",name)
.orderBy("timestamp","asc")
.get();

let logs = [];
snap.forEach(doc=>logs.push(doc.data()));

if(!logs.length){
document.getElementById("details").innerHTML="No data.";
return;
}

let pts = logs.map(x=>[x.lat,x.lon]);

routeLine = L.polyline(pts,{weight:5}).addTo(map);
map.fitBounds(routeLine.getBounds(),{padding:[30,30]});

// start / end markers
routePins.push(
L.marker(pts[0]).addTo(map).bindPopup("Start")
);

routePins.push(
L.marker(pts[pts.length-1]).addTo(map).bindPopup("End")
);

let km = totalDistance(logs);
let pay = km * RATE_PER_KM;

document.getElementById("selectedKm").textContent =
km.toFixed(2);

document.getElementById("selectedPay").textContent =
pay.toFixed(2);

let html = `<b>${name}</b><br><br>`;
html += `Rate/KM: ${RATE_PER_KM}<br>`;
html += `Total KM: <span class="green">${km.toFixed(2)}</span><br>`;
html += `Estimated Pay: <span class="green">${pay.toFixed(2)}</span><br><br>`;

logs.forEach(l=>{
html += `
<div class="item">
📍 ${l.lat.toFixed(5)}, ${l.lon.toFixed(5)}<br>
🕒 ${formatTime(l.timestamp)}<br>
📌 ${l.purpose || '-'}
</div>
`;
});

document.getElementById("details").innerHTML = html;

}

// ======================
// DISTANCE KM
// ======================
function totalDistance(arr){

let km = 0;

for(let i=1;i<arr.length;i++){
km += haversine(
arr[i-1].lat,
arr[i-1].lon,
arr[i].lat,
arr[i].lon
);
}

return km;
}

function haversine(lat1,lon1,lat2,lon2){

const R = 6371;
const dLat = toRad(lat2-lat1);
const dLon = toRad(lon2-lon1);

const a =
Math.sin(dLat/2)*Math.sin(dLat/2)+
Math.cos(toRad(lat1))*
Math.cos(toRad(lat2))*
Math.sin(dLon/2)*Math.sin(dLon/2);

const c = 2*Math.atan2(Math.sqrt(a),Math.sqrt(1-a));

return R*c;
}

function toRad(v){
return v*Math.PI/180;
}

// ======================
function clearRoute(){

if(routeLine){
map.removeLayer(routeLine);
routeLine=null;
}

routePins.forEach(p=>map.removeLayer(p));
routePins=[];
}

function formatTime(ts){
return new Date(ts).toLocaleString();
}
</script>

</body>
</html>
