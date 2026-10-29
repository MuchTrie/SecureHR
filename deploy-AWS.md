# Panduan Deploy SecureHR ke 2 AWS EC2 (Ubuntu Server)

Panduan ini digunakan jika dua instance EC2 Ubuntu sudah dibuat:

- **EC2 Frontend**: menjalankan container React/Vite hasil build dengan Nginx.
- **EC2 Backend**: menjalankan container Go API dan container MySQL.

Database untuk sementara ditempatkan di EC2 Backend. Arsitektur ini cocok untuk
latihan atau staging, bukan rancangan production final. Untuk production,
database sebaiknya dipindahkan ke Amazon RDS dan aplikasi diberi domain,
HTTPS, serta reverse proxy atau load balancer.

## 1. Arsitektur dan nilai yang perlu disiapkan

```text
Browser
   |
   | http://PUBLIC_FRONTEND_IP:5173
   v
EC2 Frontend
└── securehr-frontend (container, port 80)
   |
   | HTTP API dari browser
   | http://PUBLIC_BACKEND_IP:8080/api/v1
   v
EC2 Backend
├── securehr-backend (container, port 8080)
└── securehr-mysql (container, port 3306 internal saja)
```

Siapkan dua alamat berikut sebelum mulai:

```text
PUBLIC_FRONTEND_IP = public IPv4 EC2 Frontend
PUBLIC_BACKEND_IP  = public IPv4 EC2 Backend
```

Gunakan **private IP** atau private DNS antar-instance jika konfigurasi
jaringan AWS sudah menyediakannya. Namun, karena browser pengguna harus
mengakses API secara langsung pada tahap ini, `VITE_API_URL` tetap harus
menggunakan alamat backend yang dapat dijangkau browser.

## 2. Atur Security Group AWS

### Security Group EC2 Frontend

| Port | Protocol | Source                  | Kegunaan                |
| ---: | :------: | ----------------------- | ----------------------- |
|   22 |   TCP   | IP publik Anda `/32`  | SSH                     |
| 5173 |   TCP   | `0.0.0.0/0` sementara | Akses aplikasi frontend |

### Security Group EC2 Backend

| Port | Protocol | Source                  | Kegunaan                   |
| ---: | :------: | ----------------------- | -------------------------- |
|   22 |   TCP   | IP publik Anda `/32`  | SSH                        |
| 8080 |   TCP   | `0.0.0.0/0` sementara | API yang dipanggil browser |

Jangan membuka port `3306` pada Security Group. MySQL hanya perlu diakses oleh
container backend melalui network Docker internal. Setelah memakai domain atau
load balancer, batasi port publik sesuai desain akhir dan jangan membiarkan
port development terbuka tanpa kebutuhan.

## 3. Masuk ke EC2 Ubuntu

Jalankan dari komputer lokal. Ganti nama key dan alamat sesuai instance:

```bash
chmod 400 securehr-key.pem
ssh -i securehr-key.pem ubuntu@PUBLIC_FRONTEND_IP
ssh -i securehr-key.pem ubuntu@PUBLIC_BACKEND_IP
```

Default username Ubuntu EC2 biasanya `ubuntu`. Jika AMI yang digunakan
berbeda, gunakan username yang disediakan oleh AMI tersebut.

## 4. Instal tools di kedua EC2

Jalankan bagian ini **di EC2 Frontend dan EC2 Backend**:

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y ca-certificates curl git
```

Instal Docker Engine dan Docker Compose plugin dari repository resmi Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

Aktifkan Docker dan izinkan user `ubuntu` menjalankan Docker tanpa `sudo`:

```bash
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
```

Keluar dari SSH lalu masuk kembali agar perubahan group aktif:

```bash
exit
ssh -i securehr-key.pem ubuntu@PUBLIC_EC2_IP
```

Verifikasi instalasi:

```bash
docker --version
docker compose version
git --version
```

## 5. Ambil subfolder source code di masing-masing EC2

Jika repository private, pastikan autentikasi Git sudah disiapkan di server.
Jangan menaruh access token GitHub langsung di dalam file deployment atau
commit ke repository.

Repository ini memiliki dua subfolder aplikasi, yaitu `frontend` dan
`backend`. Karena Git melakukan clone pada level repository, gunakan sparse
checkout agar hanya subfolder yang dibutuhkan yang diambil di setiap EC2.

### EC2 Backend

Jalankan hanya di **EC2 Backend**:

```bash
cd ~
git clone --filter=blob:none --sparse https://github.com/MuchTrie/DevOps_Training.git backend
cd ~/backend
git sparse-checkout set backend
```

Source backend sekarang berada di `~/backend/backend`.

### EC2 Frontend

Jalankan hanya di **EC2 Frontend**:

```bash
cd ~
git clone --filter=blob:none --sparse https://github.com/MuchTrie/DevOps_Training.git frontend
cd ~/frontend
git sparse-checkout set frontend
```

Source frontend sekarang berada di `~/frontend/frontend`.

Jika ingin memakai branch tertentu, jalankan pada folder clone yang sesuai:

```bash
git checkout NAMA_BRANCH
git pull --ff-only
```

Jika repository diakses melalui SSH, gunakan URL clone SSH sesuai konfigurasi
akun Anda.

## 6. Konfigurasi EC2 Backend: subfolder backend + MySQL

### 6.1 Buat file Compose khusus backend

Pada **EC2 Backend**, buat file `~/backend/docker-compose.aws-backend.yml`.
File Compose berada satu level di atas source `~/backend/backend`:

```bash
cd ~/backend
nano docker-compose.aws-backend.yml
```

Isi dengan konfigurasi berikut. Ganti semua nilai bertanda `<...>` sebelum
menjalankan Compose:

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: securehr-mysql
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: securehr
      MYSQL_USER: securehr
      MYSQL_PASSWORD: <DB_APP_PASSWORD>
      MYSQL_ROOT_PASSWORD: <DB_ROOT_PASSWORD>
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -p$${MYSQL_ROOT_PASSWORD}"]
      interval: 5s
      timeout: 5s
      retries: 20

  backend:
    build:
      context: ./backend/backend
    container_name: securehr-backend
    restart: unless-stopped
    environment:
      APP_ENV: staging
      HOST: 0.0.0.0
      PORT: 8080
      GIN_MODE: release
      DATABASE_DRIVER: mysql
      DATABASE_DSN: securehr:<DB_APP_PASSWORD>@tcp(mysql:3306)/securehr?charset=utf8mb4&parseTime=True&loc=Local
      JWT_SECRET: <LONG_RANDOM_JWT_SECRET>
      ACCESS_TOKEN_MINUTES: 15
      REFRESH_TOKEN_DAYS: 7
      CORS_ALLOWED_ORIGINS: http://<PUBLIC_FRONTEND_IP>:5173
      CORS_ALLOWED_METHODS: GET,POST,PUT,PATCH,DELETE,OPTIONS
      CORS_ALLOWED_HEADERS: Origin,Content-Type,Accept,Authorization
      CORS_ALLOW_CREDENTIALS: "true"
    ports:
      - "8080:8080"
    depends_on:
      mysql:
        condition: service_healthy

volumes:
  mysql_data:
```

Pastikan `<DB_APP_PASSWORD>` sama di `MYSQL_PASSWORD` dan `DATABASE_DSN`.
`DATABASE_DSN` menggunakan hostname `mysql`, bukan `localhost` dan bukan
public IP backend, karena kedua container berada pada network Compose yang
sama.

### 6.2 Generate secret

Buat nilai secret secara aman di EC2 Backend dan salin ke file Compose:

```bash
openssl rand -base64 48
openssl rand -base64 32
```

Gunakan hasil pertama untuk password database dan hasil kedua untuk
`JWT_SECRET`, atau buat password terpisah untuk aplikasi dan root. Hindari
karakter khusus yang dapat mengubah format DSN, terutama `@`, `:`, `/`, `?`,
dan `#`, kecuali Anda memahami aturan escaping DSN.

Batasi permission file Compose karena file ini berisi password:

```bash
chmod 600 ~/backend/docker-compose.aws-backend.yml
```

### 6.3 Build dan jalankan backend

```bash
cd ~/backend
docker compose -f docker-compose.aws-backend.yml up -d --build
docker compose -f docker-compose.aws-backend.yml ps
```

Periksa log:

```bash
docker compose -f docker-compose.aws-backend.yml logs --tail=100 mysql
docker compose -f docker-compose.aws-backend.yml logs --tail=100 backend
```

Tes dari EC2 Backend:

```bash
curl -i http://127.0.0.1:8080/api/v1
```

Respons `404` dari root API masih menunjukkan bahwa port dan server dapat
dijangkau. Yang penting container backend berjalan dan tidak terus restart.
Tes endpoint login dari komputer lokal setelah konfigurasi selesai:

```bash
curl -i http://PUBLIC_BACKEND_IP:8080/api/v1/auth/login
```

## 7. Konfigurasi EC2 Frontend

Frontend memakai `VITE_API_URL` saat **build**, sehingga perubahan URL setelah
container dibuat tidak akan mengubah aplikasi yang sudah dibuild.

Pada **EC2 Frontend**, build image dengan URL backend publik:

```bash
cd ~/frontend
docker build \
  --build-arg VITE_API_URL=http://<PUBLIC_BACKEND_IP>:8080/api/v1 \
  -t securehr-frontend:staging ./frontend/frontend
```

Jalankan container frontend:

```bash
docker rm -f securehr-frontend 2>/dev/null || true
docker run -d \
  --name securehr-frontend \
  --restart unless-stopped \
  -p 5173:80 \
  securehr-frontend:staging
```

Verifikasi:

```bash
docker ps
docker logs --tail=100 securehr-frontend
curl -I http://127.0.0.1:5173
```

Frontend dan backend berada di dua Docker host berbeda. Karena itu frontend
tidak boleh menggunakan hostname `backend` atau `mysql`; browser harus
menggunakan URL:

```text
http://PUBLIC_BACKEND_IP:8080/api/v1
```

## 8. Pengujian end-to-end

Buka alamat berikut di browser:

```text
http://PUBLIC_FRONTEND_IP:5173
```

Uji alur berikut:

1. Halaman login dapat dibuka.
2. Login dengan akun seed development berhasil.
3. Dashboard memuat data.
4. Browser dapat memanggil `/auth/me`.
5. Attendance dapat dibuka.
6. Leave dapat dibaca atau diajukan.
7. Tidak ada error CORS pada Developer Tools browser.

Akun seed development yang tersedia:

```text
admin@example.com / change-me-admin
manager@example.com / change-me-manager
employee@example.com / change-me-employee
```

Akun tersebut hanya untuk latihan. Ganti atau nonaktifkan seeder sebelum
deployment production dan segera ganti password setelah pengujian.

## 9. Update aplikasi

### Update backend

Pada EC2 Backend:

```bash
cd ~/backend
git pull --ff-only
docker compose -f docker-compose.aws-backend.yml up -d --build backend
docker compose -f docker-compose.aws-backend.yml ps
```

Jangan gunakan `docker compose down -v` untuk update biasa. Opsi `-v`
menghapus volume MySQL dan seluruh data development.

### Update frontend

Pada EC2 Frontend:

```bash
cd ~/frontend
git pull --ff-only
docker build \
  --build-arg VITE_API_URL=http://<PUBLIC_BACKEND_IP>:8080/api/v1 \
  -t securehr-frontend:staging ./frontend/frontend
docker rm -f securehr-frontend
docker run -d \
  --name securehr-frontend \
  --restart unless-stopped \
  -p 5173:80 \
  securehr-frontend:staging
```

Jika public IP berubah, perbarui sekaligus:

1. `VITE_API_URL` saat build frontend.
2. `CORS_ALLOWED_ORIGINS` pada Compose backend.
3. Security Group dan URL yang digunakan pengguna.

## 10. Backup database dasar

Jalankan pada EC2 Backend. Perintah ini menulis dump ke home user, bukan ke
container:

```bash
cd ~/backend
docker compose -f docker-compose.aws-backend.yml exec -T mysql \
  mysqldump -u securehr -p'<DB_APP_PASSWORD>' securehr \
  > ~/securehr-db-$(date +%Y%m%d-%H%M%S).sql
```

Simpan hasil dump di lokasi yang aman dan jangan commit ke Git. Untuk
production, gunakan backup terjadwal dan pindahkan database ke RDS.

## 11. Troubleshooting

### Frontend terbuka tetapi login gagal

- Pastikan `VITE_API_URL` berisi `http://PUBLIC_BACKEND_IP:8080/api/v1`.
- Frontend harus dibuild ulang setelah URL berubah.
- Pastikan port `8080` dibuka pada Security Group backend.
- Periksa error CORS di Developer Tools browser.
- Pastikan `CORS_ALLOWED_ORIGINS` persis sama dengan origin frontend,
  termasuk port `5173`.

### Backend container berhenti atau restart

```bash
docker compose -f docker-compose.aws-backend.yml ps
docker compose -f docker-compose.aws-backend.yml logs --tail=200 backend
docker compose -f docker-compose.aws-backend.yml logs --tail=200 mysql
```

Pastikan MySQL sudah `healthy`, DSN memakai `mysql:3306`, dan password pada
`MYSQL_PASSWORD` sama dengan password pada `DATABASE_DSN`.

### Tidak dapat mengakses aplikasi dari internet

- Periksa status instance dan public IPv4.
- Periksa Security Group inbound.
- Pastikan container mem-publish port yang benar:

```bash
docker ps
```

- Pastikan aplikasi bind ke `0.0.0.0`, bukan hanya `127.0.0.1`.

## 12. Checklist deployment

- [ ] Dua EC2 Ubuntu sudah aktif dan memiliki public IPv4 atau DNS.
- [ ] SSH hanya terbuka dari IP administrator.
- [ ] Docker Engine dan Docker Compose plugin terpasang di kedua EC2.
- [ ] Source code sudah diambil pada kedua server.
- [ ] `docker-compose.aws-backend.yml` hanya berisi backend dan MySQL.
- [ ] Password database dan JWT secret sudah diganti dari nilai development.
- [ ] Port `3306` tidak terbuka pada Security Group.
- [ ] Port `8080` backend dapat dijangkau browser.
- [ ] `CORS_ALLOWED_ORIGINS` berisi origin frontend yang tepat.
- [ ] Frontend dibuild dengan URL API backend yang tepat.
- [ ] Volume `mysql_data` digunakan agar data tidak hilang saat container dibuat ulang.
- [ ] Login, dashboard, attendance, dan leave sudah diuji.
- [ ] Backup database sudah dibuat sebelum pengujian atau perubahan besar.

## 13. Catatan keamanan

Deployment ini masih menggunakan HTTP dan public IP. Token autentikasi dan
data aplikasi belum dilindungi TLS. Jangan gunakan konfigurasi ini sebagai
production final tanpa:

- domain dan HTTPS;
- reverse proxy atau load balancer;
- database RDS dengan backup;
- secret manager;
- monitoring dan log terpusat;
- pembatasan akses Security Group;
- penggantian akun seed dan password development.
