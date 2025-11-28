# Tài liệu Tham khảo về Môi trường

## Giới thiệu

Go sử dụng nhiều biến môi trường để điều khiển hành vi của các công cụ Go. Tài liệu này mô tả các biến môi trường quan trọng nhất.

## Xem và Thiết lập

### Xem tất cả biến môi trường Go

```bash
go env
```

### Xem biến cụ thể

```bash
go env GOPATH
go env GOPROXY
```

### Thiết lập biến

```bash
go env -w GOPRIVATE=*.corp.example.com
go env -w GOPROXY=https://proxy.golang.org,direct
```

### Xóa thiết lập

```bash
go env -u GOPRIVATE
```

## Các Biến Môi trường Chính

### GOROOT

Thư mục cài đặt Go.

```bash
GOROOT=/usr/local/go
```

Thường được tự động thiết lập khi cài đặt Go.

### GOPATH

Thư mục workspace của Go.

```bash
GOPATH=$HOME/go
```

Mặc định: `$HOME/go` (Unix) hoặc `%USERPROFILE%\go` (Windows)

Cấu trúc:
```
$GOPATH/
├── bin/     # Installed binaries
├── pkg/     # Compiled packages (cũ, pre-modules)
└── src/     # Source code (cũ, pre-modules)
```

### GOBIN

Thư mục cài đặt binaries.

```bash
GOBIN=$HOME/go/bin
```

Mặc định: `$GOPATH/bin`

### GO111MODULE

Điều khiển module mode.

```bash
GO111MODULE=on    # Luôn sử dụng modules
GO111MODULE=off   # Không sử dụng modules (deprecated)
GO111MODULE=auto  # Tự động (mặc định từ Go 1.16)
```

## Biến Module

### GOPROXY

URL của module proxy.

```bash
GOPROXY=https://proxy.golang.org,direct
```

Mặc định: `https://proxy.golang.org,direct`

Các options:
- `direct`: Tải trực tiếp từ VCS
- `off`: Không sử dụng proxy
- URL proxy: Sử dụng proxy cụ thể

### GOPRIVATE

Patterns cho private modules (không đi qua proxy hoặc checksum).

```bash
GOPRIVATE=*.corp.example.com,github.com/mycompany/*
```

### GONOPROXY

Patterns không đi qua proxy.

```bash
GONOPROXY=*.corp.example.com
```

### GONOSUMDB

Patterns không kiểm tra trong checksum database.

```bash
GONOSUMDB=*.corp.example.com
```

### GOSUMDB

URL của checksum database.

```bash
GOSUMDB=sum.golang.org
```

Mặc định: `sum.golang.org`

Tắt: `GOSUMDB=off`

## Biến Build

### GOOS

Target operating system.

```bash
GOOS=linux
GOOS=windows
GOOS=darwin
```

Các giá trị phổ biến:
| GOOS | OS |
|------|-----|
| linux | Linux |
| windows | Windows |
| darwin | macOS |
| freebsd | FreeBSD |
| js | JavaScript (WebAssembly) |

### GOARCH

Target architecture.

```bash
GOARCH=amd64
GOARCH=arm64
GOARCH=386
```

Các giá trị phổ biến:
| GOARCH | Architecture |
|--------|--------------|
| amd64 | x86-64 |
| 386 | x86 |
| arm | ARM |
| arm64 | ARM64 |
| wasm | WebAssembly |

### CGO_ENABLED

Bật/tắt cgo.

```bash
CGO_ENABLED=1  # Bật (mặc định)
CGO_ENABLED=0  # Tắt
```

### CC và CXX

C và C++ compiler cho cgo.

```bash
CC=gcc
CXX=g++
```

### GOFLAGS

Flags mặc định cho lệnh go.

```bash
GOFLAGS="-mod=vendor"
```

### GOTOOLCHAIN

Toolchain Go sử dụng (Go 1.21+).

```bash
GOTOOLCHAIN=go1.21.0
GOTOOLCHAIN=auto
GOTOOLCHAIN=local
```

## Biến Testing

### GOTEST

Testing flags mặc định.

```bash
go test -v ./...
```

### Benchmark

```bash
go test -bench=. -benchmem
```

## Biến Debug

### GODEBUG

Các flags debug.

```bash
GODEBUG=gctrace=1
GODEBUG=schedtrace=1000
GODEBUG=http2debug=1
```

Các options phổ biến:
| Flag | Mô tả |
|------|-------|
| gctrace=1 | Log GC activity |
| schedtrace=1000 | Log scheduler state mỗi 1000ms |
| http2debug=1 | HTTP/2 debug logs |
| inittrace=1 | Log package init times |

### GOMAXPROCS

Số OS threads tối đa có thể chạy goroutines đồng thời.

```bash
GOMAXPROCS=4
```

Mặc định: Số CPU cores.

### GOGC

Garbage collection target percentage.

```bash
GOGC=100  # Mặc định
GOGC=off  # Tắt GC
GOGC=50   # GC thường xuyên hơn
```

### GOMEMLIMIT

Giới hạn memory (Go 1.19+).

```bash
GOMEMLIMIT=1GiB
```

## Cross-Compilation

### Ví dụ

```bash
# Build cho Linux AMD64
GOOS=linux GOARCH=amd64 go build -o myapp-linux

# Build cho Windows AMD64
GOOS=windows GOARCH=amd64 go build -o myapp.exe

# Build cho macOS ARM64
GOOS=darwin GOARCH=arm64 go build -o myapp-darwin

# Build static binary (không cần cgo)
CGO_ENABLED=0 go build -o myapp-static
```

### Danh sách targets

```bash
go tool dist list
```

## Ví dụ cấu hình

### Development

```bash
# ~/.bashrc hoặc ~/.zshrc
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

### CI/CD

```bash
# Trong CI script
export CGO_ENABLED=0
export GOPROXY=https://proxy.golang.org,direct
go build -trimpath -ldflags="-s -w" -o app
```

### Private Modules

```bash
go env -w GOPRIVATE=github.com/mycompany/*
go env -w GOPROXY=https://proxy.golang.org,direct
```

---

*Xem tài liệu đầy đủ bằng `go help environment` hoặc tại [go.dev/ref/environment](https://go.dev/ref/environment)*
