# PWA Engineering • Elektromedis

## Deploy ke GitHub Pages
1. Buat repository baru, upload semua isi folder ini (index.html, manifest.webmanifest, sw.js, folder icons).
2. Settings → Pages → Source: `Deploy from a branch` → branch `main` / folder `/ (root)` → Save.
3. Buka `https://USERNAME.github.io/NAMA-REPO/` (wajib HTTPS, sudah otomatis).

## WAJIB: izinkan Apps Script ditampilkan di dalam PWA
Di Code.gs, pada fungsi `doGet`, tambahkan `setXFrameOptionsMode`:

```js
function doGet(e) {
  // Endpoint opsional untuk bunyi notifikasi (lihat bawah)
  if (e && e.parameter && e.parameter.action === 'count') {
    const sh = SpreadsheetApp.getActive().getSheetByName('Laporan'); // ganti nama sheet
    return ContentService.createTextOutput(
      JSON.stringify({ count: sh.getLastRow() - 1 })   // -1 jika baris 1 = header
    ).setMimeType(ContentService.MimeType.JSON);
  }
  return HtmlService.createHtmlOutputFromFile('Index')   // sesuaikan nama file HTML Anda
    .setTitle('ENGINEERING • ELEKTROMEDIS')
    .addMetaTag('viewport', 'width=device-width, initial-scale=1, viewport-fit=cover')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}
```
Lalu **Deploy → Manage deployments → Edit → New version → Deploy**. URL `/exec` tidak berubah.
Akses web app harus `Anyone` agar bisa dibuka dari PWA.

## Suara notifikasi laporan
Ada dua cara, bisa dipakai bersamaan.

**A. Saat laporan dikirim dari perangkat ini** — di HTML Apps Script, setelah laporan sukses tersimpan:
```js
window.top.postMessage({ type: 'laporan-baru', title: 'Laporan terkirim', body: 'Laporan tersimpan' }, '*');
```
(Opsional) beri tahu PWA bahwa halaman sudah termuat: `window.top.postMessage({type:'ready'}, '*');`

**B. Saat laporan dari orang lain masuk** — di `index.html` ubah `POLL_ENABLED: true`.
PWA akan memanggil `?action=count` tiap 30 detik (endpoint di atas) dan berbunyi saat jumlah laporan bertambah.

## Catatan penting
- Bunyi dan polling hanya jalan saat aplikasi terbuka/di depan. Notifikasi saat aplikasi tertutup butuh Web Push dengan server (FCM/OneSignal), tidak bisa hanya dengan GitHub Pages + Apps Script.
- iOS: PWA harus dipasang lewat Share → Tambah ke Layar Utama (iOS 16.4+ untuk notifikasi). Mode senyap (tombol samping) membisukan suara.
- Android: mode layar penuh aktif setelah dipasang. iOS memakai mode standalone (tanpa bilah Safari).
- Banner pasang otomatis hilang bila aplikasi sudah dipasang atau dibuka dari ikon layar utama.
- Jika mengubah file, naikkan `CACHE = 'elektromedis-v2'` di sw.js agar pengguna menerima versi baru.


## Sidebar & Install (v7)

**Sidebar** dikontrol oleh tombol ☰ di dalam aplikasi (Apps Script). Tidak ada lagi tombol sidebar melayang dari PWA yang menutupi header.
- HP: sidebar berupa drawer. Tutup dengan ketuk area gelap, tombol ✕, pilih menu, tombol Esc, atau tombol Kembali (Android).
- Desktop: ☰ menyembunyikan/menampilkan sidebar, pilihan diingat.
- Jembatan PWA ↔ Apps Script memakai `window.top.postMessage` (Apps Script berjalan di iframe bertingkat, `window.parent` hanya halaman pembungkus Google) dan PWA meneruskan perintah ke semua frame anak.

**Install**: tombol "Install Aplikasi" tampil di tengah-bawah selama belum terpasang dan belum login. Setelah login, tersedia sebagai menu "Pasang Aplikasi" di sidebar. Android/Chrome memakai `beforeinstallprompt`; iOS menampilkan petunjuk Tambah ke Layar Utama.

**Update**: ganti isi `Index.html` di proyek Apps Script dengan `APPS-SCRIPT-SYNC/Index.html` lalu Deploy → New version. Upload `index.html` dan `sw.js` ke GitHub Pages (cache sudah dinaikkan ke v7).
