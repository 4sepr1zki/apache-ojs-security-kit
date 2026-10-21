# 🛡️ Apache / LiteSpeed OJS Security & Performance Hardening Kit

Koleksi konfigurasi `.htaccess` tingkat lanjut khusus untuk server pengelola **Open Journal Systems (OJS 3.x / 2.4.x)** yang menggunakan **Apache Web Server** atau **LiteSpeed Web Server** (sangat cocok untuk **cPanel, WHM, Plesk, DirectAdmin, atau VPS LAMP Stack**).

---

## 📁 Struktur Berkas

```text
├── .htaccess       # File rule Apache siap pakai (Mod_Rewrite & Mod_Headers)
└── README.md       # Panduan instalasi dan dokumentasi teknis
```

---

## 🚀 Fitur Utama & Kegunaan

### 1. 🚷 Pemblokiran URL Parasit Judol `pdfJsViewer` (HTTP 410 Gone)
* **Masalah:** Spammer dan sindikat judi online sering memanfaatkan file `viewer.html` bawaan plugin PDF OJS untuk menyisipkan parameter spam (`?id=...`, `?signOut=...`, `?login=...`) sehingga ribuan URL judi terindeks di mesin pencari atas nama domain jurnal Anda.
* **Solusi di Apache:**
  - Menggunakan flag `[G,L]` (**G = Gone / HTTP 410**). Kode 410 memberi sinyal tegas kepada Googlebot bahwa halaman tersebut sengaja dimusnahkan secara permanen, mempercepat proses *de-indexing*.
  - Menyuntikkan header `X-Robots-Tag: noindex, nofollow, noarchive` via modul `mod_headers`.
  - Hanya mengizinkan parameter `file` resmi yang mengarah ke endpoint `download` naskah jurnal OJS.

### 2. 🤖 Bot Blocker & Aggressive Scraper Defense (HTTP 403 Forbidden)
* **Masalah:** Bot scraper agresif (Ahrefs, Semrush, Bytespider, crawler AI tanpa izin) dan scanner tools (sqlmap, fuzzers) sering membebani Apache/PHP hingga memori server habis.
* **Solusi di Apache:** Memblokir User-Agent jahat dan langsung mengembalikan respon `403 Forbidden` (`[F,L]`) sebelum diteruskan ke PHP/MySQL.
* **Whitelisted Bots:** Bot penting untuk indeksasi jurnal (Googlebot, Bingbot, `SciRepID`, `curl`, `GPTBot`, `ClaudeBot`) tetap diizinkan.

### 3. 🛑 Anti Similarity Calculation Trap (HTTP 403 Forbidden)
* **Masalah:** Query `articlesBySimilarityPage` di OJS sering kali memicu kalkulasi matriks kesamaan artikel yang sangat berat di database MySQL jika diserang bot secara berulang.
* **Solusi di Apache:** Menolak query tersebut seketika dengan `[F,L]`.

---

## 🛠️ Panduan Pemasangan (Tutorial Langkah demi Langkah)

### Skenario 1: Pada cPanel / Shared Hosting / LiteSpeed
1. Login ke **cPanel** akun hosting Anda.
2. Buka menu **File Manager** $\rightarrow$ buka folder root OJS (biasanya di `public_html` atau direktori sub-domain).
3. Pastikan fitur *Show Hidden Files (dotfiles)* diaktifkan di pengaturan File Manager.
4. Buka file `.htaccess` (jika belum ada, klik tombol **+ File** dan beri nama `.htaccess`).
5. Salin dan tempelkan seluruh isi berkas [`.htaccess`](.htaccess) dari repo ini ke bagian paling atas file `.htaccess` Anda.
6. Simpan perubahan. Konfigurasi **langsung aktif otomatis** tanpa perlu restart hosting!

### Skenario 2: Pada Server VPS / Dedicated Server (Ubuntu / Debian / CentOS LAMP)
1. Salin file `.htaccess` ke root direktori web OJS Anda:
   ```bash
   cp .htaccess /var/www/html/.htaccess
   # Sesuaikan kepemilikan user web server:
   chown www-data:www-data /var/www/html/.htaccess
   ```
2. Pastikan modul `rewrite` dan `headers` di Apache sudah aktif:
   ```bash
   sudo a2enmod rewrite headers
   ```
3. Pastikan virtual host Apache Anda mengizinkan `.htaccess` (`AllowOverride All`):
   ```apache
   <Directory /var/www/html>
       Options Indexes FollowSymLinks
       AllowOverride All
       Require all granted
   </Directory>
   ```
4. Reload service Apache:
   ```bash
   sudo systemctl reload apache2
   # atau pada CentOS / RHEL / AlmaLinux:
   sudo systemctl reload httpd
   ```

---

## 🧪 Cara Pengujian (Verifikasi Hasil)

Jalankan perintah ini di terminal (atau via [ReqBin / Postman]):

1. **Uji Coba Rule 410 Gone (Parasit Judol):**
   ```bash
   curl -sI "https://domain-jurnal.com/plugins/generic/pdfJsViewer/pdf.js/web/viewer.html?id=slot-gacor"
   ```
   *Ekspektasi Hasil:*
   ```http
   HTTP/1.1 410 Gone
   X-Robots-Tag: noindex, nofollow, noarchive
   ```

2. **Uji Coba Blokir Bot:**
   ```bash
   curl -sI -A "SemrushBot" "https://domain-jurnal.com/index.php"
   ```
   *Ekspektasi Hasil:* Mengembalikan `HTTP/1.1 403 Forbidden`.

3. **Uji Coba Query Trap:**
   ```bash
   curl -sI "https://domain-jurnal.com/index.php?articlesBySimilarityPage=1"
   ```
   *Ekspektasi Hasil:* Mengembalikan `HTTP/1.1 403 Forbidden`.

---

## 📄 Lisensi
Konfigurasi ini bebas digunakan untuk pengamanan server jurnal internal institusi dan akademik.
