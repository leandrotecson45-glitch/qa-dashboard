
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

/* TOP BAR */
.top{
display:flex;
gap:10px;
padding:10px;
background:#0f172a;
flex-wrap:wrap;
}

select,input{
padding:8px;
border-radius:8px;
border:none;
}

/* STATS */
.stats{
display:flex;
gap:10px;
padding:10px;
}

.card{
flex:1;
background:#111827;
padding:10px;
border-radius:10px;
text-align:center;
}

/* MAP */
#map{
height:55vh;
}

/* POPUP */
.popup-box{
font-size:13px;
max-height:220px;
overflow-y:auto;
}

.popup-header{
font-weight:bold;
margin-bottom:8px;
}

.popup-item{
padding:10px;
margin-bottom:6px;
background:#1f2937;
border-radius:10px;
display:flex;
justify-content:space-between;
}

.tag{
font-size:11px;
padding:3px 8px;
border-radius:8px;
}

.inTag{background:#22c55e;}
.outTag{background:#ef4444;}
.autoTag{background:#3b82f6;}

</style>
</head>

<body>

<!-- FILTERS -->
<div class="top">
<select id="employee">
<option value="ALL">All Employees</option>
</select>

<input type="date" id="date">
</div>

<!-- STATS -->
<div class="stats">
<div class="card"><div id="total">0</div><small>Total</small></div>
<div class="card"><div id="in">0</div><small>IN</small></div>
<div class="card"><div id="out">0</div><small>OUT</small></div>
</div>

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
let allData=[];

// FILTER CHANGE
document.getElementById("employee").onchange = render;
document.getElementById("date").onchange = render;

// REALTIME
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

allData=[];
let names = new Set();

snapshot.forEach(doc=>{
let d = doc.data();
allData.push(d);
names.add(d.name);
});

// POPULATE EMPLOYEE DROPDOWN
let select = document.getElementById("employee");
select.innerHTML = `<option value="ALL">All Employees</option>`;

names.forEach(n=>{
select.innerHTML += `<option value="${n}">${n}</option>`;
});

render();

});

function render(){

markers.forEach(m=>map.removeLayer(m));
markers=[];

let emp = document.getElementById("employee").value;
let date = document.getElementById("date").value;

let filtered = allData.filter(d=>{

let matchEmp = emp==="ALL" || d.name===emp;

let matchDate = true;
if(date){
let dDate = new Date(d.timestamp).toISOString().split("T")[0];
matchDate = dDate === date;
}

return matchEmp && matchDate;
});

// STATS
let total=filtered.length;
let inCount=filtered.filter(d=>d.type==="IN").length;
let outCount=filtered.filter(d=>d.type==="OUT").length;

document.getElementById("total").innerText=total;
document.getElementById("in").innerText=inCount;
document.getElementById("out").innerText=outCount;

// GROUP BY LOCATION
let grouped={};

filtered.forEach(d=>{
let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);
if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);
});

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

// SORT LATEST
logs.sort((a,b)=>b.timestamp - a.timestamp);

// POPUP
let html = `<div class="popup-box">`;
html += `<div class="popup-header">📍 ${logs.length} Logs</div>`;

logs.forEach(l=>{

let tagClass="autoTag";
if(l.type==="IN") tagClass="inTag";
if(l.type==="OUT") tagClass="outTag";

html += `
<div class="popup-item">
<div>
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
font-size:12px;">
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

}

</script>

</body>
</html>
