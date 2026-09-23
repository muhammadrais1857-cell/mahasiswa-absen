# Presensi Kelas - QR Code

Aplikasi presensi kelas berbasis web yang membaca QR Code mahasiswa lewat kamera, lalu mencatat kehadiran ke **Google Sheets** melalui backend **Google Apps Script**. Frontend-nya adalah satu file HTML statis (`index.html`) sehingga bisa langsung di-host gratis di **GitHub Pages**.

## Struktur Repo

```
├── index.html   # Frontend (UI + scanner QR + logika presensi)
├── Code.gs      # Backend Google Apps Script (API ke Google Sheets)
└── README.md
```

## Cara Kerja Singkat

- `index.html` memindai QR Code (berisi NIM mahasiswa) menggunakan library `html5-qrcode`.
- Data mahasiswa & rekap kehadiran diambil/disimpan lewat `CONFIG.API_URL` (Web App Apps Script) yang membaca/menulis ke Google Sheets.
- Jika koneksi ke server terputus, presensi tetap dicatat sementara di `localStorage` (mode offline) dan bisa disinkronkan lagi saat online.

## Tahap 1 — Tampilkan dulu di GitHub Pages (tanpa Google Sheets)

Frontend bisa langsung online dulu tanpa perlu setup Google Sheets/Apps Script. Kamera & tampilan UI sudah jalan; hanya bagian data mahasiswa/presensi yang nanti akan tampil "offline" sampai Tahap 2 dikerjakan — itu wajar.

1. Push `index.html` dan `README.md` ke repo GitHub:
   ```bash
   git init
   git add .
   git commit -m "Presensi kelas QR code"
   git branch -M main
   git remote add origin https://github.com/USERNAME/NAMA_REPO.git
   git push -u origin main
   ```
2. Di repo GitHub, buka **Settings > Pages**.
3. Pada **Source**, pilih branch `main` dan folder `/ (root)`, lalu Save.
4. Tunggu 1-2 menit — GitHub akan memberi URL seperti `https://USERNAME.github.io/NAMA_REPO/`.
5. Buka URL tersebut dari HP/laptop yang punya kamera. Karena GitHub Pages otomatis pakai HTTPS, akses kamera browser (`getUserMedia`) akan berfungsi normal.

## Tahap 2 — Sambungkan ke Google Sheets (kerjakan belakangan)

Kalau tampilannya sudah muncul dan Anda siap mengaktifkan penyimpanan data sungguhan:

1. Buat Google Spreadsheet baru, lalu buat 2 sheet dengan nama persis:
   - **Mahasiswa** — header baris pertama: `nim | nama | kelas | jurusan`, lalu isi data mahasiswa di bawahnya.
   - **Presensi** — header baris pertama: `tanggal | nim | nama | kelas | jurusan | mataKuliah | status | waktu` (baris data akan terisi otomatis oleh script).
2. Di spreadsheet tersebut, buka **Extensions > Apps Script**.
3. Hapus isi default, lalu tempel seluruh isi file [`Code.gs`](./Code.gs) dari repo ini.
4. Klik **Deploy > New deployment**:
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Klik **Deploy**, lalu salin **Web app URL** yang muncul (formatnya `https://script.google.com/macros/s/XXXX/exec`).
6. Catat juga **ID Spreadsheet** (bagian di URL sheet antara `/d/` dan `/edit`) beserta `gid` masing-masing sheet.
7. Buka `index.html`, cari bagian `CONFIG` di dalam tag `<script>`, lalu ganti nilainya:
   ```js
   const CONFIG = {
     API_URL: 'GANTI_DENGAN_WEB_APP_URL_ANDA',
     SHEET_URL: 'GANTI_DENGAN_URL_SPREADSHEET_ANDA',
     SHEET_GID_MAHASISWA: 'GID_SHEET_MAHASISWA',
     SHEET_GID_PRESENSI: 'GID_SHEET_PRESENSI',
     ...
   };
   ```
8. Commit & push perubahan `index.html` itu ke repo yang sama — GitHub Pages otomatis update dalam 1-2 menit.

## Menjalankan Lokal (opsional, untuk uji coba)

Karena akses kamera butuh HTTPS atau `localhost`, jangan buka `index.html` langsung dari file explorer (`file://`). Jalankan server lokal sederhana, misalnya:

```bash
python3 -m http.server 8000
```

Lalu buka `http://localhost:8000` di browser.

## Catatan

- QR Code yang dipindai berisi JSON `{ "nim": "..." }` atau teks NIM biasa.
- Tombol scan otomatis mencoba kamera belakang dulu, baru kamera depan jika gagal.
- Data mahasiswa & presensi hari ini di-cache di `localStorage` agar UI tetap responsif saat offline.
