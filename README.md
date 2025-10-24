<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="utf-8"/>
  <title>Prediksi 4 Digit — Global + Kombinasi A–T</title>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <style>
    body { font-family: Arial, sans-serif; margin:20px; background:#f6f8fa; color:#222; }
    h2 { margin-bottom:6px; }
    textarea { width:100%; height:150px; padding:8px; font-family:monospace; box-sizing:border-box; }
    .pattern-row { display:flex; flex-wrap:wrap; gap:6px; margin:10px 0; }
    .btn { padding:8px 10px; border-radius:6px; border:1px solid #cfcfcf; background:white; cursor:pointer; }
    .btn:hover { background:#f0f0f0; }
    .btn-active { background:#2d83f7; color:white; border-color:#1b62d6; }
    .controls { display:flex; gap:8px; align-items:center; margin-bottom:8px; }
    .panel { background:#fff; border:1px solid #e0e0e0; padding:10px; border-radius:6px; min-height:80px; box-sizing:border-box; }
    .result-box { font-family:monospace; white-space:nowrap; overflow:auto; padding:6px; }
    .meta { font-size:13px; color:#444; margin-top:6px; }
    .small-muted { font-size:13px; color:#666; }
    .two-horizontal { display:flex; gap:10px; flex-wrap:wrap; align-items:center; }
    .copyBtn { margin-left:8px; }
    table { width:100%; border-collapse:collapse; margin-top:8px; font-size:13px; }
    th, td { border:1px solid #ddd; padding:6px; text-align:center; }
    th { background:#f3f3f3; }
    .hit { color:green; font-weight:bold; }
    .miss { color:red; font-weight:bold; }
    .note { font-size:13px; color:#333; margin-top:8px; }
  </style>
</head>
<body>
  <h2>Mesin Prediksi 4 Digit — Global + Kombinasi (A–T)</h2>
  <div class="note">Masukkan histori 4-digit (pisahkan dengan spasi, koma, newline). Contoh: <code>9880, 4264 8840 2194 ...</code></div>
  <textarea id="historyBox" placeholder="Masukkan histori angka 4 digit..."></textarea>

  <div class="small-muted">Pilih pola (A–T) — klik untuk memilih pola aktif (klik pola juga akan langsung menjalankan Prediksi jika histori ada):</div>
  <div id="patternRow" class="pattern-row"></div>

  <div class="controls two-horizontal" style="margin-top:8px;">
    <div style="display:flex; gap:8px;">
      <button id="predictBtn" class="btn" onclick="predictGlobal()">Prediksi Global</button>
      <button id="comboBtn" class="btn" onclick="computeCombination()">Kombinasi (2 pola terbaik)</button>
      <button class="btn" onclick="resetAll()">Reset</button>
    </div>
    <div style="margin-left:auto" class="small-muted">Kombinasi hanya menghitung saat tombol ditekan.</div>
  </div>

  <h3>Hasil Prediksi Global (maks 500 angka)</h3>
  <div id="resultGlobalPanel" class="panel">
    <div id="resultGlobal" class="result-box">(Belum ada prediksi)</div>
  </div>
  <div style="display:flex; justify-content:space-between; align-items:center; gap:12px; margin-top:6px;">
    <div id="globalMeta" class="meta">Jumlah kandidat: -</div>
    <div>
      <button class="btn" onclick="copyGlobal()">Salin Hasil Global</button>
    </div>
  </div>

  <h3>Akurasi & Tabel Pengujian (100 histori terakhir — per pola yang dipilih)</h3>
  <div id="accuracyBox" class="panel small-muted">(Akurasi belum dihitung)</div>
  <div id="testTableBox" class="panel" style="margin-top:8px; font-family:inherit;"></div>

  <h3>Kotak Kombinasi Pola Terbaik (2 pola)</h3>
  <div id="comboPanel" class="panel">
    <div id="comboInfo" class="small-muted">Tekan tombol <b>Kombinasi</b> untuk menghitung 2 pola terbaik dari A–T (berdasarkan akurasi 100 histori terakhir).</div>
    <div id="comboResults" style="margin-top:10px;"></div>
  </div>

<script>
/* ============================
   Helper & state
   ============================ */
const PATTERNS = 'ABCDEFGHIJKLMNOPQRST'.split(''); // A..T
let activePattern = null;

// create pattern buttons
const patternRow = document.getElementById('patternRow');
PATTERNS.forEach(p => {
  const btn = document.createElement('button');
  btn.className = 'btn';
  btn.textContent = p;
  btn.id = 'pat_' + p;
  btn.onclick = () => selectPattern(p);
  patternRow.appendChild(btn);
});

function selectPattern(p) {
  activePattern = p;
  // highlight
  PATTERNS.forEach(q => {
    const el = document.getElementById('pat_' + q);
    if (el) el.classList.toggle('btn-active', q === p);
  });
  // update accuracy panel quick hint
  document.getElementById('accuracyBox').textContent = '(Pola aktif: ' + p + ')';
  // If there is histori, immediately run prediction for convenience
  const raw = document.getElementById('historyBox').value.trim();
  if (raw) {
    // small delay to allow button highlight render
    setTimeout(() => { try { predictGlobal(); } catch(e){ console.error(e); } }, 80);
  }
}

/* parse input */
function cleanInput(raw) {
  return raw.split(/[^0-9]+/).map(x => x.trim()).filter(x => /^\d{4}$/.test(x));
}
function valid4Digit(num) {
  return /^\d{4}$/.test(num) && !/(.)\1{2}/.test(num);
}

/* ============================
   Pattern generators A - T
   ============================ */
function generatePredictions4D(numbers, pola) {
  const allPatterns = PATTERNS;
  if (!allPatterns.includes(pola)) return [];
  let candidates = [];
  for (let num of numbers) {
    let d = num.split('').map(n => parseInt(n, 10));
    let results = [];
    const make = arr => arr.map(x => ((x % 10) + 10) % 10).join('');
    switch (pola) {
      case 'A':
        results.push(make([(d[0]+d[1]+d[2]+d[3])%10, (d[0]+d[3])%10, (d[1]+d[2])%10, Math.abs(d[0]-d[3])%10]));
        break;
      case 'B':
        results.push(make([(d[0]*d[1])%10, (d[2]*d[3])%10, (d[1]+d[3])%10, (d[0]+d[2])%10]));
        break;
      case 'C':
        results.push(make([(d[3]+d[2])%10, (d[2]-d[1]+10)%10, (d[1]+d[0])%10, Math.abs(d[0]-d[2])%10]));
        break;
      case 'D':
        let rnd = Math.floor(Math.random()*4);
        let mix = [
          [(d[0]+d[2])%10,(d[3]+d[1])%10,(d[0]*d[1])%10,(d[2]*d[3])%10],
          [(d[1]+d[3])%10,(d[2]+d[0])%10,Math.abs(d[1]-d[0])%10,(d[3]+d[2])%10],
          [(d[0]*d[3])%10,(d[1]+d[2])%10,(d[3]-d[1]+10)%10,(d[0]+d[2])%10],
          [(d[2]+d[1])%10,(d[3]+d[0])%10,(d[1]*d[3])%10,Math.abs(d[0]-d[2])%10]
        ];
        results.push(make(mix[rnd]));
        break;
      case 'E':
        let rev = d.slice().reverse().map(x => (x + 1) % 10);
        results.push(make(rev));
        break;
      case 'F':
        results.push(make([(d[0]*2 + d[2])%10, (d[1] + d[3]*3)%10, (d[0]+d[1]+d[3])%10, (d[2]*2 + d[0])%10]));
        break;
      case 'G':
        let x1 = (d[0]+d[1])%10, x2 = (d[2]+d[3])%10;
        results.push(make([(x1+x2)%10, x1, x2, Math.abs(x1-x2)%10]));
        break;
      case 'H':
        results.push(make([(d[0]+d[1]+d[2])%10, (d[1]+d[2]+d[3])%10, (d[2]+d[3]+d[0])%10, (d[3]+d[0]+d[1])%10]));
        break;
      case 'I':
        results.push(make(d.map(x => x%2===0 ? (x*2)%10 : (x+3)%10)));
        break;
      case 'J':
        results.push(make([(d[1]-d[0]+10)%10, (d[2]-d[1]+10)%10, (d[3]-d[2]+10)%10, (d[0]-d[3]+10)%10]));
        break;
      case 'K':
        results.push(make([(d[0]+0)%10, (d[1]+1)%10, (d[2]+2)%10, (d[3]+3)%10]));
        break;
      case 'L':
        results.push(make([(d[3]+5)%10, (d[2]+5)%10, (d[1]+5)%10, (d[0]+5)%10]));
        break;
      case 'M':
        let m1=(d[0]+d[1])%10, m2=(d[1]+d[2])%10, m3=(d[2]+d[3])%10, m4=(m1+m3)%10;
        results.push(make([m1,m2,m3,m4]));
        break;
      case 'N':
        results.push(make([(9-d[0])%10, (9-d[1])%10, (9-d[2])%10, (9-d[3])%10]));
        break;
      case 'O':
        results.push(make(d.map(x => (x % 2 === 1 ? (x+4) : (x-3)) % 10)));
        break;
      case 'P':
        results.push(make([(d[1]-d[0]+10)%10, (d[2]-d[1]+10)%10, (d[3]-d[2]+10)%10, (d[0]-d[3]+10)%10]));
        break;
      case 'Q':
        let S = d[0]*1 + d[1]*2 + d[2]*3 + d[3]*4;
        let sStr = String(S).padStart(4,'0').slice(-4);
        results.push(sStr);
        break;
      case 'R':
        let eRev = d.slice().reverse().map(x => (x + 1) % 10);
        let nInv = d.map(x => (9 - x) % 10);
        let avg = eRev.map((v, idx) => Math.floor((v + nInv[idx]) / 2) % 10);
        results.push(make(avg));
        break;
      case 'S':
        let sum = d.reduce((a,b)=>a+b,0);
        let out = sum % 2 === 0 ? d.slice() : d.slice().reverse();
        results.push(make(out));
        break;
      case 'T':
        let chaos = d.map(x => x + (Math.floor(Math.random()*3) - 1));
        results.push(make(chaos));
        break;
    }
    candidates.push(...results);
    if (candidates.length > 5000) break; // safety
  }
  let unique = [...new Set(candidates)].filter(valid4Digit);
  return unique.slice(0, 500);
}

/* ============================
   Predict Global (uses activePattern)
   ============================ */
function predictGlobal() {
  const raw = document.getElementById('historyBox').value.trim();
  if (!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers = cleanInput(raw);
  if (!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');
  if (!activePattern) return alert('Pilih pola (A–T) terlebih dahulu dengan klik salah satu tombol pola.');

  const preds = generatePredictions4D(numbers, activePattern);
  if (!preds.length) {
    document.getElementById('resultGlobal').textContent = '(Tidak ada kandidat valid untuk pola ' + activePattern + ')';
    document.getElementById('globalMeta').textContent = 'Jumlah kandidat: 0';
  } else {
    document.getElementById('resultGlobal').textContent = preds.join('*');
    document.getElementById('globalMeta').textContent = 'Jumlah kandidat: ' + preds.length + ' (Pola ' + activePattern + ')';
  }

  const accText = calculateAccuracyGlobal(numbers, activePattern);
  document.getElementById('accuracyBox').textContent = accText.summary;
  document.getElementById('testTableBox').innerHTML = accText.tableHtml;
}

/* ============================
   Accuracy evaluation (global)
   ============================ */
function calculateAccuracyGlobal(numbers, pola) {
  if (numbers.length < 101) {
    return {
      summary: "Histori terlalu sedikit untuk evaluasi (butuh >=101 untuk akurasi penuh).",
      tableHtml: ""
    };
  }
  let hits = 0, misses = 0;
  let rows = [];
  const testRange = numbers.slice(-100);
  for (let i = 0; i < testRange.length - 1; i++) {
    let posFromEnd = testRange.length - 1 - i;
    let subsetGlobalEnd = numbers.length - posFromEnd;
    let subsetGlobal = numbers.slice(0, subsetGlobalEnd);
    let next = testRange[i + 1];
    let predGlobal = generatePredictions4D(subsetGlobal, pola);
    let hit = predGlobal.includes(next);
    if (hit) hits++; else misses++;
    rows.push(`<tr><td>${next}</td><td class="${hit ? 'hit' : 'miss'}">${hit ? 'Hit' : 'Miss'}</td></tr>`);
  }
  let acc = ((hits / (hits + misses)) * 100).toFixed(2);
  let summary = `Global: Hit ${hits}, Miss ${misses}, Akurasi ${acc}% (pola ${pola})`;
  let tableHtml = `<table><tr><th>Hasil Real</th><th>Global (${pola})</th></tr>${rows.join('')}</table>`;
  return { summary, tableHtml };
}

/* ============================
   Evaluate all patterns & pick top 2
   ============================ */
function evaluateAllPatterns(numbers) {
  const patterns = PATTERNS.slice();
  const results = [];
  const testRange = numbers.slice(-100);
  for (let pola of patterns) {
    let hits = 0, misses = 0;
    let history = [];
    for (let i = 0; i < testRange.length - 1; i++) {
      let posFromEnd = testRange.length - 1 - i;
      let subsetGlobalEnd = numbers.length - posFromEnd;
      let subsetGlobal = numbers.slice(0, subsetGlobalEnd);
      let next = testRange[i + 1];
      let predGlobal = generatePredictions4D(subsetGlobal, pola);
      let globalHit = predGlobal.includes(next);
      history.push(globalHit ? 'H' : 'M');
      if (globalHit) hits++; else misses++;
    }
    let total = hits + misses;
    let acc = total > 0 ? (hits / total) * 100 : 0;
    let longestMiss = 0, cur = 0;
    for (let s of history) {
      if (s === 'M') { cur++; if (cur > longestMiss) longestMiss = cur; }
      else { cur = 0; }
    }
    results.push({ pola, acc: parseFloat(acc.toFixed(4)), longestMiss });
  }
  results.sort((a,b) => {
    if (b.acc !== a.acc) return b.acc - a.acc;
    if (b.longestMiss !== a.longestMiss) return b.longestMiss - a.longestMiss;
    return a.pola.localeCompare(b.pola);
  });
  return results;
}

/* ============================
   Compute Combination (top 2)
   ============================ */
function computeCombination() {
  const raw = document.getElementById('historyBox').value.trim();
  if (!raw) return alert('Masukkan histori terlebih dahulu.');
  const numbers = cleanInput(raw);
  if (!numbers.length) return alert('Tidak ditemukan histori 4-digit valid.');

  const evals = evaluateAllPatterns(numbers);
  if (!evals || evals.length < 2) return alert('Gagal mengevaluasi pola.');

  const top1 = evals[0], top2 = evals[1];
  const preds1 = generatePredictions4D(numbers, top1.pola);
  const preds2 = generatePredictions4D(numbers, top2.pola);

  const combined = [];
  const seen = new Set();
  for (let x of preds1) { if (!seen.has(x)) { combined.push(x); seen.add(x); } }
  for (let x of preds2) { if (!seen.has(x)) { combined.push(x); seen.add(x); } }

  const html = `
    <div>
      <div><b>Pola Terbaik 1:</b> ${top1.pola} — Akurasi ${(top1.acc).toFixed(2)}% — MissStreak ${top1.longestMiss}</div>
      <div><b>Pola Terbaik 2:</b> ${top2.pola} — Akurasi ${(top2.acc).toFixed(2)}% — MissStreak ${top2.longestMiss}</div>
      <div class="meta" style="margin-top:8px;"><b>Total angka gabungan (unik):</b> ${combined.length}</div>
      <div id="comboList" class="result-box" style="margin-top:8px; white-space:nowrap;">${combined.length ? combined.join('*') : '(Kosong)'}</div>
      <div style="margin-top:8px;"><button class="btn" onclick="copyCombo()">Salin Hasil Kombinasi</button></div>
    </div>
  `;
  document.getElementById('comboInfo').textContent = '';
  document.getElementById('comboResults').innerHTML = html;
}

/* ============================
   Copy functions
   ============================ */
function copyGlobal() {
  const el = document.getElementById('resultGlobal');
  const text = el.innerText || el.textContent || '';
  if (!text.trim()) return alert('Tidak ada hasil untuk disalin.');
  navigator.clipboard.writeText(text).then(() => {
    alert('Hasil prediksi global disalin ke clipboard.');
  }).catch(err => {
    alert('Gagal menyalin: ' + err);
  });
}
function copyCombo() {
  const listDiv = document.getElementById('comboList');
  if (!listDiv) return alert('Tidak ada hasil kombinasi untuk disalin.');
  const text = listDiv.innerText || listDiv.textContent || '';
  if (!text.trim()) return alert('Tidak ada hasil kombinasi untuk disalin.');
  navigator.clipboard.writeText(text).then(() => {
    alert('Hasil kombinasi disalin ke clipboard.');
  }).catch(err => {
    alert('Gagal menyalin: ' + err);
  });
}

/* ============================
   Reset initial
   ============================ */
function resetAll() {
  document.getElementById('historyBox').value = '';
  document.getElementById('resultGlobal').textContent = '(Belum ada prediksi)';
  document.getElementById('globalMeta').textContent = 'Jumlah kandidat: -';
  document.getElementById('accuracyBox').textContent = '(Akurasi belum dihitung)';
  document.getElementById('testTableBox').textContent = '';
  document.getElementById('comboInfo').textContent = 'Tekan tombol Kombinasi untuk menghitung 2 pola terbaik dari A–T (berdasarkan akurasi 100 histori terakhir).';
  document.getElementById('comboResults').innerHTML = '';
  PATTERNS.forEach(q => {
    const el = document.getElementById('pat_' + q);
    if (el) el.classList.remove('btn-active');
  });
  activePattern = null;
}
resetAll();
</script>
</body>
</html>
