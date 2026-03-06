<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>SizeCheck.io — Real-size visualizer</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
--bg: #0c0c0c; --bg2: #141414; --border: #2a2a2a;
--accent: #e8ff47; --accent2: #ff6b35; --text: #f0f0f0; --muted: #555;
--mono: 'DM Mono', monospace; --sans: 'Syne', sans-serif;
}
html, body { height: 100%; background: var(--bg); color: var(--text); font-family: var(--sans); overflow-x: hidden; }
body::before {
content: ''; position: fixed; inset: 0;
background-image: linear-gradient(rgba(255,255,255,0.025) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,0.025) 1px, transparent 1px);
background-size: 40px 40px; pointer-events: none; z-index: 0;
}
header {
position: relative; z-index: 10; display: flex; align-items: center; justify-content: space-between;
padding: 16px 32px; border-bottom: 1px solid var(--border);
background: rgba(12,12,12,0.9); backdrop-filter: blur(10px); gap: 12px;
}
.logo { font-family: var(--sans); font-weight: 800; font-size: 1.1rem; letter-spacing: -0.02em; }
.logo span { color: var(--accent); }
.header-right { display: flex; align-items: center; gap: 10px; }
.lang-switcher { display: flex; gap: 4px; align-items: center; }
.lang-btn {
width: 32px; height: 22px; border: 1.5px solid var(--border); background: transparent;
cursor: pointer; font-size: 1.1rem; line-height: 1; display: flex; align-items: center;
justify-content: center; transition: border-color 0.15s, transform 0.1s;
border-radius: 2px; opacity: 0.4; padding: 0;
}
.lang-btn:hover { opacity: 0.8; transform: scale(1.08); }
.lang-btn.active { border-color: var(--accent); opacity: 1; box-shadow: 0 0 6px rgba(232,255,71,0.3); }
.calibrate-btn {
display: flex; align-items: center; gap: 8px; padding: 8px 16px;
background: transparent; border: 1px solid var(--border); color: var(--muted);
font-family: var(--mono); font-size: 0.72rem; letter-spacing: 0.08em;
cursor: pointer; transition: all 0.2s; text-transform: uppercase;
}
.calibrate-btn:hover { border-color: var(--accent); color: var(--accent); }
.calibrate-btn.calibrated { border-color: #3ddc84; color: #3ddc84; }
.dot { width: 7px; height: 7px; border-radius: 50%; background: var(--muted); transition: background 0.3s; }
.calibrated .dot { background: #3ddc84; box-shadow: 0 0 6px #3ddc84; }
main { position: relative; z-index: 1; max-width: 900px; margin: 0 auto; padding: 40px 24px 60px; }
.inputs-section { display: flex; align-items: center; gap: 12px; margin-bottom: 36px; flex-wrap: wrap; }
.field-wrap { display: flex; flex-direction: column; gap: 6px; }
.field-label { font-family: var(--mono); font-size: 0.65rem; letter-spacing: 0.15em; text-transform: uppercase; color: var(--muted); }
.dim-input {
width: 130px; padding: 14px 16px; background: var(--bg2);
border: 1px solid var(--border); border-bottom: 2px solid var(--border);
color: var(--text); font-family: var(--mono); font-size: 1.6rem; font-weight: 500;
outline: none; transition: border-color 0.2s; -moz-appearance: textfield;
}
.dim-input::-webkit-outer-spin-button, .dim-input::-webkit-inner-spin-button { -webkit-appearance: none; }
.dim-input:focus { border-color: var(--accent); border-bottom-color: var(--accent); background: #181818; }
.dim-input::placeholder { color: #2a2a2a; }
.separator { font-family: var(--mono); font-size: 2rem; color: var(--muted); margin-top: 22px; user-select: none; }
.unit-select {
margin-top: 22px; padding: 14px 12px; background: var(--bg2);
border: 1px solid var(--border); color: var(--text);
font-family: var(--mono); font-size: 0.85rem; outline: none; cursor: pointer;
}
.unit-select:focus { border-color: var(--accent); }
.scale-warning {
display: none; align-items: center; gap: 10px; padding: 12px 16px;
background: rgba(255,107,53,0.08); border: 1px solid rgba(255,107,53,0.25);
margin-bottom: 24px; font-family: var(--mono); font-size: 0.72rem;
color: var(--accent2); letter-spacing: 0.05em;
}
.scale-warning.visible { display: flex; }
.canvas-zone { position: relative; width: 100%; min-height: 120px; margin-bottom: 32px; }
.canvas-placeholder {
display: flex; align-items: center; justify-content: center; min-height: 200px;
border: 1px dashed var(--border); color: var(--muted);
font-family: var(--mono); font-size: 0.8rem; letter-spacing: 0.08em;
}
#rect-container { position: relative; display: none; }
#main-rect {
position: relative; background: rgba(232,255,71,0.07); border: 2px solid var(--accent);
transition: width 0.15s ease, height 0.15s ease; flex-shrink: 0;
}
#main-rect::before, #main-rect::after { content: ''; position: absolute; width: 14px; height: 14px; }
#main-rect::before { top: -2px; left: -2px; border-top: 3px solid var(--accent); border-left: 3px solid var(--accent); }
#main-rect::after { bottom: -2px; right: -2px; border-bottom: 3px solid var(--accent); border-right: 3px solid var(--accent); }
.rect-dim-label { position: absolute; font-family: var(--mono); font-size: 0.65rem; color: var(--accent); letter-spacing: 0.06em; white-space: nowrap; }
.label-w { top: -22px; left: 0; }
.label-h { right: -60px; top: 0; writing-mode: vertical-rl; transform: rotate(180deg); }
.compare-overlay { position: absolute; top: 0; left: 0; pointer-events: none; }
.compare-shape { position: absolute; top: 0; left: 0; border: 1.5px dashed; opacity: 0.7; }
.compare-label-overlay { position: absolute; font-family: var(--mono); font-size: 0.58rem; letter-spacing: 0.05em; white-space: nowrap; opacity: 0.8; top: 4px; left: 4px; }
.compare-section { margin-bottom: 32px; }
.compare-title { font-family: var(--mono); font-size: 0.65rem; letter-spacing: 0.15em; text-transform: uppercase; color: var(--muted); margin-bottom: 12px; }
.compare-chips { display: flex; flex-wrap: wrap; gap: 8px; }
.chip {
padding: 7px 14px; border: 1px solid var(--border); background: transparent;
color: var(--muted); font-family: var(--mono); font-size: 0.7rem;
letter-spacing: 0.06em; cursor: pointer; transition: all 0.15s; user-select: none;
}
.chip:hover { border-color: #444; color: var(--text); }
.chip.active { border-color: var(--accent2); color: var(--accent2); background: rgba(255,107,53,0.06); }
.modal-overlay {
display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.85);
z-index: 100; align-items: center; justify-content: center; backdrop-filter: blur(4px);
}
.modal-overlay.open { display: flex; }
.modal { position: relative; background: var(--bg2); border: 1px solid var(--border); padding: 36px; max-width: 520px; width: calc(100% - 32px); }
.modal h2 { font-family: var(--sans); font-weight: 700; font-size: 1.3rem; margin-bottom: 8px; }
.modal-subtitle { font-family: var(--mono); font-size: 0.72rem; color: var(--muted); letter-spacing: 0.05em; margin-bottom: 28px; line-height: 1.7; }
.cal-tabs { display: flex; margin-bottom: 24px; border: 1px solid var(--border); }
.cal-tab { flex: 1; padding: 10px; background: transparent; border: none; color: var(--muted); font-family: var(--mono); font-size: 0.72rem; letter-spacing: 0.06em; cursor: pointer; transition: all 0.15s; text-align: center; }
.cal-tab.active { background: var(--accent); color: #000; font-weight: 500; }
.cal-area {
position: relative; height: 220px; background: #0a0a0a; border: 1px solid var(--border);
margin-bottom: 20px; overflow: hidden; display: flex; align-items: center; justify-content: center;
}
.cal-object { position: absolute; cursor: se-resize; touch-action: none; }
.cal-card { background: linear-gradient(135deg,#1a1a2e 0%,#16213e 50%,#0f3460 100%); border: 1px solid rgba(255,255,255,0.15); border-radius: 6px; user-select: none; }
.cal-coin { border-radius: 50%; background: radial-gradient(circle at 35% 35%,#f5c842,#c8962c,#8b6914); border: 2px solid #c8962c; user-select: none; }
.cal-resize-handle { position: absolute; bottom: -5px; right: -5px; width: 14px; height: 14px; background: var(--accent); cursor: se-resize; z-index: 5; }
.cal-instructions { font-family: var(--mono); font-size: 0.65rem; color: var(--muted); text-align: center; margin-bottom: 20px; letter-spacing: 0.05em; line-height: 1.6; }
.cal-instructions strong { color: var(--text); }
.modal-actions { display: flex; gap: 12px; justify-content: flex-end; }
.btn-cancel { padding: 11px 20px; background: transparent; border: 1px solid var(--border); color: var(--muted); font-family: var(--mono); font-size: 0.75rem; cursor: pointer; transition: all 0.15s; }
.btn-cancel:hover { border-color: #444; color: var(--text); }
.btn-confirm { padding: 11px 24px; background: var(--accent); border: none; color: #000; font-family: var(--mono); font-size: 0.75rem; font-weight: 500; letter-spacing: 0.06em; cursor: pointer; transition: opacity 0.15s; }
.btn-confirm:hover { opacity: 0.85; }
.ruler-h, .ruler-v { position: absolute; background: rgba(232,255,71,0.15); }
.ruler-h { height: 1px; width: 100%; top: 50%; }
.ruler-v { width: 1px; height: 100%; left: 50%; }
.live-dims { position: absolute; top: 8px; right: 8px; font-family: var(--mono); font-size: 0.62rem; color: rgba(232,255,71,0.5); letter-spacing: 0.06em; pointer-events: none; }
.footer-hint { font-family: var(--mono); font-size: 0.65rem; color: #333; letter-spacing: 0.06em; text-align: center; margin-top: 48px; }
@media (max-width: 600px) {
header { padding: 12px 16px; }
main { padding: 24px 16px 48px; }
.dim-input { width: 100px; font-size: 1.3rem; }
.modal { padding: 24px 20px; }
.cal-btn-text { display: none; }
}
</style>
</head>
<body>

<header>
<div class="logo">Size<span>Check</span><span style="color:var(--muted);font-weight:400">.io</span></div>
<div class="header-right">
<div class="lang-switcher">
<button class="lang-btn active" id="btnFR" onclick="setLang('fr')" title="Français">🇫🇷</button>
<button class="lang-btn" id="btnEN" onclick="setLang('en')" title="English">🇬🇧</button>
</div>
<button class="calibrate-btn" id="calBtn" onclick="openCalModal()">
<div class="dot"></div>
<span class="cal-btn-text" id="calBtnText"></span>
</button>
</div>
</header>

<main>
<div class="scale-warning" id="scaleWarning"></div>
<div class="inputs-section">
<div class="field-wrap">
<span class="field-label" id="labelWidth"></span>
<input class="dim-input" type="number" id="inputW" placeholder="100" min="1" max="9999">
</div>
<div class="separator">×</div>
<div class="field-wrap">
<span class="field-label" id="labelHeight"></span>
<input class="dim-input" type="number" id="inputH" placeholder="150" min="1" max="9999">
</div>
<select class="unit-select" id="unitSelect">
<option value="mm">mm</option>
<option value="cm">cm</option>
<option value="in">in</option>
<option value="px">px</option>
</select>
</div>
<div class="compare-section">
<div class="compare-title" id="compareTitle"></div>
<div class="compare-chips" id="compareChips"></div>
</div>
<div class="canvas-zone">
<div class="canvas-placeholder" id="placeholder"></div>
<div id="rect-container">
<div id="main-rect">
<span class="rect-dim-label label-w" id="dimLabelW"></span>
<span class="rect-dim-label label-h" id="dimLabelH"></span>
</div>
<div class="compare-overlay" id="compareOverlay"></div>
</div>
<div class="live-dims" id="liveDims"></div>
</div>
<div class="footer-hint" id="footerHint"></div>
</main>

<!-- MODAL -->

<div class="modal-overlay" id="calModal">
<div class="modal">
<h2 id="modalTitle"></h2>
<p class="modal-subtitle" id="modalSubtitle"></p>
<div class="cal-tabs">
<button class="cal-tab active" id="tabCard" onclick="setCalMode('card')"></button>
<button class="cal-tab" id="tabCoin" onclick="setCalMode('coin')"></button>
</div>
<div class="cal-area" id="calArea">
<div class="ruler-h"></div>
<div class="ruler-v"></div>
<div class="cal-object" id="calObject"></div>
<div class="live-dims" id="calLiveDims"></div>
</div>
<p class="cal-instructions" id="calInstructions"></p>
<div class="modal-actions">
<button class="btn-cancel" id="btnCancel" onclick="closeCalModal()"></button>
<button class="btn-confirm" id="btnConfirm" onclick="confirmCalibration()"></button>
</div>
</div>
</div>

<script>
// ── TRANSLATIONS ─────────────────────────────────────────────────────────────
const T = {
fr: {
calBtn: 'Calibrer l\'écran', calBtnDone: 'Calibré ✓',
warning: '⚠ Calibration requise pour l\'affichage à l\'échelle réelle — dimensions approximatives',
labelWidth: 'Largeur', labelHeight: 'Hauteur',
compareTitle: 'Comparer avec —',
placeholder: '↑ Entrez des dimensions pour visualiser',
scaled: '⚠ réduit à l\'affichage',
footer: 'Posez une carte bancaire sur l\'écran et calibrez une seule fois · iPhone, tablette, PC',
modalTitle: 'Calibrer l\'écran',
modalSub: 'Posez l\'objet sur votre écran, puis faites glisser le coin ▣ pour ajuster à la taille réelle.',
tabCard: '💳 Carte bancaire', tabCoin: '🪙 Pièce de 1€',
instrCard: 'Faites glisser <strong>▣</strong> jusqu\'à ce que la forme corresponde à votre carte<br><span style="opacity:.5">85.6 × 54 mm — ISO 7810</span>',
instrCoin: 'Faites glisser <strong>▣</strong> jusqu\'à ce que le cercle corresponde à votre pièce de 1€<br><span style="opacity:.5">Diamètre 23.25 mm</span>',
cancel: 'Annuler', confirm: '✓ Confirmer', resizeHint: 'Glisser pour redimensionner',
},
en: {
calBtn: 'Calibrate screen', calBtnDone: 'Calibrated ✓',
warning: '⚠ Calibration required for true real-size display — dimensions are approximate',
labelWidth: 'Width', labelHeight: 'Height',
compareTitle: 'Compare with —',
placeholder: '↑ Enter dimensions to visualize',
scaled: '⚠ scaled down to fit',
footer: 'Place a credit card on your screen and calibrate once · iPhone, tablet, PC',
modalTitle: 'Calibrate screen',
modalSub: 'Place the object on your screen, then drag corner ▣ to match its real-world size.',
tabCard: '💳 Credit card', tabCoin: '🪙 1€ coin',
instrCard: 'Drag <strong>▣</strong> until the shape matches your credit card exactly<br><span style="opacity:.5">85.6 × 54 mm — ISO 7810 standard</span>',
instrCoin: 'Drag <strong>▣</strong> until the circle matches your 1€ coin exactly<br><span style="opacity:.5">Diameter 23.25 mm</span>',
cancel: 'Cancel', confirm: '✓ Confirm calibration', resizeHint: 'Drag to resize',
}
};

const COMPARE = [
{ id:'iphone15', fr:'iPhone 15', en:'iPhone 15', w:71.5, h:147.6, color:'#60a5fa' },
{ id:'iphone15pm', fr:'iPhone 15 Pro Max', en:'iPhone 15 Pro Max', w:76.7, h:159.9, color:'#a78bfa' },
{ id:'card', fr:'Carte bancaire', en:'Credit card', w:85.6, h:54, color:'#34d399' },
{ id:'canette', fr:'Canette 33cl', en:'Soda can 33cl', w:66, h:115, color:'#fb923c' },
{ id:'a4', fr:'Feuille A4', en:'A4 Sheet', w:210, h:297, color:'#e879f9' },
{ id:'passport', fr:'Passeport', en:'Passport', w:125, h:88, color:'#f472b6' },
{ id:'macbook', fr:'MacBook Air 13"', en:'MacBook Air 13"', w:304.1, h:215, color:'#38bdf8' },
];

const OBJECTS = { card:{w:85.6,h:54}, coin:{w:23.25,h:23.25} };

// ── STATE ─────────────────────────────────────────────────────────────────────
let lang = 'fr';
let pxPerMm = 96 / 25.4;
let calMode = 'card';
let isCalibrated = false;
let activeCompare = new Set();
let calObjW = 0, calObjH = 0;

// ── LANG ──────────────────────────────────────────────────────────────────────
function setLang(l) {
lang = l;
document.documentElement.lang = l;
document.getElementById('btnFR').classList.toggle('active', l==='fr');
document.getElementById('btnEN').classList.toggle('active', l==='en');
applyT();
rebuildChips();
draw();
}

function tr(k) { return T[lang][k] || T.en[k] || k; }

function applyT() {
document.getElementById('calBtnText').textContent = isCalibrated ? tr('calBtnDone') : tr('calBtn');
document.getElementById('scaleWarning').textContent = tr('warning');
document.getElementById('labelWidth').textContent = tr('labelWidth');
document.getElementById('labelHeight').textContent = tr('labelHeight');
document.getElementById('compareTitle').textContent = tr('compareTitle');
document.getElementById('placeholder').textContent = tr('placeholder');
document.getElementById('footerHint').textContent = tr('footer');
document.getElementById('modalTitle').textContent = tr('modalTitle');
document.getElementById('modalSubtitle').textContent = tr('modalSub');
document.getElementById('tabCard').textContent = tr('tabCard');
document.getElementById('tabCoin').textContent = tr('tabCoin');
document.getElementById('btnCancel').textContent = tr('cancel');
document.getElementById('btnConfirm').textContent = tr('confirm');
refreshCalInstr();
}

function refreshCalInstr() {
const el = document.getElementById('calInstructions');
if (el) el.innerHTML = tr(calMode==='card' ? 'instrCard' : 'instrCoin');
}

// ── INIT ──────────────────────────────────────────────────────────────────────
window.addEventListener('DOMContentLoaded', () => {
document.getElementById('scaleWarning').classList.add('visible');
applyT();
rebuildChips();
buildCalObject();
document.getElementById('inputW').addEventListener('input', draw);
document.getElementById('inputH').addEventListener('input', draw);
document.getElementById('unitSelect').addEventListener('change', draw);
});

// ── UNITS ─────────────────────────────────────────────────────────────────────
function toMm(v, u) {
if (u==='mm') return v;
if (u==='cm') return v*10;
if (u==='in') return v*25.4;
if (u==='px') return v/pxPerMm;
return v;
}

// ── DRAW ──────────────────────────────────────────────────────────────────────
function draw() {
const wRaw = parseFloat(document.getElementById('inputW').value);
const hRaw = parseFloat(document.getElementById('inputH').value);
const unit = document.getElementById('unitSelect').value;
const ph = document.getElementById('placeholder');
const rc = document.getElementById('rect-container');

if (!wRaw || !hRaw || wRaw<=0 || hRaw<=0) {
ph.style.display='flex'; rc.style.display='none';
document.getElementById('liveDims').textContent=''; return;
}

const wMm=toMm(wRaw,unit), hMm=toMm(hRaw,unit);
let wPx=Math.round(wMm*pxPerMm), hPx=Math.round(hMm*pxPerMm);

const maxW=window.innerWidth*0.92, maxH=window.innerHeight*0.6;
let scaled=false;
if (wPx>maxW||hPx>maxH) {
const s=Math.min(maxW/wPx,maxH/hPx);
wPx=Math.round(wPx*s); hPx=Math.round(hPx*s); scaled=true;
}

ph.style.display='none'; rc.style.display='block';
const rect=document.getElementById('main-rect');
rect.style.width=wPx+'px'; rect.style.height=hPx+'px';
document.getElementById('dimLabelW').textContent=`${wRaw} ${unit}`;
document.getElementById('dimLabelH').textContent=`${hRaw} ${unit}`;
document.getElementById('liveDims').textContent=scaled?tr('scaled'):'';
drawComparisons(wPx,hPx,wMm,hMm);
}

// ── COMPARE ───────────────────────────────────────────────────────────────────
function rebuildChips() {
const c=document.getElementById('compareChips');
c.innerHTML='';
COMPARE.forEach(item=>{
const chip=document.createElement('button');
chip.className='chip'+(activeCompare.has(item.id)?' active':'');
chip.dataset.id=item.id;
chip.textContent=item[lang]||item.en;
if (activeCompare.has(item.id)) {
chip.style.borderColor=item.color; chip.style.color=item.color; chip.style.background=item.color+'11';
}
chip.onclick=()=>toggleCompare(item.id,chip);
c.appendChild(chip);
});
}

function toggleCompare(id,chip) {
if (activeCompare.has(id)) {
activeCompare.delete(id); chip.classList.remove('active');
chip.style.removeProperty('border-color'); chip.style.removeProperty('color'); chip.style.removeProperty('background');
} else {
activeCompare.add(id); chip.classList.add('active');
const item=COMPARE.find(x=>x.id===id);
chip.style.borderColor=item.color; chip.style.color=item.color; chip.style.background=item.color+'11';
}
draw();
}

function drawComparisons(mainWpx,mainHpx,mainWmm,mainHmm) {
const ov=document.getElementById('compareOverlay');
ov.innerHTML=''; ov.style.width=mainWpx+'px'; ov.style.height=mainHpx+'px';
const maxW=window.innerWidth*0.92, maxH=window.innerHeight*0.6;
const fW=Math.round(mainWmm*pxPerMm), fH=Math.round(mainHmm*pxPerMm);
const globalScale=(fW>maxW||fH>maxH)?Math.min(maxW/fW,maxH/fH):1;

activeCompare.forEach(id=>{
const item=COMPARE.find(x=>x.id===id); if (!item) return;
const wPx=Math.round(item.w*pxPerMm*globalScale);
const hPx=Math.round(item.h*pxPerMm*globalScale);
const shape=document.createElement('div');
shape.className='compare-shape';
shape.style.width=wPx+'px'; shape.style.height=hPx+'px';
shape.style.borderColor=item.color;
shape.style.borderRadius=(id==='canette')?'50% 50% 0 0 / 10% 10% 0 0':(id==='iphone15'||id==='iphone15pm')?'10px':'0';
const lbl=document.createElement('div');
lbl.className='compare-label-overlay'; lbl.style.color=item.color;
lbl.textContent=item[lang]||item.en; shape.appendChild(lbl);
ov.appendChild(shape);
});
}

// ── CALIBRATION ───────────────────────────────────────────────────────────────
function openCalModal() { document.getElementById('calModal').classList.add('open'); buildCalObject(); }
function closeCalModal() { document.getElementById('calModal').classList.remove('open'); }

function setCalMode(mode) {
calMode=mode;
document.getElementById('tabCard').classList.toggle('active',mode==='card');
document.getElementById('tabCoin').classList.toggle('active',mode==='coin');
buildCalObject(); refreshCalInstr();
}

function buildCalObject() {
const obj=OBJECTS[calMode];
const area=document.getElementById('calArea');
const container=document.getElementById('calObject');
const roughPx=96/25.4;
calObjW=Math.round(obj.w*roughPx); calObjH=Math.round(obj.h*roughPx);
const aW=area.clientWidth||400, aH=area.clientHeight||200;
container.style.left=Math.max(0,Math.round((aW-calObjW)/2))+'px';
container.style.top=Math.max(0,Math.round((aH-calObjH)/2))+'px';
container.style.width=calObjW+'px'; container.style.height=calObjH+'px';
const hint=tr('resizeHint');
if (calMode==='card') {
container.innerHTML=`<div class="cal-card" style="width:100%;height:100%;position:relative;">
<div style="position:absolute;left:10%;bottom:25%;width:22%;height:35%;background:linear-gradient(135deg,#d4a843,#f5c842);border-radius:3px;"></div>
</div><div class="cal-resize-handle" title="${hint}"></div>`;
} else {
container.innerHTML=`<div class="cal-coin" style="width:100%;height:100%;"></div><div class="cal-resize-handle" title="${hint}"></div>`;
}
refreshCalInstr(); updateCalDims(); setupCalDrag();
}

function updateCalDims() {
document.getElementById('calLiveDims').textContent=`${calObjW}px × ${calObjH}px`;
}

function setupCalDrag() {
const container=document.getElementById('calObject');
const area=document.getElementById('calArea');
const handle=container.querySelector('.cal-resize-handle');
if (!handle) return;
let dragging=false,startX,startW;
const onDown=(e)=>{ e.preventDefault(); dragging=true; const p=e.touches?e.touches[0]:e; startX=p.clientX; startW=container.offsetWidth; };
const onMove=(e)=>{ if (!dragging) return; e.preventDefault();
const p=e.touches?e.touches[0]:e, dx=p.clientX-startX;
const obj=OBJECTS[calMode], ar=obj.h/obj.w;
let nW=Math.max(30,startW+dx), nH=Math.round(nW*ar);
const aW=area.clientWidth,aH=area.clientHeight;
if (nW>aW-10){nW=aW-10;nH=Math.round(nW*ar);}
if (nH>aH-10){nH=aH-10;nW=Math.round(nH/ar);}
calObjW=nW; calObjH=nH;
container.style.width=nW+'px'; container.style.height=nH+'px';
const l=parseInt(container.style.left)||0,t=parseInt(container.style.top)||0;
if (l+nW>aW) container.style.left=Math.max(0,aW-nW)+'px';
if (t+nH>aH) container.style.top=Math.max(0,aH-nH)+'px';
updateCalDims();
};
const onUp=()=>{ dragging=false; };
handle.addEventListener('mousedown',onDown);
handle.addEventListener('touchstart',onDown,{passive:false});
window.addEventListener('mousemove',onMove);
window.addEventListener('touchmove',onMove,{passive:false});
window.addEventListener('mouseup',onUp);
window.addEventListener('touchend',onUp);
}

function confirmCalibration() {
pxPerMm=calObjW/OBJECTS[calMode].w;
isCalibrated=true;
document.getElementById('calBtn').classList.add('calibrated');
document.getElementById('calBtnText').textContent=tr('calBtnDone');
document.getElementById('scaleWarning').classList.remove('visible');
closeCalModal(); draw();
}
</script>

</body>
</html>
