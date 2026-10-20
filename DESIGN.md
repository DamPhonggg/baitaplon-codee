# Thiết kế Hệ thống Đăng ký Tín chỉ

## 1. Phân tích Use Case Diagram

### 1.1 Actors (Tác nhân)

| Actor | Mô tả |
|-------|-------|
| **Student** (Sinh viên) | Người dùng đăng ký học phần, xem bảng điểm |
| **Professor** (Giảng viên) | Người chọn môn dạy, nhập điểm sinh viên |
| **Registrar** (Giáo vụ) | Quản trị viên hệ thống, quản lý dữ liệu nền |

### 1.2 Use Cases & Quan hệ Include

```
                    ┌─────────────┐
                    │    Login    │  ← Bắt buộc (<<include>>) cho TẤT CẢ UC
                    └─────────────┘
                           ▲
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────┴─────┐   ┌──────┴──────┐  ┌─────┴─────┐
    │  Student  │   │  Professor  │  │ Registrar │
    └───────────┘   └─────────────┘  └───────────┘
```

| Use Case | Actor | Mô tả |
|----------|-------|-------|
| Register for courses | Student | Đăng ký học phần trong kỳ |
| View report card | Student | Xem bảng điểm các môn đã học |
| Select courses to teach | Professor | Chọn môn giảng dạy trong kỳ |
| Submit grades | Professor | Nhập điểm cho sinh viên |
| Maintain professor information | Registrar | CRUD thông tin giảng viên |
| Maintain student information | Registrar | CRUD thông tin sinh viên |
| Maintain curriculum | Registrar | CRUD chương trình đào tạo, học phần |

---

## 2. Kiến trúc hệ thống

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│  Next.js App Router + React + Tailwind CSS              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Student  │  │Professor │  │Registrar │              │
│  │ Dashboard│  │ Dashboard│  │  Panel   │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP / API Routes
┌────────────────────────▼────────────────────────────────┐
│                    Application Layer                     │
│  Auth Middleware │ Business Logic │ Validation          │
└────────────────────────┬────────────────────────────────┘
                         │ Prisma ORM
┌────────────────────────▼────────────────────────────────┐
│                      Data Layer                          │
│                    SQLite Database                       │
└─────────────────────────────────────────────────────────┘
```

### 2.1 Công nghệ

| Thành phần | Công nghệ |
|------------|-----------|
| Frontend | Next.js 14, React, Tailwind CSS |
| Backend | Next.js API Routes |
| ORM | Prisma |
| Database | SQLite |
| Auth | Session cookie + bcrypt |

---

## 3. Thiết kế Cơ sở dữ liệu (ERD)

```
┌──────────┐       ┌──────────────┐       ┌─────────────┐
│   User   │───────│  Enrollment  │───────│CourseSection│
│          │       │              │       │             │
│ id       │       │ studentId    │       │ courseId    │
│ email    │       │ sectionId    │       │ professorId │
│ password │       │ grade        │       │ semester    │
│ role     │       │ status       │       │ capacity    │
│ name     │       └──────────────┘       │ schedule    │
└──────────┘                              └──────┬──────┘
     │                                           │
     │ (professor)                               │
     └───────────────────────────────────────────┘
                                                 │
                                          ┌──────▼──────┐
                                          │   Course    │
                                          │             │
                                          │ code        │
                                          │ name        │
                                          │ credits     │
                                          │ department  │
                                          └─────────────┘
```

### 3.1 Bảng chi tiết

**User** — Người dùng (Student / Professor / Registrar)
- `id`, `email`, `passwordHash`, `name`, `role`, `studentId?`, `department?`

**Course** — Học phần trong chương trình đào tạo
- `id`, `code`, `name`, `credits`, `description`, `department`

**CourseSection** — Lớp học phần cụ thể (1 môn có nhiều lớp)
- `id`, `courseId`, `professorId?`, `semester`, `capacity`, `schedule`, `room`

**Enrollment** — Đăng ký / Kết quả học tập
- `id`, `studentId`, `sectionId`, `status` (REGISTERED/COMPLETED/DROPPED), `grade?`

---

## 4. Thiết kế API

### 4.1 Authentication
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/api/auth/login` | Đăng nhập |
| POST | `/api/auth/logout` | Đăng xuất |
| GET | `/api/auth/me` | Lấy thông tin user hiện tại |

### 4.2 Student
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/api/student/sections` | Danh sách lớp học phần mở đăng ký |
| POST | `/api/student/enroll` | Đăng ký học phần |
| DELETE | `/api/student/enroll/:id` | Hủy đăng ký |
| GET | `/api/student/report-card` | Xem bảng điểm |

### 4.3 Professor
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/api/professor/sections` | Lớp đang dạy / có thể chọn |
| POST | `/api/professor/select-section` | Chọn môn dạy |
| GET | `/api/professor/sections/:id/students` | DS sinh viên trong lớp |
| PUT | `/api/professor/grades` | Nhập điểm |

### 4.4 Registrar
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET/POST/PUT/DELETE | `/api/registrar/students` | Quản lý sinh viên |
| GET/POST/PUT/DELETE | `/api/registrar/professors` | Quản lý giảng viên |
| GET/POST/PUT/DELETE | `/api/registrar/courses` | Quản lý chương trình đào tạo |
| GET/POST/PUT/DELETE | `/api/registrar/sections` | Quản lý lớp học phần |

---

## 5. Thiết kế Giao diện

### 5.1 Luồng chung
```
Trang chủ → Login → Dashboard (theo role) → Chức năng cụ thể
```

### 5.2 Student Dashboard
- **Đăng ký học phần**: Bảng lớp mở, nút Đăng ký/Hủy, hiển thị tín chỉ đã đăng ký
- **Bảng điểm**: Bảng môn đã hoàn thành, điểm số, GPA

### 5.3 Professor Dashboard
- **Chọn môn dạy**: Danh sách lớp chưa có GV, nút Nhận dạy
- **Nhập điểm**: Chọn lớp → Bảng sinh viên → Nhập điểm → Lưu

### 5.4 Registrar Panel
- **Quản lý sinh viên**: CRUD bảng sinh viên
- **Quản lý giảng viên**: CRUD bảng giảng viên
- **Quản lý chương trình**: CRUD học phần + tạo lớp học phần

---

## 6. Quy tắc nghiệp vụ

1. **Authentication**: Mọi API (trừ login) yêu cầu session hợp lệ
2. **Authorization**: RBAC — mỗi role chỉ truy cập API của mình
3. **Đăng ký học phần**:
   - Không vượt quá `capacity` của lớp
   - Không trùng lịch học
   - Không đăng ký trùng môn
4. **Chọn môn dạy**: Một lớp chỉ có 1 giảng viên
5. **Nhập điểm**: Chỉ GV được phân công mới nhập điểm

---

## 7. Sequence Diagram — Đăng ký học phần

```
Student          Frontend         API              Database
  │                 │               │                  │
  │── Chọn lớp ────►│               │                  │
  │                 │── POST /enroll►│                  │
  │                 │               │── Check auth ───►│
  │                 │               │── Check capacity►│
  │                 │               │── Check conflict►│
  │                 │               │── Insert ───────►│
  │                 │◄── 201 OK ────│                  │
  │◄── Cập nhật UI ─│               │                  │
```

---

## 8. Cấu trúc thư mục

```
baitaplon9/
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── src/
│   ├── app/
│   │   ├── api/           # API routes
│   │   ├── login/         # Trang đăng nhập
│   │   ├── student/       # Dashboard sinh viên
│   │   ├── professor/     # Dashboard giảng viên
│   │   └── registrar/     # Panel giáo vụ
│   ├── components/        # UI components
│   └── lib/               # Auth, DB, utils
├── DESIGN.md
└── README.md
```
