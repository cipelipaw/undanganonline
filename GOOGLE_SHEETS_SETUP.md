# 📖 Panduan Menghubungkan Buku Tamu (Doa Restu) ke Google Sheets

Dengan panduan ini, setiap tamu yang mengirimkan doa restu / ucapan di website undangan **Diva & Dimas** akan langsung:
1. Tersimpan secara **real-time** di file Google Sheets Anda.
2. Muncul secara **otomatis** dan dapat dibaca oleh tamu undangan lainnya dari perangkat apa pun!

---

## Langkah 1: Buat Google Sheet Baru
1. Buka [Google Sheets](https://sheets.google.com) lalu buat Spreadsheet baru.
2. Beri nama file, misalnya: **Buku Tamu Diva & Dimas**.
3. Di baris paling atas (Baris 1), buat 4 kolom judul:
   - **Kolom A**: `Nama`
   - **Kolom B**: `Kehadiran`
   - **Kolom C**: `Pesan`
   - **Kolom D**: `Waktu`

---

## Langkah 2: Pasang Script Google Apps Script
1. Di menu atas Google Sheet, klik **Ekstensi** > **Apps Script**.
2. Hapus semua kode bawaan di editor, lalu **salin & tempel kode di bawah ini**:

```javascript
const SHEET_NAME = "Sheet1"; // Sesuaikan jika nama tab sheet Anda berbeda

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME) || SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const rows = sheet.getDataRange().getValues();
    
    // Hapus header baris pertama
    rows.shift();
    
    const wishes = rows.reverse().map(r => ({
      name: String(r[0] || ''),
      status: String(r[1] || 'Hadir'),
      message: String(r[2] || ''),
      time: r[3] ? formatDate(r[3]) : 'Baru saja'
    })).filter(w => w.name && w.message);

    return ContentService
      .createTextOutput(JSON.stringify(wishes))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify([]))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME) || SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    let data;
    if (e.postData && e.postData.contents) {
      data = JSON.parse(e.postData.contents);
    } else {
      data = e.parameter;
    }

    const name = data.name || '';
    const status = data.status || 'Hadir';
    const message = data.message || '';
    const timestamp = new Date();

    if (name && message) {
      sheet.appendRow([name, status, message, timestamp]);
    }

    return ContentService
      .createTextOutput(JSON.stringify({ status: "success" }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function formatDate(dateVal) {
  try {
    const d = new Date(dateVal);
    const now = new Date();
    const diffMs = now - d;
    const diffMinutes = Math.floor(diffMs / (1000 * 60));
    if (diffMinutes < 1) return "Baru saja";
    if (diffMinutes < 60) return diffMinutes + " menit yang lalu";
    const diffHours = Math.floor(diffMinutes / 60);
    if (diffHours < 24) return diffHours + " jam yang lalu";
    return d.toLocaleDateString("id-ID", { day: "numeric", month: "short", year: "numeric" });
  } catch (e) {
    return "Baru saja";
  }
}
```

3. Klik ikon **Simpan** (ikon disket atau `Ctrl + S`).

---

## Langkah 3: Deploy sebagai Web App
1. Di pojok kanan atas editor Apps Script, klik tombol **Terapkan (Deploy)** > **Deployment baru (New deployment)**.
2. Di bagian kiri (ikon roda gigi), pilih jenis: **Aplikasi web (Web app)**.
3. Atur pengaturannya sebagai berikut:
   - **Deskripsi**: `API Buku Tamu Diva Dimas`
   - **Jalankan sebagai (Execute as)**: **Saya (email Anda)**
   - **Siapa yang memiliki akses (Who has access)**: **Siapa saja (Anyone)** *(Penting agar tamu tanpa login Google bisa mengisi)*
4. Klik **Terapkan (Deploy)**.
5. Jika muncul permintaan izin (*Authorization required*):
   - Klik **Tinjau Izin (Review permissions)**.
   - Pilih akun Google Anda.
   - Klik **Lanjutan (Advanced)** > Klik **Buka Buku Tamu (tidak aman) / Go to project (unsafe)**.
   - Klik **Izinkan (Allow)**.
6. Salin **URL Aplikasi Web (Web app URL)** yang diberikan.
   - Format URL contoh: `https://script.google.com/macros/s/AKfycbx.../exec`

---

## Langkah 4: Masukkan URL ke Website
1. Buka file `index.html`.
2. Cari baris:
   ```javascript
   const WISHES_API_URL = "";
   ```
3. Masukkan URL Web App yang tadi disalin di dalam tanda kutip:
   ```javascript
   const WISHES_API_URL = "https://script.google.com/macros/s/AKfycbx.../exec";
   ```
4. Simpan, lakukan commit & push ke GitHub:
   ```bash
   git commit -am "Hubungkan buku tamu live Google Sheets"
   git push origin main
   ```

Selesai! Sekarang semua ucapan selamat dan doa restu akan tersimpan di Google Sheet Anda secara live dan langsung terlihat oleh seluruh tamu undangan lainnya! 🎉
