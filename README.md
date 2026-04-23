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

db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

// CLEAR
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

// GROUP PATH PER USER
if(!paths[d.name]) paths[d.name]=[];
paths[d.name].push([d.lat, d.lon]);

// LATEST STATUS
if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

});

// DRAW ROUTES
Object.keys(paths).forEach(name=>{

let coords = paths[name];

if(coords.length > 1){

let line = L.polyline(coords,{
color:"#38bdf8",
weight:4
}).addTo(map);

lines.push(line);

}

});

// DRAW MARKERS WITH OFFSET
Object.keys(paths).forEach(name=>{

let coords = paths[name];

coords.forEach((c,i)=>{

let lat = c[0] + (i * 0.00005);
let lon = c[1] + (i * 0.00005);

// GET TYPE (approx from order)
let type = "IN";
if(i > 0) type = "OUT";

let iconHTML = `
<div style="
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:white;
border:1px solid #333;
">
${type === "IN" ? "⬆ IN" : "⬇ OUT"}
</div>
`;

let customIcon = L.divIcon({
html: iconHTML,
className: "",
iconSize: [70,30]
});

let marker = L.marker([lat,lon],{
icon: customIcon
}).addTo(map);

markers.push(marker);

});

});

// STATS
document.getElementById("total").innerText = total;
document.getElementById("in").innerText = inCount;
document.getElementById("out").innerText = outCount;

// USERS
document.getElementById("users").innerHTML="";

Object.values(users).forEach(u=>{
document.getElementById("users").innerHTML += `
<div class="user">
<b>${u.name}</b><br>
${u.type} - ${u.time}<br>
📍 ${u.lat.toFixed(5)}, ${u.lon.toFixed(5)}
</div>
`;
});

});

</script>

</body>
</html>
