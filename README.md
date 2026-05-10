# 🎓 DarusTrack API — Backend Service

<div align="center">

[![Capstone Project](https://img.shields.io/badge/🎓_Capstone_Project-S1_Informatika-2E86AB?style=for-the-badge)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Production_Ready-00A651?style=for-the-badge)](https://github.com)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?logo=node.js&logoColor=white&style=flat-square)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white&style=flat-square)](https://expressjs.com)
[![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?logo=mysql&logoColor=white&style=flat-square)](https://mysql.com)
[![Sequelize](https://img.shields.io/badge/Sequelize-6.x-52B0E7?logo=sequelize&logoColor=white&style=flat-square)](https://sequelize.org)
[![Jest](https://img.shields.io/badge/Tested-Jest-C21325?logo=jest&logoColor=white&style=flat-square)](https://jestjs.io)

**RESTful API backend untuk DarusTrack — sistem pemantauan perkembangan akademik siswa secara real-time di SDIT Darussalam 01 Batam.**

*Menjembatani koordinasi antara pihak sekolah dan orang tua secara mudah, efisien, dan efektif.*

</div>

---

## 📖 Daftar Isi

- [Executive Summary](#-executive-summary)
- [Tech Stack](#️-tech-stack)
- [Arsitektur Sistem](#️-arsitektur-sistem)
- [Desain Keamanan](#-desain-keamanan)
- [Optimasi Database](#-optimasi-database)
- [Role & Akses Sistem](#-role--akses-sistem)
- [API Reference Lengkap](#-api-reference-lengkap)
- [Instalasi & Setup Lokal](#️-instalasi--setup-lokal)
- [Environment Variables](#-environment-variables)
- [Testing](#-testing)
- [Keputusan Teknis (Why)](#-keputusan-teknis-why)

---

## 📋 Executive Summary

**DarusTrack** adalah sistem pemantauan perkembangan akademik siswa berbasis web yang dirancang khusus untuk SDIT Darussalam 01 Batam. Dibangun sebagai Capstone Project S1, sistem ini menyelesaikan masalah nyata: laporan hasil pembelajaran yang selama ini hanya disampaikan setiap semester — terlambat bagi orang tua untuk memberikan intervensi yang bermakna.

### Masalah yang Diselesaikan

Sebelum DarusTrack, informasi akademik siswa hanya tersedia melalui laporan semester yang datang terlalu terlambat:

| Kondisi Sebelumnya | Solusi DarusTrack |
|---|---|
| Laporan nilai hanya tiap semester | Monitoring nilai real-time per mata pelajaran |
| Absensi tidak diketahui orang tua secara cepat | Data kehadiran dapat dipantau kapan saja |
| Tidak ada kanal koordinasi guru–orang tua | Platform terpusat dengan akses berbasis role |
| Evaluasi perkembangan siswa tidak terstruktur | Sistem evaluasi terstandarisasi per semester |

### Peran Pengembang

Dibangun oleh **Backend Developer** — bertanggung jawab penuh atas seluruh arsitektur sisi server, dari desain schema database hingga deployment:

| Pilar | Implementasi |
|---|---|
| **Keamanan API** | JWT dual-token dengan rotasi token, rate limiting berlapis dengan Redis fallback, RBAC 4 role |
| **Performa** | LRU cache untuk autentikasi, composite database index per query pattern, connection pooling |
| **Maintainability** | Layered Architecture (Route → Middleware → Controller → Model), JSDoc menyeluruh, migration-driven schema |
| **Reliability** | Jest test suite (unit + integration), global error handler, asyncHandler di setiap route |

### Scope Backend

- **60+ endpoint** (81 route definitions) diorganisir dalam 13 route file terpisah
- **4 role pengguna** dengan matriks akses berbeda: `admin`, `wali_kelas`, `kepala_sekolah`, `orang_tua`
- **18 migration file** — schema versioning lengkap dengan rollback support
- **1 migration index khusus** — 30+ composite dan single-column index untuk performa query
- **LRU cache** pada middleware autentikasi — mengurangi DB roundtrip per request
- **Redis-backed rate limiter** dengan MemoryStore fallback — tidak crash jika Redis down

---

## 🛠️ Tech Stack

### Runtime & Framework

| Teknologi | Versi | Alasan Pemilihan |
|---|---|---|
| **Node.js** | 18+ | Event-loop non-blocking cocok untuk API dengan banyak I/O concurrent (DB query + JWT verify per request) |
| **Express.js** | ^4.21 | Minimalis, middleware-chain yang fleksibel; tidak memaksakan struktur — cocok untuk layered architecture manual |
| **Sequelize ORM** | ^6.37 | Schema-as-code via migrations, model associations, parameterized query aman dari SQL injection secara default |
| **MySQL** | 8.x | Data akademik sangat relasional (siswa → kelas → semester → nilai → kategori); foreign key dan JOIN adalah kebutuhan utama |

### Keamanan & Autentikasi

| Library | Versi | Fungsi |
|---|---|---|
| **jsonwebtoken** | ^9.0.2 | JWT signing & verifikasi — access token + refresh token dengan `tokenVersion` untuk rotasi |
| **bcryptjs** | ^3.0.2 | Password hashing — implementasi bcrypt yang tidak memerlukan native bindings |
| **helmet** | ^8.1 | 11+ HTTP security headers otomatis (XSS, clickjacking, MIME sniffing, HSTS) |
| **express-rate-limit** | ^5.5.1 | Rate limiting per-IP; `loginLimiter` (10 req/15 menit) dan `apiLimiter` (300 req/menit) |
| **cors** | ^2.8.5 | Whitelist origin — hanya `localhost` dan domain Vercel production yang diizinkan |

### Performa & Infrastruktur

| Library | Versi | Fungsi |
|---|---|---|
| **lru-cache** | (built-in di accessValidation) | In-memory cache untuk data user — max 1000 entri, TTL 5 detik; menghindari DB hit per request |
| **ioredis** | ^5.6.1 | Redis client untuk distributed rate limiting di production |
| **rate-limit-redis** | ^1.7.0 | Redis store untuk express-rate-limit — sinkronisasi limit antar instance server |
| **compression** | ^1.8.0 | Gzip response compression — mengurangi bandwidth terutama untuk response list/rekap besar |
| **fastest-validator** | ^1.19.0 | Schema-based input validation — deklaratif, cepat, konsisten antar controller |

### Utilitas & Testing

| Library | Versi | Fungsi |
|---|---|---|
| **nanoid** | ^3.3.11 | Primary key ID 10 karakter — tidak bisa di-enumerate seperti auto-increment integer |
| **morgan** | ~1.9.1 | HTTP request logger — format `dev` di development, `combined` (Apache) di production |
| **jest** | ^29.7.0 | Test runner — unit test + integration test dengan `--runInBand` untuk isolasi |
| **supertest** | ^7.1.0 | HTTP assertion untuk integration test endpoint |
| **dotenv** | ^16.6.1 | Environment variable loader |
| **nodemon** | ^3.1.14 | Hot-reload development server |
| **express-async-handler** | ^1.2.0 | Wrapper async route handler — error async otomatis diteruskan ke global error handler |

---

## 🏗️ Arsitektur Sistem

### Struktur Direktori

```
darustrack-api/
│
├── app.js                          # Entry point: middleware chain, route registry,
│                                   # DB connection check, global error handler
├── bin/www                         # HTTP server bootstrap (port binding)
│
├── config/
│   ├── database.js                 # Sequelize instance + connection pool config
│   └── config.js                   # Sequelize CLI config per environment
│
├── controllers/                    # Business logic — satu file per domain/fitur
│   ├── authController.js           # Login, refresh token, profil, update profil
│   ├── academicYearController.js   # CRUD tahun ajaran
│   ├── semesterController.js       # Aktivasi & query semester
│   ├── userController.js           # CRUD pengguna (admin only)
│   ├── classController.js          # CRUD kelas + penugasan wali kelas
│   ├── classSummaryController.js   # Summary kelas untuk kepala sekolah
│   ├── studentController.js        # CRUD siswa
│   ├── studentClassController.js   # Penempatan siswa ke kelas
│   ├── subjectController.js        # CRUD mata pelajaran & kurikulum
│   ├── curriculumController.js     # Manajemen kurikulum
│   ├── scheduleController.js       # Jadwal pelajaran per kelas
│   ├── teacherAttendanceController.js  # Input & rekap absensi siswa
│   ├── teacherEvaluationController.js  # Evaluasi perkembangan siswa
│   ├── teacherGradeController.js   # Input & manajemen nilai
│   ├── parentController.js         # Akses data anak oleh orang tua
│   └── academicCalendarController.js   # Kalender akademik
│
├── middlewares/
│   ├── accessValidation.js         # JWT verify + LRU cache user data (TTL 5 detik)
│   ├── roleValidation.js           # Whitelist role (factory middleware)
│   ├── loadActiveSemester.js       # Inject semester aktif ke req.semester
│   ├── asyncHandler.js             # Wrapper async error handling
│   ├── rateLimiter.js              # loginLimiter + apiLimiter (Redis/MemoryStore)
│   └── errorHandler.js             # Global 4-parameter error handler
│
├── models/                         # Sequelize models — schema-as-code
│   ├── index.js                    # Auto-discovery + association loader
│   ├── user.js                     # User (nanoid PK, role, token_version)
│   ├── academic_year.js            # Tahun ajaran
│   ├── semester.js                 # Semester (is_active flag)
│   ├── class.js                    # Kelas (FK: teacher_id, academic_year_id)
│   ├── student.js                  # Siswa (FK: parent_id)
│   ├── student_class.js            # Pivot: siswa ↔ kelas per tahun ajaran
│   ├── subject.js                  # Mata pelajaran
│   ├── curriculum.js               # Kurikulum (jumlah jam, target pencapaian)
│   ├── schedule.js                 # Jadwal pelajaran
│   ├── attendance.js               # Data absensi harian
│   ├── evaluation.js               # Template evaluasi perkembangan
│   ├── student_evaluation.js       # Nilai evaluasi per siswa
│   ├── grade_category.js           # Kategori penilaian (UH, UTS, UAS)
│   ├── grade_detail.js             # Detail komponen nilai
│   ├── student_grade.js            # Nilai per siswa per komponen
│   ├── academic_calendar.js        # Event kalender akademik
│   └── password_reset.js           # Token reset password
│
├── routes/                         # Thin routes — hanya mapping endpoint ke middleware + controller
│   ├── index.js                    # Health check
│   ├── authRoutes.js               # /auth/*
│   ├── academicYearRoutes.js       # /academic-years/* (admin)
│   ├── semesterRoutes.js           # /semesters/*
│   ├── userRoutes.js               # /users/* (admin)
│   ├── classRoutes.js              # /classes/*
│   ├── studentRoutes.js            # /students/* (admin)
│   ├── teacherRoutes.js            # /teachers/* (wali_kelas)
│   ├── parentRoutes.js             # /parents/* (orang_tua)
│   ├── headmasterRoutes.js         # /headmaster/* (kepala_sekolah)
│   ├── curriculumRoutes.js         # /curriculums/*
│   ├── subjectRoutes.js            # /subjects/*
│   └── academicCalendarRoutes.js   # /academic-calendar/*
│
├── utils/
│   └── tokenUtils.js               # generateAccessToken + generateRefreshToken
│                                   # (token rotation via tokenVersion)
│
├── migrations/                     # 18 migration file — schema versioning lengkap
│   ├── 20260509164437-add-create-academic-years-table.js
│   ├── 20260509164611-add-create-users-table.js
│   ├── ...
│   └── 20260509181107-add-performance-indexes.js   # 30+ index (single + composite)
│
└── seeders/                        # Data awal untuk development & testing
```

### Request Lifecycle

```
Incoming HTTP Request
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│  CORS Policy                                                 │
│  → Whitelist: localhost:3000, darustrack.vercel.app          │
│  → Preflight OPTIONS ditangani otomatis                      │
│                                                              │
│  Helmet                                                      │
│  → 11+ HTTP security headers (XSS, clickjacking, dll)       │
│                                                              │
│  Morgan Logger (per-environment)                             │
│  → development: 'dev' format (colorized)                     │
│  → production: 'combined' Apache format                      │
│                                                              │
│  Body Parser (JSON limit 1MB) + Compression (Gzip)           │
│  → Limit 1MB mencegah memory pressure attack                 │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────────────────────────────────────┐
│  Express Router                                                       │
│                                                                       │
│  GET /                                                                │
│    → Health check — publik                                            │
│                                                                       │
│  POST /auth/login, /auth/refresh-token                                │
│    → loginLimiter (10 req / 15 menit / IP)                            │
│    → refreshLimiter (terpisah)                                        │
│    → authController (tanpa autentikasi)                               │
│                                                                       │
│  GET /auth/profile, PUT /auth/profile                                 │
│    → accessValidation (JWT verify + LRU cache)                        │
│    → authController                                                   │
│                                                                       │
│  /academic-years, /users, /classes, /students                         │
│    → accessValidation → roleValidation(['admin'])                     │
│    → Controller → Sequelize ORM → MySQL                               │
│                                                                       │
│  /teachers                                                            │
│    → accessValidation → roleValidation(['wali_kelas'])                │
│    → (beberapa endpoint) loadActiveSemester                           │
│    → Teacher/Attendance/Evaluation/Grade Controllers                  │
│                                                                       │
│  /parents                                                             │
│    → accessValidation → roleValidation(['orang_tua'])                 │
│    → parentController                                                 │
│                                                                       │
│  /headmaster                                                          │
│    → accessValidation → roleValidation(['kepala_sekolah'])            │
│    → classSummaryController                                           │
│                                                                       │
│  /semesters, /curriculums, /subjects, /academic-calendar              │
│    → accessValidation (semua role terautentikasi)                     │
│    → Controller masing-masing                                         │
└───────────────────────────────────────────────────────────────────────┘
        │
        ▼
Response JSON { message, data, ... }
        │
        ▼ (jika terjadi error tidak tertangani)
Global Error Handler (errorHandler.js)
→ Log stack trace di server
→ Pesan generik ke client di production
```

### Database Schema (Entity Relationships)

```
academic_years
  └── semesters (is_active)
        │
        └── classes (teacher_id → users)
              │
              └── student_classes ──── students (parent_id → users)
                        │
          ┌─────────────┼──────────────────────┐
          │             │                       │
      attendances  student_evaluations    student_grades
          │             │                       │
          │         evaluations          grade_details
          │                                     │
          └────────────────────────── grade_categories
                                             │
                                 schedules ──┤── subjects
                                             │
                                        curriculums
```

---

## 🔒 Desain Keamanan

### 1. Dual-Token Authentication dengan Token Rotation

```
[POST /auth/login]
  Server generate:
    - access_token  : JWT berumur pendek (4-8 jam sesuai role)
    - refresh_token : JWT 7 hari, disimpan di httpOnly cookie
  Keduanya menyematkan tokenVersion dari kolom token_version user di DB

[Setiap API Request]
  Header: Authorization: Bearer <access_token>
  accessValidation → JWT verify → LRU cache (5 detik TTL) → DB lookup

[access_token kadaluarsa]
  POST /auth/refresh-token { refresh_token }
  Server: verify JWT → bandingkan decoded.tokenVersion vs user.token_version di DB
    → SAMA    → naikkan token_version → terbitkan token baru
    → BERBEDA → tolak (token sudah dirotasi atau direvoke)

[Logout / Ganti Password]
  Server: naikkan token_version → semua token lama langsung tidak valid
```

**Mengapa `tokenVersion`, bukan blocklist?**
Blocklist memerlukan storage yang terus berkembang. `tokenVersion` cukup satu kolom integer per user — revokasi instan tanpa storage overhead.

### 2. Password Security

| Aspek | Implementasi |
|---|---|
| Algoritma | bcryptjs — portable, tanpa native bindings, mendukung salt rounds yang dapat dikonfigurasi |
| Validasi | fastest-validator schema di controller — min length, complexity rules |
| Token rotation | `token_version` dinaikkan setiap ganti password — semua sesi lain otomatis logout |

### 3. Rate Limiting Berlapis (Redis + MemoryStore Fallback)

```
loginLimiter  : 10 request / 15 menit / IP
              → /auth/login, /auth/refresh-token
              → Jika Redis down: fallback MemoryStore (sistem tidak crash)

apiLimiter    : 300 request / menit / IP  →  seluruh endpoint

Redis Strategy:
  → REDIS_URL tersedia + konek  : RedisStore (terdistribusi, multi-instance)
  → REDIS_URL tidak ada / gagal : MemoryStore (per-process, tetap aktif)
  → Redis error saat runtime    : fail-open (request tetap lewat, tidak crash)
```

### 4. LRU Cache di Middleware Autentikasi

```javascript
// accessValidation.js
const userCache = new LRU({ max: 1000, ttl: 5000 }); // 5 detik

// Setiap request terautentikasi:
// 1. Cek cache → jika ada, skip DB query
// 2. Cache miss → query DB → simpan ke cache
// 3. TTL 5 detik: perubahan role/nonaktifkan user berlaku dalam ≤5 detik
```

Tanpa cache, setiap dari 60+ endpoint yang memerlukan autentikasi akan hit database dua kali (JWT verify + user lookup). Dengan cache, request ke endpoint yang sama dalam 5 detik hanya hit DB sekali.

### 5. HTTP Security Headers (via Helmet)

Dipasang otomatis: `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`, `X-XSS-Protection`, `Content-Security-Policy`, dan lainnya.

### 6. Input & Query Security

| Mekanisme | Implementasi |
|---|---|
| SQL Injection | Semua query via Sequelize ORM → parameterized queries otomatis |
| Input validation | fastest-validator schema di controller layer |
| ID enumeration | Nanoid 10 karakter sebagai PK — tidak bisa ditebak/di-enumerate |
| Body overflow | `express.json({ limit: '1mb' })` — mencegah memory pressure attack |
| CORS | Whitelist ketat: `localhost:3000` dan `darustrack.vercel.app` |
| Error leakage | `[BUG #37]` — JWT error detail tidak dikirim ke client, hanya dicatat di server |

---

## ⚡ Optimasi Database

### Performance Index Migration

Index diorganisir dalam satu migration terakhir (`20260509181107-add-performance-indexes.js`) — memberi fleksibilitas untuk iterate index tanpa mengubah migration table awal:

| Tabel | Index |
|---|---|
| `users` | `idx_users_nip`, `idx_users_role`, `idx_users_token_version` |
| `students` | `idx_students_parent_id`, `idx_students_nisn` |
| `semesters` | `idx_semesters_academic_year_id`, `idx_semesters_is_active` |
| `classes` | `idx_classes_teacher_id`, `idx_classes_academic_year_id`, **composite** `idx_classes_year_name` |
| `student_classes` | `idx_sc_student_id`, `idx_sc_class_id`, **unique constraint** `(student_id, class_id)` |
| `attendances` | `idx_att_student_class_id`, `idx_att_semester_id`, `idx_att_date`, **composite** `idx_att_semester_date_class` |
| `schedules` | `idx_sch_class_id`, `idx_sch_subject_id`, `idx_sch_day`, **composite** `idx_sch_class_day_time` |
| `grade_categories` | **composite** `idx_gc_class_subject_semester` |
| `student_grades` | `idx_sg_student_class_id`, `idx_sg_grade_detail_id`, **unique constraint** `(student_class_id, grade_detail_id)` |
| `evaluations` | `idx_ev_class_id`, `idx_ev_semester_id` |
| `student_evaluations` | `idx_se_student_class_id`, `idx_se_evaluation_id` |
| `password_resets` | `idx_pr_token`, `idx_pr_user_id`, `idx_pr_expires_at` |
| `academic_calendar` | `idx_ac_start_date`, `idx_ac_end_date` |

**Contoh composite index yang kritis:**

```sql
-- Query rekap absensi: sering difilter oleh 3 kolom bersamaan
CREATE INDEX idx_attendances_semester_date_class
  ON attendances (semester_id, date, student_class_id);

-- Query jadwal: overlap checking membutuhkan 4 kolom
CREATE INDEX idx_schedules_class_day_time
  ON schedules (class_id, day, start_time, end_time);

-- Filter nilai per semester (pola query paling umum di teacher grade controller)
CREATE INDEX idx_gc_class_subject_semester
  ON grade_categories (class_id, subject_id, semester_id);
```

### `loadActiveSemester` Middleware

Endpoint guru yang memerlukan semester aktif menggunakan middleware `loadActiveSemester` — query semester aktif dilakukan sekali di middleware dan hasilnya tersedia di `req.semester` untuk semua handler di chain. Ini mencegah query berulang di setiap controller yang memerlukan data semester.

---

## 👥 Role & Akses Sistem

### Empat Role Pengguna

| Role | Kemampuan Utama | Contoh Endpoint |
|---|---|---|
| `admin` | CRUD semua master data, manajemen user & kelas | `/users`, `/students`, `/classes`, `/academic-years` |
| `wali_kelas` | Input nilai, absensi, evaluasi untuk kelas sendiri | `/teachers/attendances`, `/teachers/grades/*` |
| `kepala_sekolah` | View summary semua kelas, monitoring akademik | `/headmaster/classes`, `/headmaster/classes/:id` |
| `orang_tua` | View data anak: jadwal, absensi, nilai, evaluasi | `/parents/student`, `/parents/grades/*` |

### Implementasi Middleware Autentikasi

```javascript
// accessValidation.js — verifikasi JWT + LRU cache
router.get('/profile', accessValidation, authController.getProfile);

// roleValidation.js — factory function whitelist role
app.use('/teachers', accessValidation, roleValidation(['wali_kelas']), teachersRouter);
app.use('/parents',  accessValidation, roleValidation(['orang_tua']),  parentsRouter);
app.use('/headmaster', accessValidation, roleValidation(['kepala_sekolah']), headmasterRouter);
app.use('/users',    accessValidation, roleValidation(['admin']),      usersRouter);

// Middleware loadActiveSemester untuk endpoint yang butuh context semester
router.get('/attendances', loadActiveSemester, attendanceCtrl.getAttendances);
router.post('/attendances', loadActiveSemester, attendanceCtrl.createAttendance);
```

---

## 📡 API Reference Lengkap

**Legenda kolom Auth:**

| Simbol | Arti |
|---|---|
| `Publik` | Tidak memerlukan token |
| `Publik + RL` | Publik + rate limiter (10 req / 15 menit) |
| `Login` | `Authorization: Bearer <access_token>` |
| `Admin` | Token dengan role `admin` |
| `Guru` | Token dengan role `wali_kelas` |
| `Ortu` | Token dengan role `orang_tua` |
| `Kasek` | Token dengan role `kepala_sekolah` |
| `Semua` | Semua role yang terautentikasi |

---

### 🔑 Autentikasi — `/auth`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `POST` | `/login` | `Publik + RL` | Login → access token + refresh token (httpOnly cookie). Token expiry per role. |
| `POST` | `/refresh-token` | `Publik + RL` | Tukar refresh token (tokenVersion check) → access token baru. Rotasi otomatis. |
| `GET` | `/profile` | `Login` | Profil user saat ini dari DB. |
| `PUT` | `/profile` | `Login` | Update profil. Semua field opsional kecuali validasi format. |

---

### 🏫 Tahun Ajaran & Semester — `/academic-years`, `/semesters`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/academic-years` | `Admin` | Daftar semua tahun ajaran. |
| `POST` | `/academic-years` | `Admin` | Buat tahun ajaran baru. |
| `PUT` | `/academic-years/:id` | `Admin` | Update tahun ajaran. |
| `DELETE` | `/academic-years/:id` | `Admin` | Hapus tahun ajaran (guard: ada data terkait). |
| `PUT` | `/academic-years/semesters/:id` | `Admin` | Aktifkan/nonaktifkan semester. |
| `GET` | `/semesters` | `Semua` | Daftar semester aktif. |

---

### 👤 Manajemen User — `/users`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/users` | `Admin` | Daftar user. Query: search, role, page, limit. |
| `GET` | `/users/:id` | `Admin` | Detail satu user. |
| `POST` | `/users` | `Admin` | Buat akun (admin, guru, ortu, kasek). |
| `PUT` | `/users/:id` | `Admin` | Update parsial (nama, role, dll). |
| `DELETE` | `/users/:id` | `Admin` | Hapus user. Guard: tidak bisa hapus diri sendiri. |

---

### 🏷️ Kelas & Jadwal — `/classes`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/classes` | `Admin` | Daftar kelas aktif tahun ajaran berjalan. |
| `GET` | `/academic-years/:id/classes` | `Admin` | Daftar kelas per tahun ajaran. |
| `POST` | `/academic-years/:id/classes` | `Admin` | Buat kelas baru + assign wali kelas. |
| `PUT` | `/academic-years/classes/:classId` | `Admin` | Update kelas. |
| `DELETE` | `/academic-years/classes/:classId` | `Admin` | Hapus kelas. |
| `GET` | `/classes/:class_id/schedule` | `Admin` | Jadwal kelas. |
| `POST` | `/classes/:class_id/schedule` | `Admin` | Tambah jadwal. Validasi overlap waktu. |
| `PUT` | `/classes/schedule/:schedule_id` | `Admin` | Update jadwal. |
| `DELETE` | `/classes/schedule/:schedule_id` | `Admin` | Hapus jadwal. |

---

### 🎓 Siswa — `/students`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/students` | `Admin` | Daftar semua siswa. |
| `POST` | `/students` | `Admin` | Daftarkan siswa baru + assign ke orang tua (user). |
| `PUT` | `/students/:id` | `Admin` | Update data siswa. |
| `DELETE` | `/students/:id` | `Admin` | Hapus siswa. |
| `GET` | `/academic-years/:id/students` | `Admin` | Siswa per tahun ajaran. |
| `POST` | `/academic-years/:id/students` | `Admin` | Masukkan siswa ke tahun ajaran (student_class). |
| `DELETE` | `/academic-years/:id/students/:studentId` | `Admin` | Keluarkan siswa dari tahun ajaran. |

---

### 📚 Mata Pelajaran & Kurikulum — `/subjects`, `/curriculums`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/subjects` | `Semua` | Daftar mata pelajaran. Query: search. |
| `GET` | `/subjects/:id` | `Semua` | Detail mata pelajaran. |
| `POST` | `/subjects` | `Admin` | Buat mata pelajaran baru. |
| `PUT` | `/subjects/:id` | `Admin` | Update mata pelajaran. |
| `DELETE` | `/subjects/:id` | `Admin` | Hapus mata pelajaran. |
| `GET` | `/curriculums` | `Semua` | Daftar kurikulum aktif. |
| `PUT` | `/curriculums/:id` | `Admin` | Update target kurikulum per mata pelajaran. |

---

### 👨‍🏫 Portal Guru (Wali Kelas) — `/teachers`

**Absensi**

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/teachers/my-class` | `Guru` | Data kelas yang diampu. |
| `GET` | `/teachers/schedules` | `Guru` | Jadwal mengajar. |
| `GET` | `/teachers/semesters` | `Guru` | Semester yang tersedia. |
| `GET` | `/teachers/attendances/rekap` | `Guru` | Rekap absensi seluruh siswa di kelas (semester aktif). |
| `GET` | `/teachers/attendances` | `Guru` | Data absensi harian (filter: tanggal). |
| `POST` | `/teachers/attendances` | `Guru` | Input absensi. Satu entry per siswa per hari. |
| `PUT` | `/teachers/attendances` | `Guru` | Update absensi yang sudah diinput. |
| `DELETE` | `/teachers/attendances` | `Guru` | Hapus record absensi. |

**Evaluasi Perkembangan**

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/teachers/semesters/:semester_id/evaluations` | `Guru` | Daftar template evaluasi semester ini. |
| `POST` | `/teachers/semesters/:semester_id/evaluations` | `Guru` | Buat template evaluasi baru. |
| `GET` | `/teachers/evaluations/:id` | `Guru` | Detail evaluasi + nilai per siswa. |
| `PUT` | `/teachers/evaluations/:id` | `Guru` | Update template evaluasi. |
| `DELETE` | `/teachers/evaluations/:id` | `Guru` | Hapus evaluasi. |
| `PUT` | `/teachers/student-evaluations/:id` | `Guru` | Input/update nilai evaluasi per siswa. |

**Nilai Akademik**

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/teachers/grades/subjects` | `Guru` | Mata pelajaran yang diajarkan di kelas ini. |
| `GET` | `/teachers/grades/:subject_id/:semester_id/categories` | `Guru` | Kategori penilaian (UH, UTS, UAS). |
| `POST` | `/teachers/grades/:subject_id/:semester_id/categories` | `Guru` | Buat kategori baru. |
| `PUT` | `/teachers/grades/categories/:category_id` | `Guru` | Update nama/bobot kategori. |
| `DELETE` | `/teachers/grades/categories/:category_id` | `Guru` | Hapus kategori. |
| `GET` | `/teachers/grades/categories/:category_id/details` | `Guru` | Detail komponen nilai dalam kategori. |
| `POST` | `/teachers/grades/categories/:category_id/details` | `Guru` | Buat detail komponen baru. |
| `PUT` | `/teachers/grades/details/:detail_id` | `Guru` | Update komponen. |
| `DELETE` | `/teachers/grades/details/:detail_id` | `Guru` | Hapus komponen. |
| `GET` | `/teachers/grades/details/:detail_id/students` | `Guru` | Nilai siswa untuk satu komponen. |
| `PATCH` | `/teachers/grades/students/:student_grade_id` | `Guru` | Input/update satu nilai siswa. |

---

### 👪 Portal Orang Tua — `/parents`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/parents/student` | `Ortu` | Profil siswa milik orang tua yang login. |
| `GET` | `/parents/schedule` | `Ortu` | Jadwal pelajaran anak. |
| `GET` | `/parents/attendances/:semesterId` | `Ortu` | Data absensi anak per semester. |
| `GET` | `/parents/evaluations/:semesterId` | `Ortu` | Daftar judul evaluasi semester ini. |
| `GET` | `/parents/evaluations/:semesterId/:evaluationId` | `Ortu` | Detail nilai evaluasi anak. |
| `GET` | `/parents/grades/:semesterId/subjects` | `Ortu` | Mata pelajaran di semester ini. |
| `GET` | `/parents/grades/:semesterId/:subjectId/categories` | `Ortu` | Kategori nilai per mapel. |
| `GET` | `/parents/grades/categories/:gradeCategoryId/details` | `Ortu` | Detail nilai anak per komponen. |

---

### 🏫 Portal Kepala Sekolah — `/headmaster`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/headmaster/classes` | `Kasek` | Summary semua kelas (jumlah siswa, rata-rata kehadiran). |
| `GET` | `/headmaster/classes/:classId` | `Kasek` | Detail satu kelas (daftar siswa, rekap nilai, evaluasi). |

---

### 📅 Kalender Akademik — `/academic-calendar`

| Method | Endpoint | Auth | Deskripsi |
|---|---|---|---|
| `GET` | `/academic-calendar/upcoming` | `Semua` | Event mendatang (diurutkan berdasarkan tanggal). |
| `GET` | `/academic-calendar` | `Semua` | Semua event kalender. |
| `GET` | `/academic-calendar/:id` | `Semua` | Detail satu event. |
| `POST` | `/academic-calendar` | `Admin` | Buat event kalender. |
| `PUT` | `/academic-calendar/:id` | `Admin` | Update event. |
| `DELETE` | `/academic-calendar/:id` | `Admin` | Hapus event. |

---

## ⚙️ Instalasi & Setup Lokal

### Prasyarat

- Node.js >= 18.x
- npm >= 9.x
- MySQL 8.x (lokal atau cloud)
- Redis (opsional — untuk distributed rate limiting)

### Langkah Instalasi

```bash
# 1. Clone repository
git clone https://github.com/[username]/darustrack-api.git
cd darustrack-api

# 2. Install dependencies
npm install

# 3. Buat file environment
cp .env.example .env
# Edit .env sesuai konfigurasi Anda

# 4. Buat database
mysql -u root -p -e "CREATE DATABASE darustrack;"

# 5. Jalankan semua migrations
npm run migrate
# Atau: npx sequelize-cli db:migrate

# 6. (Opsional) Jalankan seeders untuk data awal
npm run seed

# 7. Jalankan development server
npm run dev
# Server berjalan di http://localhost:3000
```

### Scripts yang Tersedia

```bash
npm start           # Production server (node ./bin/www)
npm run dev         # Development dengan hot-reload (nodemon)
npm run migrate     # Jalankan semua pending migrations
npm run migrate:undo # Rollback satu migration terakhir
npm run seed        # Jalankan semua database seeders
npm test            # Jalankan test suite (Jest --runInBand)
npm run test:watch  # Test mode watch
npm run test:coverage # Test dengan laporan coverage
```

---

## 🔧 Environment Variables

```env
# ─── Aplikasi ─────────────────────────────────────────────────────────────────
PORT=3000
NODE_ENV=development          # development | test | production

# ─── Database ─────────────────────────────────────────────────────────────────
DB_HOST=localhost
DB_PORT=3306
DB_NAME=darustrack
DB_USER=root
DB_PASSWORD=password

# ─── JWT & Token ──────────────────────────────────────────────────────────────
# Generate: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
JWT_SECRET=ganti_dengan_string_random_minimal_64_karakter
REFRESH_TOKEN_SECRET=ganti_dengan_string_random_berbeda
REFRESH_TOKEN_EXPIRES_IN=7d

# ─── Rate Limiting ────────────────────────────────────────────────────────────
LOGIN_RATE_LIMIT_WINDOW_MS=900000     # 15 menit (dalam milidetik)
LOGIN_RATE_LIMIT_MAX=10               # 10 percobaan per window

# ─── Redis (opsional — fallback ke MemoryStore jika tidak diset) ──────────────
REDIS_URL=redis://localhost:6379

# ─── CORS ─────────────────────────────────────────────────────────────────────
ALLOWED_ORIGINS=http://localhost:5173,https://darustrack.vercel.app
```

---

## 🚀 Deployment Checklist

- [ ] `NODE_ENV=production`
- [ ] JWT secrets diganti nilai random kuat (minimal 64 karakter)
- [ ] `RATE_LIMIT_SKIP` tidak di-set atau `false`
- [ ] Database production sudah dikonfigurasi
- [ ] Semua 18 migration sudah berjalan (`npm run migrate`)
- [ ] CORS `ALLOWED_ORIGINS` hanya domain production
- [ ] `trust proxy` aktif (sudah otomatis di `app.js` saat production)
- [ ] Redis dikonfigurasi untuk distributed rate limiting
- [ ] Morgan format `combined` aktif (otomatis saat `NODE_ENV=production`)

---

## 💡 Keputusan Teknis (Why)

### Mengapa Layered Architecture (Route → Controller → Model)?

Di DarusTrack, ada 4 role dengan kebutuhan akses yang berbeda terhadap data yang sama. Misalnya, data absensi siswa diakses oleh guru (untuk input), orang tua (untuk monitoring), dan kepala sekolah (untuk rekap). Dengan layered architecture, logika query di `teacherAttendanceController.js` dan `parentController.js` terpisah secara natural — perubahan pada cara guru menginput absensi tidak memengaruhi cara orang tua melihat data, karena mereka menggunakan controller berbeda meskipun berinteraksi dengan model yang sama.

### Mengapa Controller dan Service tidak dipisah lebih jauh?

DarusTrack adalah proyek sekolah tunggal dengan domain yang relatif terisolasi — tidak ada layanan eksternal yang kompleks, tidak ada queue, tidak ada event system. Menambahkan layer Service di antara Controller dan Model akan menambah indirection tanpa nilai nyata untuk skala ini. Keputusan ini bisa direvisi jika sistem berkembang menjadi multi-tenant atau memerlukan logika bisnis yang lebih kompleks.

### Mengapa LRU Cache di `accessValidation`?

Setiap dari 81 endpoint yang memerlukan autentikasi harus memvalidasi token sekaligus memverifikasi bahwa user masih ada di DB (bukan sekadar percaya payload JWT). Tanpa cache, ini berarti satu DB query per request. Dengan LRU cache TTL 5 detik, request berulang dari user yang sama dalam rentang 5 detik hanya hit DB sekali. TTL 5 detik dipilih secara sadar: cukup singkat sehingga perubahan role atau deaktivasi user berlaku hampir instan, namun cukup panjang untuk memberikan dampak performa yang signifikan.

### Mengapa `tokenVersion` untuk refresh token revocation?

Alternatif paling umum adalah token blocklist (simpan token yang sudah di-revoke di Redis/DB). Masalahnya: blocklist terus berkembang dan memerlukan cleanup berkala. `tokenVersion` lebih elegan — satu kolom integer per user, increment saat logout atau ganti password, dan validasi dilakukan dengan satu DB query. Trade-off: tidak bisa revoke token spesifik (misal: logout dari satu perangkat saja); semua token user di-revoke sekaligus. Untuk use case DarusTrack (sekolah, bukan aplikasi multi-device enterprise), ini adalah trade-off yang dapat diterima.

### Mengapa performance index dipisah ke migration sendiri?

Ketika mengembangkan proyek iteratif, query pattern sering berubah. Memisahkan index ke migration terakhir memudahkan iterasi: bisa menambah, mengubah, atau menghapus index tanpa menyentuh migration table-creation awal yang lebih stabil. Ini juga memudahkan review — satu file berisi seluruh strategi indexing dapat di-review sekaligus sebagai satu unit.

### Mengapa Redis rate limiter dengan MemoryStore fallback?

Ketergantungan keras pada Redis akan menjadikan Redis sebagai single point of failure. Jika Redis down, semua request ditolak — buruk untuk pengalaman pengguna. Dengan fallback ke MemoryStore, sistem tetap berfungsi (dengan rate limiting per-proses, bukan per-cluster) bahkan saat Redis tidak tersedia. Ini adalah contoh penerapan graceful degradation: keamanan tetap terjaga, performa sedikit berkurang, tapi sistem tidak crash.

### Mengapa Nanoid sebagai primary key?

Integer auto-increment memudahkan attacker melakukan ID enumeration (`GET /students/1`, `/students/2`, dst.). Nanoid 10 karakter (1.7 × 10^17 kemungkinan kombinasi) secara praktis tidak bisa ditebak. Biaya: sedikit lebih besar penyimpanannya dan tidak bisa digunakan untuk pagination berbasis ID. Untuk skala sekolah seperti DarusTrack, biaya ini tidak signifikan.

---

*Capstone Project S1 — Backend Developer | Node.js + MySQL + Sequelize*
