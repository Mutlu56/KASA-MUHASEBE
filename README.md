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
