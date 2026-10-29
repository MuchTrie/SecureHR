# SecureHR — Rancangan Aplikasi FE, BE, dan Database

## 1. Tujuan

SecureHR adalah aplikasi **HR Employee Management System** untuk mengelola data karyawan, struktur organisasi, attendance, dan leave management.

Pada tahap ini, scope proyek mencakup:

- Frontend React
- Backend Go
- Database MySQL
- Integrasi Frontend ↔ Backend ↔ Database
- Containerisasi lokal dengan Docker Compose

Hal-hal berikut **belum termasuk dalam rancangan tahap ini**:

- Deployment ke AWS/EC2
- VPS/server
- Kubernetes
- Docker production hardening
- CI/CD
- DevSecOps
- Cloud
- Monitoring infrastructure

Fokus utama adalah menghasilkan aplikasi yang berjalan dengan baik secara lokal.

---

# 2. Arsitektur Aplikasi

Arsitektur aplikasi menggunakan pola:

```text
┌──────────────────────┐
│      React FE        │
│   TypeScript + Vite  │
└──────────┬───────────┘
           │
           │ REST API / JSON
           ▼
┌──────────────────────┐
│       Go BE          │
│        Gin           │
└──────────┬───────────┘
           │
           │ SQL / ORM
           ▼
┌──────────────────────┐
│       MySQL          │
└──────────────────────┘
```

Backend menggunakan pendekatan **modular monolith**.

Alur request:

```text
React
  │
  ▼
HTTP Request
  │
  ▼
Go Router
  │
  ▼
Handler
  │
  ▼
GORM Query
  │
  ▼
MySQL atau SQLite fallback
```

## 2.1 Arsitektur container lokal

Untuk pengujian lokal, frontend dan backend dibungkus dalam container
terpisah. MySQL juga dijalankan sebagai container melalui Compose:

```text
┌──────────────────────┐
│ Browser              │
│ localhost:5173       │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ Frontend container   │
│ Nginx :80            │
└──────────┬───────────┘
           │ Browser API URL
           ▼
┌──────────────────────┐
│ Backend container    │
│ Go/Gin :8080         │
└──────────┬───────────┘
           │ mysql:3306
           ▼
┌──────────────────────┐
│ MySQL container      │
│ Volume mysql_data    │
└──────────────────────┘
```

File terkait:

- `docker-compose.yml` di root workspace.
- `backend/Dockerfile`.
- `frontend/Dockerfile`.
- `frontend/nginx.conf`.

Perintah latihan:

```powershell
docker compose up -d --build
docker compose ps
docker compose down
```

MySQL dapat diakses dari host pada `localhost:3306` atau melalui CLI di
dalam container:

```powershell
docker compose exec mysql mysql -u securehr -p securehr
```

Hostname `mysql` hanya digunakan oleh backend di dalam Docker network.
Frontend browser menggunakan `VITE_API_URL=http://localhost:8080/api/v1`.

---

# 3. Teknologi

## Frontend

- React
- TypeScript
- Vite
- React Router
- Axios
- TanStack Query
- Tailwind CSS
- React Hook Form
- Zod

## Backend

- Go
- Gin
- GORM
- MySQL Driver
- JWT
- Argon2id atau bcrypt
- Validator

## Database

- MySQL 8.x

---

# 4. Modul Aplikasi

MVP terdiri dari:

```text
SecureHR
│
├── Authentication
│
├── Dashboard
│
├── Employee Management
│
├── Department Management
│
├── Position Management
│
├── Attendance
│
├── Leave Management
│
└── User Management
```

Modul yang belum masuk MVP:

```text
Payroll
Performance Review
Employee Documents
Advanced Reporting
```

Modul tersebut dapat ditambahkan setelah MVP selesai.

---

# 5. Role Pengguna

Tiga role utama:

```text
ADMIN
MANAGER
EMPLOYEE
```

## 5.1 ADMIN

Admin dapat:

- Mengelola employee
- Mengelola department
- Mengelola position
- Melihat attendance
- Mengelola leave
- Mengelola user
- Melihat dashboard

## 5.2 MANAGER

Manager dapat:

- Melihat employee dalam scope-nya
- Melihat attendance team
- Melihat leave team
- Approve leave
- Reject leave

## 5.3 EMPLOYEE

Employee dapat:

- Melihat profile sendiri
- Mengubah informasi tertentu
- Check-in
- Check-out
- Melihat attendance sendiri
- Mengajukan leave
- Melihat leave sendiri

---

# 6. Database Design

## 6.1 Entity

Database utama:

```text
users
roles
employees
departments
positions
attendance
leave_requests
```

Relasi:

```text
roles
  │
  ▼
users
  │
  ▼
employees
  │
  ├────────── departments
  │
  ├────────── positions
  │
  ├────────── attendance
  │
  └────────── leave_requests
```

---

# 7. Tabel Database

## 7.1 roles

```text
roles
--------------------
id
name
created_at
updated_at
```

Contoh:

```text
1 | ADMIN
2 | MANAGER
3 | EMPLOYEE
```

---

## 7.2 users

```text
users
--------------------
id
employee_id
email
password_hash
role_id
is_active
created_at
updated_at
```

Relasi:

```text
users.role_id
    ↓
roles.id
```

```text
users.employee_id
    ↓
employees.id
```

---

## 7.3 employees

```text
employees
--------------------
id
employee_number
first_name
last_name
email
phone
address
date_of_birth
gender
department_id
position_id
manager_id
join_date
status
created_at
updated_at
```

Status:

```text
ACTIVE
INACTIVE
RESIGNED
```

---

## 7.4 departments

```text
departments
--------------------
id
name
description
manager_id
created_at
updated_at
```

Contoh:

```text
Engineering
Human Resources
Finance
Marketing
Operations
```

---

## 7.5 positions

```text
positions
--------------------
id
name
description
level
created_at
updated_at
```

Contoh:

```text
Junior Software Engineer
Software Engineer
Senior Software Engineer
Engineering Manager
HR Specialist
```

---

## 7.6 attendance

```text
attendance
--------------------
id
employee_id
date
check_in
check_out
status
created_at
updated_at
```

Status:

```text
PRESENT
LATE
ABSENT
```

---

## 7.7 leave_requests

```text
leave_requests
--------------------
id
employee_id
start_date
end_date
reason
status
approved_by
approved_at
created_at
updated_at
```

Status:

```text
PENDING
APPROVED
REJECTED
CANCELLED
```

Fitur aturan cuti menambahkan entitas berikut:

```text
leave_policies
--------------------
id
leave_type
quota_days
max_consecutive_days
allow_weekend
required_approval_levels

leave_balances
--------------------
id
employee_id
leave_type
year
quota_days
used_days
remaining_days

company_holidays
--------------------
id
date
name

leave_status_histories
--------------------
id
leave_request_id
from_status
to_status
changed_by
changed_at

approval_delegations
--------------------
id
manager_id
delegate_id
start_date
end_date
```

Saldo leave dihitung berdasarkan employee, jenis cuti, dan tahun berjalan.
Pengajuan `PENDING` dan `APPROVED` ikut mencadangkan quota. Backend juga
memvalidasi weekend, company holiday, overlap, maksimum hari berturut-turut,
multi-level approval, serta periode delegasi.

---

# 8. Backend Architecture

Struktur backend aktual:

```text
backend/
│
├── cmd/
│   └── server/
│       └── main.go
│
├── internal/
│   │
│   ├── config/
│   │   └── config.go
│   │
│   ├── database/
│   │   ├── database.go
│   │   └── bootstrap.go
│   │
│   ├── model/
│   │   └── models.go
│   │
│   ├── seeder/
│   │   └── seeder.go
│   │
│   ├── middleware/
│   │   └── auth.go
│   │
│   └── server/
│       ├── handlers.go
│       ├── routes.go
│       └── server.go
│
├── go.mod
└── go.sum
```

---

# 9. Backend Responsibility

Implementasi saat ini mengelompokkan HTTP route dan handler dalam package
`internal/server`. Pemisahan service dan repository belum digunakan. Tanggung
jawab utama dibagi sebagai berikut:

```text
main.go
   │
   ▼
database bootstrap
   │
   ▼
Server dan routes
   │
   ▼
HTTP handlers + GORM queries
```

## Entry point dan bootstrap

`cmd/server/main.go` bertanggung jawab terhadap:

- loading environment dan konfigurasi,
- membuka koneksi database,
- menjalankan versioned migration dan seeder,
- membuat server dan menjalankan router.

`internal/database/` menangani koneksi database dan versioned bootstrap.

## Server

`internal/server/routes.go` mendaftarkan route, middleware, CORS, dan role
authorization. `internal/server/server.go` menyediakan konstruktor server dan
router.

## Handler

`internal/server/handlers.go` menangani:

- HTTP request dan response,
- parsing serta validasi parameter,
- role scope,
- business rule,
- query GORM,
- HTTP status code.

---

# 10. Authentication Backend

Endpoint:

```http
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

Flow:

```text
Login
  │
  ▼
Validate Request
  │
  ▼
Find User
  │
  ▼
Verify Password
  │
  ▼
Generate Token
  │
  ▼
Return User Session
```

Password disimpan sebagai hash:

```text
Password
   │
   ▼
Argon2id / bcrypt
   │
   ▼
password_hash
   │
   ▼
MySQL
```

---

# 11. Employee API

```http
GET    /api/v1/employees
POST   /api/v1/employees
GET    /api/v1/employees/:id
PUT    /api/v1/employees/:id
DELETE /api/v1/employees/:id
```

Fitur:

- List employee
- Search
- Filter
- Pagination
- Detail
- Create
- Update
- Delete

Query list menggunakan server-side filtering:

```http
GET /api/v1/employees?page=1&per_page=10&search=andi&department_id=1&position_id=2&gender=MALE&status=ACTIVE&join_date_from=2026-01-01&join_date_to=2026-12-31&sort_by=first_name&sort_order=ASC
```

Parameter yang tersedia:

- `page`: nomor halaman, default `1`.
- `per_page`: hanya `10`, `25`, atau `50`.
- `search`: nama, email, atau employee number.
- `department_id`, `position_id`, `gender`, dan `status`.
- `join_date_from` dan `join_date_to` dengan format `YYYY-MM-DD`.
- `sort_by` dan `sort_order` dengan whitelist kolom backend.

Response menggunakan bentuk:

```json
{
  "items": [],
  "total": 0,
  "page": 1,
  "per_page": 10,
  "total_pages": 0
}
```

---

# 12. Department API

```http
GET    /api/v1/departments
POST   /api/v1/departments
GET    /api/v1/departments/:id
PUT    /api/v1/departments/:id
DELETE /api/v1/departments/:id
```

---

# 13. Position API

```http
GET    /api/v1/positions
POST   /api/v1/positions
GET    /api/v1/positions/:id
PUT    /api/v1/positions/:id
DELETE /api/v1/positions/:id
```

---

# 14. Attendance API

```http
POST /api/v1/attendance/check-in
POST /api/v1/attendance/check-out
GET  /api/v1/attendance
GET  /api/v1/attendance/:employee_id
```

Fitur:

- Check-in
- Check-out
- History
- Filter tanggal
- Filter employee

---

# 15. Leave API

```http
POST /api/v1/leaves
GET  /api/v1/leaves
GET  /api/v1/leaves/:id

POST /api/v1/leaves/:id/approve
POST /api/v1/leaves/:id/reject
POST /api/v1/leaves/:id/cancel
```

Flow:

```text
Employee
   │
   ▼
Request Leave
   │
   ▼
PENDING
   │
   ├── APPROVED
   │
   └── REJECTED
```

---

# 16. API Response Format

Success:

```json
{
  "success": true,
  "data": {}
}
```

Error:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Invalid request"
  }
}
```

Format response harus konsisten di seluruh endpoint.

---

# 17. Frontend Architecture

Struktur:

```text
frontend/
│
├── src/
│   │
│   ├── app/
│   │   ├── router.tsx
│   │   └── providers.tsx
│   │
│   ├── components/
│   │   ├── ui/
│   │   ├── layout/
│   │   ├── table/
│   │   ├── modal/
│   │   └── form/
│   │
│   ├── features/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── employees/
│   │   ├── departments/
│   │   ├── positions/
│   │   ├── attendance/
│   │   ├── leave/
│   │   └── users/
│   │
│   ├── hooks/
│   ├── lib/
│   │   ├── axios.ts
│   │   └── auth.ts
│   │
│   ├── types/
│   ├── utils/
│   └── main.tsx
│
├── public/
├── package.json
└── vite.config.ts
```

---

# 18. Frontend Routes

## Authentication

```text
/login
/forgot-password
```

## Dashboard

```text
/dashboard
```

## Employee

```text
/employees
/employees/new
/employees/:id
/employees/:id/edit
```

## Department

```text
/departments
/departments/new
/departments/:id
```

## Position

```text
/positions
/positions/new
/positions/:id
```

## Attendance

```text
/attendance
/attendance/history
```

## Leave

```text
/leaves
/leaves/request
/leaves/:id
```

## User

```text
/users
/users/:id
```

## Profile

```text
/profile
```

---

# 19. Frontend Authentication

Login page:

```text
+--------------------------------+
|           SecureHR             |
|                                |
| Email                          |
| [________________________]     |
|                                |
| Password                       |
| [________________________]     |
|                                |
| [          LOGIN          ]    |
|                                |
| Forgot Password?               |
+--------------------------------+
```

Setelah login berhasil:

```text
Login
  │
  ▼
Authentication Success
  │
  ▼
Dashboard
```

Frontend harus memiliki:

- Login form
- Form validation
- Loading state
- Error state
- Protected routes
- Logout
- Current user

---

# 20. Dashboard

## Admin

```text
+------------------------------------------------+
| Dashboard                                      |
+------------------------------------------------+
| Employees | Departments | Present | Leave      |
|    120    |      8      |   108   | Pending 7 |
+------------------------------------------------+
```

## Manager

Menampilkan:

- Jumlah anggota team
- Attendance team
- Pending leave
- Informasi team

## Employee

Menampilkan:

- Attendance hari ini
- Status check-in
- Status check-out
- Leave
- Profile

---

# 21. Employee UI

Route:

```text
/employees
```

Tampilan:

```text
+-------------------------------------------------------+
| Employees                              [Add Employee] |
+-------------------------------------------------------+
| Search [____________] Department [______] Status [__] |
| Position [______] Gender [____] Join date [__]–[__]   |
| Sort [First name] [Ascending]          [Clear filters]|
+-------------------------------------------------------+
| No | Employee | Department | Position | Status        |
+-------------------------------------------------------+
| 01 | John Doe | Engineering| Engineer | ACTIVE        |
| 02 | Jane Doe | HR         | HR Staff | ACTIVE        |
+-------------------------------------------------------+
```

Fitur:

- Search
- Filter
- Pagination
- Sorting ascending/descending
- Filter position, gender, dan rentang join date
- Pilihan page size `10`, `25`, atau `50`
- Default status `ACTIVE`
- Search debounce sekitar 300 ms
- Detail
- Create
- Edit
- Delete

---

# 22. Department UI

```text
/departments
```

Fitur:

- List department
- Create department
- Edit department
- Delete department
- Detail department
- Employee count

---

# 23. Position UI

```text
/positions
```

Fitur:

- List position
- Create position
- Edit position
- Delete position
- Position level

---

# 24. Attendance UI

Employee:

```text
+---------------------------+
| Today's Attendance        |
+---------------------------+
| Check In  : 08:01         |
| Check Out : --            |
|                           |
| [ CHECK OUT ]             |
+---------------------------+
```

Attendance history:

```text
+---------------------------------------------+
| Date       | Check In | Check Out | Status  |
+---------------------------------------------+
| 2026-10-01 | 08:02    | 17:01     | PRESENT |
| 2026-10-02 | 08:15    | 17:04     | LATE    |
+---------------------------------------------+
```

---

# 25. Attendance Calendar UI

Route frontend:

```text
/attendance/calendar
```

Kalender menggunakan:

```text
GET /api/v1/attendance?date_from=YYYY-MM-DD&date_to=YYYY-MM-DD
GET /api/v1/holidays?year=YYYY
```

Fitur:

- Grid kalender bulanan dengan navigasi bulan sebelumnya/berikutnya.
- Warna status `PRESENT`, `LATE`, dan `ABSENT`.
- Detail employee, check-in, check-out, dan status ketika tanggal dipilih.
- Tampilan company holiday pada tanggal terkait.
- Filter employee dan department untuk Admin/Manager.
- Layout responsive untuk mobile dan desktop.

Cakupan data:

- Admin: seluruh attendance.
- Manager: direct reports.
- Employee: attendance sendiri.

Status `IZIN` dan `SAKIT` belum masuk rancangan status attendance karena
belum didukung oleh model backend.

---

# 26. Company Holiday Management UI

Route Admin:

```text
/holidays
```

Endpoint:

```text
GET    /api/v1/holidays?year=YYYY
POST   /api/v1/holidays
PUT    /api/v1/holidays/:id
DELETE /api/v1/holidays/:id
```

Halaman menampilkan daftar holiday berdasarkan tahun, form tanggal dan nama,
aksi edit/delete untuk Admin, loading state, dan empty state. Role non-Admin
tidak mendapat aksi write. Tanggal holiday memiliki unique index dan dipakai
oleh validasi leave serta kalender attendance.

---

# 27. Leave UI

Employee:

```text
/leaves/request
```

Form:

```text
Leave Type
[________________]

Start Date
[________________]

End Date
[________________]

Reason
[________________________]
[________________________]

[ Submit Request ]
```

Halaman Leave aktual menampilkan:

- Header dan action row untuk `Aturan Cuti` serta `Request Leave`.
- Kartu saldo per jenis cuti yang berisi quota, pemakaian, dan sisa hari.
- Filter status pengajuan.
- Tabel responsive dengan label jenis cuti, tanggal readable, alasan yang
  dipotong secara aman, status, dan action sesuai role.
- Modal aturan cuti yang mengambil policy aktif dari API.
- Pesan error validasi dalam bahasa yang mudah dipahami pengguna.

Manager/Admin:

```text
+------------------------------------------------+
| Leave Requests                                 |
+------------------------------------------------+
| Employee | Period | Reason | Status | Action  |
+------------------------------------------------+
| John     | 10-12  | Family | PENDING| Approve |
| Jane     | 13-14  | Sick   | PENDING| Reject  |
+------------------------------------------------+
```

# 28. Dashboard Statistik

Dashboard menggunakan endpoint agregasi:

```text
GET /api/v1/dashboard/stats?preset=this_month
```

Preset periode:

```text
this_month
last_3_months
this_year
```

Cakupan data mengikuti role:

- Admin: seluruh employee dan organisasi.
- Manager: direct reports berdasarkan `manager_id`.
- Employee: data employee yang sedang login.

Dashboard menampilkan kartu ringkasan, employee per department, status
attendance, leave per bulan, status employee/leave, dan jumlah keterlambatan.
Frontend menyediakan loading, error dengan tombol retry, serta empty state
untuk agregasi yang belum memiliki data.

Kontrak response utama:

```json
{
  "preset": "this_month",
  "period_start": "2026-10-01",
  "period_end": "2026-10-21",
  "total_employees": 10,
  "total_departments": 4,
  "total_users": 10,
  "present_today": 8,
  "absent_today": 2,
  "pending_leaves": 1,
  "late_attendance": 3,
  "department_counts": {},
  "attendance_status_counts": {},
  "leave_monthly": {},
  "leave_status_counts": {},
  "employee_status_counts": {}
}
```

`period_start` dan `period_end` mengikuti preset. Kartu hadir dan tidak hadir
berdasarkan tanggal hari ini, sedangkan agregasi attendance, leave, dan
keterlambatan mengikuti periode yang dipilih.

Detail UI dashboard:

- KPI card menggunakan grid responsive dengan tinggi seragam dan ruang label
  yang konsisten agar label dua baris tidak mengubah tinggi card.
- Card Employee/Manager dapat mengarahkan pengguna ke halaman employee atau
  pending leave sesuai hak akses.
- Panel visual utama menampilkan employee per department dan attendance
  status.
- Dashboard Employee menampilkan recent attendance dalam tabel dengan tanggal
  `dd MMM yyyy`, waktu `HH:mm`, status badge, dan empty state.
- Loading KPI menggunakan skeleton card yang memiliki placeholder icon, label,
  dan nilai.
- Sidebar tidak menampilkan heading `Menu`; daftar navigasi tetap difilter
  berdasarkan role.

---

# 29. Frontend ↔ Backend Integration

Komunikasi:

```text
React
  │
  │ Axios
  ▼
Go REST API
  │
  ▼
MySQL
```

API client:

```text
src/lib/axios.ts
```

Feature API:

```text
features/auth/api.ts
features/employees/api.ts
features/departments/api.ts
features/positions/api.ts
features/attendance/api.ts
features/leave/api.ts
```

TanStack Query digunakan untuk:

- Fetch data
- Cache
- Mutation
- Loading state
- Error state
- Refetch

---

# 27. Testing Aplikasi

Testing dilakukan setelah FE dan BE terintegrasi.

## Backend

- Unit test service
- Repository test
- Handler/API test
- Authentication test
- Authorization test

## Frontend

- Component test
- Form validation test
- API integration test
- Protected route test

## API Testing

Endpoint diuji menggunakan:

- Postman
- Insomnia
- Bruno

Target:

```text
Authentication
Employee
Department
Position
Attendance
Leave
User
```

---

# 28. Repository Project

Struktur repository utama:

```text
securehr/
│
├── frontend/
│
├── backend/
│
├── database/
│   └── migrations/
│
├── docs/
│   ├── requirements/
│   ├── database/
│   └── api/
│
├── README.md
└── .gitignore
```

Pada tahap ini repository hanya berisi kebutuhan pembangunan aplikasi.

---

# 29. Development Roadmap

## Phase 1 — Requirement

- Tentukan kebutuhan aplikasi
- Tentukan role
- Tentukan fitur
- Buat use case
- Buat flow aplikasi

## Phase 2 — Database

- Buat ERD
- Buat table
- Tentukan primary key
- Tentukan foreign key
- Tentukan relationship
- Buat migration
- Seed data awal

## Phase 3 — Backend Foundation

- Setup Go
- Setup Gin
- Setup MySQL
- Setup GORM
- Setup configuration
- Setup routing
- Setup error handling

## Phase 4 — Authentication

- User model
- Role
- Login
- Password hashing
- JWT
- Middleware
- Current user

## Phase 5 — Master Data

- Department
- Position
- Employee

## Phase 6 — Transaction

- Attendance
- Leave request
- Leave approval
- Leave rejection

## Phase 7 — Frontend

- Setup React
- Routing
- Layout
- Login
- Dashboard
- Employee
- Department
- Position
- Attendance
- Leave
- User

## Phase 8 — Integration

```text
React
  ↕
REST API
  ↕
Go
  ↕
MySQL
```

Integrasikan seluruh fitur.

## Phase 9 — Testing

- Backend testing
- Frontend testing
- API testing
- Integration testing
- Bug fixing

## Phase 10 — MVP

MVP selesai ketika seluruh fitur utama dapat digunakan:

```text
Login
  ↓
Dashboard
  ↓
Employee
  ↓
Department
  ↓
Position
  ↓
Attendance
  ↓
Leave
```

---

# 30. Scope MVP

### Wajib

- Authentication
- Role
- Dashboard
- Employee CRUD
- Department CRUD
- Position CRUD
- Attendance
- Leave
- API
- MySQL
- React
- Go

### Belum dikerjakan

- Payroll
- Performance Review
- Employee Documents
- Advanced Reporting
- Deployment
- Deployment Docker production
- Kubernetes
- CI/CD
- DevSecOps
- Cloud

---

# 31. Target Hasil Tahap Pertama

Tahap pertama dianggap selesai apabila:

1. MySQL memiliki schema yang sesuai.
2. Backend Go dapat menjalankan REST API.
3. Frontend React dapat mengakses REST API.
4. User dapat login.
5. Role dapat membatasi akses fitur.
6. Admin dapat mengelola employee.
7. Admin dapat mengelola department dan position.
8. Employee dapat melakukan check-in/check-out.
9. Employee dapat mengajukan leave.
10. Manager/Admin dapat memproses leave.
11. Data tersimpan dan dibaca dari MySQL.
12. FE dan BE terintegrasi.
13. Test dasar telah dibuat dan dijalankan.

Setelah semua poin tersebut selesai, aplikasi dianggap mencapai **MVP**.

Tahap setelah MVP akan dibahas sebagai proyek terpisah.
