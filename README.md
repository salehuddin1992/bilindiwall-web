# BilindiWall Web

Versi **web (landscape)** dari aplikasi Android **BilindiWall** — Dinding Pembelajaran Sekolah
**SMP Negeri Sinombayuga**, Kec. Posigadan, Kab. Bolaang Mongondow Selatan, Sulawesi Utara.

> Hanya **tampilan** yang berbeda. Seluruh fungsi, logika, aturan moderasi, dan **database**
> sama persis dengan aplikasi Android — data yang dibuat di web langsung muncul di Android,
> dan sebaliknya.

---

## 1. Ringkasan

| Aspek | Android | Web (repo ini) |
|---|---|---|
| Tampilan | Portrait, navigasi bawah | **Landscape**, 3 kolom + rail kiri/kanan |
| Teknologi | Kotlin + Jetpack Compose | HTML + CSS + JavaScript (ES Modules) |
| Database | Cloud Firestore `bilindiwall-7c8e9` | **Cloud Firestore yang sama** |
| Upload media | Cloudinary `utohfzmw` | **Cloudinary yang sama** |
| Build step | Gradle | **Tidak ada** — cukup di-hosting sebagai file statis |
| Dependensi npm | — | **Tidak ada** |

Tampilan otomatis menyesuaikan: di layar lebar tampil 3 kolom, di layar sempit
(HP/tablet) rail disembunyikan dan muncul navigasi bawah seperti versi Android.

---

## 2. Cara menjalankan di komputer sendiri

Karena memakai ES Modules, file **tidak bisa** dibuka lewat `file://`. Jalankan server lokal:

```bash
# Python (sudah ada di kebanyakan komputer)
python3 -m http.server 8080

# atau Node.js
npx serve .
```

Lalu buka `http://localhost:8080`.

---

## 3. Deploy ke GitHub Pages (paling mudah, gratis)

```bash
git init
git add .
git commit -m "BilindiWall versi web"
git branch -M main
git remote add origin https://github.com/<username>/bilindiwall-web.git
git push -u origin main
```

Lalu di GitHub:

1. Buka **Settings → Pages**
2. Bagian **Build and deployment → Source**, pilih **GitHub Actions**
3. Selesai. Workflow `.github/workflows/deploy.yml` akan berjalan otomatis setiap `git push`.

Alamat situs: `https://<username>.github.io/bilindiwall-web/`

### Alternatif: Firebase Hosting (satu domain dengan project Firestore)

```bash
npm install -g firebase-tools
firebase login
firebase use bilindiwall-7c8e9
firebase deploy
```

`firebase.json` dan `firestore.rules` sudah disiapkan di repo ini.

---

## 4. ⚠️ Dua langkah WAJIB agar database benar-benar terhubung

Tanpa dua langkah ini, aplikasi tetap berjalan tetapi **datanya hanya lokal** (mode offline).

### Langkah A — Daftarkan aplikasi Web di Firebase

`google-services.json` milik Android hanya berisi konfigurasi untuk Android.
Web membutuhkan `appId` tersendiri.

1. Buka [Firebase Console](https://console.firebase.google.com/) → project **bilindiwall-7c8e9**
2. **Project Settings** (ikon gerigi) → gulir ke **Your apps** → klik ikon **Web `</>`**
3. Beri nama misalnya `BilindiWall Web` → **Register app**
4. Salin nilai `firebaseConfig` yang muncul
5. Tempel ke **`js/config.js`**:

```js
export const FIREBASE_CONFIG = {
  apiKey: 'AIza...',                  // dari Firebase Console
  authDomain: 'bilindiwall-7c8e9.firebaseapp.com',
  projectId: 'bilindiwall-7c8e9',     // WAJIB sama dengan Android
  storageBucket: 'bilindiwall-7c8e9.firebasestorage.app',
  messagingSenderId: '901564670259',
  appId: '1:901564670259:web:xxxxxxxxxxxx',  // ← isi dari Firebase Console
};
```

> `projectId` **harus** tetap `bilindiwall-7c8e9`. Itulah yang membuat data web dan Android sama.

6. Masih di Firebase Console → **Authentication → Settings → Authorized domains**,
   tambahkan domain situs Anda (`<username>.github.io`).

### Langkah B — Pasang Security Rules Firestore

Buka **Firestore Database → Rules**, tempel isi berkas `firestore.rules` dari repo ini,
lalu klik **Publish**.

Kalau langkah ini dilewati, konsol browser akan menampilkan `PERMISSION_DENIED`
dan lencana awan di kanan atas berwarna merah (offline).

---

## 5. Cek koneksi database

Lencana awan kecil di samping logo (pojok kiri atas):

| Warna | Arti |
|---|---|
| 🟢 Hijau | Terhubung ke Cloud Firestore — data tersinkron dengan Android |
| 🔴 Merah | Offline — periksa Langkah A & B di atas |

Detail lengkapnya juga tampil di **Pengaturan → Status Sinkronisasi Database**.

---

## 6. Akun untuk uji coba

Semua akun bawaan memakai kata sandi **`123`**:

| Username | Nama | Peran | Kelas / Mapel |
|---|---|---|---|
| `salehuddin` | Salehuddin, S.Pd | Guru | IPA |
| `ahmad` | Drs. H. Ahmad | Kepala Sekolah | — |
| `andi` | Andi Pratama | Siswa | VII-A |
| `budi` | Budi Santoso | Siswa | VII-A |
| `siti` | Siti Aminah | Siswa | VIII-B |

Alias cepat juga berlaku seperti di Android: ketik `guru`, `siswa`, atau `kepsek`.

**Kode otorisasi pendaftaran akun Guru:** `GURU2026`, `SINOMBAYUGA`, atau `123456`.

---

## 7. Struktur berkas

```
bilindiwall-web/
├── index.html                  Halaman utama
├── firebase.json               Konfigurasi Firebase Hosting
├── firestore.rules             Security Rules Firestore
├── .github/workflows/deploy.yml  Deploy otomatis ke GitHub Pages
├── assets/logo.jpg             Logo bawaan (dari ic_logo_bilindi.jpg)
├── css/style.css               Palet warna + tata letak landscape
└── js/
    ├── config.js               Firebase & Cloudinary
    ├── seed.js                 Data awal (salinan dari Repository.kt)
    ├── store.js                Logika bisnis  ← padanan BilindiWallRepository.kt
    ├── sync.js                 Firestore + upload  ← padanan FirestoreSyncService.kt
    ├── ui.js                   Helper DOM, ikon, toast, modal
    ├── postcard.js             Kartu postingan  ← padanan PostCardItem.kt
    ├── dialogs.js              Semua dialog  ← padanan TeacherCreateModal.kt dll
    ├── app.js                  Kerangka aplikasi  ← padanan BilindiWallApp.kt
    └── views/
        ├── login.js      ← LoginScreen.kt
        ├── feed.js       ← FeedScreen.kt
        ├── reels.js      ← ReelsScreen.kt
        ├── learn.js      ← TeacherDashboardScreen.kt
        ├── classes.js    ← ClassesScreen.kt
        ├── profile.js    ← ProfileScreen.kt
        ├── messenger.js  ← MessengerScreen.kt
        └── settings.js   ← SettingsScreen.kt
```

---

## 8. Koleksi Firestore

Enam koleksi pertama **dipakai bersama** Android dengan skema dokumen yang identik:

| Koleksi | Isi |
|---|---|
| `posts` | Postingan, modul terstruktur, tugas, status moderasi |
| `users` | Akun siswa, guru, kepala sekolah |
| `comments` | Komentar pada postingan |
| `reels` | Reels edukasi |
| `stories` | Cerita 24 jam |
| `settings` | Preferensi per pengguna (mode gelap) |

Empat koleksi berikut **ditambahkan versi web** supaya data tidak hilang saat halaman
di-refresh (di Android data ini hanya disimpan di memori). Aplikasi Android tidak
terpengaruh sama sekali:

`notifications`, `submissions`, `classes`, `messages`

---

## 9. Fungsi yang sudah diverifikasi sama dengan Android

- Login (username, nama, alias `guru`/`siswa`/`kepsek`, sandi universal `123`)
- Registrasi siswa (menunggu persetujuan) & guru (langsung aktif, butuh kode otorisasi)
- Beranda terkunci pada kelas siswa; guru & kepala sekolah bisa ganti filter kelas
- Filter kategori Semua / Materi / Tugas / Siswa, pencarian, dan filter tagar
- Moderasi otomatis: postingan siswa → `PENDING` → notifikasi ke guru
- Guru menyetujui / menolak / meminta revisi, siswa menerima notifikasi balik
- Modul terstruktur 4 bagian: Pemantik, Tujuan, Materi Inti, Asesmen
- Tugas dengan tagar otomatis `#Tugas<Mapel><Kelas><Judul>` dan tenggat waktu
- Pengumpulan tugas otomatis terdata ketika siswa memakai tagar
- Penilaian karya siswa (nilai 0–100 + umpan balik)
- Cerita (foto/video), Reels edukasi, Messenger, Kelas, Profil, Lencana
- Mode gelap tersimpan di Firestore per pengguna
- Upload foto/video/PDF ke Cloudinary yang sama dengan Android

---

## 10. Catatan

**Tagar otomatis.** Mata pelajaran `IPA (Ilmu Pengetahuan Alam)` menghasilkan tagar
`#TugasIPA(...` karena rumusnya mengambil 4 karakter pertama setelah spasi dihapus.
Ini **perilaku asli dari aplikasi Android** dan sengaja dipertahankan agar tagar yang
dibuat di web dan di Android tetap sama. Jika ingin dirapikan, ubah **di kedua aplikasi
sekaligus** (`js/dialogs.js` baris `cleanSubject` dan `TeacherCreateModal.kt` baris 108).

**Kunci di dalam kode.** `apiKey` Firebase dan `upload_preset` Cloudinary memang dirancang
untuk dipakai di sisi klien (sama seperti di aplikasi Android). Keamanan data ditentukan
oleh Firestore Security Rules, bukan oleh menyembunyikan kunci.

---

© 2026 TIM IT SMP Negeri Sinombayuga
