# 📋 Aplikasi Reservasi (PHP + Bootstrap + MySQL)

Aplikasi reservasi berbasis web dengan Docker Compose.

## ✨ Fitur

- Sistem reservasi online
- Responsive design dengan Bootstrap
- Database MySQL/MariaDB
- Auto-create database dan tabel

## 📁 Struktur File

```
appreservasi/
├── index.php         # Halaman utama
├── config.php        # Konfigurasi database
├── save.php          # Endpoint untuk menyimpan data
├── css/
├── js/
├── img/
└── README.md
```

## ⚙️ Instalasi & Menjalankan

### Metode 1: Docker Compose (Rekomendasi)

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
   git clone https://github.com/paknux/appReservasi.git
   cd appReservasi
   ```

2. **Cek apakah port 8081 sudah digunakan:**
   ```bash
   sudo lsof -i :8081
   ```
   Jika port 8081 sudah digunakan, edit `docker-compose.yml` dan ubah port mapping:
   ```yaml
   ports:
     - "8082:80"  # atau port lain yang tersedia
   ```

3. **Buat file Dockerfile** (jika belum ada):
   ```bash
   sudo nano Dockerfile
   ```
   Isi dengan:
   ```dockerfile
   FROM php:8.2-apache

   # Install mysqli extension untuk koneksi MySQL/MariaDB
   RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

   # Enable Apache mod_rewrite
   RUN a2enmod rewrite

   # Set working directory
   WORKDIR /var/www/html

   # Copy semua file aplikasi ke container
   COPY . /var/www/html/

   # Set permission
   RUN chown -R www-data:www-data /var/www/html

   # Expose port 80
   EXPOSE 80

   # Start Apache di foreground
   CMD ["apache2-foreground"]
   ```

4. **Buat file docker-compose.yml** (jika belum ada):
   ```bash
   sudo nano docker-compose.yml
   ```
   Isi dengan:
   ```yaml
   version: '3.8'

   services:
     webserver:
       build: .
       container_name: appreservasi-web
       ports:
         - "8081:80"
       depends_on:
         - dbserver
       environment:
         - DB_HOST=dbserver
         - DB_PORT=3306
         - DB_USER=root
         - DB_PASS=rootpassword
         - DB_NAME=db_reservasi
       networks:
         - app-network

     dbserver:
       image: mariadb:11-jammy
       container_name: appreservasi-db
       environment:
         MYSQL_ROOT_PASSWORD: rootpassword
         MYSQL_DATABASE: db_reservasi
       volumes:
         - db_data:/var/lib/mysql
       networks:
         - app-network

   networks:
     app-network:
       driver: bridge

   volumes:
     db_data:
   ```

5. **Update config.php untuk support environment variables:**
   ```bash
   sudo nano config.php
   ```
   Ubah konfigurasi database menjadi:
   ```php
   // Support environment variables untuk Docker deployment
   define('DB_HOST', getenv('DB_HOST') ?: 'localhost');
   define('DB_PORT', getenv('DB_PORT') ?: '3306');
   define('DB_USER', getenv('DB_USER') ?: 'root');
   define('DB_PASS', getenv('DB_PASS') ?: '');
   define('DB_NAME', getenv('DB_NAME') ?: 'db_reservasi');
   ```

6. **Build dan jalankan containers:**
   ```bash
   docker-compose up -d --build
   ```

7. **Cek status containers:**
   ```bash
   docker-compose ps
   ```

8. **Lihat logs jika ada error:**
   ```bash
   docker-compose logs webserver
   docker-compose logs dbserver
   ```

9. **Akses aplikasi di browser:**
   ```
   http://localhost:8081
   ```
   Atau dari device lain di jaringan yang sama:
   ```
   http://<IP-KOMPUTER-ANDA>:8081
   ```

10. **Perintah manajemen Docker Compose:**
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

### Metode 2: Setup Tradisional (Apache + PHP + MariaDB Manual)

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
   sudo git clone https://github.com/paknux/appReservasi.git /var/www/html/appreservasi/
   cd /var/www/html/appreservasi/
   ```

3. **Setup dan konfigurasi MariaDB:**
   ```bash
   # Start MariaDB service
   sudo systemctl start mariadb
   sudo systemctl enable mariadb

   # Secure MariaDB installation (set password root, dll)
   sudo mysql_secure_installation
   ```
   Saat diminta:
   - Set root password? **Y** (masukkan password yang aman)
   - Remove anonymous users? **Y**
   - Disallow root login remotely? **Y** (opsional, untuk development bisa N)
   - Remove test database? **Y**
   - Reload privilege tables now? **Y**

   Atau untuk setup cepat tanpa mysql_secure_installation:
   ```bash
   sudo mysql
   ```
   Lalu jalankan perintah SQL berikut:
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'password_aman_anda';
   FLUSH PRIVILEGES;
   EXIT;
   ```

4. **Buka file `config.php`, sesuaikan kredensial database:**
   ```bash
   sudo nano config.php
   ```
   Edit bagian konfigurasi database sesuai password yang sudah diset:
   ```php
   define('DB_HOST', 'localhost');
   define('DB_PORT', '3306');
   define('DB_USER', 'root');
   define('DB_PASS', 'password_aman_anda');  // Ganti dengan password yang diset
   define('DB_NAME', 'db_reservasi');
   ```
   > Anda **tidak perlu** membuat database/tabel secara manual — aplikasi
   > akan membuatnya otomatis saat pertama kali diakses/menyimpan data.

5. **Set permission folder:**
   ```bash
   sudo chown -R www-data:www-data /var/www/html/appreservasi
   sudo chmod -R 755 /var/www/html/appreservasi
   ```

6. **Restart Apache:**
   ```bash
   sudo systemctl restart apache2
   ```

7. **Akses aplikasi dari browser:**
   ```
   http://localhost/appreservasi
   ```
   Atau dari device lain di jaringan yang sama:
   ```
   http://<IP-KOMPUTER-ANDA>/appreservasi
   ```

## 🗄️ Struktur Tabel (dibuat otomatis)

Aplikasi akan membuat tabel secara otomatis saat pertama kali dijalankan.

## 🛠️ Teknologi

- PHP (native, `mysqli`)
- Bootstrap 5 (CDN)
- MySQL / MariaDB
- Vanilla JavaScript
- Docker & Docker Compose

## 📄 Lisensi

Bebas digunakan dan dimodifikasi untuk keperluan pembelajaran/internal.

## 🐛 Troubleshooting

### Error: "Access denied for user 'root'@'localhost'"
- Pastikan password di config.php sama dengan password MariaDB
- Cek koneksi: `sudo mysql -u root -p`
- Reset password jika perlu:
  ```bash
  sudo mysql
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'password_baru';
  FLUSH PRIVILEGES;
  EXIT;
  ```

### Error: Port already in use (Docker)
- Cek port yang digunakan: `sudo lsof -i :8081`
- Ubah port di docker-compose.yml ke port lain (misal 8082)

### Error: Database connection failed
- Pastikan MariaDB berjalan: `sudo systemctl status mariadb`
- Cek konfigurasi DB_HOST, DB_USER, DB_PASS di config.php
- Untuk Docker, pastikan environment variables di docker-compose.yml benar

### Permission denied
- Pastikan permission folder benar:
  ```bash
  sudo chown -R www-data:www-data /var/www/html/appreservasi
  sudo chmod -R 755 /var/www/html/appreservasi
  ```
