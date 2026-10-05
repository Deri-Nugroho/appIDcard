# 📇 Aplikasi ID Card / Passport Sederhana (PHP + Bootstrap + MySQL)

Aplikasi sederhana untuk membuat ID Card digital berbasis web, responsive,
dark mode, dan didesain khusus agar **muat dalam satu layar HP (portrait)
tanpa perlu scroll**.

## ✨ Fitur

- Input biodata: **Nama, Kelas, Absen, JK (Jenis Kelamin), Nomer HP**
- Setelah "difoto", otomatis menampilkan **ID Card / Passport** dalam mode
  portrait, dark mode, satu kartu (card) penuh tanpa header/footer.
- Menampilkan info **AWS Environment** di bagian bawah kartu: EC2 Instance
  ID, EC2 Public IP, RDS Endpoint, Region/AZ. Data ini **diambil langsung
  dari AWS EC2 Instance Metadata Service (IMDSv2)** jika aplikasi memang
  dijalankan di EC2 (mis. AWS Academy Lab). Jika tidak terdeteksi berjalan
  di EC2 (mis. testing di localhost/laptop), nilai yang tidak tersedia akan
  ditampilkan sebagai **`-`** (tidak ada data dummy/palsu yang dipaksakan),
  dan kartu akan menampilkan badge **LIVE** (data asli) atau **N/A**
  (metadata tidak terdeteksi).
- Foto pada ID Card ditampilkan **besar dan sesuai rasio asli gambar**
  (tidak dicrop paksa ke rasio tertentu).
- **Database, tabel, dan struktur tabel dibuat otomatis** oleh aplikasi
  saat pertama kali dijalankan (tidak perlu import SQL manual).
- Tombol **"Buat Ulang"** untuk kembali ke form dan input data baru.

## 📁 Struktur File

```
idcard-app/
├── konfig.php        # Semua konfigurasi (DB & info AWS dummy) + auto-create DB/table
├── index.php         # Halaman utama (single page app: form → kamera → kartu)
├── save.php          # Endpoint AJAX: simpan biodata ke DB, return JSON
├── css/
│   └── style.css     # Styling dark mode & layout fit 1 layar HP
├── js/
│   └── app.js        # Logika kamera & render kartu
├── img/
└── README.md
```

## ⚙️ Instalasi & Menjalankan

### 1. Persyaratan
- PHP 7.4+ dengan ekstensi `mysqli`
- MySQL / MariaDB Server
- Web server: Apache2

### 2. Setup

#### Prasyarat Sistem:
- PHP 7.4+ dengan ekstensi `mysqli`
- MySQL / MariaDB Server
- Web server: Apache2

#### Langkah-langkah Instalasi (Ubuntu/Debian):

1. **Update system dan install dependencies:**
   ```bash
   sudo apt update
   sudo apt install -y apache2 php php-mysqli mariadb-server git
   ```

2. **Clone repository dari GitHub ke folder web server:**
   ```bash
   sudo git clone https://github.com/Deri-Nugroho/appIDcard.git /var/www/html/idcard-app/
   cd /var/www/html/idcard-app/
   ```

3. **Buka file `konfig.php`, sesuaikan kredensial database:**
   ```bash
   sudo nano konfig.php
   ```
   Edit bagian konfigurasi database:
   ```php
   define('DB_HOST', 'localhost');
   define('DB_PORT', '3306');
   define('DB_USER', 'root');
   define('DB_PASS', '');
   define('DB_NAME', 'db_idcard');
   define('DB_TABLE', 'biodata');
   ```
   > Anda **tidak perlu** membuat database/tabel secara manual — aplikasi
   > akan membuatnya otomatis saat pertama kali diakses/menyimpan data.

4. **Set permission folder:**
   ```bash
   sudo chown -R www-data:www-data /var/www/html/idcard-app
   sudo chmod -R 755 /var/www/html/idcard-app
   ```

5. **Restart Apache:**
   ```bash
   sudo systemctl restart apache2
   ```

6. **Akses aplikasi dari browser:**
   ```
   http://localhost/idcard-app
   ```
   Atau dari device lain di jaringan yang sama:
   ```
   http://<IP-KOMPUTER-ANDA>/idcard-app
   ```

### 2b. Setup dengan Docker Compose

#### Prasyarat Sistem:
- Docker Engine terinstall
- Docker Compose terinstall

#### Langkah-langkah Instalasi Docker (Ubuntu/Debian):

1. **Install Docker:**
   ```bash
   sudo apt update
   sudo apt install -y docker.io docker-compose
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker $USER
   ```
   > Logout dan login kembali agar group docker aktif

2. **Verifikasi instalasi Docker:**
   ```bash
   docker --version
   docker-compose --version
   ```

#### Langkah-langkah Deploy Aplikasi:

1. **Clone repository:**
   ```bash
   git clone https://github.com/Deri-Nugroho/appIDcard.git
   cd appIDcard
   ```

2. **Cek apakah port 8080 sudah digunakan:**
   ```bash
   sudo lsof -i :8080
   ```
   Jika port 8080 sudah digunakan, edit `docker-compose.yml` dan ubah port mapping:
   ```yaml
   ports:
     - "8081:80"  # atau port lain yang tersedia
   ```

3. **Build dan jalankan containers:**
   ```bash
   docker-compose up -d --build
   ```

4. **Cek status containers:**
   ```bash
   docker-compose ps
   ```

5. **Lihat logs jika ada error:**
   ```bash
   docker-compose logs webserver
   docker-compose logs dbserver
   ```

6. **Akses aplikasi di browser:**
   ```
   http://localhost:8081
   ```
   Atau dari device lain di jaringan yang sama:
   ```
   http://<IP-KOMPUTER-ANDA>:8081
   ```

7. **Perintah manajemen Docker Compose:**
   - Stop containers:
     ```bash
     docker-compose stop
     ```
   - Start containers:
     ```bash
     docker-compose start
     ```
   - Hentikan dan hapus containers:
     ```bash
     docker-compose down
     ```
   - Hentikan dan hapus containers beserta volumes database:
     ```bash
     docker-compose down -v
     ```
   - Rebuild containers:
     ```bash
     docker-compose up -d --build
     ```

### 3. Cara Pakai

1. Isi form biodata (Nama, Kelas, Absen, JK, Nomer HP).
2. Tekan tombol **📷 Ambil Foto** 
3. Setelah selesai, data otomatis tersimpan ke database dan tampilan
   berpindah ke **ID Card** lengkap dengan foto (sesuai JK) dan info
   AWS Environment di bagian bawah kartu.
4. Tekan **🔄 Buat Ulang** untuk kembali ke form dan input data baru.

## ⚠️ Catatan Penting

- Info AWS (EC2 Instance ID, IP, Region/AZ) **otomatis diambil dari data
  ASLI** melalui **AWS EC2 Instance Metadata Service versi 2 (IMDSv2)**
  — lihat fungsi `getAwsInfo()` di `konfig.php`. Ini akan berfungsi
  otomatis jika aplikasi dijalankan di instance EC2 (termasuk EC2 dari
  AWS Academy Lab), tanpa perlu konfigurasi tambahan.
  - **RDS Endpoint** yang ditampilkan adalah `DB_HOST` sungguhan dari
    `konfig.php` (bukan dummy) — yaitu endpoint database yang benar-benar
    dipakai aplikasi untuk konek.
  - Jika dijalankan **bukan di EC2** (mis. di localhost/laptop biasa),
    request ke metadata service (`169.254.169.254`) akan gagal/timeout
    (percobaan dibatasi ±0.4 detik agar tidak lama), dan field yang tidak
    berhasil dibaca akan ditampilkan sebagai **`-`** (bukan nilai
    dummy/palsu). Badge pada kartu akan menunjukkan **LIVE** (data asli)
    atau **N/A** (metadata tidak terdeteksi) sesuai kondisi ini.
  - Tidak ada lagi konstanta AWS dummy di `konfig.php` — semua nilai AWS
    murni hasil pembacaan metadata real-time.

## 🗄️ Struktur Tabel `biodata` (dibuat otomatis)

| Kolom      | Tipe                          | Keterangan            |
|------------|-------------------------------|------------------------|
| id         | INT AUTO_INCREMENT PRIMARY KEY | ID unik               |
| nama       | VARCHAR(100)                  | Nama lengkap          |
| kelas      | VARCHAR(50)                   | Kelas                 |
| absen      | VARCHAR(10)                   | Nomor absen           |
| jk         | ENUM('Laki-Laki','Perempuan') | Jenis kelamin         |
| no_hp      | VARCHAR(20)                   | Nomor HP              |
| foto_path  | VARCHAR(255)                  | Path foto             |
| created_at | TIMESTAMP                     | Waktu data dibuat     |

## 🛠️ Teknologi

- PHP (native, `mysqli`)
- Bootstrap 5 (CDN)
- MySQL / MariaDB
- Vanilla JavaScript (Fetch API untuk AJAX)
- Dark mode custom CSS, layout `100dvh` agar pas 1 layar tanpa scroll

## 📄 Lisensi

Bebas digunakan dan dimodifikasi untuk keperluan pembelajaran/internal.
