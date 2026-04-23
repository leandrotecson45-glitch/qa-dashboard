<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}
#map{height:60vh;}
</style>
</head>

<body>

<h3 style="padding:10px;">QA Dashboard</h3>
<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 SAME CONFIG DAPAT SA FIELD
const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png')
.addTo(map);

db.collection("attendance")
.onSnapshot(snapshot=>{

console.log("DATA RECEIVED:", snapshot.size);

// CLEAR MAP
map.eachLayer(layer=>{
if(layer instanceof L.Marker){
map.removeLayer(layer);
}
});

// ADD MARKERS
snapshot.forEach(doc=>{

let d = doc.data();

console.log(d);

L.marker([d.lat, d.lon]).addTo(map)
.bindPopup(`
<b>${d.name}</b><br>
${d.type}<br>
${d.time}
`);

});

});

</script>

</body>
</html>
