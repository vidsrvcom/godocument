# Source Code Go

## Giới thiệu

Tài liệu này cung cấp thông tin về cách truy cập, duyệt và làm việc với mã nguồn Go.

## Repositories

### Main Go repository

```bash
# Clone từ Google Source
git clone https://go.googlesource.com/go

# Mirror trên GitHub (read-only)
git clone https://github.com/golang/go
```

### Các repositories khác

| Repository | Description | URL |
|------------|-------------|-----|
| `go` | Main Go | go.googlesource.com/go |
| `tools` | Go tools | go.googlesource.com/tools |
| `net` | Networking | go.googlesource.com/net |
| `crypto` | Cryptography | go.googlesource.com/crypto |
| `text` | Text processing | go.googlesource.com/text |
| `image` | Image processing | go.googlesource.com/image |

## Cấu trúc Go Source

### Thư mục chính

```
go/
├── src/           # Source code
│   ├── cmd/       # Commands
│   ├── runtime/   # Runtime
│   ├── internal/  # Internal packages
│   └── ...
├── doc/           # Documentation
├── test/          # Tests
├── misc/          # Miscellaneous
├── lib/           # Libraries
└── api/           # API compatibility
```

### Packages quan trọng

| Directory | Description |
|-----------|-------------|
| `src/runtime` | Go runtime |
| `src/cmd/compile` | Compiler |
| `src/cmd/link` | Linker |
| `src/cmd/go` | go command |
| `src/fmt` | Formatting |
| `src/net` | Networking |
| `src/os` | OS interface |

## Duyệt Source Code

### Online

1. **GitHub**: [github.com/golang/go](https://github.com/golang/go)
2. **Google Source**: [go.googlesource.com/go](https://go.googlesource.com/go)
3. **pkg.go.dev**: [pkg.go.dev/std](https://pkg.go.dev/std)

### Local

```bash
# Sau khi cài đặt Go
go env GOROOT
# /usr/local/go

# Duyệt
ls $(go env GOROOT)/src
```

## Build từ Source

### Prerequisites

- Một Go installation để bootstrap (Go 1.4+)
- Git

### Clone và build

```bash
# Clone
git clone https://go.googlesource.com/go goroot
cd goroot

# Checkout version
git checkout go1.21.5

# Build
cd src
./all.bash
```

### Kết quả

```bash
# Binaries trong bin/
./bin/go version
```

## Các branches

### Phát triển

| Branch | Description |
|--------|-------------|
| `master` | Development |
| `release-branch.go1.21` | Go 1.21 releases |
| `release-branch.go1.20` | Go 1.20 releases |

### Tags

```bash
# List tags
git tag | grep "go1"

# Checkout specific version
git checkout go1.21.5
```

## Reading Source Code

### Tips

1. **Start với packages đơn giản**: `strings`, `errors`
2. **Use go doc**: `go doc fmt.Printf`
3. **Follow imports**: Để hiểu dependencies
4. **Read tests**: Cho examples

### Ví dụ: Đọc fmt package

```bash
# Xem documentation
go doc fmt

# Xem source
less $(go env GOROOT)/src/fmt/print.go

# Xem tests
less $(go env GOROOT)/src/fmt/fmt_test.go
```

## Công cụ hữu ích

### go doc

```bash
# Package docs
go doc fmt

# Function docs
go doc fmt.Printf

# With source
go doc -src fmt.Printf
```

### godoc

```bash
# Install
go install golang.org/x/tools/cmd/godoc@latest

# Run server
godoc -http=:6060
```

### Source code navigation

```bash
# Trong editor với gopls
# Ctrl+Click để go to definition
```

## Standard Library

### Cấu trúc

```
src/
├── archive/       # Archive formats
├── bufio/         # Buffered I/O
├── bytes/         # Byte manipulation
├── compress/      # Compression
├── container/     # Data structures
├── context/       # Context
├── crypto/        # Cryptography
├── database/      # Database
├── encoding/      # Encoding
├── errors/        # Errors
├── flag/          # Flags
├── fmt/           # Formatting
├── go/            # Go tools
├── hash/          # Hash
├── html/          # HTML
├── image/         # Image
├── io/            # I/O
├── log/           # Logging
├── math/          # Math
├── mime/          # MIME
├── net/           # Networking
├── os/            # OS
├── path/          # Paths
├── reflect/       # Reflection
├── regexp/        # Regex
├── runtime/       # Runtime
├── sort/          # Sorting
├── strconv/       # Conversions
├── strings/       # Strings
├── sync/          # Synchronization
├── syscall/       # System calls
├── testing/       # Testing
├── text/          # Text
├── time/          # Time
├── unicode/       # Unicode
└── unsafe/        # Unsafe
```

## Contributing to Source

### 1. Find issue

```bash
# Issues labeled "help wanted"
# https://github.com/golang/go/labels/help%20wanted
```

### 2. Set up

```bash
git clone https://go.googlesource.com/go
cd go
git checkout -b my-fix
```

### 3. Make changes

```bash
# Edit source
vim src/fmt/format.go

# Build
cd src
./make.bash

# Test
./run.bash --no-rebuild
```

### 4. Submit

```bash
git add .
git commit -m "fmt: fix something"
git push origin HEAD:refs/for/master
```

## Debugging Go Source

### Với Delve

```bash
# Install
go install github.com/go-delve/delve/cmd/dlv@latest

# Debug
dlv debug .
```

### Với GDB

```bash
go build -gcflags="all=-N -l" -o prog .
gdb ./prog
```

## Resources

### Documentation

- [Go Source](https://cs.opensource.google/go/go)
- [pkg.go.dev](https://pkg.go.dev/std)
- [Go internal docs](https://github.com/golang/go/tree/master/src/cmd/compile/doc.go)

### Videos

- GopherCon talks về internals
- Go team presentations

---

*Xem thêm tại [go.googlesource.com/go](https://go.googlesource.com/go)*
