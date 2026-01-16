<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Atharv Ultra AI - Relaxed Login</title>
<style>
body {
    margin: 0;
    font-family: "Segoe UI", sans-serif;
    background: #0d1117;
    color: #e5e5e5;
    padding: 20px;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    min-height: 100vh;
}

.hidden { display: none; }

.login-box, #app-interface {
    max-width: 750px;
    width: 100%;
    margin: auto;
    background: #161b22;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 0 10px #000;
}

input, select, button {
    padding: 10px;
    margin: 5px 0;
    border-radius: 6px;
    border: none;
    width: 70%;
}

button { cursor: pointer; font-weight: bold; }

.glow-btn {
    background: linear-gradient(90deg, #0263ff, #a855f7, #10b981, #ef4444);
    background-size: 300% 300%;
    animation: gradientMove 3s ease infinite;
    border: none;
    color: #000;
}

@keyframes gradientMove {
    0%,100% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
}

#taskList .task-item {
    background: #1f1f1f;
    margin: 5px 0;
    padding: 10px;
    border-radius: 6px;
    display: flex;
    justify-content: space-between;
}

#chat-window {
    background: #1f1f1f;
    height: 300px;
    overflow-y: auto;
    padding: 10px;
    border-radius: 6px;
    margin-bottom: 10px;
}

.bubble {
    padding: 10px 14px;
    border-radius: 10px;
    margin: 8px 0;
    max-width: 80%;
    line-height: 1.4em;
}

.user { background: #0263ff; color: #fff; margin-left: auto; }
.ai { background: #2b2b2b; color: #eee; margin-right: auto; }

.typing {
    width: 60px;
    height: 15px;
    background: url("https://i.imgur.com/6pKcQ3F.gif");
    background-size: contain;
}

.password-wrap { position: relative; }
.eye {
    position: absolute;
    right: 12px;
    top: 50%;
    transform: translateY(-50%);
    cursor: pointer;
    user-select: none;
}
.logout { background: #a40e26; color: white; margin-top:10px; width: 100%; padding:10px; border-radius:6px; border:none; cursor:pointer; }
</style>
</head>
<body>

<!-- LOGIN -->
<div class="login-box" id="loginBox">
  <h2>Atharv Ultra AI - Login</h2>
  <input type="text" id="username" placeholder="Enter username"><br>
  
  <label>Password:</label>
  <div class="password-wrap">
      <input type="password" id="passwordInput" placeholder="Enter password">
      <span class="eye" id="togglePass">👁</span>
  </div>

  <button class="glow-btn" onclick="login()">Login</button>
</div>

<!-- MAIN APP -->
<div id="app-interface" class="hidden">
  <h2>Welcome, <span id="user-name"></span></h2>

  <!-- LOGOUT -->
  <button class="logout" onclick="logout()">Logout</button>

  <!-- TASKS -->
  <h3>Tasks</h3>
  <input id="taskInput" placeholder="Task">
  <select id="subjectSelect">
    <option>General</option> <option>Math</option> <option>Science</option>
    <option>English</option> <option>Social</option> <option>Language</option>
  </select>
  <select id="difficultySelect">
    <option>Easy</option> <option>Medium</option> <option>Difficult</option>
  </select>
  <input type="date" id="taskDate">
  <button class="glow-btn" onclick="addTask()">Add</button>
  <div id="taskList"></div>

  <!-- CHAT -->
  <h3>AI Chat</h3>
  <div id="chat-window"></div>
  <input id="chatInput" placeholder="Say something...">
  <button class="glow-btn" onclick="sendChat()">Send</button>
</div>

<script>
// -----------------------------
// CONFIG — USERNAME + PASSWORD
// -----------------------------
const REAL_USER = "atharv";
const REAL_PASS = "1234"; // simple relaxed password

// -----------------------------
// LOGIN / LOGOUT
// -----------------------------
function login() {
    const name = document.getElementById("username").value.trim();
    const pass = document.getElementById("passwordInput").value;
    if(name === REAL_USER && pass === REAL_PASS){
        localStorage.setItem("loggedIn","true");
        document.getElementById("user-name").textContent = name;
        showApp();
    } else {
        alert("Incorrect username or password ❌");
    }
}

function logout() {
    localStorage.removeItem("loggedIn");
    showLogin();
}

window.onload = function(){
    if(localStorage.getItem("loggedIn")==="true"){
        showApp();
        document.getElementById("user-name").textContent = REAL_USER;
    }
}

function showApp(){
    document.getElementById("loginBox").style.display="none";
    document.getElementById("app-interface").classList.remove("hidden");
}
function showLogin(){
    document.getElementById("loginBox").style.display="block";
    document.getElementById("app-interface").classList.add("hidden");
}

// -----------------------------
// SHOW / HIDE PASSWORD
// -----------------------------
document.getElementById("togglePass").onclick = function(){
    const field = document.getElementById("passwordInput");
    field.type = field.type==="password"?"text":"password";
}

// -----------------------------
// APP STATE
// -----------------------------
let state = { tasks:[], chat:[] };

// -----------------------------
// TASKS
// -----------------------------
function addTask(){
    const t = document.getElementById("taskInput").value.trim();
    if(!t) return;
    state.tasks.push({
        title: t,
        subject: subjectSelect.value,
        difficulty: difficultySelect.value,
        due: taskDate.value
    });
    renderTasks();
    document.getElementById("taskInput").value="";
}

function renderTasks(){
    const box = document.getElementById("taskList");
    box.innerHTML="";
    state.tasks.forEach((t,i)=>{
        const div=document.createElement("div");
        div.className="task-item";
        div.innerHTML=`<span>${t.title} — ${t.subject} / ${t.difficulty} ${t.due ? "(Due: "+t.due+")":""}</span>
        <button class="glow-btn" onclick="removeTask(${i})">Done</button>`;
        box.appendChild(div);
    });
}

function removeTask(i){ state.tasks.splice(i,1); renderTasks(); }

// -----------------------------
// CHAT — OpenAI API DIRECT
// -----------------------------
let typingDiv;
async function sendChat(){
    const msg=chatInput.value.trim();
    if(!msg) return;
    chatInput.value="";
    state.chat.push({role:"user", text:msg});
    renderChat();
    showTyping();
    const data={ model:"gpt-4o-mini", messages:[{role:"user", content:msg}] };
    try{
        const res=await fetch("https://api.openai.com/v1/chat/completions",{
            method:"POST",
            headers:{
                "Content-Type":"application/json",
                "Authorization":"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"
            },
            body: JSON.stringify(data)
        });
        const result=await res.json();
        hideTyping();
        const aiReply=result.choices?.[0]?.message?.content||"Error: No reply";
        state.chat.push({role:"ai", text:aiReply});
        renderChat();
    }catch(e){
        hideTyping();
        state.chat.push({role:"ai", text:"⚠️ Error connecting to AI."});
        renderChat();
    }
}

function renderChat(){
    const win=document.getElementById("chat-window");
    win.innerHTML="";
    state.chat.forEach(m=>{
        const div=document.createElement("div");
        div.className="bubble "+(m.role==="user"?"user":"ai");
        div.textContent=m.text;
        win.appendChild(div);
    });
    win.scrollTop=win.scrollHeight;
}

function showTyping(){
    const win=document.getElementById("chat-window");
    typingDiv=document.createElement("div");
    typingDiv.className="bubble ai";
    typingDiv.innerHTML=`<div class="typing"></div>`;
    win.appendChild(typingDiv);
    win.scrollTop=win.scrollHeight;
}

function hideTyping(){ if(typingDiv) typingDiv.remove(); }
</script>

</body>
</html>
