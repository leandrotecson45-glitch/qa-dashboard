<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body { margin:0; font-family:Arial; }
#map { height:100vh; }
</style>
</head>

<body>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 SAME CONFIG DITO
const firebaseConfig = {
  apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2",
  storageBucket: "attendance1-697b2.firebasestorage.app",
  messagingSenderId: "911862803622",
  appId: "1:911862803622:web:63f0bafabaec74f2665867",
  measurementId: "G-WR38DZW80B"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// MARKERS STORAGE
let markers = {};

// 🔥 REALTIME LISTENER
db.collection("attendance").onSnapshot(snapshot => {

    snapshot.docChanges().forEach(change => {

        const data = change.doc.data();
        const id = change.doc.id;

        if(!markers[id]){

            const marker = L.marker([data.lat,data.lon]).addTo(map);

            marker.bindPopup(`
                <b>${data.name}</b><br>
                ${data.type}<br>
                🕒 ${data.time}
            `);

            markers[id] = marker;
        }

    });

});
</script>

</body>
</html>
