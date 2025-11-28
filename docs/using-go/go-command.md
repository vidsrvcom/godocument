# Tham chiếu Lệnh Go

## Giới thiệu

Công cụ `go` là chương trình chính để quản lý mã nguồn Go. Nó cung cấp các lệnh để biên dịch, test, format và nhiều tác vụ khác.

## Cú pháp chung

```bash
go <command> [arguments]
```

## Các lệnh chính

### go build

Biên dịch packages và dependencies.

```bash
# Biên dịch package hiện tại
go build

# Biên dịch file cụ thể
go build main.go

# Biên dịch với tên output
go build -o myapp

# Biên dịch cho nền tảng khác
GOOS=linux GOARCH=amd64 go build
```

### go run

Biên dịch và chạy chương trình Go.

```bash
go run main.go
go run .
go run main.go arg1 arg2
```

### go install

Biên dịch và cài đặt packages.

```bash
# Cài đặt package hiện tại
go install

# Cài đặt package từ xa
go install golang.org/x/tools/gopls@latest
```

### go test

Chạy tests.

```bash
# Test package hiện tại
go test

# Test với verbose
go test -v

# Test tất cả packages
go test ./...

# Test cụ thể
go test -run TestFunctionName

# Test với coverage
go test -cover

# Benchmark
go test -bench=.
```

### go get

Thêm dependencies vào module hiện tại.

```bash
# Thêm dependency
go get golang.org/x/text

# Thêm phiên bản cụ thể
go get golang.org/x/text@v0.3.0

# Cập nhật dependency
go get -u golang.org/x/text

# Cập nhật tất cả
go get -u ./...
```

### go mod

Quản lý modules.

```bash
go mod init example.com/mymodule
go mod tidy
go mod download
go mod vendor
go mod verify
go mod graph
go mod why <package>
```

### go fmt

Format mã nguồn.

```bash
# Format package hiện tại
go fmt

# Format tất cả files
go fmt ./...
```

### go vet

Kiểm tra lỗi có thể xảy ra.

```bash
go vet
go vet ./...
```

### go doc

Hiển thị documentation.

```bash
# Doc cho package
go doc fmt

# Doc cho function
go doc fmt.Println

# Doc chi tiết
go doc -all fmt
```

### go generate

Chạy các lệnh generate được định nghĩa trong comments.

```bash
go generate
go generate ./...
```

### go clean

Xóa files được tạo bởi build.

```bash
go clean
go clean -cache      # Xóa build cache
go clean -testcache  # Xóa test cache
go clean -modcache   # Xóa module cache
```

### go env

Hiển thị/thiết lập biến môi trường Go.

```bash
# Hiển thị tất cả
go env

# Hiển thị biến cụ thể
go env GOPATH

# Thiết lập biến
go env -w GOPRIVATE=*.corp.example.com
```

### go version

Hiển thị phiên bản Go.

```bash
go version
```

### go list

Liệt kê packages hoặc modules.

```bash
# Liệt kê packages
go list ./...

# Liệt kê modules
go list -m all

# Format output
go list -json ./...
```

### go work (Go 1.18+)

Quản lý workspaces.

```bash
go work init ./module1 ./module2
go work use ./module3
go work sync
```

## Build Flags

### -o

Chỉ định tên file output.

```bash
go build -o myapp
```

### -v

Verbose output.

```bash
go build -v
```

### -race

Bật race detector.

```bash
go build -race
go test -race
```

### -ldflags

Truyền flags cho linker.

```bash
go build -ldflags="-s -w"  # Giảm kích thước binary
go build -ldflags="-X main.version=1.0.0"  # Inject variables
```

### -tags

Chỉ định build tags.

```bash
go build -tags=prod
```

### -trimpath

Xóa đường dẫn file từ binary.

```bash
go build -trimpath
```

## Biến môi trường

| Biến | Mô tả |
|------|-------|
| `GOPATH` | Thư mục workspace |
| `GOROOT` | Thư mục cài đặt Go |
| `GOPROXY` | Module proxy URL |
| `GOPRIVATE` | Patterns cho private modules |
| `GOBIN` | Thư mục cài đặt binaries |
| `GOOS` | Target operating system |
| `GOARCH` | Target architecture |

## Cross-compilation

```bash
# Linux AMD64
GOOS=linux GOARCH=amd64 go build

# Windows AMD64
GOOS=windows GOARCH=amd64 go build

# macOS ARM64
GOOS=darwin GOARCH=arm64 go build
```

## Ví dụ Workflow

```bash
# Khởi tạo project
mkdir myproject && cd myproject
go mod init example.com/myproject

# Thêm dependencies
go get github.com/gin-gonic/gin

# Build và test
go build ./...
go test ./...

# Format và vet
go fmt ./...
go vet ./...

# Cài đặt
go install
```

---

*Xem tài liệu đầy đủ bằng `go help` hoặc tại [go.dev/cmd/go](https://go.dev/cmd/go)*
