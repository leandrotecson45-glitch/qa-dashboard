
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

.popup{
max-height:150px;
overflow:auto;
font-size:12px;
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

let markers=[];

db.collection("attendance")
.orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped = {};
let total=0,inCount=0,outCount=0;

// 🔥 GROUP BY LOCATION (rounded)
snapshot.forEach(doc=>{

const d = doc.data();

total++;
if(d.type==="IN") inCount++;
if(d.type==="OUT") outCount++;

// 🔥 ROUND COORDS (group same spots)
let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);

});

// 🔥 CREATE ONE MARKER PER LOCATION
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

// 🔥 BUILD POPUP LIST
let html = `<div class="popup">`;

logs.forEach(l=>{
html += `
<div>
<b>${l.name}</b><br>
${l.type} - ${l.time}
<hr>
</div>
`;
});

html += `</div>`;

// 🔥 COLOR BASED ON LAST ENTRY
let last = logs[logs.length-1];
let color = last.type==="IN" ? "green" : "red";

let marker = L.circleMarker([lat,lon],{
radius:12,
color:color,
fillColor:color,
fillOpacity:0.9
}).addTo(map);

marker.bindPopup(html);

markers.push(marker);

});

// COUNTERS
document.getElementById("total").innerText = total;
document.getElementById("in").innerText = inCount;
document.getElementById("out").innerText = outCount;

});

</script>

</body>
</html>
