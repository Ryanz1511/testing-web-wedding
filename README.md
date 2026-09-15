# Undangan Digital — Paket REGULAR (Rp50.000)

Andi & Salsa — 12 Desember 2026

## Apa ini

Satu file statis (`index.html`) yang berisi seluruh undangan: HTML, CSS, dan
JavaScript. Tidak ada proses build yang dibutuhkan — tinggal upload ke hosting
statis mana pun (Vercel, Netlify, cPanel, dsb.) dan langsung bisa diakses.

Ini konsisten dengan posisi paket REGULAR: sederhana, ringan, murah untuk
dirawat, tapi tetap terlihat profesional.

## Struktur komponen (secara konseptual)

Walau di dalam satu file, kode dipisah per bagian dengan komentar yang jelas,
mengikuti arsitektur komponen berikut — jika platform ini nanti dipecah
menjadi banyak file React/Next.js untuk paket yang lebih tinggi, potongan ini
tinggal dipindah 1:1 menjadi komponen sungguhan:

```
Invitation
├── OpeningScreen        → guest name dari ?to=, tombol "Buka Undangan"
├── HeroSection           → nama pasangan, foto, intro singkat
├── CoupleSection         → profil mempelai (layout editorial, bukan kartu)
├── EventSection          → Akad & Resepsi, tombol "Lihat Lokasi"
├── CountdownSection      → hitung mundur real-time ke countdownDate
├── GallerySection        → grid foto + lightbox ringan
├── RSVPSection           → form RSVP dengan validasi
├── WishesSection         → form ucapan + daftar ucapan (guestbook)
├── ShareSection          → Web Share API + fallback salin tautan
├── MusicControl          → tombol musik mengambang, aman dari autoplay block
└── Footer
```

## Data model

Semua data spesifik pelanggan berada di satu objek di bagian atas `<script>`:

```js
const INVITATION_DATA = {
  slug, couple, events: { akad, reception }, gallery, countdownDate
};
```

Tidak ada data pelanggan yang ditulis langsung di dalam logic komponen —
untuk mengganti pasangan lain, cukup ubah objek ini (dan konten HTML statis
yang menampilkan nama/lokasi di masing-masing section).

## Guest personalization

```
index.html?to=Budi%20Santoso
```

Nama tamu diambil dari query string dan dirender dengan `textContent`
(bukan `innerHTML`), sehingga aman dari XSS. Parameter `to` **tidak pernah**
dipakai untuk otentikasi/otorisasi — hanya untuk sapaan personal.

## Yang termasuk (sesuai batasan paket REGULAR)

Buka undangan · hero · profil mempelai · info acara (Akad & Resepsi) +
Google Maps · countdown · galeri (lightbox) · musik latar · RSVP · ucapan/
guestbook · bagikan/salin tautan · desain responsif mobile-first · animasi
ringan yang menghormati `prefers-reduced-motion`.

## Yang sengaja TIDAK termasuk

Sesuai batasan resmi paket REGULAR: domain custom, dashboard admin,
statistik RSVP lanjutan, galeri custom tak terbatas, layout sepenuhnya
custom, sistem hadiah digital, QR code, dsb. Semua itu ada di paket
PREMIUM ke atas.

## Backend integration points (wajib sebelum go-live)

Saat ini RSVP & Ucapan berjalan di sisi klien saja (data ucapan tersimpan
sementara di memori browser dan akan hilang saat halaman di-refresh) —
ini demo front-end. Untuk produksi, sambungkan dua form ini ke API:

```
POST /api/events/:slug/rsvp
  body: { name, attendance, guests, message }

POST /api/events/:slug/wishes
  body: { name, message }

GET  /api/events/:slug/wishes
  → daftar ucapan untuk ditampilkan
```

Titik integrasi sudah ditandai dengan komentar `-- Integration point --` di
dalam `index.html`. Aturan wajib di sisi server:

- Validasi ulang semua input (jangan percaya validasi client).
- Sanitasi teks ucapan sebelum disimpan/ditampilkan (cegah XSS/HTML injection).
- Rate-limit endpoint RSVP dan ucapan per IP untuk mencegah spam.
- Jangan gunakan parameter `to` di URL sebagai identitas tamu yang sah.

## Kustomisasi cepat

1. Ganti `INVITATION_DATA` dan teks di masing-masing `<section>`.
2. Ganti URL foto (hero, mempelai, galeri) dengan foto asli pelanggan.
3. Ganti `audio/wedding-theme.mp3` dengan file musik pilihan (letakkan di
   folder `audio/` di samping `index.html`).
4. Ganti link Google Maps di tombol "Lihat Lokasi" dengan lokasi asli.

## Catatan performa & aksesibilitas

- Semua gambar galeri memakai `loading="lazy"`.
- Kontras warna, focus state, label form, dan alt text sudah disiapkan.
- Animasi dinonaktifkan otomatis jika pengguna mengaktifkan
  `prefers-reduced-motion`.
