# MySQL (MariaDB) & phpMyAdmin with Docker Compose

Setup cepat lingkungan database **MariaDB (MySQL 10.6)** dan **phpMyAdmin** (berbasis FPM + Nginx) menggunakan Docker Compose. Siap digunakan untuk kebutuhan pengembangan lokal (*local development*) maupun akses jaringan lokal (LAN).

---

## 📋 Daftar Isi
- [Fitur Utama](#-fitur-utama)
- [Prasyarat](#-prasyarat)
- [Panduan Memulai (Step-by-Step)](#-panduan-memulai-step-by-step)
  - [1. Clone Repository](#1-clone-repository)
  - [2. Konfigurasi Environment (.env)](#2-konfigurasi-environment-env)
  - [3. Jalankan Container](#3-jalankan-container)
- [Cara Mengakses](#-cara-mengakses)
  - [Web Interface phpMyAdmin](#web-interface-phpmyadmin)
  - [Koneksi Database Langsung (DBeaver / Navicat / Backend App)](#koneksi-database-langsung-dbeaver--navicat--backend-app)
- [Akses dari Jaringan Lokal / LAN (Opsional)](#-akses-dari-jaringan-lokal--lan-opsional)
- [Perintah Berguna (Docker Commands)](#-perintah-berguna-docker-commands)
- [Struktur Folder](#-struktur-folder)
- [Troubleshooting / Masalah Umum](#-troubleshooting--masalah-umum)

---

## ✨ Fitur Utama
- **MariaDB 10.6** stabil dengan volume persisten data.
- **phpMyAdmin (FPM + Nginx)** ringan dan cepat.
- **Konfigurasi Fleksibel** via `.env` (custom port & bind address).
- **Mendukung Import Database Besar**:
  - `UPLOAD_LIMIT: 2G` pada phpMyAdmin.
  - `max_allowed_packet=1G` dan `innodb_log_file_size=1G` pada [my.cnf](my.cnf).

---

## 🛠 Prasyarat
Sebelum memulai, pastikan perangkat Anda telah terpasang:
- [Git](https://git-scm.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows / macOS) atau [Docker Engine](https://docs.docker.com/engine/install/) & Docker Compose (Linux).

Pastikan Docker service dalam kondisi berjalan (`docker info`).

---

## 🚀 Panduan Memulai (Step-by-Step)

### 1. Clone Repository
Buka terminal dan clone repository ini ke komputer Anda:

```bash
git clone <URL_REPOSITORY_ANDA>
cd mysql-phpmyadmin
```

### 2. Konfigurasi Environment (`.env`)
Salin file template `.env.sample` menjadi `.env`:

**Windows (PowerShell):**
```powershell
Copy-Item .env.sample .env
```

**Linux / macOS / Git Bash:**
```bash
cp .env.sample .env
```

Buka file `.env` dan sesuaikan nilainya:

```env
# Port MySQL di komputer host (misal: 3306 atau 3307 jika 3306 sudah dipakai)
MYSQL_EXPOSED_PORT=3306

# Password untuk user root database (Wajib diisi!)
MYSQL_ROOT_PASSWORD=rahasia123

# Bind address MySQL (127.0.0.1 untuk hanya PC sendiri, 0.0.0.0 untuk semua IP / LAN)
MYSQL_BIND_ADDRESS=127.0.0.1

# Port web phpMyAdmin di komputer host
PHPMYADMIN_SERVER_PORT=1111

# Bind address phpMyAdmin (127.0.0.1 atau 0.0.0.0)
PHPMYADMIN_BIND_ADDRESS=127.0.0.1
```

> [!WARNING]
> Jangan pernah membagikan atau meng-commit file `.env` asli ke GitHub jika mengandung kredensial sensitif. File `.env` sudah dimasukkan ke `.gitignore`.

### 3. Jalankan Container
Jalankan perintah berikut di folder project:

```bash
docker compose up -d
```

Periksa apakah semua container berjalan normal:

```bash
docker compose ps
```
Semua container (`mysql-dev`, `phpmyadmin-dev`, dan `phpmyadmin_server-dev`) harus berstatus `running` / `Up`.

---

## 🌐 Cara Mengakses

### Web Interface phpMyAdmin
Buka browser favorit Anda dan akses:
```text
http://localhost:1111
```
*(Sesuaikan port jika Anda mengubah `PHPMYADMIN_SERVER_PORT` di `.env`)*

**Login phpMyAdmin:**
- **Server**: biarkan default / `mariadb`
- **Username**: `root`
- **Password**: *(sesuai dengan `MYSQL_ROOT_PASSWORD` di `.env`)*

---

### Koneksi Database Langsung (DBeaver / Navicat / Backend App)
Jika ingin menghubungkan aplikasi Anda (Node.js, Laravel, Python, dll.) atau database client GUI dari komputer host:
- **Host**: `127.0.0.1` atau `localhost`
- **Port**: `3306` *(sesuai `MYSQL_EXPOSED_PORT` di `.env`)*
- **User**: `root`
- **Password**: *(sesuai `MYSQL_ROOT_PASSWORD`)*

---

## 📶 Akses dari Jaringan Lokal / LAN (Opsional)
Jika Anda ingin mengakses phpMyAdmin atau MySQL dari HP / laptop lain dalam satu jaringan WiFi / LAN:

1. Ubah nilai di file `.env` menjadi:
   ```env
   MYSQL_BIND_ADDRESS=0.0.0.0
   PHPMYADMIN_BIND_ADDRESS=0.0.0.0
   ```
2. Restart container:
   ```bash
   docker compose down
   docker compose up -d
   ```
3. Cari IP address PC host Anda (misal `192.168.1.50`).
4. Akses dari perangkat lain melalui:
   ```text
   http://192.168.1.50:1111/
   ```
5. *Catatan:* Jika tidak bisa dibuka, pastikan Windows Firewall / firewall OS Anda mengizinkan koneksi port masuk (*inbound rules*) untuk port yang digunakan.

---

## ⚙️ Perintah Berguna (Docker Commands)

| Perintah | Deskripsi |
|---|---|
| `docker compose up -d` | Menjalankan semua service di latar belakang (*detached mode*). |
| `docker compose down` | Menghentikan dan menghapus container (data database tetap aman di volume). |
| `docker compose restart` | Merestart semua service container. |
| `docker compose logs -f` | Melihat log output secara real-time untuk debugging. |
| `docker compose ps` | Melihat status container yang sedang berjalan. |
| `docker compose down -v` | ⚠️ **Hati-hati:** Menghapus container beserta volume data (database akan terhapus bersih). |

---

## 📁 Struktur Folder
```text
mysql-phpmyadmin/
├── nginx/
│   └── default.conf       # Konfigurasi reverse proxy Nginx untuk phpMyAdmin FPM
├── .env.sample            # Template variabel environment
├── .gitignore             # Mengabaikan file .env dan cache
├── docker-compose.yml     # Definisi service MariaDB, phpMyAdmin, dan Nginx
├── my.cnf                 # Konfigurasi kustom MariaDB (packet size, buffer)
└── README.md              # Dokumentasi project
```

---

## ❓ Troubleshooting / Masalah Umum

### 1. Port sudah digunakan (*Port already in use*)
- **Penyebab**: Port `3306` atau `1111` sudah dipakai oleh aplikasi lain (misalnya XAMPP atau MySQL native di Windows).
- **Solusi**: Ubah nilai `MYSQL_EXPOSED_PORT` atau `PHPMYADMIN_SERVER_PORT` di `.env` ke port lain (misal: `3307` dan `1112`), lalu jalankan `docker compose up -d`.

### 2. Gagal Login di phpMyAdmin (`Access denied for user 'root'`)
- Pastikan password yang diketik sama persis dengan `MYSQL_ROOT_PASSWORD` di `.env`.
- Jika Anda baru saja mengganti password di `.env` setelah container pertama kali dibuat, MariaDB tidak akan otomatis mengubah password root volume lama. Solusinya, reset volume dengan `docker compose down -v` lalu jalankan kembali (perhatian: data lama akan hilang).
