
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

#map{height:50vh;}

.section{padding:10px;}

.user{
background:#111827;
padding:10px;
margin-bottom:8px;
border-radius:10px;
}

.status-in{color:#22c55e;}
.status-out{color:#ef4444;}

.popup{
max-height:150px;
overflow:auto;
font-size:12px;
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

db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped = {};
let users = {};

let total=0,inCount=0,outCount=0;

snapshot.forEach(doc=>{

const d = doc.data();

total++;
if(d.type==="IN") inCount++;
if(d.type==="OUT") outCount++;

let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);
if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);

if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

});

// MAP
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

let html = `<div class="popup">`;

logs.forEach(l=>{
html += `<b>${l.name}</b><br>${l.type} - ${l.time}<br><hr>`;
});

html += `</div>`;

let last = logs[logs.length-1];
let color = last.type==="IN" ? "#22c55e" : "#ef4444";

let marker = L.circleMarker([lat,lon],{
radius:12,
color:color,
fillColor:color,
fillOpacity:0.9
}).addTo(map);

marker.bindPopup(html);

markers.push(marker);

});

// STATS
document.getElementById("total").innerText = total;
document.getElementById("in").innerText = inCount;
document.getElementById("out").innerText = outCount;

// USERS
document.getElementById("users").innerHTML="";

Object.values(users).forEach(u=>{

let statusClass = u.type==="IN" ? "status-in" : "status-out";

document.getElementById("users").innerHTML += `
<div class="user">
<b>${u.name}</b><br>
Status: <span class="${statusClass}">${u.type}</span><br>
Time: ${u.time}<br>
📍 ${u.lat.toFixed(5)}, ${u.lon.toFixed(5)}
</div>
`;

});

});

</script>

</body>
</html>
