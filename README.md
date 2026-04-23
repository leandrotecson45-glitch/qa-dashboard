
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
margin:0;
font-family:Arial;
background:#0b1220;
color:white;
}

#map{
height:60vh;
}

/* POPUP DESIGN */
.popup-box{
font-size:13px;
max-height:220px;
overflow-y:auto;
}

.popup-header{
font-weight:bold;
margin-bottom:8px;
font-size:14px;
}

.popup-item{
padding:10px;
margin-bottom:6px;
background:#1f2937;
border-radius:10px;
display:flex;
justify-content:space-between;
align-items:center;
}

.popup-left{
line-height:1.3;
}

.tag{
font-size:11px;
padding:3px 8px;
border-radius:8px;
font-weight:bold;
}

.inTag{background:#22c55e;}
.outTag{background:#ef4444;}
.autoTag{background:#3b82f6;}

</style>
</head>

<body>

<h3 style="padding:10px;">QA Dashboard</h3>
<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 FIREBASE CONFIG
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

// REALTIME
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped = {};

// GROUP BY LOCATION
snapshot.forEach(doc=>{
let d = doc.data();
let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);
});

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

// SORT LATEST FIRST
logs.sort((a,b)=>b.timestamp - a.timestamp);

// POPUP UI
let html = `<div class="popup-box">`;
html += `<div class="popup-header">📍 ${logs.length} Logs</div>`;

logs.forEach(l=>{

let tagClass = "autoTag";
if(l.type==="IN") tagClass="inTag";
if(l.type==="OUT") tagClass="outTag";

html += `
<div class="popup-item">
<div class="popup-left">
<b>${l.name}</b><br>
<small>${l.time}</small>
</div>
<div class="tag ${tagClass}">
${l.type}
</div>
</div>
`;

});

html += `</div>`;

// LABEL
let iconHTML = `
<div style="
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
color:white;
border:1px solid #333;">
📍 ${logs.length}
</div>
`;

let icon = L.divIcon({
html:iconHTML,
className:"",
iconSize:[60,30]
});

let marker = L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

});

</script>

</body>
</html>
