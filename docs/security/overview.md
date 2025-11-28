# Tổng quan về Bảo mật Go

## Giới thiệu

Go được thiết kế với sự chú trọng đến bảo mật. Tài liệu này cung cấp tổng quan về các tài nguyên bảo mật cho các nhà phát triển Go.

## Tính năng Bảo mật Tích hợp

### Memory Safety

Go cung cấp memory safety thông qua:
- Bounds checking cho arrays và slices
- Garbage collection
- Không có pointer arithmetic
- Nil pointer checks

```go
// Bounds checking tự động
arr := []int{1, 2, 3}
// arr[10] sẽ panic với index out of range
```

### Type Safety

Go là strongly typed:

```go
var x int = 42
// x = "hello" // Compile error!
```

### Concurrency Safety

Go cung cấp race detector:

```bash
go test -race ./...
go build -race
```

## Các Công cụ Bảo mật

### govulncheck

Kiểm tra vulnerabilities trong dependencies:

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
```

### Go Vet

Phát hiện các vấn đề tiềm ẩn:

```bash
go vet ./...
```

### Staticcheck

Phân tích tĩnh nâng cao:

```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
```

### gosec

Scanner bảo mật chuyên dụng:

```bash
go install github.com/securego/gosec/v2/cmd/gosec@latest
gosec ./...
```

## Thực hành Tốt nhất

### Input Validation

```go
func validateEmail(email string) error {
    if len(email) == 0 {
        return errors.New("email required")
    }
    if !strings.Contains(email, "@") {
        return errors.New("invalid email format")
    }
    return nil
}
```

### SQL Injection Prevention

```go
// Tốt: Sử dụng parameterized queries
db.Query("SELECT * FROM users WHERE id = ?", userID)

// Xấu: String concatenation
// db.Query("SELECT * FROM users WHERE id = " + userID)
```

### Password Handling

```go
import "golang.org/x/crypto/bcrypt"

// Hash password
hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)

// Verify password
err := bcrypt.CompareHashAndPassword(hash, []byte(password))
```

### HTTPS/TLS

```go
import "crypto/tls"

tlsConfig := &tls.Config{
    MinVersion: tls.VersionTLS12,
}

server := &http.Server{
    Addr:      ":443",
    TLSConfig: tlsConfig,
}
server.ListenAndServeTLS("cert.pem", "key.pem")
```

### Secure Random

```go
import "crypto/rand"

// Tốt: crypto/rand cho security-sensitive
token := make([]byte, 32)
rand.Read(token)

// Xấu: math/rand cho security
// math_rand.Read(token)
```

## Tài nguyên

### Go Security Policy

- [Chính sách Bảo mật Go](security-policy.md)
- Cách báo cáo vulnerabilities
- Quy trình xử lý issues

### Vulnerability Database

- [vuln.go.dev](https://vuln.go.dev)
- Cơ sở dữ liệu vulnerabilities cho Go modules

### Security Advisories

- [GitHub Security Advisories](https://github.com/golang/go/security/advisories)
- Thông báo bảo mật chính thức

## Checklist Bảo mật

### Trước khi Release

- [ ] Chạy `govulncheck ./...`
- [ ] Chạy `gosec ./...`
- [ ] Chạy `go test -race ./...`
- [ ] Review dependencies
- [ ] Kiểm tra secrets không bị commit

### Trong Code

- [ ] Validate tất cả input
- [ ] Sử dụng parameterized queries
- [ ] Hash passwords với bcrypt/argon2
- [ ] Sử dụng crypto/rand cho random
- [ ] Xử lý errors đúng cách

### Infrastructure

- [ ] Sử dụng TLS 1.2+
- [ ] Rotate secrets định kỳ
- [ ] Audit logs
- [ ] Rate limiting

---

*Xem thêm tại [go.dev/security](https://go.dev/security)*
