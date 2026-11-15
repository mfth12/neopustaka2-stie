# NeoPustaka STIE

Fork [SLiMS 9 Bulian](https://github.com/slims/slims9_bulian) `v9.8.0` untuk Perpustakaan STIE. Aplikasi mengelola koleksi, sirkulasi, keanggotaan, dan stok opname.

SLiMS dilisensikan di bawah GNU GPL versi 3. Baca `GPL-3.0 License.txt`.

## Kebutuhan

- PHP >= 8.1, dengan ekstensi `gd` (GIF, JPEG, PNG), `pdo_mysql`, `mbstring`, `gettext`, dan `fileinfo`
- MariaDB atau MySQL 5.7+
- Composer

## Pengembangan lokal

Database lokal sudah berisi impor dari staging. Jangan menjalankan `/install/`, dan jangan memakai `docker compose` di repo ini. Compose itu membuat MariaDB baru bernama `senayan`, bukan database STIE.

| | |
| --- | --- |
| Host | `127.0.0.1` |
| Port | `3666` |
| Database | `db-dev-neopustaka2` |
| User | `root` |
| Storage engine | `InnoDB` |

### 1. Dependency

```bash
composer install
```

### 2. Konfigurasi

Salin sampel lalu isi koneksi lokal. Kedua file ini diabaikan Git.

```bash
cp config/database.sample.php config/database.php
cp config/env.sample.php config/env.php
```

Di `config/database.php`, node `SLiMS`:

```php
'host' => '127.0.0.1',
'database' => 'db-dev-neopustaka2',
'port' => 3666,
'username' => 'root',
'password' => '<password lokal>',
'options' => [
    'storage_engine' => 'InnoDB'
]
```

Di `config/env.php` set `$env` dan `$conditional_environment` ke `development`, serta `$based_on_ip` ke `false`. Mode `development` menampilkan error PHP. Di staging dan produksi biarkan `production`.

### 3. Folder yang harus bisa ditulis

```bash
mkdir -p files/reports files/att images/content cache
```

PHP perlu menulis ke `files/`, `images/` (termasuk `barcodes`, `persons`, `docs`, `labels`, `cache`, `content`), `repository/`, dan `cache/`.

Dump SQL tidak membawa sampul, foto anggota, atau lampiran. Salin `images/`, `files/`, dan `repository/` dari staging bila berkas itu diperlukan.

### 4. Menjalankan

```bash
php -S 127.0.0.1:8080
```

Jika port 8080 sudah dipakai, ganti portnya, misalnya:

```bash
php -S 127.0.0.1:8081
```

- OPAC: `http://127.0.0.1:8081/`
- Login petugas: `http://127.0.0.1:8081/index.php?p=login`

Akun petugas sama dengan akun di staging. Membuka `/admin/` tanpa sesi akan diarahkan ke halaman login.

Setelah login, cek pengaturan yang ikut terimpor dari staging supaya mesin lokal tidak mengirim email sungguhan atau menghubungi server produksi (SMTP, UCS, P2P).

## Kustom

Tambahan kampus sebaiknya berupa plugin di `plugins/` (lihat `plugins/README.md`) atau template, bukan suntingan inti SLiMS. `.gitignore` saat ini mengabaikan `plugins/*` dan `template/*`. Saat kode kustom pertama dibuat, longgarkan pola itu supaya kode ikut version control. `config/database.php` dan `config/env.php` tetap diabaikan.
