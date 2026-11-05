# InvenScan AI

Aplikasi Android untuk absensi dan audit kelengkapan aset ruangan dengan AI Vision. Operator memilih
ruangan, lalu mengarahkan kamera ke sekeliling ruangan. Model YOLOv8 di perangkat mendeteksi barang
dan mencentang checklist secara otomatis. Setiap barang yang ditemukan dibuktikan dengan satu foto
berstempel waktu, koordinat GPS, dan lokasi.

Aplikasi ini bisa dipakai oleh perusahaan mana pun, cukup dengan akun Google, dan berjalan tanpa
server. Aplikasi didistribusikan sebagai **APK** lewat MDM atau dipasang langsung (sideload),
tidak melalui Play Store. Data setiap perusahaan tersimpan di Google Sheets dan Google Drive miliknya sendiri.

> **Status:** desain sistem dan 14 layar UI sudah selesai. Enam layar baru (pendaftaran,
> undangan, profil perusahaan, penghapusan Operator, dan pemberitahuan privasi) masih perlu
> dirancang, dan layar kamera (S22) serta Konfirmasi Audit (S20) perlu dirancang ulang.
> Repo ini belum berisi code; code akan dibangun bertahap setelah uji teknis (Fase 0) selesai.

---

## Fitur utama

- **Daftar perusahaan dengan panduan.** Calon Admin lebih dulu membuat Google Cloud project
  perusahaannya sendiri lewat wizard langkah demi langkah di aplikasi, lalu login dengan Google, mengisi profil
  perusahaan (nama, telepon, logo, email kontak, alamat, bidang usaha, website), lalu otomatis
  menjadi Admin. Seluruh struktur Drive dan Sheets dibuat otomatis.
- **Undang Operator.** Admin cukup memasukkan email. Operator menerima **kode undangan** (lewat
  email dari Google Drive, QR, atau WhatsApp) yang juga membawa konfigurasi Google Cloud perusahaan,
  memasukkan kode itu sebelum login, dan langsung menjadi
  Operator tanpa mengisi apa pun.
- **Kendali keanggotaan di tangan Admin.** Operator tidak bisa keluar dari perusahaan sendiri.
  Admin dapat menonaktifkan Operator sementara, atau menghapusnya. Saat Operator dihapus, aplikasi
  Admin mengambil alih kepemilikan semua foto yang masih tertunda; data audit tetap utuh.
- **Deteksi aset real-time on-device.** YOLOv8 menampilkan bounding box dan persentase confidence.
  Hanya aset yang terdaftar di ruangan yang dicentang; barang lain diabaikan.
  Barcode dipakai untuk mengenali unit secara spesifik, termasuk lewat scanner hardware Zebra.
- **Konfirmasi per barang.** Sebelum dikirim, pengguna meninjau semua barang ruangan di layar
  Konfirmasi Audit: hasil kamera (AI, barcode, manual, atau tidak terdeteksi) dicatat terpisah
  dari status yang dipilih pengguna (Ada, Tidak ditemukan, Hilang, Dipinjam, … atau ketik sendiri),
  apakah barang berfungsi, dan keterangan (wajib untuk setiap status selain Ada dan untuk barang
  tidak berfungsi; keterangan yang sama bisa diisi sekaligus untuk semua barang Tidak ditemukan).
  Sesi pindai boleh diakhiri sebelum semua barang tercentang; sisanya otomatis berstatus Tidak
  ditemukan. Barang yang terdeteksi kamera tidak bisa
  ditandai Hilang/Tidak ditemukan. Barang yang tidak dikenali AI tetap bisa diaudit lewat barcode
  atau centang manual.
- **Foto bukti berstempel per barang.** Setiap barang yang ditemukan (lewat AI, barcode, atau
  centang manual) difoto otomatis dari kamera. Tanggal-jam, koordinat GPS, ruangan, dan alamat
  tercetak di foto sekaligus tersimpan sebagai data. Foto disimpan lokal, lalu diunggah langsung ke
  folder perusahaan di Google Drive Admin dan kepemilikannya diserahkan otomatis ke Admin. Setiap
  audit juga wajib punya satu foto ruangan sudut lebar, diambil saat sesi pindai diakhiri.
- **GPS wajib.** Audit tidak bisa dimulai tanpa lokasi yang cukup akurat (batasnya diatur Admin per
  perusahaan, default 50 m), dan lokasi palsu (mock location) ditolak.
- **Audit tidak boleh diganggu.** Selama memindai dan mengonfirmasi, layar dikunci (pin layar; di
  perangkat MDM/Zebra otomatis). Keluar hanya lewat Kirim atau Buang audit, dan draf darurat
  disimpan bila aplikasi tertutup paksa.
- **Satu audit per ruangan per hari.** Audit kedua di hari yang sama ditolak; bila terjadi saat
  offline, yang dimulai paling awal berlaku dan sisanya ditandai duplikat.
- **Audit terisolasi per ruangan.** Alurnya: pilih ruangan → pindai → konfirmasi → kembali ke Beranda.
  Dari konfirmasi, pengguna bisa kembali ke kamera untuk ruangan yang sama selama belum dikirim.
  Setelah dikirim, hasil tidak bisa diubah.
- **Offline-first.** Audit tetap bisa dimulai dan dilakukan tanpa sinyal, tanpa batas waktu. Sinkronisasi berjalan otomatis
  ketika perangkat online, termasuk lewat data seluler karena ukuran foto kecil.
- **Manajemen Admin.** Admin mengelola master barang, ruangan beserta checklist, dan pengguna
  (undang, kirim ulang/batalkan undangan, nonaktifkan, hapus). Master ruangan dan barang juga bisa
  diimpor massal dari template Excel dengan pemeriksaan (dry-run) sebelum ditulis. Tidak ada
  penugasan: Operator bebas mengaudit ruangan mana pun.
- **Riwayat, verifikasi, dan laporan.** Semua anggota bisa membuka detail setiap audit (foto
  ruangan, semua barang beserta foto, integritas). Admin dapat menandai audit diverifikasi, dan
  membuat laporan PDF/Excel per periode, per ruangan, per Operator, atau daftar barang bermasalah.
- **Integritas bukti.** Setiap foto dicatat sidik jarinya (MD5) dan diperiksa berkala oleh aplikasi
  Admin. Jam atau zona waktu perangkat yang tidak otomatis/wajar dan lokasi palsu ditandai, dan label QR ruangan bisa
  diwajibkan sebelum audit dimulai.
- **Privasi.** Pemberitahuan privasi ditampilkan di aplikasi saat pertama kali dibuka, ditambah
  pengingat sebelum mengambil foto. Lokasi (GPS) dan alamat saat audit disebutkan dengan jelas.
- **Backup dan restore.** Data (semua kolom, termasuk lokasi) bisa diekspor ke `.xlsx`/`.csv` dan dipulihkan kembali ke Google
  Sheets dengan validasi (dry-run). Header setiap sheet dilindungi agar hanya Admin yang bisa
  mengubahnya.
- **Gratis dan open source.** Tanpa biaya lisensi, fitur berbayar, maupun batas pemakaian dari
  aplikasi.

## Arsitektur

```
                 ┌─────────────────────── Perangkat (Android / Zebra) ───────────────────────┐
                 │  Kamera ─▶ YOLOv8 TFLite ─▶ Matcher ─▶ Checklist ─▶ Frame berstempel JPG  │
                 │  GPS ─────────────────────────────────────────────────┘   │               │
                 │                      SQLite terenkripsi + Outbox ◀────────┘               │
                 └──────────────────────────────────┬────────────────────────────────────────┘
                                                    │ Sheets API + Drive API
                                                    ▼
  Drive ADMIN perusahaan (dibuat otomatis; semua file diserahkan ke Admin, memakai kuota Admin)
  /InvenScan_System_Storage/
    01_MASTER_DATA/  Profil_Perusahaan (+ _registry) · User_Otorisasi       Operator: Viewer
                     Master_Ruangan · Master_Barang
    02_AUDIT_LOG/    Audit_Log_YYYY · Audit_Detail_YYYY · Anggota_Registry  Operator: Editor
                     Anggota_<email>.json (penanda keanggotaan, diserahkan ke Admin)
    03_BUKTI_FOTO/   AUDIT_[RUANG_ID]_[YYYYMMDD_HHMMSS]/…_[ASET_ID].jpg      Operator: Editor
                     (1 foto per barang; Operator mengunggah, Admin menerima kepemilikan)
                     <email>_<YYYYMMDD>/ salinan cadangan dari Operator terhapus
```

- **Tanpa backend dan tanpa domain.** Aplikasi memanggil Google API langsung dengan token
  pengguna. Tidak ada server, website, maupun halaman yang di-host.
- **Satu perusahaan = satu Admin + banyak Operator**, dan satu akun Google hanya tergabung di satu
  perusahaan (antar Google Cloud project hal ini tidak bisa dicek).
- **Google Cloud project per perusahaan.** Setiap perusahaan memakai project, client OAuth, dan
  kuota API-nya sendiri. Konfigurasinya disimpan di perangkat; pemilik aplikasi tidak mengelola
  project Google Cloud.
- **Akses minimum.** Aplikasi hanya meminta akses ke file yang dibuatnya sendiri (`drive.file`).
  Aplikasi tidak meminta akses ke seluruh isi Drive pengguna. Aplikasi tidak diverifikasi Google; bila uji teknis menunjukkan akses Google
  Sheets tambahan tetap diperlukan, jumlah pengguna dibatasi 100 per perusahaan.
- **Semua data di Drive Admin.** Operator mengunggah ke folder Admin lalu menawarkan kepemilikan;
  aplikasi Admin menerimanya otomatis saat dibuka. Setelah itu tidak ada file milik Operator
  (sebelumnya file sementara milik Operator), dan semua foto memakai kuota Drive Admin. Sisa kuota
  tampil di aplikasi Admin; foto tidak dihapus otomatis.

## Peran dan hak akses

| Modul                                     |  Admin   |       Operator        |
| ----------------------------------------- | :------: | :-------------------: |
| Daftar perusahaan, profil perusahaan      |    ✓     |   lihat saja          |
| Undang / nonaktifkan / hapus Operator     |    ✓     |           —           |
| Keluar dari perusahaan sendiri            |    —     |           —           |
| Login, pilih ruangan, pindai, hasil       |    ✓     |           ✓           |
| Master Barang / Ruangan & Checklist       |  CRUD    |       lihat saja      |
| Skema Drive, kebijakan, label QR ruangan, integritas bukti |    ✓     |           —           |
| Backup lokal                              |    ✓     |   sesuai kebijakan    |
| Restore ke Google Sheets                  |    ✓     |           —           |

## Stack

Expo (React Native, JavaScript) · expo-router · NativeWind · react-native-vision-camera ·
react-native-fast-tflite (YOLOv8) · react-native-skia · expo-location · Google OAuth (PKCE, Custom Tab) · Google Sheets API v4 ·
Google Drive API v3 · expo-sqlite (SQLCipher) · expo-secure-store

> **Lisensi:** aplikasi ini open source berlisensi **AGPL-3.0**, karena memakai model YOLO
> Ultralytics (AGPL-3.0). Siapa pun boleh memakai dan memodifikasinya; versi modifikasi yang
> dibagikan wajib menyertakan source-nya. Teks lengkap ada di [LICENSE](LICENSE).

## Riwayat perubahan

Lihat [CHANGELOG.md](CHANGELOG.md).

## Dukungan

InvenScan AI adalah proyek open source **tanpa layanan dukungan resmi**. Siapa pun boleh memakai,
mempelajari, dan memodifikasi source code ini sesuai lisensi [AGPL-3.0](LICENSE).

## Struktur proyek (target)

```
src/app/        route expo-router (layar tipis, tanpa logika)
src/features/   satu folder per use case: hook, provider, UI khusus fitur
src/components/ komponen UI reusable
src/core/       google, tenant (peran/perusahaan/undangan), db, sync, vision, media (frame + stempel), location, rbac, backup, logger
src/config/     env, skema sheet, drive, vision, rbac, tema, locale
assets/models/  model YOLOv8 .tflite + label
```
