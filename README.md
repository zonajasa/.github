# ZonaJasa — Pencarian Jasa Berdasarkan Radius

## Ringkasan

ZonaJasa adalah layanan pencarian penyedia jasa di sekitar lokasi pengguna berdasarkan radius (jarak). Backend dibangun dengan Laravel (API RESTful) dan frontend mobile dibuat dengan Flutter. Aplikasi memanfaatkan lokasi GPS perangkat dan kueri geospasial (Haversine atau PostGIS) untuk mengurutkan dan memfilter hasil berdasarkan jarak.

## Fitur Utama

- Pencarian penyedia jasa di sekitar lokasi pengguna (radius configurable)
- Urutkan hasil berdasarkan jarak (terdekat terlebih dahulu)
- Detail penyedia jasa: nama, kategori, alamat, jarak, rating, kontak
- Otentikasi dasar untuk penyedia (API token / JWT)
- Dukungan untuk MySQL (Haversine) atau PostgreSQL + PostGIS

## Tech Stack

- Backend: Laravel (PHP 8+)
- Database: MySQL atau PostgreSQL (+ PostGIS untuk akurasi lebih baik)
- Mobile frontend: Flutter
- API: RESTful JSON

## Arsitektur Singkat

- Mobile (Flutter) meminta permission lokasi, kemudian memanggil endpoint API `GET /api/services/nearby` dengan parameter `lat`, `lng`, `radius`.
- Laravel meng-handle request, melakukan kueri geospasial ke database, menghitung jarak, lalu mengembalikan daftar service yang berada di dalam radius.

## Contoh Endpoint API

- GET /api/services/nearby?lat={lat}&lng={lng}&radius={meters}

Contoh request:

GET /api/services/nearby?lat=-6.200000&lng=106.816666&radius=5000

Contoh respons (JSON):

{
"data": [
{
"id": 12,
"name": "Tukang Ledeng Budi",
"category": "Plumbing",
"lat": -6.201234,
"lng": 106.817890,
"distance": 132.4,
"rating": 4.7
}
],
"meta": {
"count": 1
}
}

## Implementasi Geospasial (Pilihan)

- MySQL (Haversine): jalankan kueri raw SQL yang menghitung jarak menggunakan rumus Haversine. Contoh (raw query di Laravel):

SELECT id, name, lat, lng,
(6371000 _ ACOS(
COS(RADIANS(:lat)) _ COS(RADIANS(lat)) _ COS(RADIANS(lng) - RADIANS(:lng)) +
SIN(RADIANS(:lat)) _ SIN(RADIANS(lat))
)) AS distance
FROM services
HAVING distance <= :radius
ORDER BY distance ASC;

- PostgreSQL + PostGIS (lebih akurat & indeks geospasial): gunakan `ST_DistanceSphere` atau `ST_DWithin` untuk performa dan presisi lebih baik.

## Petunjuk Instalasi — Backend (Laravel)

1. Clone repo backend.
2. Copy `.env.example` ke `.env` dan atur `DB_*`, `APP_URL`, `MAPS_API_KEY` jika diperlukan.
3. Install dependencies:

```bash
composer install
php artisan key:generate
```

4. Migrasi dan seeding:

```bash
php artisan migrate --seed
```

5. Jalankan server lokal:

```bash
php artisan serve
```

6. Pastikan environment support spatial (untuk PostGIS) atau gunakan Haversine jika MySQL.

## Petunjuk Instalasi — Frontend (Flutter)

1. Pastikan Flutter SDK terpasang.
2. Set `API_BASE_URL` di konfigurasi aplikasi (mis. `lib/config.dart`).
3. Install dependency dan run:

```bash
flutter pub get
flutter run
```

## Praktik di Mobile

- Minta permission lokasi dari pengguna.
- Ambil koordinat (`latitude`, `longitude`).
- Kirim request ke endpoint `nearby` dengan `radius` dalam meter.
- Tampilkan hasil terurut berdasarkan `distance`.

## Env Vars Penting

- `APP_URL`
- `DB_CONNECTION`, `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`
- `MAPS_API_KEY` (opsional, jika menampilkan peta di frontend)

## Testing

- Backend: `php artisan test` (unit & feature tests untuk endpoint)
- Frontend: `flutter test` untuk unit/widget tests

## Tips & Catatan

- Untuk dataset besar, gunakan indeks geospasial (PostGIS atau spatial index di MySQL) dan batasi hasil dengan bounding-box sebelum hitung jarak akurat.
- Pertimbangkan caching dan rate-limiting pada endpoint untuk mencegah penyalahgunaan.
- Pertimbangkan format `radius` default (mis. 5000 = 5 km) dan batas maksimum (mis. 50 km).

## Kontribusi

Silakan buka issue atau PR. Sertakan deskripsi singkat, langkah reproduksi, dan testing yang dilakukan.

## Lisensi

Tentukan lisensi proyek (mis. MIT) di file `LICENSE`.

## Kontak

Jika butuh bantuan lanjutan atau integrasi peta/optimisasi, hubungi tim pengembang.

---

Dokumentasi ini adalah titik awal — beri tahu saya jika Anda ingin menambahkan contoh kode konkretnya (controller Laravel, query Eloquent, atau widget Flutter).
