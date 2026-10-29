# Product Requirements Document (PRD)

## SecureHR — Pengembangan Fitur Lanjutan

**Status:** Draft
**Versi:** 1.1
**Tanggal:** 16 Oktober 2026
**Pemilik:** Tim Pengembang SecureHR
**Platform:** Web application
**Target pengguna:** Admin, Manager, dan Employee

---

## 1. Ringkasan produk

SecureHR adalah aplikasi pengelolaan sumber daya manusia yang membantu
perusahaan mengelola data karyawan, department, posisi, pengguna, absensi, dan
pengajuan cuti.

Pengembangan lanjutan akan dilakukan secara bertahap menggunakan branch Git
terpisah untuk setiap fitur. Fitur pertama yang menjadi prioritas adalah
**Approval Cuti**, kemudian dilanjutkan dengan pencarian karyawan, dashboard
statistik, kalender absensi, laporan, notifikasi, dan audit log.

Dokumen ini menjadi acuan untuk:

- Menentukan tujuan dan batasan setiap fitur.
- Menentukan kebutuhan pengguna dan aturan bisnis.
- Memecah pekerjaan menjadi branch dan Pull Request yang terukur.
- Menyamakan kriteria selesai sebelum sebuah fitur digabungkan ke `main`.

---

## 2. Latar belakang dan masalah

Versi aplikasi saat ini sudah memiliki fondasi untuk data karyawan, absensi,
dan cuti. Namun, proses bisnis lanjutan masih perlu dikembangkan:

1. Pengajuan cuti belum memiliki alur persetujuan yang lengkap.
2. Pengguna kesulitan menemukan karyawan tertentu ketika data bertambah.
3. Pengelola belum mendapatkan ringkasan statistik HR secara cepat.
4. Riwayat absensi dan cuti belum mudah dibaca dalam bentuk kalender atau
   laporan.
5. Aktivitas penting pengguna belum dapat ditelusuri melalui audit log.

Tanpa fitur tersebut, aplikasi hanya berfungsi sebagai pencatatan data dan
belum mendukung workflow HR yang lengkap.

---

## 3. Tujuan produk

### Tujuan utama

Membangun workflow HR yang lebih terstruktur, aman, dan mudah digunakan dengan
memprioritaskan proses approval cuti.

### Tujuan pembelajaran Git

Setiap fitur harus dikerjakan pada branch terpisah dan digabungkan melalui Pull
Request:

```text
main
  ├── feature/leave-approval
  ├── feature/employee-filter
  ├── feature/dashboard-statistics
  ├── feature/attendance-calendar
  ├── feature/report-export
  ├── feature/notifications
  └── feature/audit-log
```

### Indikator keberhasilan

- Pengguna dapat mengajukan dan memproses cuti sesuai role.
- Setiap fitur memiliki branch, commit, dan PR yang fokus.
- Akses endpoint terlindungi oleh autentikasi dan authorization.
- Validasi dilakukan di frontend dan backend.
- Perubahan tidak merusak fitur yang sudah tersedia.
- Fitur dapat diuji berdasarkan acceptance criteria yang jelas.

---

## 4. Persona dan role

### 4.1 Employee

Karyawan yang mengelola informasi pribadi, absensi, dan pengajuan cuti miliknya
sendiri.

### 4.2 Manager

Atasan yang melihat pengajuan cuti dari anggota team yang dikelolanya dan
memberikan keputusan.

### 4.3 Admin

Pengguna dengan kewenangan mengelola data master, pengguna, seluruh data
karyawan, serta membantu proses administrasi.

### Matriks akses ringkas

| Kemampuan                 | Employee |      Manager | Admin |
| ------------------------- | -------: | -----------: | ----: |
| Melihat profil sendiri    |       Ya |           Ya |    Ya |
| Mengajukan cuti sendiri   |       Ya |           Ya |    Ya |
| Melihat pengajuan sendiri |       Ya |           Ya |    Ya |
| Menyetujui cuti team      |    Tidak |           Ya |    Ya |
| Mengelola data karyawan   |    Tidak |     Terbatas |    Ya |
| Melihat seluruh laporan   |    Tidak | Team sendiri |    Ya |
| Melihat audit log         |    Tidak |        Tidak |    Ya |

---

## 5. Ruang lingkup roadmap

### Fase 1 — MVP: Approval Cuti

Branch utama:

```text
feature/leave-approval
```

Mencakup pengajuan, validasi, daftar pengajuan, persetujuan, penolakan, dan
riwayat status cuti.

### Fase 2 — Efisiensi data karyawan

Branch:

```text
feature/employee-filter
feature/dashboard-statistics
```

Mencakup pencarian, filter, pagination, sorting, dan statistik ringkas HR.

### Fase 3 — Pengalaman absensi

Branch:

```text
feature/company-holidays
feature/attendance-calendar
feature/work-schedule
```

Mencakup kalender absensi, hari libur, jam kerja, keterlambatan, dan rekap
durasi kerja.

### Fase 4 — Pelaporan dan komunikasi

Branch:

```text
feature/report-export
feature/notifications
```

Mencakup export data dan notifikasi aktivitas penting.

### Fase 5 — Kontrol dan integritas data

Branch:

```text
feature/audit-log
feature/employee-import
```

Mencakup pencatatan aktivitas dan import data karyawan secara batch.

> Fase setelah MVP dapat disesuaikan berdasarkan hasil evaluasi dan tidak
> wajib dikerjakan seluruhnya dalam satu iterasi.

---

## 6. Spesifikasi MVP: Approval Cuti

### 6.1 Tujuan fitur

Menyediakan proses pengajuan dan persetujuan cuti yang dapat dilacak, dengan
akses berbeda sesuai role pengguna.

### 6.2 User stories

#### Employee

- Sebagai employee, saya ingin mengajukan cuti dengan jenis, tanggal, dan
  alasan agar permintaan saya tercatat.
- Sebagai employee, saya ingin melihat status pengajuan agar mengetahui
  apakah cuti saya menunggu, disetujui, atau ditolak.
- Sebagai employee, saya ingin membatalkan pengajuan yang masih `PENDING`.

#### Manager

- Sebagai manager, saya ingin melihat pengajuan dari anggota team saya.
- Sebagai manager, saya ingin menyetujui pengajuan yang valid.
- Sebagai manager, saya ingin menolak pengajuan dengan memberikan alasan.

#### Admin

- Sebagai admin, saya ingin melihat seluruh pengajuan cuti.
- Sebagai admin, saya ingin menyetujui atau menolak pengajuan jika diperlukan
  untuk keperluan administrasi.
- Sebagai admin, saya ingin memfilter pengajuan berdasarkan status, employee,
  dan rentang tanggal.

### 6.3 Alur utama

```text
Employee mengisi form cuti
  -> Backend memvalidasi input
  -> Status dibuat PENDING
  -> Manager/Admin melihat pengajuan
  -> APPROVED atau REJECTED
  -> Employee melihat hasil keputusan
```

### 6.4 Status pengajuan

| Status        | Keterangan               | Perubahan yang diizinkan                                       |
| ------------- | ------------------------ | -------------------------------------------------------------- |
| `PENDING`   | Menunggu keputusan       | Manager/Admin dapat approve atau reject; employee dapat cancel |
| `APPROVED`  | Pengajuan disetujui      | Tidak dapat diubah melalui alur normal                         |
| `REJECTED`  | Pengajuan ditolak        | Tidak dapat diubah melalui alur normal                         |
| `CANCELLED` | Dibatalkan oleh employee | Status akhir                                                   |

Transisi yang valid:

```text
PENDING -> APPROVED
PENDING -> REJECTED
PENDING -> CANCELLED
```

### 6.5 Aturan bisnis

1. Hanya pengguna yang terautentikasi yang dapat membuat pengajuan.
2. Employee hanya dapat membuat dan melihat pengajuan miliknya sendiri.
3. Manager hanya dapat memproses pengajuan anggota team yang menjadi
   tanggung jawabnya.
4. Admin dapat melihat dan memproses seluruh pengajuan.
5. `start_date` wajib lebih kecil atau sama dengan `end_date`.
6. `leave_type`, tanggal, dan `reason` wajib diisi.
7. Pengajuan yang sudah `APPROVED`, `REJECTED`, atau `CANCELLED` tidak dapat
   diproses ulang.
8. Hanya pengajuan `PENDING` yang dapat disetujui, ditolak, atau dibatalkan.
9. Penolakan wajib menyimpan alasan keputusan.
10. `approved_by` dan `approved_at` diisi ketika manager atau admin memberikan
    keputusan.
11. Pengguna tidak boleh menyetujui pengajuan miliknya sendiri.
12. Sistem harus mencegah pengajuan yang tanggalnya bertabrakan dengan
    pengajuan aktif milik employee yang sama jika aturan tersebut diaktifkan.

### 6.6 Perubahan data yang diperlukan

Model `LeaveRequest` saat ini sudah memiliki `Status`, `ApprovedBy`, dan
`ApprovedAt`. Untuk mendukung MVP, tambahkan secara terukur:

```text
rejection_reason  nullable text
cancelled_at     nullable datetime
```

Pertimbangkan constraint atau validasi untuk:

- Status yang hanya memiliki nilai yang dikenal.
- Tanggal mulai dan tanggal selesai.
- Relasi employee dan approver.

Perubahan database harus dibuat sebagai migration baru, bukan mengubah migration
lama yang sudah pernah dijalankan.

### 6.7 Rancangan endpoint

Prefix API: `/api/v1`

| Method    | Endpoint                        | Pengguna              | Tujuan                  |
| --------- | ------------------------------- | --------------------- | ----------------------- |
| `GET`   | `/leave-requests`             | Semua role            | Daftar sesuai hak akses |
| `GET`   | `/leave-requests/:id`         | Pemilik/Manager/Admin | Detail pengajuan        |
| `POST`  | `/leave-requests`             | Semua role            | Membuat pengajuan       |
| `PATCH` | `/leave-requests/:id/approve` | Manager/Admin         | Menyetujui pengajuan    |
| `PATCH` | `/leave-requests/:id/reject`  | Manager/Admin         | Menolak pengajuan       |
| `PATCH` | `/leave-requests/:id/cancel`  | Pemilik/Admin         | Membatalkan pengajuan   |

Parameter daftar yang direncanakan:

```text
status
employee_id
start_date
end_date
page
limit
```

Contoh payload membuat pengajuan:

```json
{
  "leave_type": "ANNUAL",
  "start_date": "2026-11-02",
  "end_date": "2026-11-04",
  "reason": "Keperluan keluarga"
}
```

Contoh payload menolak pengajuan:

```json
{
  "reason": "Periode tersebut bertepatan dengan jadwal penting team."
}
```

### 6.8 Rancangan halaman frontend

Tambahkan atau kembangkan halaman:

```text
frontend/src/features/leave/LeavePage.tsx
```

Komponen yang disarankan:

- `LeaveRequestForm`
- `LeaveRequestTable`
- `LeaveRequestFilters`
- `LeaveRequestDetailModal`
- `LeaveStatusBadge`
- `LeaveDecisionModal`

Kebutuhan UX:

- Form menampilkan validasi field yang jelas.
- Status menggunakan badge dengan warna konsisten.
- Tombol approve, reject, dan cancel hanya muncul jika tindakan diizinkan.
- Alasan penolakan wajib diisi dalam modal reject.
- Tampilkan loading, error state, empty state, dan success feedback.
- Setelah tindakan berhasil, daftar dan detail harus diperbarui.

### 6.9 Acceptance criteria MVP

- [ ] Employee dapat membuat pengajuan cuti valid.
- [ ] Input kosong dan rentang tanggal tidak valid ditolak.
- [ ] Employee hanya melihat pengajuan miliknya.
- [ ] Manager hanya melihat pengajuan dari anggota team yang sesuai.
- [ ] Admin dapat melihat seluruh pengajuan.
- [ ] Manager/Admin dapat approve pengajuan `PENDING`.
- [ ] Manager/Admin dapat reject dengan alasan wajib.
- [ ] Employee dapat cancel pengajuan `PENDING`.
- [ ] Pengajuan final tidak dapat diproses ulang.
- [ ] Detail menyimpan approver dan waktu keputusan.
- [ ] Error permission mengembalikan response yang sesuai dan tidak membocorkan
  data.
- [ ] Test backend mencakup happy path, validation, authorization, dan
  invalid status transition.
- [ ] Frontend berhasil melakukan build tanpa error.

---

## 7. Spesifikasi fitur fase berikutnya

### 7.1 Filter dan pencarian karyawan

**Branch:** `feature/employee-filter`

Kebutuhan:

- Search nama, employee number, dan email.
- Filter department, position, dan status.
- Pagination dan sorting.
- Query parameter dikelola pada state halaman dan dikirim ke backend.
- Response mengembalikan data dan metadata pagination.

Acceptance criteria:

- Search tidak membedakan huruf besar dan kecil.
- Filter dapat dikombinasikan.
- Halaman tetap responsif saat tidak ada hasil.
- Query yang tidak valid ditangani dengan error yang jelas.

### 7.2 Aturan cuti dan kuota cuti

**Branch:** `feature/leave-rules`

Kebutuhan:

- Policy kuota tahunan per jenis cuti.
- Perhitungan quota terpakai dan sisa quota per employee dan tahun.
- Validasi hari kerja, weekend, hari libur, overlap, dan maksimum hari berturut-turut.
- Edit pengajuan selama masih `PENDING`.
- Riwayat perubahan status cuti.
- Multi-level approval yang dapat dikonfigurasi pada policy.
- Delegasi approval manager dengan periode berlaku.

Implementasi:

- Backend menyediakan `LeavePolicy`, `LeaveBalance`, `CompanyHoliday`,
  `LeaveStatusHistory`, dan `ApprovalDelegation`.
- `GET /api/v1/leaves/balance` mengembalikan quota, used, dan remaining.
- `PUT /api/v1/leaves/:id` hanya dapat mengubah pengajuan `PENDING`.
- `GET /api/v1/leaves/:id/history` menyediakan audit perubahan status.
- Pending leave ikut mencadangkan quota agar pengajuan paralel tidak melebihi
  batas tahunan.

Acceptance criteria:

- Pengajuan melebihi quota ditolak backend.
- Weekend dan company holiday ditolak oleh policy default.
- Policy dapat mengatur maksimum hari berturut-turut dan jumlah approval.
- Manager delegate hanya dapat memproses leave pada periode delegasi.
- Perubahan status tersimpan pada riwayat.
- Frontend menampilkan saldo leave dan menyediakan edit untuk pengajuan pending.
- Frontend menyediakan modal aturan cuti yang mengambil policy aktif dari
  backend.
- Error validasi leave ditampilkan sebagai pesan yang mudah dipahami, bukan
  pesan status HTTP mentah.
- Halaman Leave memiliki layout responsive dengan header, saldo, filter, dan
  tabel yang terpisah serta tanggal yang readable.

### 7.3 Dashboard statistik

**Branch:** `feature/dashboard-statistics`

Kebutuhan:

- Total employee aktif.
- Jumlah employee per department.
- Ringkasan status absensi hari ini.
- Jumlah cuti berdasarkan status.
- Filter periode untuk statistik yang relevan.

Acceptance criteria:

- Angka pada dashboard berasal dari endpoint backend.
- Loading dan error state tersedia.
- Nilai kosong ditampilkan sebagai `0`, bukan halaman rusak.
- Statistik mengikuti hak akses pengguna.
- Admin melihat agregasi seluruh perusahaan, Manager melihat direct reports,
  dan Employee melihat data pribadi.
- Preset periode tersedia untuk bulan berjalan, tiga bulan terakhir, dan tahun
  berjalan.
- Dashboard menampilkan kartu ringkasan, grafik department, status attendance,
  leave per bulan, dan keterlambatan.
- KPI card harus responsive, memiliki tinggi yang konsisten, dan menyediakan
  navigasi langsung untuk data yang dapat dikelola oleh role terkait.
- Riwayat attendance Employee menampilkan tanggal dan waktu dalam format yang
  mudah dibaca serta menyediakan empty state.
- Loading dashboard menggunakan skeleton yang mengikuti struktur KPI card.

Kontrak endpoint:

```text
GET /api/v1/dashboard/stats?preset=this_month
```

Preset yang valid adalah `this_month`, `last_3_months`, dan `this_year`.
Backend mengembalikan batas periode dan agregasi
`department_counts`, `attendance_status_counts`, `leave_monthly`,
`leave_status_counts`, `employee_status_counts`, serta `late_attendance`.

### 7.4 Kalender absensi dan hari libur

**Branch:** `feature/attendance-calendar` dan `feature/company-holidays`

Kebutuhan:

- Tampilan absensi per bulan pada route `/attendance/calendar`.
- Navigasi bulan sebelumnya dan berikutnya.
- Warna status `PRESENT`, `LATE`, dan `ABSENT`.
- Detail check-in, check-out, dan status saat tanggal dipilih.
- Filter employee dan department untuk Admin/Manager.
- CRUD hari libur untuk Admin.
- Hari libur tampil pada kalender.

Implementasi tahap ini:

- Kalender memakai endpoint attendance dan holidays yang sudah tersedia.
- ADMIN melihat seluruh data, MANAGER melihat direct reports, dan EMPLOYEE
  melihat data sendiri.
- Layout kalender responsive dan memiliki empty state untuk tanggal tanpa
  attendance.
- Status `IZIN` dan `SAKIT` belum dapat ditampilkan karena belum menjadi status
  valid pada model attendance backend.

### 7.4.1 Company holiday management

Implementasi fitur `feature/company-holidays`:

- Admin memiliki halaman `/holidays` untuk melihat holiday per tahun.
- Admin dapat membuat, mengubah, dan menghapus holiday.
- Nama dan tanggal holiday wajib diisi.
- Tanggal holiday tidak boleh duplikat.
- Endpoint write hanya dapat digunakan role `ADMIN`.
- Holiday digunakan oleh validasi working-day leave dan ditampilkan di
  attendance calendar.

### 7.5 Export laporan

**Branch:** `feature/report-export`

Kebutuhan:

- Export CSV karyawan, absensi, dan cuti.
- Export mengikuti filter aktif.
- Nama file mengandung jenis laporan dan tanggal export.
- Endpoint tidak mengirim data di luar hak akses pengguna.

### 7.6 Notifikasi

**Branch:** `feature/notifications`

Kebutuhan:

- Notifikasi pengajuan cuti baru.
- Notifikasi keputusan cuti.
- Penanda sudah dibaca atau belum.
- Badge jumlah notifikasi belum dibaca.

### 7.7 Audit log

**Branch:** `feature/audit-log`

Kebutuhan:

- Mencatat login dan perubahan data penting.
- Menyimpan actor, action, entity, entity ID, dan waktu.
- Hanya Admin dapat melihat audit log.
- Log tidak boleh dapat diubah melalui endpoint biasa.

---

## 8. Rencana branch dan commit

### Branch per fitur

```bash
git switch main
git pull origin main
git switch -c feature/leave-approval
```

### Contoh commit yang disarankan

```text
feat: menambahkan migration field keputusan cuti
feat: menambahkan endpoint pengajuan cuti
feat: menambahkan authorization approval cuti
feat: menambahkan tabel approval cuti
test: menambahkan test workflow approval cuti
docs: memperbarui dokumentasi endpoint cuti
```

### Aturan Pull Request

Setiap PR harus:

- Memiliki satu tujuan fitur yang jelas.
- Menyebutkan branch sumber dan target.
- Menjelaskan perubahan dan cara pengujian.
- Menyertakan screenshot untuk perubahan UI.
- Menyebutkan migration atau perubahan environment jika ada.
- Tidak menggabungkan fitur yang tidak berhubungan.

---

## 9. Non-functional requirements

### Keamanan

- Semua endpoint fitur harus memeriksa autentikasi.
- Authorization dilakukan di backend, bukan hanya menyembunyikan tombol
  frontend.
- Password, token, dan data sensitif tidak boleh masuk response atau log.
- Input dari pengguna harus divalidasi dan query harus menggunakan mekanisme
  aman dari ORM.

### Performa

- Endpoint daftar wajib mendukung pagination.
- Hindari query berulang untuk relasi employee, department, dan approver.
- Dashboard tidak boleh mengambil seluruh data mentah jika agregasi dapat
  dilakukan di database.

### Maintainability

- Ikuti struktur folder dan pola handler/service/repository yang sudah dipakai
  project.
- Gunakan konstanta atau tipe terkontrol untuk status penting.
- Migration diberi nomor dan nama yang jelas.
- Error response menggunakan format yang konsisten.

### Kompatibilitas

- Perubahan tidak boleh menghapus data lama.
- Data `LeaveRequest` lama harus tetap dapat dibaca.
- Frontend dan backend harus dirilis secara kompatibel saat ada perubahan API.

---

## 10. Strategi pengujian

### Backend

- Unit test validasi tanggal dan status.
- Test authorization untuk Employee, Manager, dan Admin.
- Test repository atau query filter.
- Integration test endpoint create, approve, reject, dan cancel.
- Test invalid transition, data tidak ditemukan, dan akses tanpa izin.

### Frontend

- Test form dengan input valid dan invalid.
- Test tombol berdasarkan status dan role.
- Test loading, error, empty, dan success state.
- Test refresh data setelah approve, reject, atau cancel.
- Build production untuk memastikan TypeScript dan bundling valid.

### Pengujian manual

1. Login sebagai Employee dan buat pengajuan.
2. Pastikan Employee tidak dapat membuka pengajuan milik orang lain.
3. Login sebagai Manager dan pastikan hanya pengajuan team yang muncul.
4. Approve satu pengajuan.
5. Reject pengajuan lain tanpa alasan dan pastikan ditolak oleh validasi.
6. Reject dengan alasan dan pastikan status tersimpan.
7. Login sebagai Admin dan verifikasi akses seluruh data.
8. Coba memproses ulang pengajuan yang sudah final.
9. Periksa data lama setelah migration dijalankan.

---

## 11. Risiko dan mitigasi

| Risiko                                 | Dampak                         | Mitigasi                                              |
| -------------------------------------- | ------------------------------ | ----------------------------------------------------- |
| Authorization hanya dibuat di frontend | Data dapat diakses melalui API | Validasi role dan ownership di setiap handler         |
| Conflict migration                     | Deployment gagal               | Buat migration baru dan uji pada database development |
| Branch fitur terlalu besar             | Review dan merge sulit         | Pecah menjadi commit atau PR yang fokus               |
| Status dapat berubah sembarangan       | Data workflow tidak konsisten  | Buat daftar transisi status yang valid                |
| Query daftar lambat                    | UX menurun saat data bertambah | Pagination, index, dan preload yang terukur           |
| Perubahan API mematahkan frontend      | Fitur lama rusak               | Jaga response kompatibel dan uji end-to-end           |

---

## 12. Definition of Done

Sebuah fitur dianggap selesai apabila:

- [ ] Requirement dan acceptance criteria sudah terpenuhi.
- [ ] Backend dan frontend terintegrasi.
- [ ] Validasi dan authorization sudah diterapkan.
- [ ] Migration sudah diuji jika ada perubahan database.
- [ ] Test yang relevan sudah dibuat atau diperbarui.
- [ ] Build dan lint berhasil.
- [ ] Tidak ada secret atau data development sensitif yang ikut di-commit.
- [ ] Dokumentasi endpoint atau perilaku fitur sudah diperbarui.
- [ ] PR sudah direview dan tidak memiliki conflict.
- [ ] Fitur sudah diuji pada branch sebelum digabung ke `main`.

---

## 13. Urutan implementasi yang direkomendasikan

```text
1. feature/leave-approval
2. feature/employee-filter
3. feature/dashboard-statistics
4. feature/company-holidays
5. feature/attendance-calendar
6. feature/work-schedule
7. feature/report-export
8. feature/notifications
9. feature/audit-log
10. feature/employee-import
```

Setiap item dikerjakan dengan pola:

```text
main terbaru
  -> buat branch fitur
  -> implementasi backend
  -> implementasi frontend
  -> test dan build
  -> push branch
  -> Pull Request
  -> review
  -> merge ke main
```
