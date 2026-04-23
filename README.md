<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Debug</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;background:#111;color:white;font-family:Arial;}
#map{height:60vh;}
#log{padding:10px;font-size:12px;background:#000;}
</style>
</head>

<body>

<h3 style="padding:10px;">QA DEBUG</h3>
<div id="map"></div>
<div id="log"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 SAME CONFIG
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

function log(msg){
document.getElementById("log").innerHTML += msg + "<br>";
}

// 🔥 REALTIME LISTENER
db.collection("attendance")
.onSnapshot(snapshot=>{

log("🔥 DATA COUNT: " + snapshot.size);

snapshot.forEach(doc=>{

let d = doc.data();

log(JSON.stringify(d));

// 🔥 CHECK LAT LON
if(!d.lat || !d.lon){
log("❌ Missing lat/lon");
return;
}

// 🔥 FORCE NUMBER
let lat = parseFloat(d.lat);
let lon = parseFloat(d.lon);

// 🔥 CHECK INVALID
if(isNaN(lat) || isNaN(lon)){
log("❌ Invalid coordinates");
return;
}

// ADD MARKER
L.marker([lat,lon]).addTo(map)
.bindPopup(`
<b>${d.name}</b><br>
${d.type}<br>
${d.time}
`);

});

}, err=>{
log("❌ ERROR: " + err.message);
});

</script>

</body>
</html>
