<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css"/>

<style>
body{margin:0;font-family:Arial;}
#map{height:100vh;}
</style>
</head>

<body>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

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

// 🔥 CLUSTER GROUP (KEY FIX)
const markers = L.markerClusterGroup({
    spiderfyOnMaxZoom: true,
    showCoverageOnHover: false
});

map.addLayer(markers);

// 🔥 SMALL OFFSET (para visible agad kahit di pa click)
function offset(lat, lon, i){
    let shift = i * 0.00003;
    return [lat + shift, lon + shift];
}

// LOAD DATA
db.collection("attendance").onSnapshot(snapshot=>{

markers.clearLayers(); // refresh safely

let i = 0;

snapshot.forEach(doc=>{

const d = doc.data();

let color = d.type==="TIME IN" ? "blue" : "red";

// 🔥 APPLY OFFSET
const [lat, lon] = offset(d.lat, d.lon, i);

const marker = L.circleMarker([lat,lon],{
radius:10,
color:color,
fillColor:color,
fillOpacity:0.9
});

marker.bindPopup(`
<b>${d.name}</b><br>
${d.type}<br>
${d.time}
`);

markers.addLayer(marker);

i++;

});

});

</script>

</body>
</html>
