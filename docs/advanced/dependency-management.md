# Quản lý phụ thuộc

## Giới thiệu

Khi mã của bạn sử dụng các package bên ngoài, các package đó (được phân phối dưới dạng module) trở thành các phụ thuộc.

## Module Basics

### Khởi tạo module

```bash
go mod init example.com/myproject
```

### Thêm dependency

```bash
go get github.com/gin-gonic/gin
```

### Cập nhật dependencies

```bash
go get -u ./...
go mod tidy
```

## go.mod và go.sum

### go.mod

```
module example.com/myproject

go 1.21

require (
    github.com/gin-gonic/gin v1.9.0
)
```

### go.sum

Chứa checksums để verify integrity.

## Versioning

### Semantic Versioning

```
v1.2.3
v1 = major (breaking changes)
2 = minor (new features)
3 = patch (bug fixes)
```

### Chọn version

```bash
go get pkg@v1.2.3      # Specific version
go get pkg@latest      # Latest version
go get pkg@master      # Branch
go get pkg@abc123      # Commit
```

## Best Practices

1. **go mod tidy thường xuyên**
2. **Review dependencies trước khi thêm**
3. **Kiểm tra vulnerabilities với govulncheck**
4. **Vendor khi cần reproducible builds**

---

*Xem thêm tại [go.dev/doc/modules](https://go.dev/doc/modules)*
