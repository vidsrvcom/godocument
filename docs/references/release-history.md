# Bản phát hành Go

## Giới thiệu

Trang này tổng hợp các bản phát hành chính của Go, bao gồm các tính năng mới, cải tiến và thay đổi quan trọng.

## Chính sách phát hành

Go tuân theo chu kỳ phát hành 6 tháng:
- Mỗi bản phát hành mới bao gồm các tính năng, cải tiến và sửa lỗi
- Hai phiên bản gần nhất được hỗ trợ với các bản vá bảo mật

## Go 1.22 (Tháng 2, 2024)

### Thay đổi ngôn ngữ
- `for` loop variables giờ được tạo lại mỗi iteration
- Range over integers: `for i := range 10 { }`

### Thư viện chuẩn
- `math/rand/v2`: API mới cho random numbers
- `net/http`: Cải tiến routing patterns

### Công cụ
- Cải tiến `go mod` commands
- Build cache improvements

## Go 1.21 (Tháng 8, 2023)

### Thay đổi ngôn ngữ
- Thêm built-in functions: `min`, `max`, `clear`
- Cải tiến type inference cho generics

### Thư viện chuẩn
- `log/slog`: Structured logging mới
- `slices`: Functions cho slice operations
- `maps`: Functions cho map operations
- `cmp`: Compare functions

### Công cụ
- `go work vendor` support
- Profile-guided optimization (PGO) mặc định

## Go 1.20 (Tháng 2, 2023)

### Thay đổi ngôn ngữ
- Chuyển đổi slice thành array
- Comparable constraint cho generics

### Thư viện chuẩn
- `errors.Join`: Kết hợp nhiều errors
- `context.WithCancelCause`
- `crypto/ecdh`: ECDH implementation

### Công cụ
- Coverage profiling cho applications (không chỉ tests)
- `go build -pgo` support

## Go 1.19 (Tháng 8, 2022)

### Thay đổi ngôn ngữ
- Không có thay đổi ngôn ngữ

### Thư viện chuẩn
- `sync/atomic` types mới
- Memory model documentation updates

### Công cụ
- Cải tiến doc comments
- `GOAMD64` environment variable

## Go 1.18 (Tháng 3, 2022)

### Thay đổi ngôn ngữ - Generics!
```go
func Print[T any](s []T) {
    for _, v := range s {
        fmt.Println(v)
    }
}
```

### Thư viện chuẩn
- `constraints` package (nay deprecated, sử dụng `cmp.Ordered`)
- `any` alias cho `interface{}`
- `comparable` constraint

### Công cụ
- **Fuzzing** built-in
- **Workspaces**: `go.work` files
- `go mod graph` improvements

## Go 1.17 (Tháng 8, 2021)

### Thay đổi ngôn ngữ
- Chuyển đổi slice thành array pointer

### Thư viện chuẩn
- `runtime/cgo`: Cải tiến cgo support

### Công cụ
- Module graph pruning
- Lazy module loading

## Go 1.16 (Tháng 2, 2021)

### Thay đổi ngôn ngữ
- `go:embed` directive

```go
//go:embed hello.txt
var s string
```

### Thư viện chuẩn
- `io/fs`: Filesystem abstractions
- `embed` package
- `io.ReadAll`, `io.NopCloser` (moved from ioutil)

### Công cụ
- Modules mặc định (`GO111MODULE=on`)
- `go install pkg@version`

## Go 1.15 (Tháng 8, 2020)

### Cải tiến
- Smaller binaries
- Allocator improvements
- `time.Ticker` improvements

## Go 1.14 (Tháng 2, 2020)

### Thư viện chuẩn
- `hash/maphash`: Deterministic hashing
- `testing.T.Cleanup`

### Cải tiến
- Goroutine preemption
- Faster defer

## Go 1.13 (Tháng 9, 2019)

### Thay đổi ngôn ngữ
- Number literals mới: `0b1010`, `0o755`, `1_000_000`

### Thư viện chuẩn
- `errors.Is`, `errors.As`, `errors.Unwrap`
- `fmt.Errorf` với `%w` verb

```go
if errors.Is(err, os.ErrNotExist) {
    // ...
}

err := fmt.Errorf("failed: %w", originalErr)
```

### Công cụ
- Go Modules proxy mặc định

## Các phiên bản cũ hơn

| Version | Release Date | Notable Features |
|---------|--------------|------------------|
| Go 1.12 | Feb 2019 | TLS 1.3 mặc định |
| Go 1.11 | Aug 2018 | Go Modules (experimental) |
| Go 1.10 | Feb 2018 | Test caching |
| Go 1.9 | Aug 2017 | Type aliases |
| Go 1.8 | Feb 2017 | HTTP/2 push |
| Go 1.7 | Aug 2016 | Context package |
| Go 1.6 | Feb 2016 | HTTP/2 support |
| Go 1.5 | Aug 2015 | Garbage collector rewrite |
| Go 1.4 | Dec 2014 | go generate |
| Go 1.3 | Jun 2014 | Sync pool |
| Go 1.2 | Dec 2013 | Three-index slices |
| Go 1.1 | May 2013 | Race detector |
| Go 1 | Mar 2012 | First stable release |

## Cài đặt phiên bản cũ hơn

```bash
go install golang.org/dl/go1.20@latest
go1.20 download
go1.20 version
```

## Kiểm tra phiên bản

```bash
go version
```

---

*Xem release notes đầy đủ tại [go.dev/doc/devel/release](https://go.dev/doc/devel/release)*
