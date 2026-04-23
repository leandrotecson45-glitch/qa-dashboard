
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

/* MAP */
#map{
height:55vh;
}

/* POPUP DESIGN */
.popup-box{
font-size:13px;
max-height:240px;
overflow-y:auto;
}

.popup-header{
font-weight:bold;
margin-bottom:8px;
}

.popup-item{
padding:10px;
margin-bottom:8px;
background:lightgray;
border-radius:10px;
}

.popup-top{
display:flex;
justify-content:space-between;
margin-bottom:5px;
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

/* PURPOSE STYLE */
.purpose{
margin-top:6px;
padding:6px;
background:#111827;
border-left:3px solid #3b82f6;
border-radius:6px;
font-size:12px;
}

</style>
</head>

<body>

<div class="top">
<select id="employee">
<option value="ALL">All Employees</option>
</select>

<input type="date" id="date">
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
let allData=[];

// FILTERS
document.getElementById("employee").onchange=render;
document.getElementById("date").onchange=render;

// REALTIME
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

allData=[];
let names=new Set();

snapshot.forEach(doc=>{
let d=doc.data();
allData.push(d);
names.add(d.name);
});

// EMPLOYEE LIST
let select=document.getElementById("employee");
select.innerHTML=`<option value="ALL">All Employees</option>`;
names.forEach(n=>{
select.innerHTML+=`<option value="${n}">${n}</option>`;
});

render();

});

function render(){

markers.forEach(m=>map.removeLayer(m));
markers=[];

let emp=document.getElementById("employee").value;
let date=document.getElementById("date").value;

let filtered=allData.filter(d=>{

let matchEmp = emp==="ALL" || d.name===emp;

let matchDate = true;
if(date){
let dDate = new Date(d.timestamp).toISOString().split("T")[0];
matchDate = dDate===date;
}

return matchEmp && matchDate;
});

// GROUP
let grouped={};

filtered.forEach(d=>{
let key=d.lat.toFixed(5)+","+d.lon.toFixed(5);
if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);
});

// MARKERS
Object.keys(grouped).forEach(key=>{

let logs=grouped[key];
let lat=logs[0].lat;
let lon=logs[0].lon;

// SORT LATEST
logs.sort((a,b)=>b.timestamp-a.timestamp);

// POPUP
let html=`<div class="popup-box">`;
html+=`<div class="popup-header">📍 ${logs.length} Logs</div>`;

logs.forEach(l=>{

let tagClass="autoTag";
if(l.type==="IN") tagClass="inTag";
if(l.type==="OUT") tagClass="outTag";

html+=`
<div class="popup-item">

<div class="popup-top">
<b>${l.name}</b>
<div class="tag ${tagClass}">${l.type}</div>
</div>

<small>${l.time}</small>

<div class="purpose">
📌 ${l.purpose || "No purpose"}
</div>

</div>
`;

});

html+=`</div>`;

// PIN LABEL
let iconHTML=`
<div style="
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;">
📍 ${logs.length}
</div>
`;

let icon=L.divIcon({
html:iconHTML,
className:"",
iconSize:[60,30]
});

let marker=L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

}

</script>

</body>
</html>
