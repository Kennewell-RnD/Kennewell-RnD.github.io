<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>R&D Hours Tracker</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700&family=DM+Mono&display=swap" rel="stylesheet" />
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --orange: #E8B000;
    --orange-d: #B88A00;
    --orange-l: #FFF3CC;
    --grey: #3D3D3D;
    --grey-m: #6B6B6B;
    --grey-l: #F4F4F2;
    --border: #E2E2E0;
    --red: #c0392b;
    --red-l: #fdecea;
    --green: #2a7a4b;
    --dark: #1a1a1a;
  }
  body { font-family: 'DM Sans', system-ui, sans-serif; background: var(--grey-l); color: var(--grey); min-height: 100vh; }
  .app { max-width: 960px; margin: 0 auto; padding: 1.5rem 1rem 4rem; }
  .banner { background: var(--dark); border-radius: 16px; padding: 1.5rem 2rem; margin-bottom: 1.75rem; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem; }
  .banner-left { display: flex; align-items: center; gap: 1.25rem; }
  .banner-logo { height: 48px; width: auto; display: block; }
  .banner-divider { width: 1px; height: 40px; background: rgba(255,255,255,0.2); }
  .banner-text h1 { font-size: 20px; font-weight: 700; color: #fff; margin-bottom: 2px; }
  .banner-text p { font-size: 12px; color: rgba(255,255,255,.6); }
  .admin-badge { background: rgba(255,255,255,.12); color: #fff; font-size: 12px; font-weight: 700; padding: 6px 14px; border-radius: 20px; cursor: pointer; border: none; font-family: inherit; white-space: nowrap; }
  .admin-badge:hover { background: rgba(255,255,255,.22); }
  .nav { display: flex; gap: 4px; background: #fff; border: 1px solid var(--border); border-radius: 12px; padding: 4px; width: fit-content; margin-bottom: 1.75rem; }
  .nav-btn { font-family: inherit; font-size: 13px; font-weight: 500; border: none; padding: 8px 18px; border-radius: 9px; cursor: pointer; background: none; color: var(--grey-m); transition: all .15s; }
  .nav-btn.active { background: var(--dark); color: #fff; }
  .card { background: #fff; border: 1px solid var(--border); border-radius: 16px; padding: 1.75rem; margin-bottom: 1.25rem; }
  .reminder-box { background: #fffbea; border: 1.5px solid #ffe066; border-radius: 12px; padding: .9rem 1.25rem; margin-bottom: 1.5rem; display: flex; align-items: center; gap: .75rem; font-size: 13px; color: #7a6000; font-weight: 500; }
  .reminder-box .reminder-icon { font-size: 18px; flex-shrink: 0; }
  label.field-label { font-size: 11px; font-weight: 700; letter-spacing: .05em; color: var(--grey-m); text-transform: uppercase; display: block; margin-bottom: 6px; }
  input, select, textarea { font-family: inherit; font-size: 14px; padding: 10px 14px; border: 1.5px solid var(--border); border-radius: 10px; background: #fff; color: var(--grey); outline: none; width: 100%; transition: border-color .15s; }
  input:focus, select:focus, textarea:focus { border-color: var(--orange-d); }
  input[readonly] { background: var(--grey-l); color: var(--grey-m); }
  textarea { min-height: 90px; line-height: 1.6; resize: vertical; }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.25rem; }
  .full { grid-column: 1 / -1; }
  .btn { font-family: inherit; font-size: 15px; font-weight: 700; padding: 14px 0; background: var(--dark); color: #fff; border: none; border-radius: 12px; cursor: pointer; width: 100%; margin-top: 1.25rem; letter-spacing: .01em; transition: all .15s; }
  .btn:hover { background: #333; }
  .btn:disabled { background: var(--border); color: var(--grey-m); cursor: not-allowed; }
  .btn-note { text-align: center; font-size: 12px; color: var(--grey-m); margin-top: 12px; }
  .stat-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; margin-bottom: 1.5rem; }
  .stat-card { background: #fff; border: 1px solid var(--border); border-radius: 12px; padding: 1.25rem; }
  .stat-val { font-size: 30px; font-weight: 700; color: var(--dark); font-family: 'DM Mono', monospace; margin-bottom: 2px; }
  .stat-lbl { font-size: 12px; color: var(--grey-m); font-weight: 500; }
  .section-title { font-size: 11px; font-weight: 700; letter-spacing: .06em; color: var(--grey-m); text-transform: uppercase; margin-bottom: 1rem; margin-top: 1.5rem; }
  .table-wrap { background: #fff; border: 1px solid var(--border); border-radius: 14px; overflow: hidden; margin-bottom: 1.5rem; overflow-x: auto; }
  table { width: 100%; border-collapse: collapse; }
  th { padding: 11px 14px; text-align: left; font-weight: 700; font-size: 11px; letter-spacing: .04em; color: #fff; background: var(--dark); white-space: nowrap; }
  td { padding: 11px 14px; font-size: 13px; border-bottom: 1px solid var(--grey-l); color: var(--grey); vertical-align: middle; }
  tr:last-child td { border-bottom: none; }
  tr:nth-child(even) td { background: var(--grey-l); }
  .tag { display: inline-block; padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 700; background: var(--orange-l); color: var(--orange-d); }
  .mono { font-family: 'DM Mono', monospace; }
  .bold { font-weight: 700; }
  .accent { color: var(--dark); }
  .muted { color: var(--grey-m); font-size: 12px; }
  .filter-bar { display: flex; gap: .75rem; flex-wrap: wrap; margin-bottom: 1.5rem; align-items: center; }
  .filter-bar select { width: auto; font-size: 13px; padding: 8px 12px; border-radius: 8px; }
  .filter-label { font-size: 11px; font-weight: 700; color: var(--grey-m); text-transform: uppercase; letter-spacing: .04em; }
  .empty { text-align: center; padding: 3rem; color: var(--grey-m); font-size: 14px; }
  .bar-bg { height: 6px; background: var(--grey-l); border-radius: 3px; overflow: hidden; min-width: 80px; }
  .bar-fill { height: 100%; background: var(--orange); border-radius: 3px; }
  .action-btn { font-family: inherit; font-size: 11px; font-weight: 700; padding: 5px 10px; border-radius: 6px; cursor: pointer; border: none; transition: all .15s; white-space: nowrap; }
  .edit-btn { background: var(--orange-l); color: var(--orange-d); }
  .edit-btn:hover { background: var(--orange-d); color: #fff; }
  .delete-btn { background: var(--red-l); color: var(--red); margin-left: 4px; }
  .delete-btn:hover { background: var(--red); color: #fff; }
  .action-cell { display: flex; gap: 4px; }
  .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,.5); z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 1rem; }
  .modal-overlay.hidden { display: none; }
  .modal { background: #fff; border-radius: 16px; padding: 2rem; width: 100%; max-width: 520px; max-height: 90vh; overflow-y: auto; }
  .modal h2 { font-size: 18px; font-weight: 700; margin-bottom: 1.5rem; }
  .modal-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .modal-full { grid-column: 1/-1; }
  .modal-actions { display: flex; gap: .75rem; margin-top: 1.5rem; }
  .modal-actions .btn { margin-top: 0; flex: 1; padding: 12px 0; font-size: 14px; }
  .cancel-btn { background: var(--grey-l); color: var(--grey); }
  .cancel-btn:hover { background: var(--border); color: var(--grey); }
  .admin-banner { background: #1a1a2e; border-radius: 12px; padding: 1rem 1.25rem; margin-bottom: 1.5rem; display: flex; align-items: center; gap: .75rem; }
  .admin-banner span { color: #fff; font-size: 13px; font-weight: 600; flex: 1; }
  .logout-btn { font-family: inherit; font-size: 12px; font-weight: 700; padding: 6px 14px; background: rgba(255,255,255,.15); color: #fff; border: none; border-radius: 8px; cursor: pointer; }
  .logout-btn:hover { background: rgba(255,255,255,.25); }
  .login-modal { background: #fff; border-radius: 16px; padding: 2rem; width: 100%; max-width: 380px; }
  .login-modal h2 { font-size: 18px; font-weight: 700; margin-bottom: .5rem; }
  .login-modal p { font-size: 13px; color: var(--grey-m); margin-bottom: 1.5rem; }
  .login-error { color: var(--red); font-size: 13px; margin-top: .75rem; display: none; }
  .toast { position: fixed; bottom: 2rem; left: 50%; transform: translateX(-50%) translateY(20px); background: var(--green); color: #fff; padding: 12px 24px; border-radius: 40px; font-size: 14px; font-weight: 600; opacity: 0; transition: all .3s; pointer-events: none; z-index: 9999; white-space: nowrap; }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
  .toast.error { background: var(--red); }
  .tab { display: none; }
  .tab.active { display: block; }
  @media (max-width: 600px) {
    .grid2, .modal-grid { grid-template-columns: 1fr; }
    .full, .modal-full { grid-column: 1; }
    .stat-grid { grid-template-columns: 1fr 1fr; }
    .banner-divider { display: none; }
  }
</style>
</head>
<body>
<div class="app">
  <div class="banner">
    <div class="banner-left">
      <img class="banner-logo" src="data:image/png;base64,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" alt="Kennewell" />
      <div class="banner-divider"></div>
      <div class="banner-text">
        <h1>R&D Hours Tracker</h1>
        <p>All entries are shared — every team member sees the same data in real time</p>
      </div>
    </div>
    <button class="admin-badge" id="admin-toggle-btn" onclick="toggleAdminLogin()">🔐 Admin</button>
  </div>

  <div class="admin-banner" id="admin-banner" style="display:none">
    <span>🛡️ Admin mode active — you can edit and delete entries in the All Entries tab</span>
    <button class="logout-btn" onclick="logoutAdmin()">Log out</button>
  </div>

  <div class="nav">
    <button class="nav-btn active" onclick="switchTab('log', this)">Log Hours</button>
    <button class="nav-btn" onclick="switchTab('dashboard', this)">Dashboard</button>
    <button class="nav-btn" onclick="switchTab('data', this)">All Entries</button>
  </div>

  <!-- LOG TAB -->
  <div id="tab-log" class="tab active">
    <div class="reminder-box">
      <span class="reminder-icon">📅</span>
      <span>Please submit your R&D hours at the end of every week in which R&D work was conducted.</span>
    </div>
    <div class="card">
      <div class="grid2">
        <div>
          <label class="field-label">Your Name</label>
          <select id="log-name">
            <option value="">— Select your name —</option>
            <option>Mitchell</option><option>Brett</option><option>Darren</option>
            <option>Orlando</option><option>Phong</option><option>Rory</option>
            <option>Klose</option><option>Adam</option><option>Jamie</option><option>Ben</option>
            <option>Lachlan</option><option>Indiyah</option><option>Tahlia</option><option>Damien</option>
          </select>
        </div>
        <div>
          <label class="field-label">Week Ending Date</label>
          <input type="date" id="log-date" />
        </div>
        <div>
          <label class="field-label">R&D Hours This Week</label>
          <input type="number" id="log-hours" min="0" max="168" step="0.5" placeholder="e.g. 7.5" />
        </div>
        <div>
          <label class="field-label">Financial Year (auto)</label>
          <input type="text" id="log-fy" readonly />
        </div>
        <div class="full">
          <label class="field-label">Description of R&D Activity</label>
          <textarea id="log-desc" placeholder="Brief description of the R&D work performed this week…"></textarea>
        </div>
      </div>
      <button class="btn" id="submit-btn" onclick="submitEntry()">✔  Submit Entry</button>
      <p class="btn-note">Entry is saved instantly and visible to everyone on the team</p>
    </div>
  </div>

  <!-- DASHBOARD TAB -->
  <div id="tab-dashboard" class="tab">
    <div class="filter-bar">
      <span class="filter-label">Filter:</span>
      <select id="d-month" onchange="renderDashboard()"><option value="">All Months</option></select>
      <select id="d-fy" onchange="renderDashboard()"><option value="">All Financial Years</option></select>
      <select id="d-person" onchange="renderDashboard()"><option value="">All People</option></select>
    </div>
    <div class="stat-grid">
      <div class="stat-card"><div class="stat-val" id="d-total">0.0</div><div class="stat-lbl">Total R&D Hours</div></div>
      <div class="stat-card"><div class="stat-val" id="d-count">0</div><div class="stat-lbl">Entries Logged</div></div>
      <div class="stat-card"><div class="stat-val" id="d-avg">0.0</div><div class="stat-lbl">Avg Hours / Entry</div></div>
    </div>
    <p class="section-title">Hours by Person</p>
    <div class="table-wrap"><table id="person-table"><thead><tr>
      <th>Person</th><th>Total Hours</th><th>Entries</th><th>Last Entry</th><th>Share of Total</th>
    </tr></thead><tbody></tbody></table></div>
    <p class="section-title">Monthly Breakdown</p>
    <div class="table-wrap"><table id="month-table"><thead><tr>
      <th>Month</th><th>Financial Year</th><th>Total Hours</th><th>Entries</th>
    </tr></thead><tbody></tbody></table></div>
  </div>

  <!-- ALL ENTRIES TAB -->
  <div id="tab-data" class="tab">
    <div class="filter-bar">
      <span class="filter-label">Filter:</span>
      <select id="e-month" onchange="renderEntries()"><option value="">All Months</option></select>
      <select id="e-fy" onchange="renderEntries()"><option value="">All Financial Years</option></select>
      <select id="e-person" onchange="renderEntries()"><option value="">All People</option></select>
    </div>
    <div class="table-wrap"><table id="entries-table"><thead><tr>
      <th>Person</th><th>Week Ending</th><th>Hours</th><th>Month</th><th>Fin. Year</th><th>Description</th>
      <th id="actions-th" style="display:none">Actions</th>
    </tr></thead><tbody></tbody></table></div>
  </div>
</div>

<!-- Admin Login Modal -->
<div class="modal-overlay hidden" id="login-modal-overlay">
  <div class="login-modal">
    <h2>🔐 Admin Login</h2>
    <p>Enter the admin password to enable edit and delete access.</p>
    <label class="field-label">Password</label>
    <input type="password" id="admin-password-input" placeholder="Enter admin password" onkeydown="if(event.key==='Enter')attemptAdminLogin()" />
    <p class="login-error" id="login-error">Incorrect password. Try again.</p>
    <div style="display:flex;gap:.75rem;margin-top:1.25rem">
      <button class="btn cancel-btn" style="margin-top:0;flex:1;padding:12px 0;font-size:14px" onclick="closeLoginModal()">Cancel</button>
      <button class="btn" style="margin-top:0;flex:1;padding:12px 0;font-size:14px" onclick="attemptAdminLogin()">Login</button>
    </div>
  </div>
</div>

<!-- Edit Entry Modal -->
<div class="modal-overlay hidden" id="edit-modal-overlay">
  <div class="modal">
    <h2>✏️ Edit Entry</h2>
    <input type="hidden" id="edit-id" />
    <div class="modal-grid">
      <div>
        <label class="field-label">Name</label>
        <select id="edit-name">
          <option>Mitchell</option><option>Brett</option><option>Darren</option>
          <option>Orlando</option><option>Phong</option><option>Rory</option>
          <option>Klose</option><option>Adam</option><option>Jamie</option><option>Ben</option>
          <option>Lachlan</option><option>Indiyah</option><option>Tahlia</option><option>Damien</option>
        </select>
      </div>
      <div>
        <label class="field-label">Week Ending Date</label>
        <input type="date" id="edit-date" />
      </div>
      <div>
        <label class="field-label">Hours</label>
        <input type="number" id="edit-hours" min="0" max="168" step="0.5" />
      </div>
      <div>
        <label class="field-label">Financial Year (auto)</label>
        <input type="text" id="edit-fy" readonly />
      </div>
      <div class="modal-full">
        <label class="field-label">Description</label>
        <textarea id="edit-desc"></textarea>
      </div>
    </div>
    <div class="modal-actions">
      <button class="btn cancel-btn" onclick="closeEditModal()">Cancel</button>
      <button class="btn" id="save-edit-btn" onclick="saveEdit()">Save Changes</button>
    </div>
  </div>
</div>

<!-- Confirm Delete Modal -->
<div class="modal-overlay hidden" id="delete-modal-overlay">
  <div class="modal" style="max-width:400px">
    <h2>🗑️ Delete Entry?</h2>
    <p style="color:var(--grey-m);font-size:14px;margin-bottom:1.5rem">This will permanently delete the entry. This cannot be undone.</p>
    <div id="delete-preview" style="background:var(--grey-l);border-radius:10px;padding:1rem;font-size:13px;margin-bottom:1.5rem;line-height:1.8"></div>
    <div class="modal-actions">
      <button class="btn cancel-btn" onclick="closeDeleteModal()">Cancel</button>
      <button class="btn" style="background:var(--red)" id="confirm-delete-btn" onclick="confirmDelete()">Yes, Delete</button>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
const SUPABASE_URL = "https://hyxdrdqgkyittgkkqzqp.supabase.co";
const SUPABASE_KEY = "sb_publishable_lql8v4QDV_R_G2lXE07g3w_fvDPEzcL";
const ADMIN_PASSWORD = "cncadmin";
const TEAM = ["Mitchell","Brett","Darren","Orlando","Phong","Rory","Klose","Adam","Jamie","Ben","Lachlan","Indiyah","Tahlia","Damien"];

const { createClient } = supabase;
const db = createClient(SUPABASE_URL, SUPABASE_KEY);

let allEntries = [];
let isAdmin = false;
let pendingDeleteId = null;

function getFY(dateStr) {
  if (!dateStr) return "";
  const d = new Date(dateStr), m = d.getMonth() + 1, y = d.getFullYear();
  return m >= 7 ? `${y}-${y+1}` : `${y-1}-${y}`;
}
function getMonth(dateStr) {
  if (!dateStr) return "";
  return new Date(dateStr).toLocaleDateString("en-AU", { month: "short", year: "numeric" });
}
function fmtDate(dateStr) {
  if (!dateStr) return "";
  const [y, m, d] = dateStr.split("-");
  return `${d}/${m}/${y}`;
}
function lastFriday() {
  const t = new Date(), dow = t.getDay();
  const diff = dow >= 5 ? dow - 5 : dow + 2;
  const f = new Date(t);
  f.setDate(t.getDate() - diff);
  return f.toISOString().split("T")[0];
}
function showToast(msg, type = "success") {
  const el = document.getElementById("toast");
  el.textContent = msg;
  el.className = "toast show" + (type === "error" ? " error" : "");
  setTimeout(() => el.className = "toast", 2800);
}
function populateFilters() {
  const months = [...new Set(allEntries.map(e => e.month))].sort((a,b) => new Date("1 "+a) - new Date("1 "+b));
  const fys = [...new Set(allEntries.map(e => e.fy))].sort();
  ["d-month","e-month"].forEach(id => {
    const sel = document.getElementById(id), val = sel.value;
    sel.innerHTML = '<option value="">All Months</option>' + months.map(m => `<option${m===val?' selected':''}}>${m}</option>`).join('');
  });
  ["d-fy","e-fy"].forEach(id => {
    const sel = document.getElementById(id), val = sel.value;
    sel.innerHTML = '<option value="">All Financial Years</option>' + fys.map(f => `<option${f===val?' selected':''}}>${f}</option>`).join('');
  });
  ["d-person","e-person"].forEach(id => {
    const sel = document.getElementById(id), val = sel.value;
    sel.innerHTML = '<option value="">All People</option>' + TEAM.map(t => `<option${t===val?' selected':''}}>${t}</option>`).join('');
  });
}
function filtered(mId, fyId, pId) {
  const m = document.getElementById(mId).value;
  const fy = document.getElementById(fyId).value;
  const p = document.getElementById(pId).value;
  return allEntries.filter(e => (!m || e.month === m) && (!fy || e.fy === fy) && (!p || e.name === p));
}
function switchTab(name, btn) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
  document.getElementById('tab-'+name).classList.add('active');
  btn.classList.add('active');
  if (name === 'dashboard') loadEntries().then(renderDashboard);
  if (name === 'data') loadEntries().then(renderEntries);
}
async function loadEntries() {
  const { data, error } = await db.from("rd_entries").select("*").order("date", { ascending: false });
  if (error) { showToast("Error loading data", "error"); return; }
  allEntries = (data || []).map(e => ({ ...e, month: e.month || getMonth(e.date), fy: e.fy || getFY(e.date) }));
  populateFilters();
}
async function submitEntry() {
  const name = document.getElementById("log-name").value;
  const date = document.getElementById("log-date").value;
  const hours = parseFloat(document.getElementById("log-hours").value);
  const desc = document.getElementById("log-desc").value.trim();
  if (!name) return showToast("Please select your name.", "error");
  if (!date) return showToast("Please select a week ending date.", "error");
  if (!hours || hours <= 0) return showToast("Please enter valid hours.", "error");
  if (!desc) return showToast("Please add a description.", "error");
  const btn = document.getElementById("submit-btn");
  btn.disabled = true; btn.textContent = "Saving…";
  const { error } = await db.from("rd_entries").insert({ name, date, hours, description: desc, month: getMonth(date), fy: getFY(date) });
  btn.disabled = false; btn.textContent = "✔  Submit Entry";
  if (error) return showToast("Error saving entry: " + error.message, "error");
  showToast(`Saved! ${name} — ${fmtDate(date)} — ${hours} hrs`);
  document.getElementById("log-name").value = "";
  document.getElementById("log-hours").value = "";
  document.getElementById("log-desc").value = "";
  document.getElementById("log-date").value = lastFriday();
  document.getElementById("log-fy").value = getFY(lastFriday());
}
function toggleAdminLogin() {
  if (isAdmin) { logoutAdmin(); return; }
  document.getElementById("login-modal-overlay").classList.remove("hidden");
  document.getElementById("admin-password-input").value = "";
  document.getElementById("login-error").style.display = "none";
  setTimeout(() => document.getElementById("admin-password-input").focus(), 100);
}
function closeLoginModal() { document.getElementById("login-modal-overlay").classList.add("hidden"); }
function attemptAdminLogin() {
  const val = document.getElementById("admin-password-input").value;
  if (val === ADMIN_PASSWORD) {
    isAdmin = true;
    closeLoginModal();
    document.getElementById("admin-banner").style.display = "flex";
    document.getElementById("admin-toggle-btn").textContent = "🛡️ Admin ON";
    document.getElementById("actions-th").style.display = "";
    showToast("Admin mode enabled");
    renderEntries();
  } else {
    document.getElementById("login-error").style.display = "block";
    document.getElementById("admin-password-input").value = "";
    document.getElementById("admin-password-input").focus();
  }
}
function logoutAdmin() {
  isAdmin = false;
  document.getElementById("admin-banner").style.display = "none";
  document.getElementById("admin-toggle-btn").textContent = "🔐 Admin";
  document.getElementById("actions-th").style.display = "none";
  showToast("Logged out of admin mode");
  renderEntries();
}
function openEditModal(id) {
  const e = allEntries.find(x => x.id === id);
  if (!e) return;
  document.getElementById("edit-id").value = e.id;
  document.getElementById("edit-name").value = e.name;
  document.getElementById("edit-date").value = e.date;
  document.getElementById("edit-hours").value = e.hours;
  document.getElementById("edit-desc").value = e.description;
  document.getElementById("edit-fy").value = e.fy;
  document.getElementById("edit-modal-overlay").classList.remove("hidden");
}
function closeEditModal() { document.getElementById("edit-modal-overlay").classList.add("hidden"); }
document.getElementById("edit-date").addEventListener("change", e => {
  document.getElementById("edit-fy").value = getFY(e.target.value);
});
async function saveEdit() {
  const id = parseInt(document.getElementById("edit-id").value);
  const name = document.getElementById("edit-name").value;
  const date = document.getElementById("edit-date").value;
  const hours = parseFloat(document.getElementById("edit-hours").value);
  const description = document.getElementById("edit-desc").value.trim();
  if (!name || !date || !hours || !description) return showToast("All fields are required.", "error");
  const btn = document.getElementById("save-edit-btn");
  btn.disabled = true; btn.textContent = "Saving…";
  const { error } = await db.from("rd_entries").update({ name, date, hours, description, month: getMonth(date), fy: getFY(date) }).eq("id", id);
  btn.disabled = false; btn.textContent = "Save Changes";
  if (error) return showToast("Error saving: " + error.message, "error");
  closeEditModal();
  showToast("Entry updated successfully!");
  await loadEntries();
  renderEntries();
}
function openDeleteModal(id) {
  const e = allEntries.find(x => x.id === id);
  if (!e) return;
  pendingDeleteId = id;
  document.getElementById("delete-preview").innerHTML =
    `<strong>${e.name}</strong> &mdash; ${fmtDate(e.date)}<br>${Number(e.hours).toFixed(1)} hrs &mdash; ${e.month} (${e.fy})<br><span style="color:var(--grey-m)">${e.description}</span>`;
  document.getElementById("delete-modal-overlay").classList.remove("hidden");
}
function closeDeleteModal() { document.getElementById("delete-modal-overlay").classList.add("hidden"); pendingDeleteId = null; }
async function confirmDelete() {
  if (!pendingDeleteId) return;
  const btn = document.getElementById("confirm-delete-btn");
  btn.disabled = true; btn.textContent = "Deleting…";
  const { error } = await db.from("rd_entries").delete().eq("id", pendingDeleteId);
  btn.disabled = false; btn.textContent = "Yes, Delete";
  if (error) return showToast("Error deleting: " + error.message, "error");
  closeDeleteModal();
  showToast("Entry deleted.");
  await loadEntries();
  renderEntries();
}
function renderDashboard() {
  const data = filtered("d-month","d-fy","d-person");
  const totalHrs = data.reduce((s,e) => s + Number(e.hours), 0);
  const avg = data.length ? totalHrs / data.length : 0;
  document.getElementById("d-total").textContent = totalHrs.toFixed(1);
  document.getElementById("d-count").textContent = data.length;
  document.getElementById("d-avg").textContent = avg.toFixed(1);
  const personRows = TEAM.map(p => {
    const pe = data.filter(e => e.name === p);
    const hrs = pe.reduce((s,e) => s + Number(e.hours), 0);
    const last = pe.length ? fmtDate([...pe].sort((a,b) => b.date.localeCompare(a.date))[0].date) : "—";
    return { name: p, hrs, cnt: pe.length, last };
  });
  const maxHrs = Math.max(...personRows.map(r => r.hrs), 1);
  document.querySelector("#person-table tbody").innerHTML = personRows.map(r => `
    <tr>
      <td class="bold">${r.name}</td>
      <td class="mono bold accent">${r.hrs.toFixed(1)}</td>
      <td>${r.cnt}</td>
      <td class="muted">${r.last}</td>
      <td><div class="bar-bg"><div class="bar-fill" style="width:${r.hrs/maxHrs*100}%"></div></div></td>
    </tr>`).join('');
  const monthMap = {};
  data.forEach(e => {
    const k = e.month+"||"+e.fy;
    if (!monthMap[k]) monthMap[k] = { month: e.month, fy: e.fy, hrs: 0, cnt: 0 };
    monthMap[k].hrs += Number(e.hours); monthMap[k].cnt++;
  });
  const monthRows = Object.values(monthMap).sort((a,b) => new Date("1 "+a.month) - new Date("1 "+b.month));
  document.querySelector("#month-table tbody").innerHTML = monthRows.length
    ? monthRows.map(r => `
      <tr>
        <td><span class="tag">${r.month}</span></td>
        <td class="muted">${r.fy}</td>
        <td class="mono bold accent">${r.hrs.toFixed(1)}</td>
        <td>${r.cnt}</td>
      </tr>`).join('')
    : '<tr><td colspan="4" class="empty">No data for selected filters</td></tr>';
}
function renderEntries() {
  const rows = filtered("e-month","e-fy","e-person").sort((a,b) => b.date.localeCompare(a.date));
  document.querySelector("#entries-table tbody").innerHTML = rows.length
    ? rows.map(e => `
      <tr>
        <td class="bold">${e.name}</td>
        <td class="mono muted">${fmtDate(e.date)}</td>
        <td class="mono bold accent">${Number(e.hours).toFixed(1)}</td>
        <td><span class="tag">${e.month}</span></td>
        <td class="muted">${e.fy}</td>
        <td class="muted" style="max-width:220px">${e.description}</td>
        ${isAdmin
          ? `<td><div class="action-cell">
               <button class="action-btn edit-btn" onclick="openEditModal(${e.id})">✏️ Edit</button>
               <button class="action-btn delete-btn" onclick="openDeleteModal(${e.id})">🗑️ Delete</button>
             </div></td>`
          : `<td style="display:none"></td>`}
      </tr>`).join('')
    : `<tr><td colspan="${isAdmin ? 7 : 6}" class="empty">No entries yet — be the first to log hours!</td></tr>`;
}
document.getElementById("log-date").value = lastFriday();
document.getElementById("log-fy").value = getFY(lastFriday());
document.getElementById("log-date").addEventListener("change", e => {
  document.getElementById("log-fy").value = getFY(e.target.value);
});
document.getElementById("login-modal-overlay").addEventListener("click", function(e) { if (e.target === this) closeLoginModal(); });
document.getElementById("edit-modal-overlay").addEventListener("click", function(e) { if (e.target === this) closeEditModal(); });
document.getElementById("delete-modal-overlay").addEventListener("click", function(e) { if (e.target === this) closeDeleteModal(); });
</script>
</body>
</html>
