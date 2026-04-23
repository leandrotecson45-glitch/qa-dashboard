<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}

#map{height:60vh;}

.popup-item{
padding:6px;
margin-bottom:4px;
background:white;
border-radius:6px;
}

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

<h3 style="padding:10px;">QA Dashboard</h3>
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

// GROUP BY LOCATION
snapshot.forEach(doc=>{
const d = doc.data();

let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);
if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);

// latest user status
if(!users[d.name] || users[d.name].timestamp < d.timestamp){
users[d.name] = d;
}

});

// CREATE ONE MARKER PER LOCATION
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

// 👉 POPUP LIST (LAHAT LALABAS)
let html = `<div style="font-size:13px;">`;

logs.forEach(l=>{
html += `
<div class="popup-item">
<b>${l.name}</b><br>
${l.type} - ${l.time}
</div>
`;
});

html += `</div>`;

// 👉 LABEL = COUNT
let iconHTML = `
<div style="
background:darkgreen;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:white;
border:1px solid #333;
">
📍 ${logs.length}
</div>
`;

let customIcon = L.divIcon({
html: iconHTML,
className: "",
iconSize: [60,30]
});

let marker = L.marker([lat,lon],{
icon: customIcon
}).addTo(map);

marker.bindPopup(html);

markers.push(marker);

});

// USERS LIST
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
