<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Kasa Kontrol Fişi</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <div class="container">
    <header>
      <h1>Kasa Kontrol Programı</h1>
      <p class="muted">Basit, çevrimdışı çalışır. Veriler tarayıcıda saklanır. CSV/JSON ile yedekleyin.</p>
    </header>

    <section class="form-section">
      <form id="fisForm" onsubmit="return false;">
        <div class="row">
          <label for="tur">İşlem Türü</label>
          <select id="tur" required>
            <option value="">-- Seçiniz --</option>
            <option value="gelir">Gelir (Giriş)</option>
            <option value="gider">Gider (Çıkış)</option>
          </select>
        </div>

        <div class="row">
          <label for="aciklama">Açıklama</label>
          <input type="text" id="aciklama" required />
        </div>

        <div class="row">
          <label for="tutar">Tutar (₺)</label>
          <input type="number" id="tutar" step="0.01" min="0.01" required />
        </div>

        <div class="row">
          <label for="tarih">Tarih</label>
          <input type="date" id="tarih" required />
        </div>

        <div class="row">
          <label for="yetkili">Yetkili</label>
          <input type="text" id="yetkili" required />
        </div>

        <div class="actions">
          <button id="kaydetBtn" type="button">Kaydet</button>
          <button id="temizleForm" type="button" class="secondary">Temizle</button>
        </div>
      </form>
    </section>

    <section class="controls">
      <input id="search" placeholder="Açıklama/Yetkili ara..." />
      <div class="export-buttons">
        <button id="exportCsv">CSV İndir</button>
        <button id="exportJson">JSON İndir</button>
        <input id="importFile" type="file" accept=".json,.csv" />
        <button id="clearAll" class="danger">Hepsini Sil</button>
      </div>
    </section>

    <section class="table-section">
      <table id="fisTablosu">
        <thead>
          <tr>
            <th data-sort="tur">Tür</th>
            <th data-sort="aciklama">Açıklama</th>
            <th data-sort="tutar">Tutar</th>
            <th data-sort="tarih">Tarih</th>
            <th data-sort="yetkili">Yetkili</th>
            <th>İşlem</th>
          </tr>
        </thead>
        <tbody></tbody>
        <tfoot>
          <tr class="toplam">
            <td colspan="2">Toplam Gelir</td>
            <td id="topGelir">0.00 ₺</td>
            <td colspan="3"></td>
          </tr>
          <tr class="toplam">
            <td colspan="2">Toplam Gider</td>
            <td id="topGider">0.00 ₺</td>
            <td colspan="3"></td>
          </tr>
          <tr class="toplam">
            <td colspan="2">Kasa Bakiyesi</td>
            <td id="net">0.00 ₺</td>
            <td colspan="3"></td>
          </tr>
        </tfoot>
      </table>
    </section>

    <footer>
      <small>Hazırlayan: Otomatik dönüştürücü · Veriler tarayıcıda saklanır (localStorage)</small>
    </footer>
  </div>

  <script src="app.js"></script>
</body>
</html>
// Basit veritabanı: localStorage
const STORAGE_KEY = "kasa_fisleri_v1";

let fisler = [];

// DOM
const tbody = document.querySelector("#fisTablosu tbody");
const topGelirEl = document.getElementById("topGelir");
const topGiderEl = document.getElementById("topGider");
const netEl = document.getElementById("net");
const form = document.getElementById("fisForm");
const kaydetBtn = document.getElementById("kaydetBtn");
const temizleBtn = document.getElementById("temizleForm");
const searchInput = document.getElementById("search");
const exportCsvBtn = document.getElementById("exportCsv");
const exportJsonBtn = document.getElementById("exportJson");
const importFile = document.getElementById("importFile");
const clearAllBtn = document.getElementById("clearAll");

// Yükle
loadFromStorage();
render();

// Eventler
kaydetBtn.addEventListener("click", handleSave);
temizleBtn.addEventListener("click", () => form.reset());
searchInput.addEventListener("input", () => render());
exportCsvBtn.addEventListener("click", exportCSV);
exportJsonBtn.addEventListener("click", exportJSON);
importFile.addEventListener("change", handleImport);
clearAllBtn.addEventListener("click", () => {
  if (!confirm("Tüm kayıtlar silinecek. Devam edilsin mi?")) return;
  fisler = [];
  saveToStorage();
  render();
});

// Fonksiyonlar
function handleSave(){
  const tur = document.getElementById("tur").value;
  const aciklama = document.getElementById("aciklama").value.trim();
  const tutarRaw = document.getElementById("tutar").value;
  const tutar = parseFloat(tutarRaw);
  const tarih = document.getElementById("tarih").value;
  const yetkili = document.getElementById("yetkili").value.trim();

  if (!tur || !aciklama || isNaN(tutar) || !tarih || !yetkili) {
    alert("Lütfen tüm alanları doldurun!");
    return;
  }

  const kayit = {
    id: Date.now().toString(),
    tur,
    aciklama,
    tutar: Number(tutar.toFixed(2)),
    tarih,
    yetkili
  };

  fisler.push(kayit);
  // Tarihe göre küçükten büyüğe sırala
  fisler.sort((a,b)=> new Date(a.tarih) - new Date(b.tarih));

  saveToStorage();
  render();
  form.reset();
}

function render(){
  // filtre
  const q = searchInput.value.trim().toLowerCase();
  const goster = fisler.filter(f=>{
    if (!q) return true;
    return f.aciklama.toLowerCase().includes(q) || f.yetkili.toLowerCase().includes(q) || f.tur.toLowerCase().includes(q);
  });

  tbody.innerHTML = "";
  let gelir = 0, gider = 0;
  for (const f of goster){
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${f.tur === "gelir" ? "Gelir" : "Gider"}</td>
      <td>${escapeHtml(f.aciklama)}</td>
      <td style="text-align:right">${f.tutar.toFixed(2)} ₺</td>
      <td>${f.tarih}</td>
      <td>${escapeHtml(f.yetkili)}</td>
      <td style="text-align:center">
        <button data-id="${f.id}" class="delete">Sil</button>
      </td>
    `;
    tbody.appendChild(tr);
    if (f.tur === "gelir") gelir += f.tutar;
    else gider += f.tutar;
  }

  topGelirEl.innerText = gelir.toFixed(2) + " ₺";
  topGiderEl.innerText = gider.toFixed(2) + " ₺";
  netEl.innerText = (gelir - gider).toFixed(2) + " ₺";

  // sil butonları
  tbody.querySelectorAll("button.delete").forEach(btn=>{
    btn.addEventListener("click", ()=>{
      const id = btn.getAttribute("data-id");
      if (!confirm("Kayıt silinsin mi?")) return;
      fisler = fisler.filter(x => x.id !== id);
      saveToStorage();
      render();
    });
  });
}

function saveToStorage(){
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(fisler));
  } catch(e){
    console.error("Storage hatası", e);
  }
}

function loadFromStorage(){
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) fisler = JSON.parse(raw);
    else fisler = [];
  } catch(e){
    fisler = [];
    console.error("Storage okuma hatası", e);
  }
}

function exportCSV(){
  if (!fisler.length){ alert("İhracat için kayıt yok."); return; }
  const rows = [["tur","aciklama","tutar","tarih","yetkili"]];
  fisler.forEach(f => rows.push([f.tur, f.aciklama.replace(/"/g,'""'), f.tutar, f.tarih, f.yetkili]));
  const csv = rows.map(r => r.map(c => `"${String(c).replace(/\n/g,' ')}"`).join(",")).join("\n");
  downloadFile(csv, "kasa_fisleri.csv", "text/csv;charset=utf-8;");
}

function exportJSON(){
  if (!fisler.length){ alert("İhracat için kayıt yok."); return; }
  const json = JSON.stringify(fisler, null, 2);
  downloadFile(json, "kasa_fisleri.json", "application/json;charset=utf-8;");
}

function downloadFile(content, filename, mime){
  const blob = new Blob([content], {type:mime});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url; a.download = filename; document.body.appendChild(a); a.click();
  setTimeout(()=>{ URL.revokeObjectURL(url); a.remove(); }, 5000);
}

function handleImport(e){
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (ev)=>{
    const text = ev.target.result;
    try {
      if (file.name.toLowerCase().endsWith(".json")){
        const data = JSON.parse(text);
        if (!Array.isArray(data)) throw new Error("JSON array bekleniyor");
        // basit doğrulama
        data.forEach(d => {
          if (!d.id) d.id = Date.now().toString() + Math.random().toString(36).slice(2,8);
        });
        fisler = fisler.concat(data);
      } else if (file.name.toLowerCase().endsWith(".csv")){
        const parsed = parseCSV(text);
        // beklenen sütunlar: tur,aciklama,tutar,tarih,yetkili
        const objs = parsed.map(r => ({
          id: Date.now().toString() + Math.random().toString(36).slice(2,8),
          tur: r[0],
          aciklama: r[1],
          tutar: parseFloat(r[2]) || 0,
          tarih: r[3],
          yetkili: r[4]
        }));
        fisler = fisler.concat(objs);
      } else {
        alert("Desteklenmeyen dosya türü. CSV veya JSON yükleyin.");
        return;
      }
      // Tarihe göre sırala
      fisler.sort((a,b)=> new Date(a.tarih) - new Date(b.tarih));
      saveToStorage();
      render();
      alert("İçe aktarma tamamlandı.");
    } catch(err){
      console.error(err);
      alert("İçe aktarma hatası: " + err.message);
    }
  };
  reader.readAsText(file, "utf-8");
  // input'u temizle ki aynı dosya tekrar seçilebilsin
  e.target.value = "";
}

function parseCSV(text){
  // Basit CSV parser (çift tırnakları destekler)
  const rows = [];
  let inQuotes = false;
  let field = "";
  for (let i=0;i<text.length;i++){
    const ch = text[i];
    const nxt = text[i+1];
    if (ch === '"' ){
      if (inQuotes && nxt === '"'){ field += '"'; i++; continue; }
      inQuotes = !inQuotes;
      continue;
    }
    if (ch === ',' && !inQuotes){ field += "\u0001"; continue; }
    if ((ch === '\n' || ch === '\r') && !inQuotes){
      if (field.length){
        rows.push(field.split("\u0001").map(c=>c.replace(/\u0001/g, "").trim().replace(/\u0000/g,'')));
        field = "";
      }
      if (ch === '\r' && text[i+1] === '\n') i++;
      continue;
    }
    field += ch;
  }
  if (field.length) rows.push(field.split("\u0001").map(c=>c.replace(/\u0001/g, "").trim()));
  if (rows.length && rows[0].some(h => /tur|aciklama|tutar|tarih|yetkili/i.test(h))) rows.shift();
  return rows;
}

function escapeHtml(s){
  return String(s).replace(/[&<>"']/g, function(m){
    return ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]);
  });
}
