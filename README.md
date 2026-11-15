# abs-guru-alhijrah
abs guru alhijrah 15112026

# Petunjuk Penggunaan - Aplikasi Absensi Guru (revisi 3: klaim banding, poin tanpa keterangan, kalender akademik, kop surat)

Aplikasi terdiri dari satu file `index.html` (HTML + CSS + JS) yang langsung terhubung ke Firebase Firestore, tanpa server tambahan.

## 1. Mencoba dulu (Mode Preview)

Buka `index.html` langsung (atau tempel ke Blogger). Selama `const IS_PREVIEW = true;` aplikasi berjalan di memori sementara tanpa Firebase.

- Admin: `admin@demo.com` / `admin123`
- Guru: `guru1` / `guru123` (punya contoh poin) dan `guru2` / `guru123` (contoh alpha dan izin menunggu)

Kamera dan GPS butuh **HTTPS** (atau `localhost`). Saat uji coba di luar radius sekolah, tombol "Demo: jadikan posisi saya sebagai lokasi sekolah" muncul di halaman Absen.

## 2. Membuat Project Firebase

1. Buka https://console.firebase.google.com lalu **Add project**.
2. **Build > Firestore Database > Create database** (mode produksi, pilih region terdekat, mis. `asia-southeast2`).
3. **Build > Authentication > Get started > Sign-in method > Email/Password > Enable**.
4. **Authentication > Users > Add user**: buat akun admin (email + password). Email ini dipakai login admin.
5. **Project settings (ikon roda gigi) > Your apps > Web (`</>`)**: daftarkan aplikasi web, lalu salin objek `firebaseConfig`.

## 3. Memasang Security Rules

1. Buka **Firestore Database > Rules**.
2. Tempel seluruh isi `firestore.rules` (yang baru, karena ada tambahan aturan untuk mode absen, pulang tanpa masuk, foto profil, dan dokumen kop surat yang hanya bisa dibaca admin).
3. Ganti `GANTI_DENGAN_EMAIL_ADMIN@contoh.com` dengan email admin dari langkah 2.4, **huruf kecil semua**. Untuk beberapa admin, pisahkan dengan koma.
4. Klik **Publish**.

Jika aplikasi menampilkan "Akses ditolak oleh Firestore", hampir selalu karena email admin di rules belum diganti atau Rules belum di-Publish.

## 4. Mengisi konfigurasi dan beralih ke produksi

Di dalam `index.html`, cari bagian paling atas `<script type="module">`:

```javascript
const IS_PREVIEW = false;   // ubah dari true menjadi false

const firebaseConfig = {    // ganti dengan config project Anda
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

Dengan `IS_PREVIEW = false`: data dan akun demo hilang, tidak ada banner preview, tidak ada data yang ditulis otomatis ke Firestore, dan seluruh data dibaca/ditulis langsung ke Firestore (tanpa localStorage).

## 5. Memasang aplikasi

**Blogger:** buat Halaman/Postingan baru, mode **HTML**, tempel semua isi `index.html` dari tag `<style>` pertama sampai `</script>` (tanpa `<html>`, `<head>`, `<body>`). Aplikasi tampil layar penuh di atas template Blogger.

**Hosting lain** (Firebase Hosting, Vercel, Netlify, GitHub Pages): unggah `index.html` apa adanya. Semuanya otomatis HTTPS.

## 6. Pengaturan awal oleh admin (urutan yang disarankan)

Semua ada di menu **Pengaturan**:

1. **Logo & Sekolah:** isi nama sekolah dan unggah logo (tampil di halaman login, header, dan laporan cetak).
2. **Jam Absen:** jam absen masuk dibuka, batas tepat waktu, **absen masuk ditutup**, toleransi, jam pulang resmi, jam pulang ditutup, dan hari kerja.
3. **Mode Absen:** pilih **Absen di Sekolah** (lokasi dicek) atau **Absen di Rumah (WFH)** (lokasi dimatikan). Guru melihat mode aktif di dashboard mereka.
4. **Poin Tanpa Keterangan:** atur poin per menit terlambat (bawaan 1), maksimal poin terlambat (225), tidak absen pulang (225), dan alpha (450).
5. **Lokasi:** klik peta atau geser penanda ke posisi sekolah, atau tekan "Gunakan lokasi saya sekarang", atur radius, lalu simpan. Posisi dapat diubah kapan saja.
6. **Kalender Akademik (menu Kalender):** libur nasional dan cuti bersama (SKB 3 Menteri) **2026 dan 2027 sudah terisi otomatis**, begitu pula hari peringatan nasional (Kartini, Hardiknas, Hari Guru, dst., hanya informasi). Matikan tombol pada tanggal tertentu bila sekolah tetap masuk (mis. cuti bersama). Tahun lain hanya berisi libur tetap (1 Jan, 1 Mei, 1 Jun, 17 Agu, 25 Des), jadi tambahkan libur keagamaan secara manual. Admin juga dapat menambah **agenda sekolah** (hari tetap kerja) atau **libur sekolah** (tidak dihitung alpha) yang tampil di kalender guru. **Penting:** hari yang bukan libur dan bukan akhir pekan dianggap hari kerja.
7. **Kop & Tanda Tangan:** isi baris instansi, alamat, kontak, kota (titimangsa), jabatan, nama dan NIP kepala sekolah. Logo dan nama sekolah diambil dari tab Logo & Sekolah. Semua laporan PDF memakai kop surat, tanda tangan, dan titimangsa ini. Datanya tersimpan di `settings/kop` yang hanya bisa dibaca admin.
8. **Guru:** tambah akun satu per satu, atau **Impor CSV** (kolom `username,nama,nip,password`, ada tombol Template CSV).

## 7. Aturan absen dan poin tanpa keterangan

Poin dihitung otomatis per bulan dari data absensi dan ditampilkan di dashboard guru, riwayat guru, rekap admin, dan dashboard admin.

| Kondisi | Poin |
|---|---|
| Terlambat | 1 poin per menit, maksimal 225 per hari (terlambat 350 menit tetap 225) |
| Tidak absen masuk tetapi absen pulang | 225 (sama dengan maksimal poin terlambat) |
| Absen masuk tetapi tidak absen pulang (dihitung setelah jam pulang ditutup) | 225 |
| Alpha (tidak absen masuk dan pulang sama sekali, dihitung setelah jam pulang ditutup) | 450 |
| Izin, sakit, cuti, dinas luar yang disetujui | 0 |
| Pulang cepat, hari libur, hari non-kerja | 0 |

- Setelah jam **absen masuk ditutup**, tombol absen masuk terkunci. Guru yang belum absen masuk **tetap bisa absen pulang** (kena 225 poin), tidak ada penguncian absen pulang.
- Hari ini baru dinyatakan alpha atau tidak absen pulang setelah jam pulang ditutup. Sebelum itu statusnya "Belum absen".
- Alpha tidak ditambah poin lain pada hari yang sama.
- Perubahan nilai poin di Pengaturan berlaku untuk semua data lama, karena poin dihitung ulang dari data absensi.
- Guru yang dibuat hari ini baru dihitung mulai tanggal pembuatan akunnya.

## 8. Mode WFH dan dinas luar

- **WFH:** saat admin mengaktifkan mode WFH, semua guru dapat absen dari mana saja. GPS tidak diminta, tetapi selfie tetap wajib dan aturan jam dan poin tetap berlaku.
- **Dinas luar:** guru mengajukan izin berjenis **Dinas Luar** (boleh dengan foto surat tugas). Setelah admin **menyetujui**, pada tanggal tersebut guru otomatis memakai mode tanpa lokasi, tetap bisa absen, dan tidak dikenai poin. Tidak ada tombol manual per guru.
- Izin/Sakit/Cuti yang disetujui membuat guru tidak perlu absen dan tidak dikenai poin.

## 8b. Klaim banding

Dipakai bila poin tanpa keterangan muncul karena **kendala teknis** (aplikasi error, jaringan, GPS, kamera, perangkat rusak).

- **Guru** membuka menu **Banding > Ajukan Klaim Banding**, memilih tanggal (hanya tanggal yang berpoin, dalam batas hari yang diatur admin, dan belum diajukan), jenis kendala, perbaikan yang diminta (hapus poin atau revisi menjadi Hadir), penjelasan, dan bukti foto opsional. Daftar klaim punya tab **Semua / Menunggu / Disetujui / Ditolak**. Klaim berstatus Menunggu dapat dibatalkan. Klaim yang ditolak boleh diajukan ulang.
- **Admin** membuka menu **Banding** (tab yang sama, bawaan Menunggu). Detail klaim menampilkan catatan absensi asli, poin, foto absen, dan bukti guru. Admin memilih **Tolak** atau **Setujui** dengan hasil: *hapus poin* (status asli tetap, poin 0) atau *revisi menjadi Hadir* (dihitung hadir tepat waktu, poin 0), plus catatan untuk guru. Keputusan dapat dikembalikan ke Menunggu bila salah.
- Poin dihitung ulang otomatis dari data absensi dan klaim yang disetujui, sehingga dashboard, riwayat, kalender, rekap, laporan, dan grafik langsung ikut berubah. Data absensi aslinya tidak diubah.
- Batas pengajuan (bawaan 14 hari) diatur di **Pengaturan > Poin Tanpa Keterangan**. Dashboard admin menampilkan pemberitahuan bila ada klaim menunggu.

## 9. Fitur guru

- **Kalender:** kalender kehadiran bulanan (hijau/kuning/merah), titik libur/agenda/peringatan, dan daftar agenda terdekat. Guru hanya dapat melihat; admin yang mengubah.
- **Beranda:** foto profil, jam saat ini, mode absen (sekolah/WFH/dinas luar), absen masuk/pulang hari ini, kata motivasi (tepat di bawah jam/profil), absen masuk/pulang, poin bulan ini, ringkasan agenda terdekat, dan rincian poin. Kata motivasi juga tampil sebagai sapaan setiap kali login.
- **Riwayat:** rincian per hari (jam masuk/pulang, menit terlambat, mode, poin), filter ada poin/alpha/izin/hadir, dan ringkasan bulanan. Ketuk satu hari untuk melihat foto dan lokasi.
- **Izin:** ajukan dan pantau izin, sakit, cuti, dinas luar.
- **Profil:** ganti foto dan password, serta jadwal absen dan aturan poin tanpa keterangan (digabung dalam satu kartu).

## 10. Fitur admin

- **Dashboard:** statistik hari ini,  mode absen aktif, poin terbanyak bulan ini, dan **Laporan Harian Guru Tidak Hadir** per tanggal dengan tombol **Cetak PDF**.
- **Rekap:** pilih rentang tanggal dan guru. Tab *Ringkasan & Grafik* berisi diagram guru dengan poin terbanyak, komposisi kehadiran, tren harian, dan tabel per guru dengan kolom poin. Tab *Rincian Harian* berisi daftar per hari beserta poin. Keduanya bisa diekspor CSV atau dicetak PDF.
- **Kalender:** kalender akademik (angka bukan indikator absensi). Ketuk tanggal untuk melihat, menambah, atau menghapus agenda; daftar libur nasional/cuti bersama dengan tombol on/off tersedia di bawahnya.
- **Izin, Banding, Guru, Setelan (pengaturan):** seperti dijelaskan di atas.

**Cara menyimpan PDF:** semua laporan berkop surat dan bertanda tangan kepala sekolah. Tombol Cetak PDF membuka dialog cetak browser. Pilih tujuan **Simpan sebagai PDF** (Save as PDF).

## 11. Catatan penting

- **Keamanan:** karena guru login tanpa Firebase Auth, aturan Firestore tidak bisa mengetahui "siapa" guru yang membaca. Daftar guru dan rekap hanya bisa dibaca admin, namun dokumen yang alamatnya diketahui (misalnya `users/<username>`) dapat dibaca. Password disimpan sebagai hash SHA-256, bukan teks asli. Gunakan password guru yang tidak mudah ditebak. Untuk keamanan yang lebih ketat, perlu beralih ke Firebase Auth untuk guru.
- **Validasi lokasi dan jam** dilakukan di aplikasi. Rules memastikan format data dan mencatat waktu server; rekap admin menandai absensi yang jam perangkatnya berbeda lebih dari 10 menit dari jam server. Aplikasi tidak dapat mendeteksi aplikasi lokasi palsu.
- **Sesi guru** tidak disimpan di perangkat, jadi guru perlu login ulang bila halaman dimuat ulang. Sesi admin dipulihkan oleh Firebase Auth.
- **Data lama:** absensi yang dibuat dengan versi sebelumnya tidak memiliki field `mode`. Absensi tersebut tetap terbaca di aplikasi dan dianggap mode sekolah.
- **Kuota:** paket gratis Firebase (Spark) cukup untuk puluhan guru. Dashboard dan riwayat guru membaca satu dokumen per hari dalam sebulan, hasilnya disimpan sementara di memori selama 1 menit. Foto selfie dan lampiran adalah bagian terbesar penggunaan penyimpanan.
