
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}

.top{
padding:15px;
background:#0f172a;
display:flex;
justify-content:space-around;
}

.card{text-align:center;}

#map{height:55vh;}

.section{
padding:10px;
}

.list{
max-height:25vh;
overflow:auto;
}

.user{
padding:10px;
margin-bottom:8px;
border-radius:8px;
background:#111827;
}
</style>
</head>

<body>

<div class="top">
<div class="card">
<div id="total">0</div>
<small>Total Logs</small>
</div>

<div class="card">
<div id="in">0</div>
<small>Total IN</small>
</div>

<div class="card">
<div id="out">0</div>
<small>Total OUT</small>
</div>
</div>

<div id="map"></div>

<div class="section">
<h3>📍 Latest Status (Per Employee)</h3>
<div class="list" id="users"></div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
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
const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let markers = [];

// 🔥 OFFSET para hindi magpatong
function offset(lat, lon, i){
let shift = i * 0.00003;
return [lat + shift, lon + shift];
}

// REALTIME
db.collection("attendance")
.orderBy("timestamp")
.onSnapshot(snapshot=>{

// clear old markers
markers.forEach(m=>map.removeLayer(m));
markers=[];

let total=0,inCount=0,outCount=0;
let users={};
let i=0;

snapshot.forEach(doc=>{

const d = doc.data();

total++;

if(d.type==="IN") inCount++;
if(d.type==="OUT") outCount++;

// 🔥 LATEST PER USER
if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

// 🔥 OFFSET PARA HINDI MAGPATONG
let [lat,lon] = offset(d.lat, d.lon, i);

let color = d.type==="IN" ? "green" : "red";

let marker = L.circleMarker([lat,lon],{
radius:8,
color:color,
fillColor:color,
fillOpacity:0.9
}).addTo(map);

marker.bindPopup(`
<b>${d.name}</b><br>
${d.type}<br>
${d.time}<br>
${d.date}
`);

markers.push(marker);
i++;

});

// 🔥 UPDATE COUNTERS
document.getElementById("total").innerText = total;
document.getElementById("in").innerText = inCount;
document.getElementById("out").innerText = outCount;

// 🔥 RENDER LATEST USERS
document.getElementById("users").innerHTML="";

Object.values(users).forEach(u=>{

document.getElementById("users").innerHTML += `
<div class="user">
<b>${u.name}</b><br>
Status: ${u.type}<br>
Time: ${u.time}
</div>
`;

});

});

</script>

</body>
</html>
