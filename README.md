<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;}
#map{height:70vh;}
#log{height:30vh;overflow:auto;background:#111;color:#0f0;font-size:12px;padding:10px;}
</style>
</head>

<body>

<div id="map"></div>
<div id="log">QA DEBUG:</div>

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

function log(msg){
document.getElementById("log").innerHTML += "<br>"+msg;
console.log(msg);
}

log("🔄 Listening Firebase...");

db.collection("attendance").onSnapshot(snapshot=>{

log("🔥 DATA COUNT: "+snapshot.size);

snapshot.docChanges().forEach(change=>{

const d=change.doc.data();

log("📍 "+d.name+" "+d.type);

if(change.type==="added"){
L.marker([d.lat,d.lon]).addTo(map)
.bindPopup(d.name+" "+d.type+"<br>"+d.time);
}

});

}, err=>{
log("❌ SNAPSHOT ERROR: "+err.message);
});

</script>

</body>
</html>
