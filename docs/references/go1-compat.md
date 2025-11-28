# Cam kết Tương thích Go 1

## Giới thiệu

Go 1 định nghĩa một ngôn ngữ và một bộ thư viện cốt lõi. Mục tiêu của cam kết tương thích Go 1 là các chương trình được viết cho Go 1 sẽ tiếp tục biên dịch và chạy đúng, không thay đổi, trong suốt vòng đời của Go 1.

## Cam kết

### Những gì được đảm bảo

1. **Ngôn ngữ**: Các chương trình Go 1 hợp lệ sẽ tiếp tục biên dịch.

2. **Thư viện chuẩn**: API công khai sẽ không thay đổi theo cách phá vỡ code hiện có.

3. **Lệnh Go**: Cú pháp và ngữ nghĩa của các lệnh go sẽ được duy trì.

### Những gì KHÔNG được đảm bảo

1. **Bugs**: Sửa lỗi có thể thay đổi hành vi nếu hành vi cũ là sai.

2. **Undocumented behavior**: Hành vi không được document có thể thay đổi.

3. **Internal APIs**: Các package bắt đầu bằng `internal/` không được đảm bảo.

4. **Unsafe package**: Sử dụng `unsafe` có thể bị phá vỡ.

5. **Performance**: Hiệu suất có thể thay đổi.

## Ví dụ về tương thích

### Code Go 1.0 vẫn hoạt động

```go
// Đây là code Go 1.0 hợp lệ, vẫn hoạt động trong Go 1.22+
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### Tính năng mới không phá vỡ code cũ

```go
// Go 1.18 thêm generics, nhưng code không dùng generics vẫn hoạt động
func Sum(a, b int) int {
    return a + b
}

// Code mới có thể dùng generics
func SumGeneric[T int | float64](a, b T) T {
    return a + b
}
```

## Các thay đổi được phép

### 1. Thêm tính năng mới

Go có thể thêm:
- Từ khóa mới (cẩn thận để không xung đột)
- Types và functions mới trong thư viện chuẩn
- Methods mới cho types hiện có

### 2. Sửa lỗi

```go
// Nếu phát hiện bug, có thể sửa dù thay đổi hành vi
// Ví dụ: time.Parse sửa lỗi parsing timezone
```

### 3. Cải tiến hiệu suất

Thay đổi implementation để cải thiện hiệu suất là được phép.

### 4. Deprecation

API cũ có thể bị đánh dấu deprecated nhưng vẫn hoạt động:

```go
// Deprecated: Use io.ReadAll instead.
func ioutil.ReadAll(r io.Reader) ([]byte, error)
```

## GODEBUG

Từ Go 1.21, `GODEBUG` cho phép kiểm soát hành vi:

```bash
GODEBUG=httpmuxgo121=1 go run main.go
```

### Ví dụ

```go
// go.mod
module example.com/myapp

go 1.21

godebug default=go1.21
```

## Các trường hợp đặc biệt

### 1. Security fixes

```go
// Sửa lỗi bảo mật có thể phá vỡ code dựa vào hành vi không an toàn
// Ví dụ: TLS mặc định bật verification
```

### 2. Operating system changes

```go
// Nếu OS thay đổi API, Go có thể phải điều chỉnh
// Ví dụ: macOS notarization requirements
```

### 3. Compiler optimizations

```go
// Compiler có thể tối ưu theo cách thay đổi memory layout
// Không nên dựa vào sizeof hoặc layout cụ thể
```

## Best Practices

### 1. Sử dụng API được document

```go
// Tốt: Sử dụng API công khai
import "encoding/json"

// Tránh: Sử dụng internal packages
// import "internal/something"
```

### 2. Không dựa vào undocumented behavior

```go
// Không nên giả định về:
// - Thứ tự iteration của map
// - Memory layout của structs
// - Garbage collection timing
```

### 3. Tránh unsafe khi có thể

```go
// Chỉ sử dụng unsafe khi thực sự cần thiết
// và hiểu rõ rủi ro
import "unsafe"
```

### 4. Kiểm tra build tags

```go
//go:build go1.18

// Code này chỉ build với Go 1.18+
func UseGenerics[T any](x T) T {
    return x
}
```

## Kiểm tra tương thích

### go vet

```bash
go vet ./...
```

### staticcheck

```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
```

### Cập nhật dependencies

```bash
go get -u ./...
go mod tidy
```

## Lịch sử cam kết

- **Go 1 (2012)**: Cam kết tương thích bắt đầu
- **Go 1.11 (2018)**: Modules được giới thiệu
- **Go 1.18 (2022)**: Generics được thêm (tương thích ngược)
- **Go 1.21 (2023)**: GODEBUG settings được mở rộng

---

*Xem tài liệu đầy đủ tại [go.dev/doc/go1compat](https://go.dev/doc/go1compat)*
