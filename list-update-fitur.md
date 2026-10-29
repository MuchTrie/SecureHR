# Roadmap Pengembangan Fitur SecureHR

Dokumen ini digunakan sebagai daftar rencana pengembangan fitur lanjutan
SecureHR sekaligus latihan Git dengan memisahkan setiap fitur ke branch yang
berbeda.

## Cara menggunakan daftar ini

1. Pilih satu fitur yang belum selesai.
2. Buat branch khusus untuk fitur tersebut.
3. Kerjakan backend dan frontend secara terukur.
4. Jalankan test, build, dan lint yang relevan.
5. Push branch ke remote.
6. Buat Pull Request menuju `main`.
7. Tandai checklist setelah fitur selesai di-merge.

Contoh:

```bash
git switch main
git pull origin main
git switch -c feature/employee-filter
```

---

## Fitur yang sudah diimplementasikan

### Authentication

- [X] Login
- [X] Refresh token
- [X] Logout
- [X] Current user/profile
- [X] Role `ADMIN`
- [X] Role `MANAGER`
- [X] Role `EMPLOYEE`
- [X] Aktivasi dan deaktivasi user

### Employee management

- [X] Melihat daftar employee
- [X] Melihat detail employee
- [X] Membuat employee
- [X] Mengubah employee
- [X] Menonaktifkan employee
- [X] Relasi department
- [X] Relasi position
- [X] Relasi manager

### Master data

- [X] CRUD department
- [X] CRUD position
- [X] Manager department
- [X] Jumlah employee per department

### Attendance

- [X] Check-in
- [X] Check-out
- [X] Status `PRESENT`
- [X] Status `LATE`
- [X] Daftar attendance
- [X] Filter attendance berdasarkan employee
- [X] Filter attendance berdasarkan tanggal

### Leave request dan approval

- [X] Employee membuat pengajuan cuti
- [X] Employee melihat pengajuan miliknya
- [X] Manager melihat pengajuan direct report
- [X] Admin melihat seluruh pengajuan
- [X] Approve cuti
- [X] Reject cuti dengan alasan
- [X] Cancel cuti berstatus `PENDING`
- [X] Validasi tanggal
- [X] Validasi overlap cuti
- [X] Authorization berdasarkan role
- [X] Status `PENDING`
- [X] Status `APPROVED`
- [X] Status `REJECTED`
- [X] Status `CANCELLED`

### Dashboard

- [X] Total employee
- [X] Total department
- [X] Total user
- [X] Employee hadir hari ini
- [X] Employee tidak hadir hari ini
- [X] Jumlah pending leave

### Struktur backend

- [X] Entry point ringkas di `cmd/server/main.go`
- [X] Package server di `internal/server`
- [X] Package database di `internal/database`
- [X] Package config di `internal/config`
- [X] Package middleware di `internal/middleware`
- [X] Package model di `internal/model`
- [X] Package seeder di `internal/seeder`
- [X] Versioned database bootstrap
- [X] Dokumentasi backend dan frontend diperbarui

---

## Prioritas tinggi

## 1. Filter, pencarian, dan pagination employee

**Branch:**

```text
feature/employee-filter
```

**Tujuan:** Memudahkan pengguna menemukan employee ketika jumlah data
bertambah.

Checklist:

- [X] Search berdasarkan nama
- [X] Search berdasarkan email
- [X] Search berdasarkan employee number
- [X] Filter berdasarkan department
- [X] Filter berdasarkan position
- [X] Filter berdasarkan gender
- [X] Filter berdasarkan status employee
- [X] Filter berdasarkan rentang tanggal bergabung
- [X] Pagination di database
- [X] Ukuran halaman 10, 25, dan 50
- [X] Sorting berdasarkan seluruh kolom employee yang diizinkan
- [X] Query parameter tidak disimpan di URL
- [X] Search dengan debounce sekitar 300 ms
- [X] Empty state ketika tidak ada hasil
- [X] Backend validation untuk query parameter
- [X] Role scope Admin, Manager, dan Employee
- [ ] Test query filter dan pagination

---

## 2. Aturan cuti dan kuota cuti

**Branch:**

```text
feature/leave-rules
```

**Tujuan:** Mengembangkan workflow cuti agar sesuai kebijakan HR perusahaan.

Checklist:

- [X] Kuota cuti tahunan employee
- [X] Perhitungan sisa cuti
- [X] Jenis cuti dengan aturan berbeda
- [X] Validasi jumlah hari cuti
- [X] Validasi weekend
- [X] Validasi hari libur melalui company holiday
- [X] Pencegahan cuti melebihi kuota
- [X] Edit pengajuan selama masih `PENDING`
- [X] Riwayat perubahan status cuti
- [X] Multi-level approval melalui konfigurasi policy
- [X] Delegasi approval manager berbasis periode
- [X] Test aturan bisnis cuti dasar (quota, weekend, dan company holiday)
- [X] Modal aturan cuti pada halaman pengajuan
- [X] Pesan error pengajuan cuti yang mudah dipahami pengguna
- [X] Perapian responsive halaman Leave, kartu saldo, filter, dan tabel
- [X] Format tanggal leave yang readable pada tabel
- [X] Validasi production build frontend setelah perubahan UI

---

## 3. Dashboard statistik lanjutan

**Branch:**

```text
feature/dashboard-statistics
```

Checklist:

- [X] Grafik employee per department
- [X] Grafik status attendance
- [X] Statistik leave per bulan
- [X] Statistik employee aktif dan nonaktif
- [X] Statistik keterlambatan
- [X] Filter preset berdasarkan periode
- [X] Dashboard berdasarkan role
- [X] Endpoint agregasi khusus dashboard
- [X] Loading state
- [X] Error state
- [X] Empty state

Catatan implementasi:

- Backend dashboard menggunakan `GET /api/v1/dashboard/stats` dengan preset
  `this_month`, `last_3_months`, dan `this_year`.
- Admin melihat agregasi seluruh perusahaan, Manager melihat direct reports,
  dan Employee melihat data pribadi.
- Frontend menampilkan kartu ringkasan, bar chart department, status
  attendance, leave bulanan, status employee, serta tombol retry ketika API
  gagal.
- UI dashboard menggunakan KPI card responsive dengan tinggi seragam,
  typography konsisten, dan link langsung ke halaman employee atau pending
  leave untuk role yang memiliki akses.
- Tabel recent attendance Employee menampilkan tanggal `dd MMM yyyy`, waktu
  `HH:mm`, status badge, dan empty state.
- Sidebar diringkas dengan menghapus heading `Menu` tanpa mengubah role scope
  navigasi.
- Loading dashboard menggunakan skeleton card khusus yang menyerupai bentuk
  KPI aktual.

---

## Prioritas menengah

## 4. Kalender attendance

**Branch:**

```text
feature/attendance-calendar
```

Checklist:

- [X] Tampilan attendance dalam kalender
- [X] Navigasi bulan
- [X] Rekap record attendance dalam bulan aktif
- [X] Warna status hadir
- [X] Warna status terlambat
- [X] Warna status tidak hadir
- [ ] Warna status izin
- [ ] Warna status sakit
- [X] Detail check-in dan check-out
- [X] Filter employee
- [X] Filter department

Catatan implementasi:

- Route frontend: `/attendance/calendar`.
- Kalender memakai `GET /api/v1/attendance` dengan `date_from` dan `date_to`,
  serta `GET /api/v1/holidays?year=YYYY`.
- ADMIN melihat seluruh data, MANAGER melihat direct reports, dan EMPLOYEE
  melihat data miliknya sendiri.
- Status `IZIN` dan `SAKIT` belum tersedia pada model/status backend, sehingga
  pewarnaannya menunggu pengembangan status attendance tambahan.

---

## 5. Hari libur perusahaan

**Branch:**

```text
feature/company-holidays
```

Checklist:

- [x] Model `CompanyHoliday`
- [x] Migration atau update schema
- [x] CRUD hari libur
- [x] Nama hari libur
- [x] Tanggal hari libur
- [x] Validasi tanggal duplikat
- [x] Halaman admin hari libur
- [x] Integrasi dengan kalender attendance
- [x] Integrasi dengan pengajuan cuti
- [x] Authorization khusus Admin

Catatan implementasi:

- Model dan migration sudah tersedia melalui versioned bootstrap leave rules.
- Endpoint yang tersedia: `GET /holidays`, `POST /holidays`,
  `PUT /holidays/:id`, dan `DELETE /holidays/:id`.
- Halaman frontend berada pada route `/holidays` dan menu ditampilkan untuk
  Admin. Admin dapat menambah, mengubah, dan menghapus holiday; role lain
  hanya membaca holiday melalui endpoint dan kalender attendance.
- Tanggal duplikat ditolak oleh unique index database dan response conflict.
- Holiday dipakai oleh validasi pengajuan cuti dan ditampilkan pada kalender
  attendance.

---

## 6. Pengaturan jam kerja

**Branch:**

```text
feature/work-schedule
```

Checklist:

- [ ] Pengaturan jam masuk
- [ ] Pengaturan jam pulang
- [ ] Toleransi keterlambatan
- [ ] Perhitungan durasi kerja
- [ ] Status pulang lebih awal
- [ ] Shift kerja
- [ ] Jadwal berbeda per department
- [ ] Integrasi aturan dengan check-in
- [ ] Test perhitungan waktu

---

## 7. Export laporan

**Branch:**

```text
feature/report-export
```

Checklist:

- [ ] Export employee ke CSV
- [ ] Export attendance ke CSV
- [ ] Export leave ke CSV
- [ ] Filter laporan berdasarkan tanggal
- [ ] Filter laporan berdasarkan department
- [ ] Filter laporan berdasarkan employee
- [ ] Filter laporan berdasarkan status
- [ ] Nama file export yang informatif
- [ ] Download file dari frontend
- [ ] Export PDF
- [ ] Print laporan
- [ ] Authorization data laporan

---

## 8. Manajemen password

**Branch:**

```text
feature/password-management
```

Checklist:

- [ ] Ubah password sendiri
- [ ] Validasi password lama
- [ ] Konfirmasi password baru
- [ ] Admin reset password user
- [ ] Invalidasi refresh token lama
- [ ] Pesan sukses dan error
- [ ] Validasi kekuatan password
- [ ] Test password workflow

---

## Prioritas lanjutan

## 9. Notifikasi aplikasi

**Branch:**

```text
feature/notifications
```

Checklist:

- [ ] Model notification
- [ ] Notifikasi pengajuan cuti baru
- [ ] Notifikasi cuti disetujui
- [ ] Notifikasi cuti ditolak
- [ ] Notifikasi perubahan profile
- [ ] Daftar notifikasi
- [ ] Badge jumlah belum dibaca
- [ ] Tandai satu notifikasi sudah dibaca
- [ ] Tandai seluruh notifikasi sudah dibaca
- [ ] Polling atau refresh notifikasi

---

## 10. Audit log

**Branch:**

```text
feature/audit-log
```

Checklist:

- [ ] Model audit log
- [ ] Mencatat login
- [ ] Mencatat perubahan employee
- [ ] Mencatat approval cuti
- [ ] Mencatat perubahan status user
- [ ] Menyimpan actor
- [ ] Menyimpan action
- [ ] Menyimpan entity dan entity ID
- [ ] Menyimpan waktu aksi
- [ ] Menyimpan IP address jika diperlukan
- [ ] Halaman audit log untuk Admin
- [ ] Filter audit log
- [ ] Audit log tidak dapat diubah melalui API biasa

Contoh field:

```text
actor_id
action
entity
entity_id
description
ip_address
created_at
```

---

## 11. Import employee dari CSV

**Branch:**

```text
feature/employee-import
```

Checklist:

- [ ] Upload file CSV
- [ ] Validasi ukuran file
- [ ] Validasi tipe file
- [ ] Validasi header CSV
- [ ] Preview data sebelum disimpan
- [ ] Validasi email
- [ ] Validasi employee number
- [ ] Validasi department
- [ ] Validasi position
- [ ] Menampilkan baris yang gagal
- [ ] Menyimpan data valid dengan transaction
- [ ] Menampilkan ringkasan hasil import

---

## 12. Profile management

**Branch:**

```text
feature/profile-management
```

Checklist:

- [ ] Edit nomor telepon
- [ ] Edit alamat
- [ ] Edit tanggal lahir
- [ ] Edit jenis kelamin
- [ ] Upload foto profile
- [ ] Validasi field profile
- [ ] Preview foto sebelum upload
- [ ] Hapus foto profile

---

## 13. Organization chart

**Branch:**

```text
feature/organization-chart
```

Checklist:

- [ ] Tampilan struktur manager dan employee
- [ ] Organization chart
- [ ] Filter berdasarkan department
- [ ] Detail direct report
- [ ] Navigasi ke detail employee
- [ ] Empty state untuk employee tanpa manager

---

## 14. Employee history

**Branch:**

```text
feature/employee-history
```

Checklist:

- [ ] Riwayat department
- [ ] Riwayat position
- [ ] Riwayat manager
- [ ] Riwayat attendance
- [ ] Riwayat leave
- [ ] Riwayat perubahan data
- [ ] Filter riwayat berdasarkan periode
- [ ] Timeline perubahan employee

---

## Testing dan kualitas kode

## Containerisasi lokal dengan Docker

Status: **Selesai untuk development lokal**

File containerisasi:

- `docker-compose.yml` di root workspace.
- `backend/Dockerfile` dan `backend/.dockerignore`.
- `frontend/Dockerfile`, `frontend/nginx.conf`, dan
  `frontend/.dockerignore`.

Stack lokal terdiri dari:

```text
frontend:5173 → backend:8080 → mysql:3306
```

Perintah utama:

```powershell
docker compose up -d --build
docker compose ps
docker compose logs -f backend
docker compose down
```

MySQL menggunakan named volume `mysql_data`. Akses CLI:

```powershell
docker compose exec mysql mysql -u securehr -p securehr
```

Containerisasi ini baru mencakup pengujian lokal. Deployment EC2, RDS, Nginx
reverse proxy, HTTPS, dan CI/CD belum termasuk implementasi fitur ini.

## 15. Test leave approval

**Branch:**

```text
test/leave-approval
```

Checklist backend:

- [ ] Test membuat leave valid
- [ ] Test tanggal tidak valid
- [ ] Test overlap leave
- [ ] Test employee hanya melihat leave sendiri
- [ ] Test manager hanya melihat direct report
- [ ] Test admin melihat seluruh leave
- [ ] Test approve leave
- [ ] Test reject tanpa alasan
- [ ] Test reject dengan alasan
- [ ] Test cancel leave
- [ ] Test invalid status transition

Checklist frontend:

- [ ] Test form leave
- [ ] Test validasi tanggal
- [ ] Test reject modal
- [ ] Test tombol berdasarkan role
- [ ] Test loading state
- [ ] Test error state
- [ ] Test success state

---

## 16. Test authentication

**Branch:**

```text
test/authentication
```

Checklist:

- [ ] Test login valid
- [ ] Test login dengan password salah
- [ ] Test login user nonaktif
- [ ] Test access token expired
- [ ] Test refresh token
- [ ] Test refresh token expired
- [ ] Test logout
- [ ] Test role middleware

---

## 17. Test employee management

**Branch:**

```text
test/employee-management
```

Checklist:

- [ ] Test membuat employee
- [ ] Test employee number duplikat
- [ ] Test email duplikat
- [ ] Test update employee
- [ ] Test menonaktifkan employee
- [ ] Test akses Admin
- [ ] Test akses non-Admin

---

## Urutan implementasi yang direkomendasikan

```text
1. feature/employee-filter
2. feature/leave-rules
3. feature/dashboard-statistics
4. feature/attendance-calendar
5. feature/company-holidays
6. feature/work-schedule
7. feature/report-export
8. feature/password-management
9. feature/notifications
10. feature/audit-log
11. feature/employee-import
12. feature/profile-management
13. feature/organization-chart
14. feature/employee-history
15. test/leave-approval
16. test/authentication
17. test/employee-management
```

## Template pengerjaan setiap fitur

### 1. Mulai dari `main`

```bash
git switch main
git pull origin main
```

### 2. Buat branch fitur

```bash
git switch -c feature/nama-fitur
```

### 3. Implementasi

- [ ] Update backend
- [ ] Update frontend
- [ ] Update model atau database jika diperlukan
- [ ] Update API wrapper
- [ ] Update type frontend
- [ ] Update dokumentasi `APPLICATION_CONTEXT.md`
- [ ] Tambahkan test

### 4. Validasi

```bash
# Backend
cd backend
go test ./...

# Frontend
cd frontend
npm run build
npm run lint
```

### 5. Commit

```bash
git status
git diff
git add <file-yang-diubah>
git commit -m "feat: deskripsi fitur"
```

### 6. Push dan Pull Request

```bash
git push -u origin feature/nama-fitur
```

Buat Pull Request dari `feature/nama-fitur` menuju `main`, lalu lakukan review
sebelum merge.

### 7. Setelah merge

```bash
git switch main
git pull origin main
git branch -d feature/nama-fitur
git push origin --delete feature/nama-fitur
```
