
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Dashboard</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}

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

/* SUMMARY */
.summary{
display:flex;
gap:10px;
padding:10px;
overflow-x:auto;
}

.sum-card{
background:#111827;
padding:10px;
border-radius:10px;
min-width:120px;
text-align:center;
font-size:13px;
}

/* MAP */
#map{height:55vh;}

/* POPUP */
.popup-box{
font-size:13px;
max-height:240px;
overflow-y:auto;
}

.popup-item{
padding:10px;
margin-bottom:8px;
background:lightgray;
border-radius:10px;
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
margin-top:6px;
padding:6px;
background:white;
border-left:3px solid #3b82f6;
border-radius:6px;
font-size:12px;
}

</style>
</head>

<body>

<!-- FILTER -->
<div class="top">
<select id="employee"><option value="ALL">All Employees</option></select>
<input type="date" id="date">
<select id="purposeFilter"><option value="ALL">All Purpose</option></select>
</div>

<!-- SUMMARY -->
<div class="summary" id="summary"></div>

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

let allData=[];
let markers=[];

// FILTER EVENTS
["employee","date","purposeFilter"].forEach(id=>{
document.getElementById(id).onchange = render;
});

// REALTIME
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

allData=[];
let names=new Set();
let purposes=new Set();

snapshot.forEach(doc=>{
let d=doc.data();
allData.push(d);
names.add(d.name);
if(d.purpose) purposes.add(d.purpose);
});

// EMPLOYEE
let emp=document.getElementById("employee");
emp.innerHTML=`<option value="ALL">All Employees</option>`;
names.forEach(n=>emp.innerHTML+=`<option>${n}</option>`);

// PURPOSE
let pf=document.getElementById("purposeFilter");
pf.innerHTML=`<option value="ALL">All Purpose</option>`;
purposes.forEach(p=>pf.innerHTML+=`<option>${p}</option>`);

render();

});

function render(){

markers.forEach(m=>map.removeLayer(m));
markers=[];

let emp=document.getElementById("employee").value;
let date=document.getElementById("date").value;
let purposeF=document.getElementById("purposeFilter").value;

// FILTER
let filtered = allData.filter(d=>{

let matchEmp = emp==="ALL" || d.name===emp;

let matchDate = true;
if(date){
let dDate=new Date(d.timestamp).toISOString().split("T")[0];
matchDate=dDate===date;
}

let matchPurpose = purposeF==="ALL" || d.purpose===purposeF;

return matchEmp && matchDate && matchPurpose;
});

// 🔥 SUMMARY PURPOSE COUNT
let summaryBox=document.getElementById("summary");
summaryBox.innerHTML="";

let countMap={};

filtered.forEach(d=>{
let p=d.purpose || "No Purpose";
if(!countMap[p]) countMap[p]=0;
countMap[p]++;
});

Object.keys(countMap).forEach(p=>{
summaryBox.innerHTML+=`
<div class="sum-card">
<b>${countMap[p]}</b><br>
${p}
</div>
`;
});

// GROUP LOCATION
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

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html=`<div class="popup-box">`;

logs.forEach(l=>{

let tagClass="autoTag";
if(l.type==="IN") tagClass="inTag";
if(l.type==="OUT") tagClass="outTag";

html+=`
<div class="popup-item">
<b>${l.name}</b><br>
<small>${l.time}</small>
<div class="tag ${tagClass}">${l.type}</div>
<div class="purpose">📌 ${l.purpose || "No purpose"}</div>
</div>
`;

});

html+=`</div>`;

// PIN
let iconHTML=`
<div style="background:#111827;padding:6px 10px;border-radius:20px;font-size:12px;">
📍 ${logs.length}
</div>`;

let icon=L.divIcon({html:iconHTML,className:"",iconSize:[60,30]});

let marker=L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

}

</script>

</body>
</html>
