# CFM-56-7-SERIES-PERFORMANCE-AND-HEALTH-MONITORING-DASHBOARD


<html lang="en">
<head>
<meta charset="UTF-8">
<title>Aircraft Engine Performance Monitoring Dashboard</title>

<style>
body{
    font-family: Arial;
    padding:20px;
    background:#f4f6f7;
}

h1{color:#2c3e50;}

.cycle-input{
    background:white;
    padding:10px;
    margin-bottom:10px;
    border-radius:6px;
    box-shadow:0 1px 3px rgba(0,0,0,0.1);
}

table{
    border-collapse:collapse;
    width:100%;
    margin-top:20px;
    background:white;
}

th,td{
    border:1px solid #ccc;
    padding:8px;
    text-align:center;
}

th{
    background:#2c3e50;
    color:white;
}

.status-normal{
    color:green;
    font-weight:bold;
}

.status-monitor{
    color:orange;
    font-weight:bold;
}

.status-alert{
    color:red;
    font-weight:bold;
}

@media print{
    button{display:none;}
    input{border:none;}
}
</style>
</head>

<body>

<h1>✈️ Aircraft Engine Performance Monitoring Dashboard</h1>

<h2>Flight Information</h2>

Tanggal
<input type="date" id="flightDate"><br><br>

Aircraft Registration
<input type="text" id="aircraftReg" placeholder="PK-XXX"><br><br>

Engine Number
<input type="text" id="engineNo" placeholder="Engine 1"><br><br>

Jumlah Flight Cycles
<input type="number" id="numCycles" min="1" max="10" value="3">

<button onclick="generateInputs()">Generate Input</button>

<div id="inputsContainer"></div>

<br>

<button onclick="analyzeCycles()">Analyze Engine Health</button>

<h2>Engine Health Summary</h2>

<table id="summaryTable">
<thead>
<tr>
<th>Cycle</th>
<th>EGT</th>
<th>N1</th>
<th>N2</th>
<th>Oil Pressure</th>
<th>Oil Temp</th>
<th>Vibration</th>
<th>Fuel Flow</th>
<th>Status</th>
<th>Alerts</th>
<th>Maintenance Recommendation</th>
</tr>
</thead>
<tbody></tbody>
</table>

<br><br>

<button onclick="printReport()" style="padding:10px 20px;font-size:16px;">
🖨 Print Engine Health Report
</button>

<script>
const LIMITS = {
    EGT:950,
    N1:5382,
    N2:15183,
    OilPressureMin:13,
    OilPressureMax:60,
    OilTemp:140,
    Vibration:3.8,
    FuelFlow:5160
};

function generateInputs(){
    let num=document.getElementById("numCycles").value;
    let container=document.getElementById("inputsContainer");
    container.innerHTML="";
    for(let i=1;i<=num;i++){
        let div=document.createElement("div");
        div.className="cycle-input";
        div.innerHTML=`
<h3>Cycle ${i}</h3>
EGT (°C)
<input type="number" id="egt${i}" value="850"><br><br>
N1 RPM
<input type="number" id="n1${i}" value="5200"><br><br>
N2 RPM
<input type="number" id="n2${i}" value="15000"><br><br>
Oil Pressure
<input type="number" id="op${i}" value="45"><br><br>
Oil Temp
<input type="number" id="ot${i}" value="120"><br><br>
Vibration
<input type="number" step="0.1" id="vib${i}" value="1.5"><br><br>
Fuel Flow (FFPH)
<input type="number" id="ff${i}" value="5000">
`;
        container.appendChild(div);
    }
}

function analyzeCycles(){
    let num=document.getElementById("numCycles").value;
    let tbody=document.querySelector("#summaryTable tbody");
    tbody.innerHTML="";

    for(let i=1;i<=num;i++){
        let egt=parseFloat(document.getElementById("egt"+i).value);
        let n1=parseFloat(document.getElementById("n1"+i).value);
        let n2=parseFloat(document.getElementById("n2"+i).value);
        let op=parseFloat(document.getElementById("op"+i).value);
        let ot=parseFloat(document.getElementById("ot"+i).value);
        let vib=parseFloat(document.getElementById("vib"+i).value);
        let ff=parseFloat(document.getElementById("ff"+i).value);

        let score=0;
        let alerts=[];
        let recommendation=[];

        // EGT
        if(egt>LIMITS.EGT){
            score+=2;
            alerts.push("EGT OVER LIMIT");
            recommendation.push("Immediate Engine Inspection");
        } else if(egt>900){
            score+=1;
            alerts.push("High EGT");
            recommendation.push("Engine Performance Check");
        }

        // N1
        if(n1>LIMITS.N1){
            score+=1;
            alerts.push("N1 OVER LIMIT");
        }

        // N2
        if(n2>LIMITS.N2){
            score+=1;
            alerts.push("N2 OVER LIMIT");
        }

        // Oil Pressure
        if(op<LIMITS.OilPressureMin || op>LIMITS.OilPressureMax){
            score+=2;
            alerts.push("Oil Pressure OVER LIMIT");
            recommendation.push("Inspect Oil System");
        } else if(op<40 || op>50){
            score+=1;
            alerts.push("Oil Pressure Out of Optimal Range");
        }

        // Oil Temp
        if(ot>LIMITS.OilTemp){
            score+=2;
            alerts.push("Oil Temp OVER LIMIT");
            recommendation.push("Check Oil Cooling System");
        } else if(ot>135){
            score+=1;
            alerts.push("Oil Temp Increasing");
        }

        // Vibration
        if(vib>LIMITS.Vibration){
            score+=2;
            alerts.push("Vibration OVER LIMIT");
            recommendation.push("Fan Balance Inspection"); // 🔥 sudah masuk rekomendasi
        } else if(vib>3.5){
            score+=1;
            alerts.push("Vibration Increasing");
            recommendation.push("Monitor Fan Balance"); // 🔥 rekomendasi mendekati limit
        }

        // Fuel Flow
        if(ff>LIMITS.FuelFlow){
            score+=2;
            alerts.push("Fuel Flow OVER LIMIT");
            recommendation.push("Engine Wash / Compressor Inspection");
        } else if(ff>5000){
            score+=1;
            alerts.push("Fuel Flow Increasing");
            recommendation.push("Monitor Engine Performance");
        }

        // Status
        let statusText="";
        let statusClass="";
        if(score==0){
            statusText="Normal";
            statusClass="status-normal";
        } else if(score<=2){
            statusText="Monitor";
            statusClass="status-monitor";
        } else {
            statusText="Maintenance Required";
            statusClass="status-alert";
        }

        if(recommendation.length===0){
            recommendation.push("None");
        }

        let row=`
<tr>
<td>${i}</td>
<td>${egt}</td>
<td>${n1}</td>
<td>${n2}</td>
<td>${op}</td>
<td>${ot}</td>
<td>${vib}</td>
<td>${ff}</td>
<td class="${statusClass}">${statusText}</td>
<td>${alerts.join(", ")}</td>
<td>${recommendation.join(", ")}</td>
</tr>
`;
        tbody.innerHTML+=row;
    }
}

function printReport(){
    window.print();
}
</script>

