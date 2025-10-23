<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Penentuan Jadwal Kerja Kelompok</title>
  <style>
    :root{
      --bg:#f6f7fb; --card:#ffffff; --muted:#6b7280; --accent:#2563eb;
      --success:#16a34a; --danger:#dc2626; --glass: rgba(37,99,235,0.06);
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{margin:0;background:var(--bg);color:#111827;padding:28px}
    .container{max-width:1100px;margin:0 auto;display:grid;grid-template-columns:320px 1fr;gap:20px}
    header{grid-column:1/-1;margin-bottom:6px}
    h1{margin:0;font-size:20px}
    p.lead{margin:6px 0 0;color:var(--muted);font-size:13px}

    .card{background:var(--card);padding:16px;border-radius:12px;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
    .small{font-size:13px;color:var(--muted)}
    .muted{color:var(--muted)}
    .btn{display:inline-block;padding:8px 10px;border-radius:8px;border:0;cursor:pointer;font-weight:600}
    .btn-primary{background:var(--accent);color:white}
    .btn-ghost{background:transparent;border:1px solid #e6e9ef}
    .btn-danger{background:var(--danger);color:white}
    input[type="text"], textarea{width:100%;padding:8px;border-radius:8px;border:1px solid #e6e9ef}
    .members-list{margin-top:12px;max-height:380px;overflow:auto}
    .member-item{display:flex;justify-content:space-between;align-items:center;padding:8px;border-radius:8px}
    .member-item:hover{background:#fbfdff}
    .selected{background:var(--glass);border:1px solid rgba(37,99,235,0.15)}

    /* slots grid */
    .slots-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-top:12px}
    .slot{padding:10px;border-radius:8px;border:1px solid #eef2f7;cursor:pointer;text-align:left;font-size:13px}
    .slot.checked{background:#e8f0ff;border-color:rgba(37,99,235,0.3)}

    /* stats */
    .stats-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:12px}
    .list-vertical{max-height:220px;overflow:auto;border-radius:8px;border:1px solid #eef2f7;padding:8px}
    .row{display:flex;justify-content:space-between;padding:6px 4px;border-bottom:1px dashed #f3f4f6}
    footer{grid-column:1/-1;margin-top:12px;color:var(--muted);font-size:13px}
    @media (max-width:880px){
      .container{grid-template-columns:1fr; padding:12px}
      header{margin-bottom:10px}
    }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>Penentuan Jadwal Kerja Kelompok</h1>
      <p class="lead">Semua anggota pilih waktu luang → sistem pilih waktu terbanyak → tambahkan lokasi pertemuan.</p>
    </header>

    <!-- LEFT PANEL: anggota + location -->
    <div class="card">
      <div style="display:flex;gap:8px">
        <input id="nameInput" type="text" placeholder="Nama anggota (mis: Aghni)" />
        <button id="addBtn" class="btn btn-primary">Tambah</button>
      </div>

      <div class="members-list" id="membersList" style="margin-top:12px">
        <div class="small muted">Belum ada anggota — tambahkan satu per satu.</div>
      </div>

      <div style="margin-top:12px">
        <label class="small muted">Lokasi pertemuan</label>
        <input id="locationInput" type="text" placeholder="Contoh: Lab RPL / Perpustakaan" style="margin-top:6px" />
      </div>

      <div style="margin-top:12px;display:flex;gap:8px">
        <button id="exportCsv" class="btn">Export CSV</button>
        <button id="copyData" class="btn btn-ghost">Copy data</button>
        <button id="resetAll" class="btn" title="Hapus semua data">Reset</button>
      </div>

      <div style="margin-top:12px;font-size:13px;color:var(--muted)">
        Tips: Minta setiap anggota memilih beberapa slot. Sistem akan menampilkan slot yang paling banyak dipilih.
      </div>
    </div>

    <!-- RIGHT PANEL: pemilihan slot + hasil -->
    <div class="card">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div>
          <div class="small muted">Pilih anggota lalu isi waktu luang mereka</div>
          <h2 style="margin:6px 0 0">Waktu Luang</h2>
        </div>
        <div id="selectedMemberName" class="small muted">Belum memilih anggota</div>
      </div>

      <!-- slots selection -->
      <div id="slotsSection" style="margin-top:12px">
        <div style="display:flex;gap:8px;flex-wrap:wrap">
          <div class="small muted">Hari</div>
        </div>
        <div id="slotsGrid" class="slots-grid" aria-live="polite"></div>
      </div>

      <!-- stats / results -->
      <div class="stats-grid">
        <div>
          <h3 style="margin:10px 0 6px">Slot Terbanyak</h3>
          <div id="topSlots" class="list-vertical small muted">Belum ada pilihan.</div>
        </div>
        <div>
          <h3 style="margin:10px 0 6px">Semua Slot (urut dari banyak)</h3>
          <div id="allSlotsList" class="list-vertical small muted"></div>
        </div>
      </div>

      <div style="margin-top:12px">
        <h3 style="margin:6px 0">Rekomendasi</h3>
        <div id="recommendation" style="padding:10px;border-radius:8px;border:1px solid #e6f4ea;background:#f8fffa">
          Belum ada rekomendasi — minta anggota memilih minimal 1 slot.
        </div>
      </div>
    </div>

    <footer>Versi sederhana — dibuat supaya langsung jalan tanpa instalasi. Simpan sebagai file .html lalu buka di browser.</footer>
  </div>

<script>
/*
  Penjelasan singkat:
  - Semua data disimpan di memory JS (array members).
  - Members: { name: string, avail: Set<string> }
  - ALL_SLOTS dibentuk dari DAYS x SLOTS (Pagi/Siang/Sore/Malam).
  - Fungsi utama:
      addMember, removeMember, selectMember, toggleSlotForSelectedMember
      computeCounts -> slotCounts object
      render updates UI setiap kali data berubah
*/

const DAYS = ["Senin","Selasa","Rabu","Kamis","Jumat","Sabtu","Minggu"];
const TIMES = ["Pagi","Siang","Sore","Malam"];
const ALL_SLOTS = DAYS.flatMap(d => TIMES.map(t => `${d} - ${t}`));

let members = []; // {name, avail:Set()}
let selectedMemberIndex = null;

const el = {
  nameInput: document.getElementById('nameInput'),
  addBtn: document.getElementById('addBtn'),
  membersList: document.getElementById('membersList'),
  locationInput: document.getElementById('locationInput'),
  exportCsv: document.getElementById('exportCsv'),
  copyData: document.getElementById('copyData'),
  resetAll: document.getElementById('resetAll'),
  slotsGrid: document.getElementById('slotsGrid'),
  selectedMemberName: document.getElementById('selectedMemberName'),
  topSlots: document.getElementById('topSlots'),
  allSlotsList: document.getElementById('allSlotsList'),
  recommendation: document.getElementById('recommendation'),
};

function init(){
  renderSlotsGrid();
  attachEvents();
  render();
}

function attachEvents(){
  el.addBtn.addEventListener('click', ()=> {
    const name = el.nameInput.value.trim();
    if(!name) return alert('Isi nama anggota.');
    if(members.some(m=>m.name.toLowerCase()===name.toLowerCase())) return alert('Nama sudah ada. Gunakan nama lain atau tambahkan nomor.');
    members.push({name, avail: new Set()});
    el.nameInput.value = '';
    selectedMemberIndex = members.length - 1;
    render();
  });

  el.exportCsv.addEventListener('click', ()=> {
    if(members.length===0) return alert('Belum ada data anggota.');
    const header = ['Nama', ...ALL_SLOTS].join(',');
    const rows = members.map(m => {
      const row = [m.name, ...ALL_SLOTS.map(s => m.avail.has(s) ? '1' : '0')];
      return row.join(',');
    });
    const csv = [header, ...rows].join('\\n');
    downloadFile('jadwal_kelompok.csv', csv, 'text/csv;charset=utf-8;');
  });

  el.copyData.addEventListener('click', async ()=> {
    const payload = {members: members.map(m=>({name:m.name, avail:[...m.avail]})), location: el.locationInput.value};
    try {
      await navigator.clipboard.writeText(JSON.stringify(payload, null, 2));
      alert('Data disalin ke clipboard (JSON).');
    } catch(e){
      alert('Gagal copy. Browser tidak mengizinkan clipboard.');
    }
  });

  el.resetAll.addEventListener('click', ()=> {
    if(!confirm('Reset semua data?')) return;
    members = [];
    selectedMemberIndex = null;
    el.locationInput.value = '';
    render();
  });
}

function downloadFile(filename, content, type){
  const blob = new Blob([content], {type});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename; document.body.appendChild(a);
  a.click(); a.remove(); URL.revokeObjectURL(url);
}

/* --- render UI pieces --- */

function render(){
  renderMembersList();
  renderSelectedMember();
  renderCountsAndRecommendation();
}

function renderMembersList(){
  const container = el.membersList;
  container.innerHTML = '';
  if(members.length === 0){
    const d = document.createElement('div');
    d.className = 'small muted';
    d.textContent = 'Belum ada anggota — tambahkan satu per satu.';
    container.appendChild(d);
    return;
  }
  members.forEach((m, idx) => {
    const item = document.createElement('div');
    item.className = 'member-item';
    if(idx === selectedMemberIndex) item.className += ' selected';
    const left = document.createElement('div');
    left.style.cursor = 'pointer';
    left.onclick = ()=> { selectedMemberIndex = idx; render(); };
    left.innerHTML = `<div style="font-weight:600">${m.name}</div><div class="small muted">${m.avail.size} slot dipilih</div>`;

    const right = document.createElement('div');
    right.style.display='flex';
    right.style.gap='8px';
    const btnDel = document.createElement('button');
    btnDel.className = 'btn';
    btnDel.style.fontSize='13px';
    btnDel.textContent = 'Hapus';
    btnDel.onclick = ()=> {
      if(!confirm('Hapus anggota "'+m.name+'"?')) return;
      members.splice(idx,1);
      if(selectedMemberIndex === idx) selectedMemberIndex = null;
      else if(selectedMemberIndex > idx) selectedMemberIndex--;
      render();
    };
    right.appendChild(btnDel);
    item.appendChild(left);
    item.appendChild(right);
    container.appendChild(item);
  });
}

function renderSlotsGrid(){
  // build entire slots grid (buttons). We'll assign data-slot attr.
  el.slotsGrid.innerHTML = '';
  ALL_SLOTS.forEach(slot => {
    const b = document.createElement('div');
    b.className = 'slot';
    b.setAttribute('data-slot', slot);
    b.tabIndex = 0;
    b.innerHTML = `<div style="font-weight:600;font-size:13px">${slot}</div>`;
    b.onclick = ()=> {
      toggleSlotForSelectedMember(slot);
    };
    b.onkeypress = (e)=> { if(e.key==='Enter') toggleSlotForSelectedMember(slot); };
    el.slotsGrid.appendChild(b);
  });
}

function renderSelectedMember(){
  if(selectedMemberIndex === null || !members[selectedMemberIndex]){
    el.selectedMemberName.textContent = 'Belum memilih anggota';
    // clear checkboxes
    document.querySelectorAll('.slot').forEach(s=> s.classList.remove('checked'));
    return;
  }
  const m = members[selectedMemberIndex];
  el.selectedMemberName.textContent = m.name;
  // mark checked slots
  document.querySelectorAll('.slot').forEach(elSlot => {
    const s = elSlot.getAttribute('data-slot');
    if(m.avail.has(s)) elSlot.classList.add('checked');
    else elSlot.classList.remove('checked');
  });
}

/* toggle slot for the currently selected member */
function toggleSlotForSelectedMember(slot){
  if(selectedMemberIndex === null || !members[selectedMemberIndex]){
    alert('Pilih anggota dulu di panel kiri.');
    return;
  }
  const m = members[selectedMemberIndex];
  if(m.avail.has(slot)) m.avail.delete(slot);
  else m.avail.add(slot);
  renderSelectedMember();
  renderCountsAndRecommendation();
}

/* compute counts for each slot */
function computeCounts(){
  const counts = {};
  ALL_SLOTS.forEach(s => counts[s] = 0);
  members.forEach(m => {
    m.avail.forEach(s => {
      if(counts[s] !== undefined) counts[s] += 1;
    });
  });
  return counts;
}

function renderCountsAndRecommendation(){
  const counts = computeCounts();
  // all slots list sorted
  const entries = Object.entries(counts).sort((a,b) => b[1]-a[1] || a[0].localeCompare(b[0]));
  el.allSlotsList.innerHTML = '';
  entries.forEach(([s,c])=>{
    const r = document.createElement('div');
    r.className = 'row';
    r.innerHTML = `<div style="font-size:13px">${s}</div><div class="muted">${c}</div>`;
    el.allSlotsList.appendChild(r);
  });

  // determine top slots
  const max = entries.length ? entries[0][1] : 0;
  const top = entries.filter(([s,c]) => c === max && c > 0);
  el.topSlots.innerHTML = '';
  if(top.length === 0){
    el.topSlots.textContent = 'Belum ada pilihan.';
    el.recommendation.innerHTML = 'Belum ada rekomendasi — minta anggota memilih minimal 1 slot.';
    return;
  }
  top.forEach(([s,c])=>{
    const d = document.createElement('div');
    d.className = 'row';
    d.innerHTML = `<div style="font-weight:600">${s}</div><div class="muted">${c} anggota</div>`;
    el.topSlots.appendChild(d);
  });

  // recommendation block
  const loc = el.locationInput.value.trim() || '(lokasi belum diisi)';
  el.recommendation.innerHTML = `
    <div style="font-weight:600;margin-bottom:6px">Waktu paling banyak dipilih:</div>
    <div style="display:flex;flex-direction:column;gap:6px">
      ${top.map(([s,c]) => `<div style="display:flex;justify-content:space-between"><div>${s}</div><div class="muted">${c}/${members.length} anggota</div></div>`).join('')}
    </div>
    <div style="margin-top:8px;color:var(--muted)">Lokasi: <strong>${loc}</strong></div>
  `;
}

/* initialize */
init();

</script>
