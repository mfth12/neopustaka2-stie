---
name: Setup lokal SLiMS
overview: Menyambungkan fork SLiMS 9.8.0 ini ke database lokal yang sudah diimpor dari staging, lalu menjalankannya di mesin ini tanpa Docker dan tanpa installer.
todos:
  - id: check-runtime
    content: Cek PHP 8.1+, ekstensi wajib, dan koneksi MySQL ke db-dev-neopustaka2
    status: completed
  - id: composer
    content: Jalankan composer install agar vendor/autoload.php ada
    status: completed
  - id: local-config
    content: Buat config/database.php dan config/env.php (development), gitignored
    status: completed
  - id: writable-dirs
    content: Pastikan files, images, repository, dan cache bisa ditulis
    status: completed
  - id: serve
    content: Jalankan php -S 127.0.0.1:8080 dan pastikan OPAC serta /admin/ merespons
    status: completed
isProject: false
---

# Setup lokal NeoPustaka (SLiMS 9.8.0)

Database `db-dev-neopustaka2` sudah berisi data staging, jadi installer tidak dijalankan. Yang kurang hanya konfigurasi lokal, dependency Composer, izin folder tulis, dan web server PHP. Docker Compose di repo ini memakai MariaDB sendiri (`DB_PORT` default 3307, database `senayan`) dan tidak dipakai.

## Yang sudah ada di kode

- Bootstrap membaca [`config/env.php`](config/env.php) lalu [`config/database.php`](config/database.php). Kalau `env.php` tidak ada, `ENVIRONMENT` menjadi `unavailable` dan aplikasi membalas HTTP 503 ([`sysconfig.inc.php`](sysconfig.inc.php) baris 34–47).
- Kedua file itu, plus isi `files/`, `repository/`, `images/*`, dan `cache/`, ada di [`.gitignore`](.gitignore). Aman diisi di lokal dan tidak ikut commit.
- `vendor/autoload.php` belum ada. [`Dockerfile`](Dockerfile) menjalankan `composer install`; lokal perlu langkah yang sama.
- Butuh PHP >= 8.1 dengan ekstensi `gd` (GIF/JPEG/PNG), `pdo_mysql`, `mbstring`, `gettext`, `fileinfo`.

## Yang akan disesuaikan

1. Cek PHP, ekstensi, dan koneksi MySQL ke `127.0.0.1:3306` / database `db-dev-neopustaka2` / user `root`. Password root tidak ditulis di rencana ini; kalau koneksi ditolak, password diminta sekali lalu hanya disimpan di `config/database.php`.
2. `composer install` di root proyek agar `vendor/autoload.php` terbentuk.
3. Buat [`config/database.php`](config/database.php) dari [`config/database.sample.php`](config/database.sample.php): storage engine nya ganti jadi innodb!

```php
'host' => '127.0.0.1',
'database' => 'db-dev-neopustaka2',
'port' => 3666,
'username' => 'root',
'password' => 'satuduatiga456TujuhDelapan9',
'options' => ['storage_engine' => 'MyISAM'],
```

4. Buat [`config/env.php`](config/env.php) dari [`config/env.sample.php`](config/env.sample.php) dengan `$env = 'development'` supaya error PHP terlihat saat pengembangan. Di staging/produksi biarkan `production`.
5. [`config/url.php`](config/url.php) tidak diubah selama aplikasi dilayani di root host lokal (`http://127.0.0.1:8080/`). `url.base` kosong sudah benar untuk itu.
6. Pastikan folder ini ada dan bisa ditulis proses PHP: `files/`, `images/` (termasuk `barcodes`, `persons`, `docs`, `cache`), `repository/`, `cache/`.
7. Jalankan server pengembangan bawaan PHP dari root proyek:

```bash
php -S 127.0.0.1:8080
```

Lalu buka OPAC di `http://127.0.0.1:8080/` dan admin di `http://127.0.0.1:8080/admin/`. Login memakai akun yang sama dengan staging, karena datanya hasil impor.

## Yang tidak diubah

- Jangan jalankan `/install/`. Begitu `config/database.php` ada, installer mengalihkan ke beranda.
- Jangan pakai `docker compose up`. Itu membuat database baru, bukan `db-dev-neopustaka2`.
- Jangan commit `config/database.php` atau `config/env.php`.

## Setelah aplikasi nyala

Data SQL staging tidak membawa berkas. Sampul, foto anggota, dan lampiran digital tetap kosong sampai folder `images/`, `files/`, dan `repository/` disalin dari server staging.

Pengaturan di tabel `setting` ikut terimpor. Setelah login admin, cek supaya lokal tidak menyentuh layanan kampus:

- pengiriman email (SMTP) — matikan atau arahkan ke mail lokal
- batasan IP, kalau OPAC menampilkan `Access not allowed`
- integrasi UCS / P2P / server lain yang mengarah ke host produksi

Kustom kampus sebaiknya lewat plugin di `plugins/` (lihat [`plugins/README.md`](plugins/README.md)) atau template, bukan mengedit inti SLiMS. Saat ini `.gitignore` mengabaikan `plugins/*` dan `template/*`. Saat kustom pertama dibuat, pola ignore itu perlu dilonggarkan supaya kode kustom ikut version control, sementara config lokal tetap diabaikan.