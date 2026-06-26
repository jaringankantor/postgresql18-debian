# PostgreSQL 18 Docker untuk Debian

Project ini berisi konfigurasi Docker Compose untuk menjalankan PostgreSQL 18 di Debian.

## Isi Project

- `docker-compose.yml`: konfigurasi service PostgreSQL.
- `.env.example`: contoh environment variable yang dibutuhkan container.
- `.env`: file konfigurasi lokal berisi username, password, dan nama database. File ini tidak ikut masuk Git.
- `enkripsi.env`: versi terenkripsi dari `.env`. File ini bisa diabaikan jika Anda tidak perlu mendekripsinya.
- `data/`: folder persistent data PostgreSQL. Data database tetap ada walaupun container dihentikan atau dibuat ulang.

## Prasyarat

- Docker Engine dan Docker Compose plugin sudah terpasang.
- Docker network `deb-network` sudah tersedia, karena `docker-compose.yml` memakai external network.

Buat network jika belum ada:

```bash
docker network create deb-network
```

## Setup

Gunakan salah satu cara berikut untuk menyediakan file `.env`.

Opsi 1, salin dari contoh:

```bash
cp .env.example .env
```

Opsi 2, dekripsi file terenkripsi jika Anda memiliki kunci `sops`:

```bash
sops -d enkripsi.env > .env
```

Edit `.env` sesuai kebutuhan:

```env
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_secret_password
POSTGRES_DB=your_database_name
```

## Menjalankan PostgreSQL

Jalankan container:

```bash
docker compose up -d
```

Cek status container:

```bash
docker compose ps
```

Lihat log:

```bash
docker compose logs -f postgres
```

## Koneksi Database

Default port PostgreSQL diekspos ke host:

```text
Host: localhost
Port: 5432
User: sesuai POSTGRES_USER di .env
Password: sesuai POSTGRES_PASSWORD di .env
Database: sesuai POSTGRES_DB di .env
```

Contoh koneksi dengan `psql`:

```bash
psql "host=localhost port=5432 user=$POSTGRES_USER dbname=$POSTGRES_DB"
```

## Menghentikan Container

Hentikan container:

```bash
docker compose down
```

Perintah di atas tidak menghapus data database karena data disimpan di folder `data/`.

## Persistent Data

Folder `data/` dipakai sebagai volume untuk menyimpan data PostgreSQL:

```yaml
./data:/var/lib/postgresql
```

Jangan hapus folder `data/` jika masih membutuhkan data database.

## Catatan

- Jika Anda tidak memiliki akses ke kunci `sops`, Anda dapat mengabaikan `enkripsi.env` dan memakai `.env` lokal.
- Folder `data/` berisi file internal PostgreSQL. Pastikan backup sebelum menghapus atau memindahkannya.
- Jika terjadi masalah permission di Debian, periksa ownership dan permission pada folder `data/`.
- Pastikan tidak ada service lain yang sedang memakai port `5432`.
