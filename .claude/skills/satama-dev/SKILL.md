---
name: satama-dev
description: Panduan mengembangkan aplikasi SIM-SATAMA (Bagan Organisasi Satama, satu berkas index.html tanpa build). Gunakan saat menambah fitur, memperbaiki bug, mengubah skema data karyawan/data induk, mengubah bagan, dasbor, impor/ekspor CSV, tampilan, atau menyambungkan penyimpanan ke server/Firebase di repo SIM-SATAMA.
---

# Mengembangkan SIM-SATAMA

Aplikasi HR untuk bagan organisasi PT Satama Corpora Indonesia dan anak perusahaannya.
Seluruh aplikasi ada di **satu berkas `index.html`** (HTML + CSS + JS vanilla, ES5, tanpa framework, tanpa build, tanpa dependensi).
Bahasa antarmuka, pesan, dan nama variabel domain: **Bahasa Indonesia**.

## Aturan utama

1. **Tetap satu berkas.** Jangan menambah bundler, npm, framework, atau library CDN kecuali pengguna memintanya. Satu-satunya sumber luar adalah Google Fonts (Public Sans).
2. **Ikuti gaya kode yang ada:** ES5 (`var`, `function`, tanpa arrow function/`let`/`const` kecuali konstanta yang sudah ada), satu IIFE `'use strict'`, baris padat, nama pendek. Komentar pemisah bagian berbentuk `/* ---------- NAMA ---------- */`.
3. **Selalu escape** teks pengguna yang masuk ke HTML/SVG dengan `esc()`.
4. **Jangan simpan data sensitif:** gaji, NIK, nomor BPJS, rekening bank (UU PDP 27/2022). Tolak atau tanyakan dulu jika diminta.
5. **Jangan commit data karyawan asli.** `.gitignore` memblokir `*.csv` kecuali `template-karyawan.csv`.
6. Logo dan contoh data di-*inline* (base64 / `SAMPLE`). `logo.png` hanya salinan; logo di halaman berasal dari data URI.
7. Teks UI singkat, kalimat lengkap, tanpa istilah teknis untuk pengguna HR.

## Peta berkas `index.html`

Cari dengan `grep -n` karena baris bisa sangat panjang (gunakan `cut -c1-300` saat membaca).

| Bagian | Penanda |
|---|---|
| Token warna & tema (terang/gelap) | `:root{` di `<style>`, variabel `--accent`, `--d0..--d7` (warna kategori) |
| Header, tombol ekspor | `<header class="top">`, `#bImport #bCsv #bSvg #bPng` |
| Navigasi 5 tampilan | `.nav` tombol `data-view` = `bagan`, `karyawan`, `setup`, `dasbor`, `surat` |
| Tampilan | `<section class="view" id="v-bagan|v-karyawan|v-setup|v-dasbor|v-surat">` |
| Contoh data | `var SAMPLE =` (tiap record `contoh:true`) |
| Konstanta | `COLS`, `LIGHT` (warna ekspor), `VARS`, `FONT`, ukuran kartu `W,H,PAD` |
| State global | `var S = {...}` (data `S.d`, tampilan, filter, zoom), indeks `M` |
| Helper | `esc num uid lc clean toast fdate pdate today dayDiff reindex sortedUnits nm` |
| Lapisan data | `/* ---------- data layer ---------- */`: `LocalStore`, `DbStore`, `attach`, `initStore`, `save`, `guard`, `runBatch` |
| Pohon & tata letak bagan | `makeTree`, `descendants`, `layout`, `svgMarkup`, `renderBagan` |
| Formulir karyawan | `cur`, `syncForm`, `genId`, `fillForm`, `readEmp`, `onSubmit`, `onDeleteEmp` |
| Setup data induk | `/* ---------- SETUP ---------- */`: `TABS`, `SF` (definisi kolom formulir), `usage`, `renderSetupForm`, `onSetupSubmit`, `deleteSetup` |
| Dasbor | `renderDash`, `bars`, `countBy` |
| CSV | `toCSV`, `templateCSV`, `parseCSV`, `planImport`, `finishImport`, `onFile` |
| Ekspor gambar/berkas | `exportSvg`, `exportPng`, `saveFile`, `directDownload` |
| Foto & SK (IndexedDB) | `fdb ftx fget fput fdel fkeys loadFotos avaHtml kecilkan`, form: `renderFotoPrev renderSkInfo`, perubahan tertunda di `S.pf` |
| Kontrak & pengingat | `addBulan`, `hitungAkhir`, `kirimPengingat` (mailto), panel "Pengingat kontrak" di `renderDash` |
| ZIP & cadangan | `crc32 zipBlob unzip` (tanpa kompresi), `berkasMenu unduhCadangan pulihkanCadangan` |
| Buat Surat | `/* ---------- SURAT ---------- */`: `SURAT` (template), `suCtx suNomor suFill suChanged renderSurat onSuSubmit`, `blokIsi letterDoc cetakSurat docxBlob unduhDocx` |
| Pengaturan | `CFG0`, `cfg()`, `renderCfg`, `onCfgSubmit` (dokumen `pengaturan/umum`) |
| Event | `wire()` (delegasi klik lewat atribut `data-act`) |
| Start | baris akhir: `wire();setStatus();...initStore();` |

## Model data

Delapan koleksi (`COLS`), masing-masing array objek dengan `id` (dibuat `uid()`):

- `units`: `nama, kode` (awalan ID karyawan, 2–6 huruf), `jenis` (`Induk`/`Anak`), `indukId`, `urutan`
- `departemen`: `nama, unitId`
- `divisi`: `nama, unitId, deptId`
- `jabatan`: `nama, unitId, deptId, divisiId, kuota`
- `status`: `nama, kategori` (`Tetap`/`Kontrak`/`Lainnya`; `Kontrak` memunculkan tanggal kontrak berakhir)
- `employees`: `idKaryawan` (`KODE-TAHUN-NOMOR`, unik lintas unit), `nama, unitId, deptId, divisiId, jabatanId, golongan` (`1`–`5` atau `Organ`), `statusId, atasanId` (harus satu unit), `urutan, mulai, akhir` (`YYYY-MM-DD`), `contoh`; kontrak: `durasi` (bulan), `ingat` (`''|30|60|90`), `ingatAtasan`, `ingatDibuka`; pribadi: `tempatLahir, tglLahir, alamat, email, hp`; berkas: `foto` (bool), `skNama` (isi berkas di IndexedDB `satama-berkas`, kunci `foto:<id>`/`sk:<id>`)
- `surat`: `jenis` (kode template), `empId, nomor, tanggal, judul, isi, ttdNama, ttdJabatan, diubah`
- `pengaturan`: satu dokumen `id:'umum'`: `emailHR, alamat, kontak, kota, ttdNama, ttdJabatan, formatNomor`

Record contoh diberi `contoh:true` agar bisa dihapus lewat `clearExamples`.

### Menambah kolom baru

- **Data induk:** tambahkan definisi di `SF[koleksi]` (`k` kunci, `l` label, `t` = `text|num|sel|ref`, `o` opsi, `ref`/`par` untuk relasi, `hint`, `full`). Form dan tabel Setup mengikuti otomatis; cek `parentCols` dan `renderSetupTable` bila kolom perlu tampil di tabel.
- **Karyawan:** tambah input di `#v-karyawan`, baca di `readEmp`, isi di `fillForm`, validasi di `onSubmit`, lalu perbarui **keempat** bagian CSV: `toCSV`, `templateCSV`, `planImport` (alias header di `ix([...])`), dan `template-karyawan.csv`. Perbarui tabel format CSV di `README.md`.
- Jika kolom mempengaruhi hapus data induk, perbarui `usage()`.

## Alur data

- Semua tulis lewat `save(koleksi, obj)` / `store.remove(koleksi, id)`, dibungkus `guard(...)` (status sibuk + toast galat). Banyak tulis: `runBatch(list, fn)` (4 paralel).
- `store.subscribe` mengisi `S.d[c]` lalu `renderAll()` → `reindex()` (isi `M[c][id]`) → `renderView()`.
- Dalam mode lokal, panggil `renderAll()` setelah tulis bila perlu (pola `if(S.mode==='local')renderAll()`).
- `STANDALONE = true` untuk build GitHub Pages (localStorage, kunci `satama-org-v2`). `initStore` mencoba `window.claude.use('db')` dulu (versi claude.ai), lalu jatuh ke `LocalStore`.
- **Mengganti backend** (mis. Firebase/Firestore): buat objek store baru dengan antarmuka yang sama dengan `DbStore`: `subscribe(c, f, onErr)`, `save(c, o)`, `remove(c, id)`, `canWrite()`, lalu `attach(store, true)` di `initStore`. Jangan ubah kode di luar lapisan data. Tambahkan login dan aturan keamanan sebelum data asli disimpan di server.

## Aturan bisnis yang harus dijaga

- ID karyawan otomatis `KODE-TAHUN-NOMOR` (`genId`), unik di semua unit.
- Atasan harus dari unit yang sama; hindari siklus atasan (lihat `descendants`).
- Posisi lowong = `jabatan.kuota` − jumlah karyawan pada jabatan itu (`filled`).
- Pengingat dasbor: kontrak PKWT berakhir ≤90 hari, batas PKWT 5 tahun (PP 35/2021), masa percobaan 3 bulan.
- Data induk yang masih dipakai tidak boleh dihapus (`usage`).
- Akhir kontrak = tanggal bergabung + durasi bulan − 1 hari (`addBulan`). Kolom kontrak hanya untuk status kategori Kontrak.
- Email pengingat hanya lewat `mailto:` (tanpa server). Penerima: email HR + atasan bila `ingatAtasan`.
- Foto dan SK tidak masuk CSV; cadangannya lewat ZIP. Hapus berkas IndexedDB saat karyawan dihapus.
- `.docx` dibuat sendiri (ZIP + WordprocessingML). Jaga urutan elemen sesuai skema (mis. `pPr`: pBdr, tabs, spacing, ind, jc; `rPr`: b, sz, szCs, u).

## Tampilan

- Warna hanya dari token CSS di `:root`; setiap token baru harus punya nilai gelap di **dua** tempat: `@media (prefers-color-scheme: dark)` dan `:root[data-theme="dark"]`.
- Bagan di layar memakai `VARS` (variabel CSS); ekspor SVG/PNG memakai `LIGHT` (hex tetap). Jika menambah warna di bagan, tambahkan ke keduanya.
- Pertahankan tata letak responsif (lebar ponsel) dan atribut aksesibilitas (`role`, `aria-*`, `label for`).

## Menjalankan & memeriksa

- Buka `index.html` langsung di browser (klik dua kali), atau dari folder repo: `python3 -m http.server 8080`.
- Setelah perubahan, periksa di browser:
  1. Tidak ada galat di konsol.
  2. Klik "Muat contoh Satama" (data `SAMPLE`) → bagan muncul.
  3. Uji alur yang diubah di keempat tab: Bagan, Karyawan, Setup, Dasbor.
  4. Jika menyentuh CSV: ekspor → impor ulang hasil ekspor → data sama.
  5. Cek mode gelap dan lebar ponsel bila mengubah tampilan.
- Untuk uji dengan data bersih: hapus kunci `satama-org-v2` di localStorage.

## Git

- Remote: `https://github.com/yuritebe/SIM-SATAMA.git`, branch `main`, ditayangkan lewat GitHub Pages dari root.
- Commit hanya bila pengguna meminta. Jangan commit berkas CSV berisi data asli.
