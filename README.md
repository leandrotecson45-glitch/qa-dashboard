
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

#map{height:60vh;}

.list{
padding:10px;
font-size:14px;
max-height:30vh;
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
<small>Total Today</small>
</div>

<div class="card">
<div id="active">0</div>
<small>Active (IN)</small>
</div>

<div class="card">
<div id="out">0</div>
<small>Out</small>
</div>
</div>

<div id="map"></div>

<div class="list" id="users"></div>

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

let markers = [];

db.collection("attendance").onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let today = new Date().toLocaleDateString();

let users = {};
let total=0, active=0, out=0;

snapshot.forEach(doc=>{

const d = doc.data();

if(d.date !== today) return;

total++;

// 🔥 latest per user
if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

});

// render users
document.getElementById("users").innerHTML="";

Object.values(users).forEach(u=>{

let color = u.type==="IN" ? "green" : "red";

if(u.type==="IN") active++;
else out++;

let marker = L.circleMarker([u.lat,u.lon],{
radius:10,
color:color,
fillColor:color,
fillOpacity:0.9
}).addTo(map);

marker.bindPopup(`${u.name}<br>${u.type}<br>${u.time}`);

markers.push(marker);

// USER CARD
document.getElementById("users").innerHTML += `
<div class="user">
<b>${u.name}</b><br>
Status: ${u.type}<br>
Time: ${u.time}
</div>
`;

});

document.getElementById("total").innerText = total;
document.getElementById("active").innerText = active;
document.getElementById("out").innerText = out;

});

</script>

</body>
</html>
