
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>QA Point To Point Route</title>

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
min-height:220px;
overflow:auto;
}

.panel h3{
margin:0 0 10px;
font-size:15px;
}

.item{
background:#111827;
padding:10px;
border-radius:10px;
margin-bottom:8px;
font-size:13px;
}

.btn{
margin-top:8px;
width:100%;
padding:8px;
border:none;
border-radius:10px;
background:#2563eb;
color:#fff;
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
white-space:nowrap;
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

<div class="header">QA Point To Point Route</div>

<div class="top">
<div class="card"><div class="num" id="empCount">0</div>Employees</div>
<div class="card"><div class="num" id="stops">0</div>Stops</div>
<div class="card"><div class="num" id="km">0</div>Total KM</div>
<div class="card"><div class="num" id="pay">0</div>Pay</div>
</div>

<div id="map"></div>

<div class="bottom">
<div class="panel">
<h3>Employees</h3>
<div id="employeeList"></div>
</div>

<div class="panel">
<h3>Route Details</h3>
<div id="details">Select employee to generate route.</div>
</div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>
// ====================
// FIREBASE
// ====================
const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// ====================
// SETTINGS
// ====================
const RATE_PER_KM = 15; // palitan kung gusto
const DUPLICATE_RADIUS_METERS = 50;

// ====================
// MAP
// ====================
const map = L.map("map").setView([15.5,120.9],12);

L.tileLayer(
"https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
{maxZoom:19}
).addTo(map);

let markers = [];
let routeLine = null;
let routePins = [];

// ====================
// LOAD EMPLOYEES
// ====================
db.collection("attendance")
.orderBy("timestamp","desc")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let rows = [];
snapshot.forEach(doc=>rows.push(doc.data()));

let empMap = {};
let latest = {};

rows.forEach(r=>{

if(!r.name || !r.lat || !r.lon) return;

if(!empMap[r.name]) empMap[r.name]=[];
empMap[r.name].push(r);

if(!latest[r.name]) latest[r.name]=r;

});

document.getElementById("empCount").textContent =
Object.keys(empMap).length;

document.getElementById("employeeList").innerHTML="";

Object.keys(empMap).forEach(name=>{

document.getElementById("employeeList").innerHTML += `
<div class="item">
<b>${name}</b><br>
<button class="btn" onclick="showRoute('${name}')">
Show Route
</button>
</div>
`;

let l = latest[name];

let icon = L.divIcon({
className:"",
html:`<div class="pin">📍 ${name}</div>`,
iconSize:[90,30]
});

let marker = L.marker([l.lat,l.lon],{icon})
.addTo(map)
.bindPopup(`
<b>${name}</b><br>
Latest Position
`);

markers.push(marker);

});

});

// ====================
// SHOW ROUTE
// ====================
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

// remove duplicate nearby points
let cleaned = [];

logs.forEach(p=>{

if(cleaned.length===0){
cleaned.push(p);
return;
}

let prev = cleaned[cleaned.length-1];

let meters = haversine(
prev.lat,prev.lon,
p.lat,p.lon
) * 1000;

if(meters >= DUPLICATE_RADIUS_METERS){
cleaned.push(p);
}

});

let pts = cleaned.map(x=>[x.lat,x.lon]);

// draw route
routeLine = L.polyline(pts,{weight:5}).addTo(map);
map.fitBounds(routeLine.getBounds(),{padding:[30,30]});

// markers A B C D
cleaned.forEach((p,i)=>{

let label = String.fromCharCode(65+i); // A,B,C...

let mk = L.marker([p.lat,p.lon])
.addTo(map)
.bindPopup(`
<b>Point ${label}</b><br>
${formatTime(p.timestamp)}<br>
${p.purpose || '-'}
`);

routePins.push(mk);

});

// KM
let totalKm = totalDistance(cleaned);
let totalPay = totalKm * RATE_PER_KM;

// top cards
document.getElementById("stops").textContent =
cleaned.length;

document.getElementById("km").textContent =
totalKm.toFixed(2);

document.getElementById("pay").textContent =
totalPay.toFixed(2);

// details
let html = `
<b>${name}</b><br><br>
Stops: ${cleaned.length}<br>
KM: <span class="green">${totalKm.toFixed(2)}</span><br>
Rate/KM: ${RATE_PER_KM}<br>
Pay: <span class="green">${totalPay.toFixed(2)}</span><br><br>
`;

for(let i=1;i<cleaned.length;i++){

let a = cleaned[i-1];
let b = cleaned[i];

let segKm = haversine(
a.lat,a.lon,b.lat,b.lon
);

let from = String.fromCharCode(64+i);
let to = String.fromCharCode(65+i);

html += `
<div class="item">
${from} ➜ ${to}<br>
${segKm.toFixed(2)} KM<br>
🕒 ${formatTime(b.timestamp)}
</div>
`;

}

document.getElementById("details").innerHTML = html;

}

// ====================
// HELPERS
// ====================
function totalDistance(arr){

let km = 0;

for(let i=1;i<arr.length;i++){
km += haversine(
arr[i-1].lat,arr[i-1].lon,
arr[i].lat,arr[i].lon
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
