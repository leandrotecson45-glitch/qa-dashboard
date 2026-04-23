<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}

.top{
display:flex;
gap:10px;
padding:10px;
background:#0f172a;
}

.card{
flex:1;
background:#111827;
padding:10px;
border-radius:10px;
text-align:center;
}

#map{height:60vh;}

.section{padding:10px;}

.user{
background:#111827;
padding:10px;
margin-bottom:8px;
border-radius:10px;
}
</style>
</head>

<body>

<div class="top">
<div class="card"><div id="total">0</div><small>Total</small></div>
<div class="card"><div id="in">0</div><small>IN</small></div>
<div class="card"><div id="out">0</div><small>OUT</small></div>
</div>

<div id="map"></div>

<div class="section">
<h3>Employees</h3>
<div id="users"></div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const map = L.map('map').setView([15.5,120.9],13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let markers=[];
let lines=[];

// 📏 DISTANCE FUNCTION (KM)
function getDistance(lat1, lon1, lat2, lon2){
const R = 6371;
const dLat = (lat2-lat1) * Math.PI/180;
const dLon = (lon2-lon1) * Math.PI/180;

const a =
Math.sin(dLat/2)*Math.sin(dLat/2) +
Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180) *
Math.sin(dLon/2)*Math.sin(dLon/2);

const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
return R * c;
}

db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
lines.forEach(l=>map.removeLayer(l));
markers=[];
lines=[];

let users = {};
let paths = {};
let total=0,inCount=0,outCount=0;

// PROCESS DATA
snapshot.forEach(doc=>{

const d = doc.data();

total++;
if(d.type==="IN") inCount++;
if(d.type==="OUT") outCount++;

if(!paths[d.name]) paths[d.name]=[];
paths[d.name].push(d);

if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

});

// DRAW PER USER
Object.keys(paths).forEach(name=>{

let logs = paths[name];

// SORT BY TIME
logs.sort((a,b)=>a.timestamp - b.timestamp);

// 📏 CALCULATE KM
let totalKM = 0;

for(let i=1;i<logs.length;i++){
totalKM += getDistance(
logs[i-1].lat, logs[i-1].lon,
logs[i].lat, logs[i].lon
);
}

// 🟢 START
let start = logs[0];
let startMarker = L.marker([start.lat,start.lon])
.addTo(map)
.bindPopup(`<b>${name}</b><br>🟢 START<br>${start.time}`);

markers.push(startMarker);

// 🔴 END
let end = logs[logs.length-1];
let endMarker = L.marker([end.lat,end.lon])
.addTo(map)
.bindPopup(`<b>${name}</b><br>🔴 END<br>${end.time}`);

markers.push(endMarker);

// 🔵 ROUTE
let coords = logs.map(l=>[l.lat,l.lon]);

if(coords.length>1){
let line = L.polyline(coords,{
color:"#38bdf8",
weight:4
}).addTo(map);

lines.push(line);
}

// USERS LIST
document.getElementById("users").innerHTML += `
<div class="user">
<b>${name}</b><br>
📏 KM: ${totalKM.toFixed(2)}<br>
Last: ${end.type} - ${end.time}
</div>
`;

});

// STATS
document.getElementById("total").innerText = total;
document.getElementById("in").innerText = inCount;
document.getElementById("out").innerText = outCount;

});

</script>

</body>
</html>
