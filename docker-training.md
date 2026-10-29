# Memahami Docker Compose SecureHR

## 1. Hal penting: bukan satu container yang berisi tiga container

Pada tampilan Docker Desktop, terlihat satu kelompok bernama:

```text
pt_much_sejahtera
```

Kelompok tersebut bukan sebuah container yang di dalamnya terdapat tiga
container. `pt_much_sejahtera` adalah nama **project Docker Compose**.

Di dalam satu project Compose terdapat tiga container yang berdiri sendiri:

```text
Project Docker Compose: pt_much_sejahtera
│
├── Container: mysql
├── Container: backend
└── Container: frontend
```

Docker Desktop mengelompokkan ketiganya agar lebih mudah dilihat dan dikelola.
Secara teknis, masing-masing tetap merupakan container terpisah dengan proses,
filesystem, dan konfigurasi sendiri.

## 2. Apa itu container?

Container adalah lingkungan terisolasi untuk menjalankan satu aplikasi atau
satu service. Container tidak sama dengan virtual machine penuh. Container
berbagi kernel sistem operasi host, tetapi proses dan filesystem aplikasinya
dipisahkan.

Dalam project SecureHR:

| Container    | Tugas                                      | Port container |
| ------------ | ------------------------------------------ | -------------: |
| `mysql`    | Menyimpan data aplikasi                    |       `3306` |
| `backend`  | Menjalankan API Go/Gin                     |       `8080` |
| `frontend` | Menyajikan hasil build React melalui Nginx |         `80` |

Setiap container memiliki tanggung jawab berbeda. Inilah alasan frontend,
backend, dan database dipisahkan, bukan dimasukkan ke satu container besar.

## 3. Apa itu Docker Compose?

Docker Compose adalah alat untuk mendefinisikan dan menjalankan beberapa
container sebagai satu aplikasi.

Pada file `docker-compose.yml`, setiap bagian di bawah `services` adalah satu
service yang biasanya akan dibuat menjadi satu container:

```yaml
services:
  mysql:
    image: mysql:8.4

  backend:
    build:
      context: ./backend

  frontend:
    build:
      context: ./frontend
```

Secara sederhana:

```text
docker-compose.yml
        │
        ├── service mysql    → container mysql
        ├── service backend  → container backend
        └── service frontend → container frontend
```

Saat menjalankan:

```powershell
docker compose up -d
```

Docker Compose akan:

1. Membaca file `docker-compose.yml`.
2. Membuat network untuk project.
3. Membuat atau mengambil image yang dibutuhkan.
4. Membuat container `mysql`.
5. Menunggu MySQL berstatus sehat.
6. Membuat dan menjalankan container `backend`.
7. Membuat dan menjalankan container `frontend`.
8. Menghubungkan ketiganya ke network yang sama.

Saat menjalankan:

```powershell
docker compose down
```

Compose akan menghentikan dan menghapus container serta network project.
Named volume database tidak dihapus kecuali perintah menggunakan `-v`.

## 4. Memahami tampilan Docker Desktop

Tampilan pada gambar dapat dibaca seperti ini:

```text
pt_much_sejahtera
│
├── mysql
│   └── mysql:8.4
│
├── backend
│   └── pt_much_sejahtera-backend
│
└── frontend
    └── pt_much_sejahtera-frontend
```

### `mysql`

```text
mysql:8.4
3306:3306
```

`mysql:8.4` adalah image resmi MySQL yang digunakan untuk membuat container
database.

Mapping `3306:3306` berarti:

```text
Port komputer host 3306 → Port MySQL di dalam container 3306
```

Karena port tersebut dipublikasikan, aplikasi dari komputer host seperti
MySQL Workbench atau DBeaver dapat mengakses MySQL melalui `localhost:3306`.

### `backend`

```text
pt_much_sejahtera-backend
8080:8080
```

Backend dibuat dari `backend/Dockerfile`. Aplikasi Go berjalan pada port `8080`
di dalam container.

Mapping tersebut berarti:

```text
Port komputer host 8080 → Port backend di dalam container 8080
```

Backend dapat diuji melalui:

```text
http://localhost:8080
```

### `frontend`

```text
pt_much_sejahtera-frontend
5173:80
```

Frontend dibuat dari `frontend/Dockerfile`. Build React disajikan oleh Nginx
yang berjalan pada port `80` di dalam container.

Mapping tersebut berarti:

```text
Port komputer host 5173 → Port Nginx di dalam container 80
```

Frontend dapat dibuka melalui:

```text
http://localhost:5173
```

Port host dan port container tidak harus sama. Contoh `5173:80` berarti
pengguna membuka port `5173`, sedangkan Nginx tetap mendengarkan port `80`
di dalam container.

## 5. Bagaimana ketiga container saling berkomunikasi?

Compose membuat network internal, misalnya:

```text
pt_much_sejahtera_default
```

Ketiga container masuk ke network tersebut:

```text
┌───────────────────────────────────────────┐
│ Docker network: pt_much_sejahtera_default │
│                                           │
│  frontend ──────── backend ──────── mysql │
└───────────────────────────────────────────┘
```

Backend tidak mengakses database menggunakan `localhost`. Di dalam container
backend, `localhost` berarti container backend itu sendiri.

Karena itu, connection string backend menggunakan hostname service Compose:

```text
tcp(mysql:3306)
```

Docker menyediakan DNS internal sehingga nama `mysql` diterjemahkan ke alamat
container MySQL.

Alur backend ke database:

```text
backend container
        │
        │ hostname: mysql
        │ port: 3306
        ▼
mysql container
```

Frontend memiliki kondisi yang sedikit berbeda. Request API dibuat oleh
browser pengguna, bukan oleh proses Nginx di dalam frontend container.
Karena itu, URL frontend menggunakan:

```text
VITE_API_URL=http://localhost:8080/api/v1
```

Browser mengakses port backend yang dipublikasikan ke host:

```text
Browser
  ├── membuka frontend: http://localhost:5173
  └── memanggil API:    http://localhost:8080/api/v1
```

## 6. Perbedaan image, container, service, dan project

Istilah-istilah ini sering membingungkan:

### Image

Image adalah template read-only untuk membuat container.

Contoh:

```text
mysql:8.4
pt_much_sejahtera-backend
pt_much_sejahtera-frontend
```

### Container

Container adalah instance yang sedang menjalankan image.

Contoh:

```text
pt_much_sejahtera-mysql-1
pt_much_sejahtera-backend-1
pt_much_sejahtera-frontend-1
```

### Service

Service adalah definisi aplikasi di `docker-compose.yml`. Satu service dapat
membuat satu atau beberapa container, walaupun pada project ini setiap service
membuat satu container.

### Project

Project adalah kumpulan service yang dikelola bersama oleh Docker Compose.
Dalam kasus ini nama project-nya `pt_much_sejahtera`.

Hubungannya:

```text
Project
  └── Service
      └── Image
          └── Container yang berjalan
```

## 7. Kenapa tidak dibuat satu container saja?

Secara teknis, ketiga aplikasi dapat dipaksa masuk ke satu container, tetapi
itu bukan pola yang disarankan.

Jika semua digabung:

```text
1 container
├── MySQL
├── Backend Go
└── Nginx frontend
```

maka akan muncul beberapa masalah:

- Satu container harus menjalankan banyak proses.
- Restart backend dapat mengganggu proses lain.
- Scaling frontend dan backend tidak dapat dilakukan secara terpisah.
- Log lebih sulit dipisahkan.
- Lifecycle database bercampur dengan lifecycle aplikasi.
- Perawatan dan debugging menjadi lebih sulit.

Dengan pemisahan container:

```text
3 container
├── mysql
├── backend
└── frontend
```

setiap bagian dapat:

- Dibuild secara terpisah.
- Direstart secara terpisah.
- Dilihat log-nya secara terpisah.
- Diuji secara terpisah.
- Diganti image-nya secara terpisah.

## 8. Apa yang terjadi ketika container dihentikan?

Container memiliki lifecycle sendiri:

```text
image
  ↓
create
  ↓
start
  ↓
running
  ↓
stop
  ↓
remove
```

Jika container MySQL dihapus tanpa volume, data database dapat hilang.
Compose menggunakan named volume:

```text
mysql_data:/var/lib/mysql
```

Volume terpisah dari lifecycle container. Jadi ketika menjalankan:

```powershell
docker compose down
```

container MySQL dihapus, tetapi data tetap tersimpan di volume.

Jika menjalankan:

```powershell
docker compose down -v
```

volume juga dihapus dan data MySQL development akan hilang.

## 9. Perintah latihan yang penting

Jalankan seluruh stack:

```powershell
docker compose up -d --build
```

Lihat semua container dalam project:

```powershell
docker compose ps
```

Lihat semua container Docker, termasuk dari project lain:

```powershell
docker ps -a
```

Lihat log satu service:

```powershell
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f mysql
```

Masuk ke shell container backend:

```powershell
docker compose exec backend sh
```

Masuk ke MySQL melalui container:

```powershell
docker compose exec mysql mysql -u securehr -p securehr
```

Password development:

```text
securehr-dev-password
```

Menghentikan stack:

```powershell
docker compose down
```

## 10. Ringkasan

Cara membaca struktur Docker SecureHR:

```text
pt_much_sejahtera
└── Docker Compose project
    ├── mysql service
    │   └── mysql container
    ├── backend service
    │   └── backend container
    └── frontend service
        └── frontend container
```

Jadi kalimat yang tepat adalah:

> Docker Compose membuat dan mengelola tiga container terpisah sebagai satu
> project aplikasi SecureHR.

Bukan:

> Satu container berisi tiga container.

Project Compose membuat ketiganya terasa seperti satu kesatuan karena mereka
dijalankan dengan satu perintah, berada dalam satu network, dan tampil dalam
satu kelompok di Docker Desktop. Namun secara teknis, ketiganya tetap
terisolasi dan dapat dikelola secara terpisah.

## 11. Penjelasan file Docker utama

Containerisasi SecureHR menggunakan tiga file utama:

```text
backend/Dockerfile
frontend/Dockerfile
docker-compose.yml
```

Masing-masing file memiliki fungsi berbeda:

| File                    | Fungsi                                         |
| ----------------------- | ---------------------------------------------- |
| `backend/Dockerfile`  | Membuat image dan menjalankan API Go           |
| `frontend/Dockerfile` | Membuat image dan menyajikan hasil build React |
| `docker-compose.yml`  | Mengatur tiga service agar berjalan bersama    |

## 12. Penjelasan `backend/Dockerfile`

Isi file:

```dockerfile
FROM golang:1.24-alpine AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /out/securehr ./cmd/server

FROM alpine:3.22

WORKDIR /app
RUN apk add --no-cache ca-certificates

COPY --from=builder /out/securehr ./securehr

EXPOSE 8080
CMD ["./securehr"]
```

### Penjelasan setiap bagian

#### `FROM golang:1.24-alpine AS builder`

Menggunakan image Go versi 1.24 berbasis Alpine Linux sebagai tahap
kompilasi. Nama `builder` adalah alias untuk tahap ini.

Tahap builder hanya digunakan untuk menghasilkan binary aplikasi, bukan
sebagai image akhir. Ini disebut **multi-stage build**.

#### `WORKDIR /src`

Menentukan folder kerja di dalam image builder. Perintah berikutnya akan
dijalankan dari folder `/src`.

#### `COPY go.mod go.sum ./`

Menyalin file dependency Go terlebih dahulu. Pemisahan ini membantu Docker
menggunakan cache build ketika source code berubah tetapi dependency tidak
berubah.

#### `RUN go mod download`

Mengunduh seluruh dependency yang tercantum dalam `go.mod` dan `go.sum`.

#### `COPY . .`

Menyalin source code backend ke folder kerja `/src`. File yang masuk tetap
dibatasi oleh `backend/.dockerignore`.

#### `RUN CGO_ENABLED=0 GOOS=linux go build ...`

Mengompilasi aplikasi Go:

- `CGO_ENABLED=0` menghasilkan binary tanpa ketergantungan CGO.
- `GOOS=linux` menargetkan sistem operasi Linux.
- `-o /out/securehr` menyimpan binary dengan nama `securehr`.
- `./cmd/server` menunjukkan package entry point aplikasi.

Hasil tahap ini adalah binary backend di:

```text
/out/securehr
```

#### `FROM alpine:3.22`

Memulai tahap image runtime baru yang lebih kecil. Source code dan compiler Go
tidak ikut dibawa ke image akhir.

#### `WORKDIR /app`

Menentukan folder kerja aplikasi di image runtime.

#### `RUN apk add --no-cache ca-certificates`

Memasang certificate authority agar aplikasi dapat melakukan koneksi HTTPS
keluar jika diperlukan. Opsi `--no-cache` mencegah cache package disimpan di
image.

#### `COPY --from=builder /out/securehr ./securehr`

Menyalin hanya binary hasil kompilasi dari tahap `builder` ke image runtime.
Inilah manfaat utama multi-stage build: image akhir tidak memuat source code
dan toolchain Go.

#### `EXPOSE 8080`

Mendokumentasikan bahwa aplikasi mendengarkan pada port `8080` di dalam
container. `EXPOSE` tidak otomatis membuat port dapat diakses dari host;
pemetaan host dilakukan di `docker-compose.yml`.

#### `CMD ["./securehr"]`

Menentukan proses utama yang dijalankan ketika container backend dimulai.

Alur backend:

```text
Source Go
   ↓
Go builder image
   ↓ go build
Binary securehr
   ↓
Alpine runtime image
   ↓
Backend container berjalan di port 8080
```

## 13. Penjelasan `frontend/Dockerfile`

Isi file:

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
ARG VITE_API_URL=http://localhost:8080/api/v1
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build

FROM nginx:1.29-alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Penjelasan setiap bagian

#### `FROM node:22-alpine AS builder`

Menggunakan Node.js 22 berbasis Alpine untuk tahap build frontend. Node.js
diperlukan untuk menjalankan npm dan Vite.

#### `WORKDIR /app`

Menentukan folder kerja frontend di dalam image builder.

#### `COPY package*.json ./`

Menyalin `package.json` dan lock file npm jika tersedia. Dependency disalin
sebelum source code agar layer `npm ci` dapat digunakan kembali oleh cache
Docker.

#### `RUN npm ci`

Menginstal dependency secara konsisten berdasarkan lock file. `npm ci`
digunakan untuk build yang lebih reproducible dibanding `npm install`.

#### `COPY . .`

Menyalin source code frontend ke image builder. `node_modules`, `dist`, dan
file yang dikecualikan berada di `frontend/.dockerignore` tidak ikut disalin.

#### `ARG VITE_API_URL=...`

Mendefinisikan build argument untuk URL API. Nilai default digunakan jika
Compose tidak memberikan nilai lain.

#### `ENV VITE_API_URL=${VITE_API_URL}`

Menyediakan build argument sebagai environment variable agar dapat dibaca oleh
Vite ketika menjalankan build.

Karena Vite mengompilasi nilai `VITE_*` ke JavaScript, perubahan URL API
membutuhkan build image frontend baru.

#### `RUN npm run build`

Menjalankan script build frontend. Hasilnya berupa file statis di:

```text
/app/dist
```

#### `FROM nginx:1.29-alpine`

Memulai tahap runtime baru menggunakan Nginx. Node.js dan source code tidak
ikut masuk ke image akhir.

#### `COPY nginx.conf ...`

Menyalin konfigurasi Nginx ke lokasi konfigurasi default. Konfigurasi tersebut
mengarahkan request route frontend yang tidak berupa file fisik kembali ke
`index.html`, sehingga React Router tetap bekerja ketika halaman direfresh.

#### `COPY --from=builder /app/dist ...`

Menyalin hasil build statis dari tahap `builder` ke folder default Nginx:

```text
/usr/share/nginx/html
```

#### `EXPOSE 80`

Mendokumentasikan bahwa Nginx mendengarkan pada port `80` di dalam container.
Compose memetakan port tersebut ke port `5173` di host.

#### `CMD ["nginx", "-g", "daemon off;"]`

Menjalankan Nginx pada foreground. Proses utama harus tetap berjalan di
foreground agar Docker menganggap container masih aktif.

Alur frontend:

```text
Source React
   ↓
Node builder image
   ↓ npm run build
Folder dist
   ↓
Nginx runtime image
   ↓
Frontend container berjalan di port 80
```

## 14. Penjelasan `docker-compose.yml`

`docker-compose.yml` tidak membuat image aplikasi secara langsung. File ini
menjadi instruksi untuk mengatur image, container, network, port, environment,
dependency startup, dan volume.

### Service `mysql`

```yaml
mysql:
  image: mysql:8.4
```

Menggunakan image MySQL resmi versi 8.4. Karena tidak memiliki bagian `build`,
Compose mengambil image dari registry jika image tersebut belum tersedia
secara lokal.

```yaml
environment:
  MYSQL_DATABASE: securehr
  MYSQL_USER: securehr
  MYSQL_PASSWORD: securehr-dev-password
  MYSQL_ROOT_PASSWORD: root-dev-password
```

Environment variable ini digunakan oleh image MySQL saat inisialisasi pertama:

- Membuat database `securehr`.
- Membuat user aplikasi `securehr`.
- Menetapkan password user aplikasi.
- Menetapkan password root.

Credential ini hanya untuk development lokal.

```yaml
ports:
  - "3306:3306"
```

Memetakan port:

```text
Port host 3306 → Port MySQL container 3306
```

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

Menyimpan data MySQL pada named volume `mysql_data`. Data tidak hilang saat
container dihentikan dengan `docker compose down`.

```yaml
healthcheck:
  test: ["CMD-SHELL", "mysqladmin ping ..."]
  interval: 5s
  timeout: 5s
  retries: 20
```

Memeriksa apakah MySQL benar-benar siap menerima koneksi, bukan hanya apakah
proses container sudah berjalan.

### Service `backend`

```yaml
backend:
  build:
    context: ./backend
```

Memerintahkan Compose membangun image menggunakan `backend/Dockerfile`.
`context: ./backend` berarti Docker dapat mengakses file dalam folder backend
sebagai konteks build.

```yaml
environment:
  APP_ENV: development
  PORT: 8080
  DATABASE_DRIVER: mysql
  DATABASE_DSN: ...@tcp(mysql:3306)/securehr...
```

Memberikan konfigurasi runtime kepada backend. Hal penting pada DSN adalah:

```text
tcp(mysql:3306)
```

`mysql` adalah nama service di Compose. Docker menyediakan DNS internal agar
backend dapat menemukan container database melalui nama tersebut.

```yaml
ports:
  - "8080:8080"
```

Memetakan port host `8080` ke port backend `8080` di dalam container.

```yaml
depends_on:
  mysql:
    condition: service_healthy
```

Meminta Compose menunggu health check MySQL berstatus sehat sebelum memulai
backend.

### Service `frontend`

```yaml
frontend:
  build:
    context: ./frontend
    args:
      VITE_API_URL: http://localhost:8080/api/v1
```

Memerintahkan Compose membangun image menggunakan `frontend/Dockerfile` dan
memberikan URL API saat build frontend.

Nilai `localhost` di sini ditujukan untuk browser pengguna. Browser mengakses
backend melalui port host `8080`, bukan melalui hostname internal Docker
`backend`.

```yaml
ports:
  - "5173:80"
```

Memetakan:

```text
Port host 5173 → Port Nginx container 80
```

Karena itu aplikasi dibuka melalui:

```text
http://localhost:5173
```

```yaml
depends_on:
  - backend
```

Menyatakan bahwa frontend bergantung pada backend untuk urutan startup.
Frontend tetap hanya menyajikan file statis; request API dilakukan browser
setelah aplikasi dibuka.

### Volume `mysql_data`

```yaml
volumes:
  mysql_data:
```

Mendeklarasikan named volume yang digunakan service MySQL. Volume merupakan
penyimpanan terpisah dari container:

```text
Container MySQL dihapus
        ↓
Volume mysql_data tetap ada
        ↓
Container MySQL berikutnya memakai data yang sama
```

Volume baru ikut dihapus jika menjalankan:

```powershell
docker compose down -v
```

## 15. Hubungan ketiga file

Ketiga file bekerja dalam urutan berikut:

```text
docker-compose.yml
       │
       ├── backend service
       │      └── membaca backend/Dockerfile
       │             └── menghasilkan backend image
       │
       ├── frontend service
       │      └── membaca frontend/Dockerfile
       │             └── menghasilkan frontend image
       │
       └── mysql service
              └── memakai image mysql:8.4
```

Ketika menjalankan:

```powershell
docker compose up -d --build
```

prosesnya menjadi:

1. Compose membaca tiga service.
2. Compose membangun image backend dari `backend/Dockerfile`.
3. Compose membangun image frontend dari `frontend/Dockerfile`.
4. Compose mengambil image `mysql:8.4`.
5. Compose membuat network project.
6. Compose membuat tiga container.
7. Compose memasang environment, port, dependency, dan volume.
8. Compose menjalankan ketiga container sesuai urutan dependency.

Dengan demikian, Dockerfile menjelaskan cara membuat image masing-masing
aplikasi, sedangkan `docker-compose.yml` menjelaskan cara menjalankan semua
image tersebut sebagai satu stack lokal.
