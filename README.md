<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CLOUT SMS</title>
<style>
 *{box-sizing:border-box;margin:0;padding:0;font-family:Arial,sans-serif}
 body{background:#0a0f1e;color:white;padding-bottom:80px}
.login{position:fixed;top:0;left:0;right:0;bottom:0;background:#1e3a8a;display:flex;justify-content:center;align-items:center;z-index:999}
.login div{background:#1f2937;padding:30px;border-radius:20px;width:90%;max-width:350px;text-align:center}
input,button{width:100%;padding:14px;margin:8px 0;border-radius:10px;border:none;font-size:16px}
button{background:#3b82f6;color:white;font-weight:bold;cursor:pointer}
.hidden{display:none}
.header{background:#1e3a8a;padding:15px;text-align:center;position:sticky;top:0}
.container{padding:15px}
.card{background:#1f2937;padding:15px;border-radius:15px;margin-bottom:15px}
.student{background:#111827;padding:12px;border-radius:10px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center}
.grade{padding:4px 10px;border-radius:20px;font-size:12px;font-weight:700}
.A{background:#10b981}.B{background:#0ea5e9}.C{background:#f59e0b}.D{background:#f97316}.F{background:#ef4444}
.nav{position:fixed;bottom:0;left:0;right:0;background:#111827;display:flex;justify-content:space-around;padding:10px}
.nav button{background:none;border:none;color:#9ca3af;font-size:12px}
.nav button.active{color:#3b82f6}
.fab{position:fixed;bottom:90px;right:20px;width:56px;height:56px;border-radius:50%;background:#3b82f6;border:none;color:white;font-size:24px}
.modal{position:fixed;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,0.8);padding:20px;overflow-y:auto;z-index:200}
.att-row{display:flex;justify-content:space-between;align-items:center;padding:10px;background:#111827;border-radius:8px;margin-bottom:8px}
.att-btn{padding:6px 10px;border:none;border-radius:6px;color:white;margin-left:5px}
.P{background:#10b981}.A{background:#ef4444}.L{background:#f59e0b}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="login" class="login">
    <div>
        <h1 style="color:#60a5fa">CLOUT SMS PRO</h1>
        <p style="color:#9ca3af;margin-bottom:15px">Password: 123456</p>
        <input type="password" id="pass" placeholder="Enter Password">
        <button onclick="login()">Login</button>
        <button onclick="resetAll()" style="background:#ef4444">Reset Everything</button>
        <p id="err" style="color:red" class="hidden">Wrong Password</p>
    </div>
</div>

<!-- MAIN APP -->
<div id="app" class="hidden">
    <div class="header"><h1 id="sname">CLOUT SMS PRO</h1></div>

    <input type="text" id="search" placeholder="Search student..." onkeyup="render()" style="width:calc(100% - 30px);margin:15px;border-radius:25px;padding:12px;background:#1f2937;color:white;border:none">

    <div class="container">
        <div id="tab-home"></div>
        <div id="tab-add" class="hidden"><div class="card"><h2>Add Student</h2><input id="n" placeholder="Full Name"><input id="c" placeholder="Class"><input id="id" placeholder="Student ID"><div id="score-box"></div><button onclick="addStu()">Save</button></div></div>
        <div id="tab-att" class="hidden"><div class="card"><h2>Attendance</h2><input type="date" id="date"><button onclick="saveAtt()">Save Attendance</button></div><div id="att-list"></div></div>
        <div id="tab-rank" class="hidden"><div class="card"><h2>Top 10</h2><table id="rank" style="width:100%"></table></div></div>
        <div id="tab-set" class="hidden">
            <div class="card"><h2>School Name</h2><input id="school" placeholder="School Name"><button onclick="saveSchool()">Save</button></div>
            <div class="card"><h2>Subjects</h2><div id="sub-list"></div><input id="newsub" placeholder="Add Subject"><button onclick="addSub()">Add</button></div>
            <div class="card"><h2>Export</h2><button onclick="exportCSV()">Export Students CSV</button></div>
            <div class="card"><h2>Danger</h2><button onclick="resetAll()" style="background:#ef4444">Reset All Data</button></div>
        </div>
    </div>

    <button class="fab" onclick="show('tab-add')">+</button>
    <div class="nav">
        <button class="active" onclick="show('tab-home')">Home</button>
        <button onclick="show('tab-att')">Attendance</button>
        <button onclick="show('tab-rank')">Ranking</button>
        <button onclick="show('tab-set')">Settings</button>
    </div>

    <div id="edit" class="modal hidden">
        <div class="card"><h2>Edit Student</h2><input id="en"><input id="ec"><input id="eid" readonly><div id="edit-scores"></div>
        <button style="background:#f59e0b" onclick="saveEdit()">Update</button>
        <button onclick="printRC()">Print Report Card</button>
        <button onclick="closeEdit()">Cancel</button>
        </div>
    </div>
</div>

<script>
// SIMPLE VERSION - NO EXTERNAL LIBRARIES
const PASSWORD = "123456";
let students = [];
let attendance = {};
let subjects = ["Maths","English","Science","Social Studies"];
let editID = null;

function resetAll(){
    if(confirm("Delete all data?")){
        students=[]; attendance={};
        sessionStorage.clear();
        alert("Reset done. Password is 123456");
        location.reload();
    }
}

function login(){
    if(document.getElementById('pass').value === PASSWORD){
        document.getElementById('login').classList.add('hidden');
        document.getElementById('app').classList.remove('hidden');
        document.getElementById('date').valueAsDate = new Date();
        loadScores();
        render();
    } else {
        document.getElementById('err').classList.remove('hidden');
    }
}

function getGrade(avg){ if(avg>=80)return"A";if(avg>=70)return"B";if(avg>=60)return"C";if(avg>=50)return"D";return"F"; }

function loadScores(){
    let html=""; subjects.forEach(s=> html+=`<label>${s}</label><input type="number" id="s_${s}" placeholder="0-100">`);
    document.getElementById('score-box').innerHTML=html;
}

function addStu(){
    let n=document.getElementById('n').value, c=document.getElementById('c').value, id=document.getElementById('id').value;
    if(!n||!c||!id)return alert("Fill all");
    let scores={}; subjects.forEach(s=> scores[s]=parseFloat(document.getElementById(`s_${s}`).value)||0);
    students.push({n,c,id,scores});
    alert("Added!");
    ['n','c','id'].forEach(x=>document.getElementById(x).value="");
    subjects.forEach(s=>document.getElementById(`s_${s}`).value="");
    show('tab-home'); render();
}

function render(){
    let filter=document.getElementById('search').value.toLowerCase();
    let html=""; students.filter(s=>s.n.toLowerCase().includes(filter)).forEach(s=>{
        let avg=Object.values(s.scores).reduce((a,b)=>a+b)/subjects.length;
        html+=`<div class="student" onclick="openEdit('${s.id}')"><div><b>${s.n}</b><br><small>${s.c} - Avg: ${avg.toFixed(1)}</small></div><span class="grade ${getGrade(avg)}">${getGrade(avg)}</span></div>`;
    });
    document.getElementById('tab-home').innerHTML=html||"<div class='card'>No students. Tap +</div>";
}

function openEdit(id){
    editID=id; let s=students.find(x=>x.id==id);
    document.getElementById('en').value=s.n; document.getElementById('ec').value=s.c; document.getElementById('eid').value=s.id;
    let html=""; subjects.forEach(sub=> html+=`<label>${sub}</label><input type="number" id="e_${sub}" value="${s.scores[sub]}">`);
    document.getElementById('edit-scores').innerHTML=html;
    document.getElementById('edit').classList.remove('hidden');
}

function closeEdit(){document.getElementById('edit').classList.add('hidden')}
function saveEdit(){let s=students.find(x=>x.id==editID); s.n=document.getElementById('en').value; s.c=document.getElementById('ec').value; subjects.forEach(sub=> s.scores[sub]=parseFloat(document.getElementById(`e_${sub}`).value)||0); closeEdit(); render();}

function show(tab){
    document.querySelectorAll('.container>div').forEach(d=>d.classList.add('hidden'));
    document.getElementById(tab).classList.remove('hidden');
    document.querySelectorAll('.nav button').forEach(b=>b.classList.remove('active'));
    event.target.classList.add('active');
    if(tab=='tab-att')renderAtt();
    if(tab=='tab-rank')renderRank();
    if(tab=='tab-set')renderSubs();
}

function renderAtt(){
    let d=document.getElementById('date').value; if(!attendance[d])attendance[d]={};
    let html=""; students.forEach(s=>{let st=attendance[d][s.id]||'P'; html+=`<div class="att-row"><div>${s.n}</div><div><button class="att-btn P ${st=='P'?'':''}" onclick="mark('${s.id}','P')">P</button><button class="att-btn A ${st=='A'?'':''}" onclick="mark('${s.id}','A')">A</button><button class="att-btn L ${st=='L'?'':''}" onclick="mark('${s.id}','L')">L</button></div></div>`});
    document.getElementById('att-list').innerHTML=html;
}
function mark(id,st){attendance[document.getElementById('date').value][id]=st; renderAtt();}
function saveAtt(){alert("Attendance Saved")}

function renderRank(){
    let sorted=students.map(s=>({...s,avg:Object.values(s.scores).reduce((a,b)=>a+b)/subjects.length})).sort((a,b)=>b.avg-a.avg).slice(0,10);
    let html="<tr><td>#</td><td>Name</td><td>Avg</td></tr>";
    sorted.forEach((s,i)=>html+=`<tr><td>${i+1}</td><td>${s.n}</td><td>${s.avg.toFixed(1)}</td></tr>`);
    document.getElementById('rank').innerHTML=html;
}

function renderSubs(){document.getElementById('sub-list').innerHTML=subjects.map(s=>`<p>• ${s}</p>`).join('')}
function addSub(){let s=document.getElementById('newsub').value; if(s&&!subjects.includes(s)){subjects.push(s); renderSubs(); loadScores();}}

function saveSchool(){document.getElementById('sname').innerText=document.getElementById('school').value||"CLOUT SMS PRO"}

function exportCSV(){
    let csv="Name,Class,ID,"+subjects.join(",")+"\n";
    students.forEach(s=> csv+=`${s.n},${s.c},${s.id},${subjects.map(x=>s.scores[x]).join(",")}\n`);
    let a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([csv])); a.download="students.csv"; a.click();
}

function printRC(){
    let s=students.find(x=>x.id==editID); let avg=Object.values(s.scores).reduce((a,b)=>a+b)/subjects.length;
    let w=window.open('','_blank');
    w.document.write(`<h1>${document.getElementById('sname').innerText}</h1><h2>Report Card</h2><p>Name: ${s.n}</p><p>Class: ${s.c}</p><hr>`);
    subjects.forEach(sub=> w.document.write(`<p>${sub}: ${s.scores[sub]}</p>`));
    w.document.write(`<hr><p>Average: ${avg.toFixed(1)}% - Grade: ${getGrade(avg)}</p>`);
    w.print(); w.close();
}
</script>
</body>
</html>
