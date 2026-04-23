
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
margin:0;
font-family:system-ui,Arial;
background:#0b1220;
color:white;
}

/* MAP */
#map{
height:60vh;
}

/* CHART CONTAINER */
.charts{
padding:10px;
}

.chart-card{
background:#111827;
padding:10px;
border-radius:12px;
margin-bottom:10px;
}

/* PIN */
.pin{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
color:white;
}

/* POPUP */
.popup-box{
max-height:250px;
overflow-y:auto;
}

.card{
background:lightgray;
padding:10px;
border-radius:10px;
margin-bottom:8px;
}

.tag{
font-size:11px;
padding:3px 8px;
border-radius:8px;
}

.inTag{background:#22c55e;}
.outTag{background:#ef4444;}
.autoTag{background:#3b82f6;}

.purpose{
margin-top:5px;
padding:6px;
background:white;
border-left:3px solid #3b82f6;
border-radius:6px;
font-size:12px;
}

</style>
</head>

<body>

<div id="map"></div>

<div class="charts">
<div class="chart-card">
<canvas id="typeChart"></canvas>
</div>

<div class="chart-card">
<canvas id="purposeChart"></canvas>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
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
const db=firebase.firestore();

// MAP
const map=L.map('map').setView([15.5,120.9],13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let markers=[];
let chart1, chart2;

// REALTIME
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let allData=[];
snapshot.forEach(doc=>allData.push(doc.data()));

// GROUP LOCATION
let grouped={};

allData.forEach(d=>{
let key=d.lat.toFixed(5)+","+d.lon.toFixed(5);
if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);
});

// MARKERS
Object.keys(grouped).forEach(key=>{

let logs=grouped[key];
let lat=logs[0].lat;
let lon=logs[0].lon;

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html=`<div class="popup-box">`;

logs.forEach(l=>{

let tag="autoTag";
if(l.type==="IN") tag="inTag";
if(l.type==="OUT") tag="outTag";

html+=`
<div class="card">
<b>${l.name}</b>
<div class="tag ${tag}">${l.type}</div>
<br>
<small>${l.time}</small>
<div class="purpose">📌 ${l.purpose||"No purpose"}</div>
</div>
`;

});

html+=`</div>`;

let icon=L.divIcon({
html:`<div class="pin">📍 ${logs.length}</div>`,
className:"",
iconSize:[60,30]
});

let marker=L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

// 📊 TYPE COUNT
let typeCount={IN:0,OUT:0,AUTO:0};

allData.forEach(d=>{
if(typeCount[d.type]!==undefined) typeCount[d.type]++;
});

// DRAW BAR CHART
if(chart1) chart1.destroy();

chart1=new Chart(document.getElementById("typeChart"),{
type:"bar",
data:{
labels:["IN","OUT","AUTO"],
datasets:[{data:[
typeCount.IN,
typeCount.OUT,
typeCount.AUTO
]}]
}
});

// 📊 PURPOSE COUNT
let purposeCount={};

allData.forEach(d=>{
let p=d.purpose||"None";
if(!purposeCount[p]) purposeCount[p]=0;
purposeCount[p]++;
});

// DRAW PIE CHART
if(chart2) chart2.destroy();

chart2=new Chart(document.getElementById("purposeChart"),{
type:"pie",
data:{
labels:Object.keys(purposeCount),
datasets:[{data:Object.values(purposeCount)}]
}
});

});

</script>

</body>
</html>
