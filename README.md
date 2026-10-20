# Hệ thống Đăng ký Tín chỉ

Web app quản lý đăng ký học phần, được thiết kế dựa trên **Use Case Diagram** với 3 actors: Sinh viên, Giảng viên, Giáo vụ.

## Use Cases đã triển khai

| Actor | Use Case | Trạng thái |
|-------|----------|------------|
| All | Login (<<include>>) | ✅ |
| Student | Register for courses | ✅ |
| Student | View report card | ✅ |
| Professor | Select courses to teach | ✅ |
| Professor | Submit grades | ✅ |
| Registrar | Maintain student information | ✅ |
| Registrar | Maintain professor information | ✅ |
| Registrar | Maintain curriculum | ✅ |

## Công nghệ

- **Next.js 14** (App Router)
- **Prisma** + SQLite
- **Tailwind CSS**
- **TypeScript**

## Cài đặt & Chạy

```bash
# Cài dependencies
npm install

# Tạo database + seed dữ liệu demo
npm run db:setup

# Chạy dev server
npm run dev
```

Mở [http://localhost:3000](http://localhost:3000)

## Tài khoản demo

Mật khẩu chung: **123456**

| Vai trò | Email |
|---------|-------|
| Sinh viên | sv1@university.edu |
| Giảng viên | gv1@university.edu |
| Giáo vụ | giaovu@university.edu |

## Cấu trúc dự án

```
src/
├── app/
│   ├── api/          # REST API theo từng role
│   ├── login/        # Trang đăng nhập
│   ├── student/      # UC: Đăng ký HP, Bảng điểm
│   ├── professor/    # UC: Chọn môn dạy, Nhập điểm
│   └── registrar/    # UC: Quản lý SV, GV, CTĐT
├── components/       # UI components
└── lib/              # Auth, DB, utils
```

## Tài liệu thiết kế

Xem chi tiết tại [DESIGN.md](./DESIGN.md):
- Phân tích UC diagram
- ERD database
- API specification
- Quy tắc nghiệp vụ
- Sequence diagram

## Quy tắc nghiệp vụ

- Mọi chức năng yêu cầu **đăng nhập** (Login <<include>>)
- Đăng ký HP: kiểm tra sĩ số, trùng lịch, trùng môn
- Một lớp chỉ có 1 giảng viên
- Chỉ GV được phân công mới nhập điểm
