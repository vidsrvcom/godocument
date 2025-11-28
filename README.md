# godocument
Tài liệu Go tiếng Việt (Vietnamese Go Documentation)

## Giới thiệu

Dự án này cung cấp bản dịch tiếng Việt **đầy đủ và chi tiết** của tài liệu chính thức ngôn ngữ lập trình Go. Mục tiêu là giúp các lập trình viên Việt Nam tiếp cận và học Go dễ dàng hơn.

## Nội dung

- [doc.md](doc.md) - Trang tài liệu chính (dịch từ [go.dev/doc](https://go.dev/doc/))

### Hướng dẫn chi tiết (Tutorials)

- [Cài đặt Go](tutorials/install.md) - Hướng dẫn tải xuống và cài đặt Go
- [Bắt đầu với Go](tutorials/getting-started.md) - Hướng dẫn "Hello, World!" để thiết lập và sử dụng Go
- [Tạo một module Go](tutorials/create-module.md) - Hướng dẫn về hàm, xử lý lỗi, mảng, map, unit testing và biên dịch
- [Multi-module workspaces](tutorials/workspaces.md) - Giới thiệu về tạo và sử dụng multi-module workspaces
- [RESTful API với Go và Gin](tutorials/web-service-gin.md) - Viết RESTful web service API với Gin Framework
- [Bắt đầu với generics](tutorials/generics.md) - Khai báo và sử dụng hàm hoặc kiểu generic
- [Bắt đầu với fuzzing](tutorials/fuzzing.md) - Tạo các đầu vào cho test để bắt các trường hợp biên
- [Truy cập cơ sở dữ liệu quan hệ](tutorials/database-access.md) - Truy cập database với package database/sql
- [Viết ứng dụng web](tutorials/web-application.md) - Xây dựng một ứng dụng web đơn giản
- [Cách viết mã Go](tutorials/how-to-write-go-code.md) - Phát triển các Go package bên trong một module
- [Tìm và sửa lỗ hổng bảo mật](tutorials/vulnerability.md) - Sử dụng govulncheck để tìm và sửa các lỗ hổng bảo mật

### Các phần chính trong tài liệu:

| Phần | Nội dung |
|------|----------|
| **Bắt đầu** | Cài đặt Go, Hello World, tạo module, workspaces, REST API, generics, fuzzing |
| **Sử dụng và hiểu Go** | Effective Go, đặc tả ngôn ngữ, thư viện chuẩn, modules, mô hình bộ nhớ, FAQ |
| **Tham chiếu** | Lịch sử phát hành, tương thích Go 1, Wiki, biến môi trường |
| **Hỗ trợ IDE** | VS Code, GoLand, Vim, gopls configuration |
| **Bảo mật** | Chính sách bảo mật, govulncheck, best practices, SQL injection prevention |
| **Công cụ chẩn đoán** | Garbage collector, profiling, PGO |
| **Truy cập Database** | CRUD, transactions, prepared statements, connection pool |
| **Cgo** | Gọi C từ Go, gọi Go từ C |
| **Chủ đề nâng cao** | Coverage, fuzzing, benchmarks, race detector, assembly |
| **Telemetry** | Cấu hình, quyền riêng tư |
| **Build và Release** | Module mirror, private modules, reproducible builds |
| **Phát triển Go** | Đóng góp mã, quy trình phát triển |
| **Cộng đồng** | Diễn đàn, Slack, meetups, tài nguyên học tập |

### Tính năng nổi bật:
- ✅ **3000+ dòng** tài liệu chi tiết
- ✅ **Hàng trăm ví dụ code** thực tế
- ✅ **Bảng thuật ngữ** Việt - Anh
- ✅ **Hướng dẫn từng bước** cho người mới
- ✅ **Best practices** cho security và performance

## Đóng góp

Chúng tôi hoan nghênh mọi đóng góp! Nếu bạn muốn cải thiện bản dịch hoặc thêm nội dung mới, vui lòng tạo Pull Request.

## Giấy phép

Dự án này được cấp phép theo giấy phép MIT - xem file [LICENSE](LICENSE) để biết chi tiết.

---

## Introduction (English)

This project provides a **comprehensive and detailed** Vietnamese translation of the official Go programming language documentation. The goal is to help Vietnamese developers access and learn Go more easily.

## Contents

- [doc.md](doc.md) - Main documentation page (translated from [go.dev/doc](https://go.dev/doc/))

### Detailed Tutorials

- [Installing Go](tutorials/install.md) - Guide to download and install Go
- [Getting Started with Go](tutorials/getting-started.md) - "Hello, World!" tutorial for setting up and using Go
- [Create a Go module](tutorials/create-module.md) - Tutorial on functions, error handling, arrays, maps, unit testing and compilation
- [Multi-module workspaces](tutorials/workspaces.md) - Introduction to creating and using multi-module workspaces
- [RESTful API with Go and Gin](tutorials/web-service-gin.md) - Writing RESTful web service API with Gin Framework
- [Getting started with generics](tutorials/generics.md) - Declaring and using generic functions or types
- [Getting started with fuzzing](tutorials/fuzzing.md) - Creating test inputs to catch edge cases
- [Accessing a relational database](tutorials/database-access.md) - Accessing database with database/sql package
- [Writing web applications](tutorials/web-application.md) - Building a simple web application
- [How to write Go code](tutorials/how-to-write-go-code.md) - Developing Go packages within a module
- [Finding and fixing vulnerabilities](tutorials/vulnerability.md) - Using govulncheck to find and fix security vulnerabilities

### Main sections in the documentation:

| Section | Content |
|---------|---------|
| **Getting Started** | Installing Go, Hello World, creating modules, workspaces, REST API, generics, fuzzing |
| **Using and Understanding Go** | Effective Go, language specification, standard library, modules, memory model, FAQ |
| **References** | Release history, Go 1 compatibility, Wiki, environment variables |
| **IDE Support** | VS Code, GoLand, Vim, gopls configuration |
| **Security** | Security policy, govulncheck, best practices, SQL injection prevention |
| **Diagnostics** | Garbage collector, profiling, PGO |
| **Database Access** | CRUD, transactions, prepared statements, connection pool |
| **Cgo** | Calling C from Go, calling Go from C |
| **Advanced Topics** | Coverage, fuzzing, benchmarks, race detector, assembly |
| **Telemetry** | Configuration, privacy |
| **Building and Releasing** | Module mirror, private modules, reproducible builds |
| **Developing Go** | Contributing code, development process |
| **Community** | Forum, Slack, meetups, learning resources |

### Key Features:
- ✅ **3000+ lines** of detailed documentation
- ✅ **Hundreds of code examples**
- ✅ **Vietnamese-English glossary**
- ✅ **Step-by-step guides** for beginners
- ✅ **Best practices** for security and performance

## Contributing

We welcome all contributions! If you want to improve translations or add new content, please create a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
