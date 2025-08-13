// --- CONFIG: set your backend API base URL here ---
const API_BASE = "https://API_BASE_REPLACE_ME"; // <<--- REPLACE this with your backend URL

// --- simple DOM helpers ---
const $ = (sel) => document.querySelector(sel);
const show = (el) => el.classList.remove("hidden");
const hide = (el) => el.classList.add("hidden");

// Views & controls
const loginView = $("#loginView");
const dashboardView = $("#dashboardView");
const btnLogin = $("#btnLogin");
const usernameInput = $("#username");
const passwordInput = $("#password");
const userArea = $("#userArea");

const p_name = $("#p_name");
const p_guildId = $("#p_guildId");
const p_joinDate = $("#p_joinDate");
const p_photo = $("#p_photo");
const btnAdd = $("#btnAdd");
const btnExport = $("#btnExport");
const addMsg = $("#addMsg");

const rosterEl = $("#roster");
const totalCount = $("#totalCount");
const latestName = $("#latestName");

// token helpers
function getToken(){ return localStorage.getItem("ff_token"); }
function setToken(t){ if(t) localStorage.setItem("ff_token", t); else localStorage.removeItem("ff_token"); }
function authHeaders(){ const t = getToken(); return t ? { "Authorization": "Bearer " + t } : {}; }

// initial flow
document.addEventListener("DOMContentLoaded", () => {
  if (getToken()) openDashboard();
  else openLogin();
  bindEvents();
  fetchRoster(); // public roster view (reads all players)
});

function openLogin(){
  show(loginView); hide(dashboardView);
  userArea.innerHTML = "";
}
function openDashboard(){
  show(dashboardView); hide(loginView);
  userArea.innerHTML = `<button id="btnLogout" class="btn">Logout</button>`;
  document.getElementById("btnLogout").addEventListener("click", () => { setToken(null); openLogin(); });
}

// bind UI handlers
function bindEvents(){
  btnLogin.addEventListener("click", login);
  btnAdd.addEventListener("click", addPlayer);
  btnExport.addEventListener("click", exportExcel);
}

// LOGIN
async function login(){
  btnLogin.disabled = true;
  const username = usernameInput.value.trim();
  const password = passwordInput.value;
  try{
    const res = await fetch(`${API_BASE}/login`, {
      method: "POST",
      headers: { "Content-Type":"application/json" },
      body: JSON.stringify({ username, password })
    });
    if(!res.ok) throw new Error("Login failed");
    const data = await res.json();
    setToken(data.token);
    openDashboard();
  }catch(err){
    alert("Login failed — check credentials & backend URL.");
    console.error(err);
  }finally{ btnLogin.disabled = false; }
}

// FETCH roster (public)
async function fetchRoster(){
  try{
    const res = await fetch(`${API_BASE}/players`);
    const list = await res.json();
    renderRoster(list);
  }catch(err){
    console.warn("Fetch roster failed", err);
    rosterEl.innerHTML = `<div class="muted">Unable to fetch roster. Is backend URL set?</div>`;
  }
}

function renderRoster(list){
  rosterEl.innerHTML = "";
  totalCount.textContent = list.length;
  latestName.textContent = list[0]?.name ?? "—";
  for(const p of list){
    const div = document.createElement("div");
    div.className = "member";
    const img = document.createElement("img");
    img.src = p.photo ? `${API_BASE}${p.photo}` : "data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///ywAAAAAAQABAAACAUwAOw==";
    img.alt = p.name;
    const meta = document.createElement("div");
    meta.className = "meta";
    meta.innerHTML = `<div style="font-weight:600">${p.name}</div><div class="muted">ID: ${p.guildId} • ${p.joinDate}</div>`;
    div.appendChild(img); div.appendChild(meta);
    rosterEl.appendChild(div);
  }
}

// ADD player (multipart + requires token)
async function addPlayer(){
  const t = getToken();
  if(!t){ alert("Login required to add players."); openLogin(); return; }

  const name = p_name.value.trim();
  const guildId = p_guildId.value.trim();
  const joinDate = p_joinDate.value || new Date().toISOString().slice(0,10);
  if(!name || !guildId){ addMsg.textContent = "Name and Guild ID required."; return; }

  const fd = new FormData();
  fd.append("name", name);
  fd.append("guildId", guildId);
  fd.append("joinDate", joinDate);
  if(p_photo.files[0]) fd.append("photo", p_photo.files[0]);

  addMsg.textContent = "Uploading ...";
  btnAdd.disabled = true;

  try{
    const res = await fetch(`${API_BASE}/players`, {
      method: "POST",
      headers: { "Authorization": "Bearer " + t },
      body: fd
    });
    if(!res.ok) throw new Error("Upload failed");
    addMsg.textContent = "Player added.";
    // refresh roster
    fetchRoster();
    // clear form
    p_name.value=""; p_guildId.value=""; p_joinDate.value=""; p_photo.value="";
  }catch(err){
    addMsg.textContent = "Failed to add player.";
    console.error(err);
  }finally{ btnAdd.disabled = false; setTimeout(()=>addMsg.textContent="",3000); }
}

// EXPORT Excel (requires token)
async function exportExcel(){
  const t = getToken();
  if(!t){ alert("Login required"); openLogin(); return; }
  btnExport.disabled = true;
  try{
    const res = await fetch(`${API_BASE}/export`, {
      headers: { "Authorization": "Bearer " + t }
    });
    if(!res.ok) throw new Error("Export failed");
    const blob = await res.blob();
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `guild-players-${new Date().toISOString().slice(0,10)}.xlsx`;
    document.body.appendChild(a);
    a.click();
    a.remove();
  }catch(err){
    alert("Export failed — check login and backend.");
    console.error(err);
  }finally{ btnExport.disabled = false; }
}
