# Bagan Organisasi Satama

Aplikasi web satu halaman untuk membuat bagan struktur organisasi PT Satama Corpora Indonesia dan unit kerjanya. Data karyawan diisi lewat formulir, bagan tersusun otomatis.

Tidak ada proses build dan tidak ada dependensi. Seluruh aplikasi ada di satu berkas, `index.html`.

## Fitur

- **Setup data induk:** Unit Kerja, Departemen, Divisi, Jabatan (dengan kuota), dan Status Kerja.
- **Formulir karyawan berjenjang:** Unit → Departemen → Divisi → Jabatan. Status Kerja diambil dari Setup.
- **ID Karyawan:** dibuat otomatis (`KODE-TAHUN-NOMOR`, misalnya `SAT-2026-001`), unik di semua unit.
- **Bagan per Unit Kerja** dan **bagan grup** (induk dan anak perusahaan).
- **Dasbor:** jumlah karyawan per unit, departemen, status, dan golongan; posisi lowong; pengingat kontrak PKWT, batas 5 tahun (PP 35/2021), dan masa percobaan 3 bulan.
- **Impor dan ekspor CSV**, ekspor bagan ke PNG dan SVG.

## Menjalankan di komputer

Buka `index.html` dengan dua kali klik. Untuk menjalankan lewat server lokal:

```bash
python3 -m http.server 8080
# buka http://localhost:8080
```

## Menayangkan lewat GitHub Pages

1. Buka repositori di GitHub, lalu **Settings → Pages**.
2. Di **Build and deployment**, pilih **Source: Deploy from a branch**.
3. Pilih branch `main` dan folder `/ (root)`, lalu klik **Save**.
4. Tunggu 1 sampai 2 menit. Alamatnya tampil di halaman yang sama, biasanya `https://<akun>.github.io/<nama-repo>/`.

Catatan: GitHub Pages dari repositori privat hanya tersedia pada paket berbayar GitHub. Repositori publik berarti kode aplikasinya bisa dilihat siapa saja. Lihat bagian keamanan di bawah.

## Penting: data tersimpan di browser

Versi di repositori ini menyimpan data di `localStorage` browser masing-masing pengguna.

- Data **tidak dibagikan** antar pengguna atau antar komputer.
- Data hilang jika data situs browser dihapus. **Ekspor CSV secara berkala** sebagai cadangan.
- Untuk dipakai bersama oleh tim HR, gunakan versi yang berjalan di claude.ai (data tersimpan di aplikasi dan bisa dibagikan lewat menu Share), atau sambungkan aplikasi ke database server (lihat bagian pengembangan lanjutan).

## Keamanan dan data pribadi

- Repositori ini hanya berisi **data contoh**. Jangan pernah meng-commit data karyawan asli. `.gitignore` sudah memblokir berkas `.csv` kecuali folder `contoh/`.
- Halaman ini tidak memiliki login. Jangan tayangkan di alamat publik jika browser yang dipakai menyimpan data karyawan asli. Data tidak dikirim ke server, tetapi siapa pun yang memakai browser yang sama dapat melihatnya.
- Aplikasi sengaja tidak menyimpan gaji, NIK, nomor BPJS, atau rekening bank. Data pribadi diatur UU Pelindungan Data Pribadi (UU 27/2022).

## Format CSV

Unduh templatenya dari aplikasi (tombol **Template CSV**) atau lihat `contoh/template-karyawan.csv`.

| Kolom | Keterangan |
|---|---|
| ID Karyawan | Opsional. Jika kosong, dibuat otomatis. ID yang sama dengan data yang ada akan memperbarui data itu. |
| Nama, Unit Kerja, Jabatan | Wajib |
| Departemen, Divisi | Jika kosong, diisi "Umum" atau sama dengan departemen |
| Golongan | 1 sampai 5, atau `Organ` |
| Status Kerja | Contoh: PKWTT, PKWT |
| ID Atasan | ID Karyawan atasan, harus dalam unit yang sama |
| Tanggal Bergabung, Kontrak Berakhir | Format `YYYY-MM-DD` |

Unit, departemen, divisi, jabatan, dan status yang belum ada dibuat otomatis saat impor.

## Pengembangan lanjutan

Rencana yang disarankan jika aplikasi akan dipakai bersama secara permanen di luar claude.ai:

1. Ganti lapisan penyimpanan (`LocalStore` di dalam `index.html`) dengan database server, misalnya Firebase Firestore atau Supabase, lengkap dengan login dan pembagian peran (HR admin dan pelihat).
2. Tambahkan riwayat mutasi dan promosi serta log perubahan.
3. Hubungkan golongan dengan skala gaji pada Kebijakan Remunerasi, dengan data gaji disimpan terpisah dan hanya bisa dilihat Direktur dan HR.

## Struktur repositori

```
index.html                    aplikasi (HTML, CSS, JavaScript dalam satu berkas)
assets/logo.png               logo Satama
contoh/template-karyawan.csv  contoh format impor
.nojekyll                     agar GitHub Pages menyajikan berkas apa adanya
.gitignore                    memblokir berkas data karyawan asli
README.md                     dokumen ini
```
