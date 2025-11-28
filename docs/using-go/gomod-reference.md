# Tham chiếu file go.mod

## Giới thiệu

File `go.mod` là file cấu hình chính của một Go module. Nó định nghĩa module path, phiên bản Go và các phụ thuộc của module.

## Cấu trúc

```
module example.com/mymodule

go 1.21

require (
    golang.org/x/text v0.3.0
    rsc.io/quote v1.5.2
)

require (
    golang.org/x/net v0.0.0-20210226172049-e18ecbb05110 // indirect
)

replace example.com/greetings => ../greetings

exclude example.com/old v1.0.0

retract v1.0.0
```

## Chỉ thị module

Khai báo module path - định danh duy nhất của module.

```
module example.com/mymodule
```

### Quy tắc đặt tên

- Thường là URL có thể tải được
- Không có khoảng trắng
- Phân biệt chữ hoa/thường

## Chỉ thị go

Khai báo phiên bản Go tối thiểu cần thiết.

```
go 1.21
```

Hoặc với minor version:

```
go 1.21.0
```

## Chỉ thị require

Khai báo các phụ thuộc của module.

### Cú pháp đơn

```
require golang.org/x/text v0.3.0
```

### Cú pháp block

```
require (
    golang.org/x/text v0.3.0
    rsc.io/quote v1.5.2
)
```

### Phụ thuộc indirect

Phụ thuộc không được import trực tiếp:

```
require golang.org/x/net v0.0.0-20210226172049-e18ecbb05110 // indirect
```

## Chỉ thị replace

Thay thế một module bằng module khác.

### Thay thế bằng đường dẫn cục bộ

```
replace example.com/greetings => ../greetings
```

### Thay thế bằng module khác

```
replace example.com/broken => example.com/fixed v1.0.0
```

### Thay thế phiên bản cụ thể

```
replace example.com/pkg v1.0.0 => example.com/pkg v1.0.1
```

## Chỉ thị exclude

Loại trừ một phiên bản cụ thể của module.

```
exclude example.com/old v1.0.0
```

### Block exclude

```
exclude (
    example.com/old v1.0.0
    example.com/old v1.0.1
)
```

## Chỉ thị retract

Đánh dấu các phiên bản không nên sử dụng.

### Retract phiên bản đơn

```
retract v1.0.0
```

### Retract khoảng phiên bản

```
retract [v1.0.0, v1.0.5]
```

### Retract với lý do

```
// Security vulnerability
retract v1.0.0
```

## Chỉ thị toolchain (Go 1.21+)

Chỉ định phiên bản toolchain cụ thể.

```
toolchain go1.21.0
```

## Chỉ thị godebug (Go 1.21+)

Thiết lập các biến GODEBUG mặc định.

```
godebug default=go1.21
```

## Phiên bản

### Semantic Versioning

```
v1.2.3
```

- `1`: Major version
- `2`: Minor version
- `3`: Patch version

### Pre-release

```
v1.2.3-alpha.1
v1.2.3-beta.1
v1.2.3-rc.1
```

### Pseudo-version

Cho commits chưa có tag:

```
v0.0.0-20210101120000-abcdef123456
```

Format: `vX.Y.Z-yyyymmddhhmmss-abcdefabcdef`

## Major Version Suffix

Từ v2 trở lên, module path phải có suffix:

```
module example.com/mymodule/v2

require example.com/dep/v3 v3.1.0
```

## Ví dụ đầy đủ

```
module github.com/example/myproject

go 1.21

toolchain go1.21.0

require (
    github.com/gin-gonic/gin v1.9.0
    github.com/go-sql-driver/mysql v1.7.0
    golang.org/x/crypto v0.9.0
)

require (
    github.com/bytedance/sonic v1.8.0 // indirect
    github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311 // indirect
)

replace github.com/example/local => ./local

exclude github.com/example/broken v1.0.0

retract (
    v1.0.0 // Published accidentally
    [v1.0.1, v1.0.3] // Security vulnerability
)
```

## Lệnh liên quan

```bash
# Khởi tạo go.mod
go mod init example.com/mymodule

# Cập nhật go.mod
go mod tidy

# Xác minh go.mod
go mod verify

# Hiển thị các thay đổi cần thiết
go mod tidy -diff
```

---

*Xem tài liệu đầy đủ tại [go.dev/doc/modules/gomod-ref](https://go.dev/doc/modules/gomod-ref)*
