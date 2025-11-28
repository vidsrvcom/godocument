# Go Modules Reference

## Giới thiệu

Go Modules là hệ thống quản lý phụ thuộc chính thức của Go, được giới thiệu từ Go 1.11 và trở thành mặc định từ Go 1.16.

## Module là gì?

Một module là một tập hợp các Go package được phát hành, phiên bản hóa và phân phối cùng nhau. Module được xác định bởi file `go.mod` ở thư mục gốc.

## Khởi tạo Module

```bash
go mod init example.com/mymodule
```

Lệnh này tạo file `go.mod`:

```
module example.com/mymodule

go 1.21
```

## File go.mod

### Cú pháp

```
module example.com/mymodule

go 1.21

require (
    golang.org/x/text v0.3.0
    rsc.io/quote v1.5.2
)

replace example.com/greetings => ../greetings

exclude example.com/old v1.0.0
```

### Chỉ thị

| Chỉ thị | Mô tả |
|---------|-------|
| `module` | Khai báo module path |
| `go` | Phiên bản Go tối thiểu |
| `require` | Khai báo phụ thuộc |
| `replace` | Thay thế module |
| `exclude` | Loại trừ phiên bản cụ thể |
| `retract` | Rút lại phiên bản đã phát hành |

## File go.sum

File `go.sum` chứa hash của các phụ thuộc để đảm bảo tính toàn vẹn:

```
golang.org/x/text v0.3.0 h1:g61tztE5qeGQ89tm6NTjjM9VPIm088od1l6aSorWRWg=
golang.org/x/text v0.3.0/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
```

## Lệnh Go Mod

### go mod init

Khởi tạo module mới:

```bash
go mod init example.com/mymodule
```

### go mod tidy

Thêm các phụ thuộc thiếu và xóa các phụ thuộc không dùng:

```bash
go mod tidy
```

### go mod download

Tải xuống các phụ thuộc:

```bash
go mod download
```

### go mod vendor

Tạo thư mục vendor:

```bash
go mod vendor
```

### go mod graph

Hiển thị đồ thị phụ thuộc:

```bash
go mod graph
```

### go mod verify

Xác minh phụ thuộc:

```bash
go mod verify
```

### go mod why

Giải thích tại sao cần một phụ thuộc:

```bash
go mod why golang.org/x/text
```

## Phiên bản

### Semantic Versioning

Go Modules sử dụng semantic versioning (v1.2.3):
- **Major** (v1): Thay đổi không tương thích
- **Minor** (v1.2): Thêm tính năng mới tương thích
- **Patch** (v1.2.3): Sửa lỗi

### Major Version Suffix

Từ v2 trở lên, module path phải có suffix:

```
module example.com/mymodule/v2
```

### Pseudo-versions

Phiên bản cho commits chưa có tag:

```
v0.0.0-20210101120000-abcdef123456
```

## Thêm Phụ thuộc

### Thêm phụ thuộc mới

```bash
go get golang.org/x/text
```

### Thêm phiên bản cụ thể

```bash
go get golang.org/x/text@v0.3.0
```

### Cập nhật phụ thuộc

```bash
go get -u golang.org/x/text
```

### Cập nhật tất cả

```bash
go get -u ./...
```

## Replace

Thay thế module cục bộ:

```go
replace example.com/greetings => ../greetings
```

Thay thế bằng fork:

```go
replace github.com/original/pkg => github.com/forked/pkg v1.0.0
```

## Private Modules

### Cấu hình GOPRIVATE

```bash
go env -w GOPRIVATE=*.corp.example.com
```

### Cấu hình GONOSUMDB

```bash
go env -w GONOSUMDB=*.corp.example.com
```

## Workspaces (Go 1.18+)

File `go.work`:

```
go 1.21

use (
    ./module1
    ./module2
)
```

### Lệnh workspace

```bash
go work init ./module1 ./module2
go work use ./module3
go work sync
```

## Module Cache

Vị trí mặc định: `$GOPATH/pkg/mod`

Xóa cache:

```bash
go clean -modcache
```

---

*Xem tài liệu đầy đủ tại [go.dev/ref/mod](https://go.dev/ref/mod)*
