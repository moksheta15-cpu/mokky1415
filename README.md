<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Tapback – Local Registration (No Firebase)</title>
  <meta name="description" content="Simple local-only registration form that generates a QR code. No backend, no Firebase – great for quick testing." />
  <style>
    :root{
      --bg:#0b1020;/* deep navy */
      --panel:#111832;/* card */
      --ink:#e7ecff;/* text */
      --muted:#9fb0ff;/* secondary text */
      --accent:#66f;/* brand */
      --good:#31d0aa;
      --warn:#ffd166;
      --err:#ff6b6b;
      --radius:18px;
      --shadow:0 6px 24px rgba(0,0,0,.35);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font:16px/1.5 system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, "Apple Color Emoji","Segoe UI Emoji";
      color:var(--ink);
      background:radial-gradient(1200px 700px at 10% -10%, #1b2550 0%, transparent 60%),
                 radial-gradient(1000px 700px at 100% 0%, #18214a 0%, transparent 60%),
                 var(--bg);
    }
    .wrap{
      max-width:1000px; margin:40px auto; padding:0 16px;
    }
    header{
      display:flex; align-items:center; justify-content:space-between; gap:12px; margin-bottom:20px;
    }
    .brand{display:flex; gap:12px; align-items:center}
    .logo{width:42px; height:42px; border-radius:12px; background:linear-gradient(135deg,#5b7cff,#7b66ff); box-shadow:var(--shadow); display:grid; place-items:center; font-weight:700}
    .brand h1{font-size:22px; margin:0}
    .sub{color:var(--muted); font-size:13px}

    .grid{display:grid; grid-template-columns:1.1fr .9fr; gap:22px}
    @media (max-width:880px){ .grid{grid-template-columns:1fr} }

    .card{background:var(--panel); border-radius:var(--radius); box-shadow:var(--shadow);}
    .card .hd{padding:18px 20px; border-bottom:1px solid rgba(255,255,255,.07); display:flex; align-items:center; justify-content:space-between}
    .card .hd h2{font-size:16px; margin:0}
    .card .bd{padding:18px 20px}

    label{display:block; font-weight:600; margin:.2rem 0 .35rem}
    input{
      width:100%; padding:12px 14px; border-radius:12px; border:1px solid rgba(255,255,255,.15);
      background:#0a1026; color:var(--ink); outline:none; box-shadow:inset 0 0 0 999px rgba(255,255,255,0);
    }
    input:focus{border-color:#88a1ff; box-shadow:0 0 0 4px rgba(102,119,255,.15)}

    .row{display:grid; grid-template-columns:1fr; gap:14px}

    .actions{display:flex; gap:10px; flex-wrap:wrap; margin-top:10px}
    button{
      cursor:pointer; border:0; padding:12px 16px; border-radius:12px; font-weight:700; box-shadow:var(--shadow);
      background:linear-gradient(135deg,#6b7bff,#5dc6ff); color:#06102a; letter-spacing:.2px
    }
    button.secondary{background:#1a244b; color:var(--ink); border:1px solid rgba(255,255,255,.12)}
    button.ghost{background:transparent; color:var(--muted); border:1px dashed rgba(255,255,255,.25)}

    .note{font-size:13px; color:var(--muted)}
    .status{margin-top:10px; font-size:14px}

    .qrBox{display:grid; place-items:center; min-height:280px;}
    .qrBox canvas, .qrBox img{border-radius:12px; padding:10px; background:#fff}

    .list{display:flex; flex-direction:column; gap:10px}
    .item{padding:12px; background:#0d1430; border:1px solid rgba(255,255,255,.08); border-radius:12px}

    .footer{margin:26px 0 10px; text-align:center; color:var(--muted); font-size:12px}
    .tag{display:inline-flex; align-items:center; gap:6px; padding:4px 8px; border-radius:999px; background:#0d1430; border:1px solid rgba(255,255,255,.08); font-size:12px}
    .tag b{color:#9fe}
    .hidden{display:none!important}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <div class="logo">TB</div>
        <div>
          <h1>Tapback – Quick Test Console</h1>
          <div class="sub">Local-only prototype: register a user (name + phone) and generate a QR. No Firebase required.</div>
        </div>
      </div>
      <span class="tag" title="Works without a backend"><b>Offline</b> friendly</span>
    </header>

    <div class="grid">
      <!-- Registration Card -->
      <section class="card" aria-labelledby="regTitle">
        <div class="hd"><h2 id="regTitle">Registration</h2></div>
        <div class="bd">
          <form id="regForm" novalidate>
            <div class="row">
              <div>
                <label for="fullName">Full name</label>
                <input id="fullName" name="fullName" type="text" placeholder="e.g., Priya Suresh" required minlength="2" />
              </div>
              <div>
                <label for="phone">Phone</label>
                <input id="phone" name="phone" type="tel" inputmode="tel" placeholder="e.g., +91 98765 43210" required pattern="^[+]?\d[\d\s-]{6,}$" />
                <div class="note">Accepts digits, spaces, dashes, optional leading +. (Basic validation for quick testing)</div>
              </div>
            </div>
            <div class="actions">
              <button type="submit" id="btnGen">Generate QR</button>
              <button type="button" class="secondary" id="btnClear">Reset</button>
              <button type="button" class="ghost" id="btnPrefill">Prefill demo</button>
            </div>
            <div id="formStatus" role="status" class="status" aria-live="polite"></div>
          </form>
        </div>
      </section>

      <!-- QR + History Card -->
      <section class="card" aria-labelledby="qrTitle">
        <div class="hd"><h2 id="qrTitle">QR Code</h2></div>
        <div class="bd">
          <div id="qr" class="qrBox" aria-live="polite"></div>
          <div class="actions">
            <button type="button" id="btnDownload" class="secondary" disabled>Download PNG</button>
            <button type="button" id="btnCopyData" class="ghost" disabled>Copy encoded data</button>
          </div>
        </div>
      </section>
    </div>

    <section class="card" aria-labelledby="histTitle" style="margin-top:22px">
      <div class="hd"><h2 id="histTitle">Recent test registrations (LocalStorage)</h2></div>
      <div class="bd">
        <div id="history" class="list" aria-live="polite"></div>
      </div>
    </section>

    <div class="footer">No network calls. Data is stored in your browser only for demo purposes.</div>
  </div>

  <!-- QRCode.js (MIT) via CDN. If you need fully offline, download this file and inline it. -->
  <script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>
  <script>
    // --- Utilities ----------------------------------------------------------
    const $$ = sel => document.querySelector(sel);
    const createId = () => (Date.now().toString(36) + Math.random().toString(36).slice(2,8)).toUpperCase();
    const storageKey = 'tapback_registrations_v1';

    function loadHistory(){
      try{ return JSON.parse(localStorage.getItem(storageKey) || '[]'); }catch{ return [] }
    }
    function saveHistory(list){ localStorage.setItem(storageKey, JSON.stringify(list)); }
    function addToHistory(entry){ const list = loadHistory(); list.unshift(entry); saveHistory(list.slice(0,10)); renderHistory(); }

    function sanitize(str){ return String(str).replace(/[\n\r\t]/g,' ').trim(); }

    function makePayload({id, name, phone}){
      // Encode a compact payload – easy to parse later if you wire up a backend
      return JSON.stringify({ t:"tapback:reg", v:1, id, name, phone, ts: Date.now() });
    }

    // --- QR handling --------------------------------------------------------
    let qrInstance = null;

    function renderQR(text){
      const el = $$('#qr');
      el.innerHTML = '';
      qrInstance = new QRCode(el, {
        text,
        width: 256,
        height: 256,
        correctLevel: QRCode.CorrectLevel.M,
      });
      enableQRButtons(true);
    }

    function enableQRButtons(on){
      $$('#btnDownload').disabled = !on;
      $$('#btnCopyData').disabled = !on;
    }

    function downloadQR(){
      if(!qrInstance) return;
      // QRCode.js renders an <img> or <canvas>. Prefer canvas if available for PNG export.
      const img = $$('#qr img');
      const cvs = $$('#qr canvas');
      let dataURL = '';
      if(cvs){ dataURL = cvs.toDataURL('image/png'); }
      else if(img){
        // If it's an <img>, draw it to a temp canvas to export
        const c = document.createElement('canvas');
        c.width = img.naturalWidth || 256; c.height = img.naturalHeight || 256;
        const ctx = c.getContext('2d');
        ctx.fillStyle = '#ffffff'; ctx.fillRect(0,0,c.width,c.height);
        ctx.drawImage(img,0,0,c.width,c.height);
        dataURL = c.toDataURL('image/png');
      }
      if(!dataURL) return;
      const a = document.createElement('a');
      a.href = dataURL;
      a.download = 'tapback-qr.png';
      a.click();
    }

    function copyEncoded(){
      const img = $$('#qr img');
      const cvs = $$('#qr canvas');
      // Retrieve the encoded text from QRCode.js instance by reading the container's dataset if needed.
      // We stored it in the last generation step too.
      const payload = window.__lastPayload || '';
      if(!payload) return;
      navigator.clipboard?.writeText(payload).then(()=>{
        setStatus('Copied encoded data to clipboard ✅', 'ok');
      }).catch(()=>{
        setStatus('Unable to copy automatically. You can select the text from history below.', 'warn');
      });
    }

    // --- UI / Form ----------------------------------------------------------
    function setStatus(msg, kind='ok'){
      const el = $$('#formStatus');
      el.textContent = msg;
      el.style.color = kind==='ok' ? 'var(--good)' : (kind==='warn' ? 'var(--warn)' : 'var(--err)');
    }

    function renderHistory(){
      const host = $$('#history');
      const list = loadHistory();
      if(!list.length){ host.innerHTML = '<div class="item">No entries yet. Generate one to see it here.</div>'; return; }
      host.innerHTML = list.map(e=>{
        const when = new Date(e.ts).toLocaleString();
        const safe = makePayload(e).replaceAll('<','&lt;');
        return `<div class="item"><div><strong>${e.name}</strong> · <span class="note">${e.phone}</span></div><div class="note">${when}</div><details><summary class="note">Encoded payload</summary><code style="word-break:break-all; font-size:12px">${safe}</code></details></div>`
      }).join('');
    }

    function prefill(){
      $$('#fullName').value = 'Alex Demo';
      $$('#phone').value = '+91 98765 43210';
      $$('#fullName').focus();
    }

    function resetAll(){
      $$('#regForm').reset();
      $$('#qr').innerHTML = '';
      enableQRButtons(false);
      setStatus('');
    }

    function onSubmit(e){
      e.preventDefault();
      const name = sanitize($$('#fullName').value);
      const phone = sanitize($$('#phone').value);

      // HTML validation first
      if(!name || name.length < 2){ setStatus('Please enter a valid full name (min 2 chars).', 'err'); return; }
      const tel = $$('#phone');
      if(!tel.checkValidity()){ setStatus('Please enter a valid phone number (digits, spaces, dashes, optional +).', 'err'); return; }

      const entry = { id: createId(), name, phone, ts: Date.now() };
      const payload = makePayload(entry);
      window.__lastPayload = payload;

      renderQR(payload);
      addToHistory(entry);
      setStatus('QR generated successfully. You can download it or copy the encoded data.', 'ok');
    }

    // --- Wire-up ------------------------------------------------------------
    document.addEventListener('DOMContentLoaded', () => {
      renderHistory();
      enableQRButtons(false);
      $$('#regForm').addEventListener('submit', onSubmit);
      $$('#btnClear').addEventListener('click', resetAll);
      $$('#btnPrefill').addEventListener('click', prefill);
      $$('#btnDownload').addEventListener('click', downloadQR);
      $$('#btnCopyData').addEventListener('click', copyEncoded);

      // If CDN fails (offline), let the user know gracefully.
      if(typeof QRCode === 'undefined'){
        $$('#qr').innerHTML = '<div class="note">QR library failed to load. If you need offline use, download qrcode.min.js and inline it in this file (see comment near the script tag).</div>';
      }
    });
  </script>
</body>
</html>
