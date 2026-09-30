# Instalasi, setup, dan penggunaan JSxtrack

JSxtrack menangkap respons HTML, JavaScript, dan source map yang melewati Caido, lalu menyimpan salinannya di komputer Anda. Program ini terdiri dari receiver/CLI Rust dan plugin Caido. Jalankan JSxtrack hanya pada sistem yang Anda berwenang untuk uji.

## 1. Yang perlu disiapkan

- Windows 10/11, Linux, atau macOS.
- Rust stable dan Cargo. Versi Rust harus mendukung edition 2024 (Rust 1.85 atau lebih baru).
- Windows: Rust toolchain MSVC dan Visual Studio Build Tools dengan workload **Desktop development with C++**. Linux: compiler C/C++ seperti `gcc` dan `build-essential`. macOS: Xcode Command Line Tools.
- Node.js 20 atau lebih baru dan pnpm 10 (Corepack bisa menjalankan pnpm tanpa instalasi global).
- Caido versi desktop, serta browser yang mengirim lalu lintas melalui proxy Caido.

Untuk memeriksa alat yang sudah terpasang:

```text
rustc --version
cargo --version
node --version
corepack pnpm@10 --version
```

Di Windows, buka PowerShell baru setelah memasang Rust agar `cargo` dan `rustc` tersedia di `PATH`.

## 2. Buka source code

Gunakan folder proyek yang disertakan bersama ekstensi ini. Buka terminal di folder yang berisi `Cargo.toml`, `crates/`, dan `caido-plugin/` (pada arsip yang Anda berikan, folder tersebut adalah `jxstrack/`). Jika Anda memakai salinan repository lain, pastikan folder `caido-plugin/` memang ada sebelum mengikuti langkah build di bawah.

Repositori proyek JSxtrack: [skysenz/JSxtrack](https://github.com/skysenz/JSxtrack). Direktori caido-plugin/ berisi extension Caido untuk proyek ini.

## 3. Pasang command `jsxtrack`

Dari folder utama proyek, build dan pasang CLI:

```bash
cargo install --path crates/jxstrack-cli
```

Untuk memperbarui versi yang sudah terpasang:

```bash
cargo install --path crates/jxstrack-cli --force
```

Pastikan folder binary Cargo masuk ke `PATH`:

- Windows: `%USERPROFILE%\.cargo\bin`
- Linux/macOS: `$HOME/.cargo/bin`

Tutup dan buka kembali terminal jika command belum ditemukan. Periksa instalasi dengan:

```text
jsxtrack --help
jsxtrack serve --help
```

Jalankan `jsxtrack` tanpa argumen untuk membuka prompt interaktif. Prompt menampilkan daftar tools:

```text
Available tools:
  serve [OPTIONS]   Start the local receiver and capture assets from Caido
  help              Show this tool list
  exit              Close the prompt
```

Ketik `serve --project default` di prompt untuk mulai menerima aset. Anda juga dapat langsung menjalankan server dari terminal tanpa membuka prompt:

```bash
jsxtrack serve --project default
```

Biarkan terminal server tetap terbuka saat pengumpulan berlangsung. Untuk menghentikannya tekan `Ctrl+C`.

## 4. Build dan pasang plugin Caido

Buka terminal lain, lalu dari folder utama proyek jalankan:

```bash
cd caido-plugin
corepack pnpm@10 install --frozen-lockfile
corepack pnpm@10 typecheck
corepack pnpm@10 build
```

Jika `corepack` tidak tersedia, pasang pnpm 10 dengan `npm install --global pnpm@10`, lalu jalankan `pnpm install --frozen-lockfile`, `pnpm typecheck`, dan `pnpm build` dari folder `caido-plugin`.

Build membuat arsip instalasi di:

```text
caido-plugin/dist/plugin_package.zip
```

Di Caido, buka **Plugins**, pilih **Install Package**, lalu pilih file `plugin_package.zip`. Aktifkan komponen backend dan frontend JSxtrack pada daftar plugin terpasang. Nama menu dapat berbeda sedikit antarversi Caido. Panduan resmi Caido: [memasang plugin](https://docs.caido.io/app/guides/plugins_installing) dan [mengaktifkan komponennya](https://docs.caido.io/app/guides/plugins_managing).

## 5. Jalankan receiver dan hubungkan Caido

Receiver bawaan menggunakan `http://127.0.0.1:3333`. Port ini milik JSxtrack dan terpisah dari port antarmuka maupun listener proxy Caido.

Di terminal pertama, jalankan:

```bash
jsxtrack serve --project default
```

Di PowerShell lain, periksa receiver:

```powershell
Invoke-RestMethod http://127.0.0.1:3333/health
```

Hasilnya harus berisi `status: ok`. Di Linux/macOS, periksa dengan:

```bash
curl http://127.0.0.1:3333/health
```

Pastikan browser menggunakan proxy Caido. Untuk situs HTTPS, browser perlu mempercayai CA certificate Caido. Saat plugin dimuat, lihat **Caido Backend Console**: plugin mencatat apakah receiver dapat dijangkau. Jika port receiver Anda berbeda, ubah `JSXTRACK_SERVER_URL` di `caido-plugin/packages/backend/src/index.ts`, lalu build dan pasang ulang plugin.

## 6. Tangkap file JavaScript

1. Pastikan server `jsxtrack serve` masih berjalan dan health check berhasil.
2. Pastikan plugin **JSxtrack** dan kedua komponennya aktif di Caido.
3. Buka situs target yang termasuk izin pengujian Anda melalui browser yang memakai proxy Caido.
4. Jelajahi atau muat ulang halaman. Plugin meneruskan respons HTML, JavaScript, dan source map yang dikenali secara otomatis.
5. Untuk mengirim ulang respons yang sudah tersimpan di **HTTP History**, pilih request, klik kanan, lalu pilih **Send selected request(s) to JSxtrack**.

Untuk memastikan JavaScript baru diminta dari server, nonaktifkan cache browser lalu muat ulang halaman. Respons `304 Not Modified` biasanya tidak memiliki body file untuk disimpan. Event plugin Caido berjalan untuk respons yang diproksikan; detailnya ada di [dokumentasi event backend Caido](https://developer.caido.io/plugins/guides/backend_events.html).

## 7. Uji penyimpanan tanpa Caido

Uji ini memastikan receiver aktif dan dapat menyimpan body JavaScript. Jalankan saat `jsxtrack serve` masih berjalan. Di PowerShell:

```powershell
$payload = @{
  request = @{
    method = 'GET'
    url = 'https://example.test/assets/test.js'
    headers = @{}
  }
  response = @{
    status = 200
    headers = @{ 'Content-Type' = 'application/javascript' }
    body = 'console.log("JSxtrack smoke test")'
  }
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
  -Uri 'http://127.0.0.1:3333/ingest' `
  -Method Post `
  -ContentType 'application/json' `
  -Body $payload

Test-Path "$HOME\jxstrack\default\original\example.test\assets\test.js"
```

`Test-Path` harus menghasilkan `True`. Request smoke test ini hanya menguji receiver dan penyimpanan lokal; untuk menguji plugin Caido, lanjutkan dengan membuka situs target melalui proxy atau gunakan menu klik kanan di HTTP History.

## 8. Lokasi hasil

Project `default` tersimpan di:

- Windows: `%USERPROFILE%\jxstrack\default`
- Linux/macOS: `~/jxstrack/default`

Folder utama yang dibuat:

```text
original/                    body HTML dan JavaScript asli
beautified/                  versi HTML dan JavaScript yang diformat
optimized/                   JavaScript untuk analisis
preanalysis/                 ringkasan analisis awal
sourcemaps/raw/              source map yang diterima atau diunduh
sourcemaps/reversed/         file sumber yang dipulihkan dari source map
sourcemaps/reports/          laporan pemetaan source map
analysis/                    hasil analisis per file
metadata/assets.jsonl        indeks aset
metadata/relationships.jsonl relasi halaman dan chunk
metadata/findings.jsonl      temuan analisis
```

Untuk memisahkan hasil per target, gunakan nama project yang berbeda:

```bash
jsxtrack serve --project nama-target
```

Untuk menyimpan di lokasi lain, gunakan `--output <folder>`; JSxtrack membuat subfolder project di dalam folder tersebut.

## 9. Opsi receiver

Contoh menjalankan receiver hanya untuk URL yang memuat `example.com`:

```bash
jsxtrack serve \
  --host 127.0.0.1 \
  --port 3333 \
  --project nama-target \
  --scope '*example.com*' \
  --rate-per-second 2 \
  --fetch-concurrency 5
```

Opsi yang umum dipakai:

- `--host` dan `--port`: alamat receiver (default `127.0.0.1:3333`).
- `--project`: nama folder project (default `default`).
- `--output`: folder dasar hasil; folder project dibuat di bawahnya.
- `--scope <pola>`: hanya proses URL yang cocok; opsi ini dapat dipakai lebih dari sekali.
- `--rate-per-second <angka>`: batas fetch chunk dan source map per detik.
- `--rate-per-minute <angka>`: batas fetch per menit; `0` menonaktifkan batas tambahan ini.
- `--fetch-concurrency <angka>`: jumlah fetch lanjutan bersamaan.
- `--max-body-bytes <angka>`: batas ukuran body respons.
- `--debug`: tampilkan log debug tambahan.

Lihat semua opsi dengan `jsxtrack serve --help`.

## Troubleshooting

- **`jsxtrack` tidak ditemukan:** periksa bahwa `%USERPROFILE%\.cargo\bin` (Windows) atau `$HOME/.cargo/bin` (Linux/macOS) ada di `PATH`, lalu buka terminal baru.
- **Health check gagal:** pastikan server berjalan di port `3333`. Pastikan juga port tersebut tidak dipakai program lain.
- **Plugin menulis “Cannot reach”:** mulai `jsxtrack serve` terlebih dahulu. Periksa bahwa plugin diarahkan ke URL receiver yang benar dan receiver berjalan pada komputer yang sama dengan Caido.
- **Tidak ada file masuk:** pastikan backend plugin aktif, browser benar-benar melalui proxy Caido, halaman tidak memakai respons cache/304, dan respons memiliki body. Periksa **Caido Backend Console** untuk error ingest.
- **Kirim manual tidak menemukan aset:** buka response di HTTP History dan pastikan body tersimpan. Pilih request yang mempunyai respons, lalu jalankan menu **Send selected request(s) to JSxtrack**.
- **HTTPS tidak muncul di Caido:** pastikan browser mempercayai CA certificate Caido dan mengirim lalu lintas melalui proxy-nya.
- **Tidak ada file hasil pemulihan:** server mungkin tidak menemukan source map, atau map tidak memiliki `sourcesContent`.
- **Plugin ZIP tidak ada:** jalankan perintah build dari folder `caido-plugin`, lalu pilih `caido-plugin/dist/plugin_package.zip`.
