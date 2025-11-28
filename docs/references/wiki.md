# Go Wiki

## Giới thiệu

Go Wiki là tài nguyên do cộng đồng duy trì chứa thông tin bổ sung về ngôn ngữ Go, công cụ và các chủ đề liên quan.

## Nội dung chính

### Bắt đầu

- **[Getting Started](https://go.dev/doc/tutorial/getting-started)**: Hướng dẫn cài đặt và sử dụng Go
- **[How to Write Go Code](https://go.dev/doc/code)**: Cách tổ chức code Go
- **[Effective Go](https://go.dev/doc/effective_go)**: Best practices

### Học Go

- **[Go Tour](https://tour.golang.org/)**: Tour tương tác
- **[Go by Example](https://gobyexample.com/)**: Học qua ví dụ
- **[Exercism Go Track](https://exercism.org/tracks/go)**: Bài tập thực hành

### IDE và Editor

- **[VSCode with Go](https://code.visualstudio.com/docs/languages/go)**: Setup VSCode
- **[GoLand](https://www.jetbrains.com/go/)**: IDE từ JetBrains
- **[Vim-go](https://github.com/fatih/vim-go)**: Plugin cho Vim

### Công cụ

| Công cụ | Mô tả |
|---------|-------|
| `gofmt` | Format code |
| `go vet` | Static analysis |
| `gopls` | Language server |
| `golint` | Style checker (deprecated, use `staticcheck`) |
| `staticcheck` | Advanced static analysis |
| `golangci-lint` | Meta-linter |

### Concurrency

- **[Share Memory By Communicating](https://go.dev/blog/codelab-share)**: Pattern cơ bản
- **[Go Concurrency Patterns](https://go.dev/blog/pipelines)**: Pipelines
- **[Context](https://go.dev/blog/context)**: Sử dụng context package

### Testing

```go
// file: math_test.go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}
```

### Modules

```bash
# Khởi tạo module
go mod init example.com/myproject

# Thêm dependency
go get github.com/pkg/errors

# Cập nhật dependencies
go mod tidy
```

## Các chủ đề phổ biến

### Error Handling

```go
// Cách tiếp cận tiêu chuẩn
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// Kiểm tra loại lỗi
if errors.Is(err, os.ErrNotExist) {
    // file không tồn tại
}
```

### Interfaces

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Kết hợp interfaces
type ReadWriter interface {
    Reader
    Writer
}
```

### Generics (Go 1.18+)

```go
func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}
```

### Context

```go
func DoSomething(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case <-time.After(1 * time.Second):
        return nil
    }
}

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

### JSON

```go
type Person struct {
    Name  string `json:"name"`
    Age   int    `json:"age,omitempty"`
    Email string `json:"-"`  // không serialize
}

// Encode
data, _ := json.Marshal(person)

// Decode
var p Person
json.Unmarshal(data, &p)
```

## Tài nguyên bên ngoài

### Sách

- **The Go Programming Language** - Donovan & Kernighan
- **Go in Action** - Kennedy, Ketelsen & St. Martin
- **Concurrency in Go** - Katherine Cox-Buday
- **Learning Go** - Jon Bodner

### Blogs

- [The Go Blog](https://go.dev/blog/)
- [Dave Cheney's Blog](https://dave.cheney.net/)
- [Ardan Labs Blog](https://www.ardanlabs.com/blog/)

### Podcasts

- [Go Time](https://changelog.com/gotime)

### Conferences

- **GopherCon**: Hội nghị chính thức
- **GopherCon Europe**
- **GoLab**
- **dotGo**

## Cộng đồng

### Diễn đàn và Chat

- [Go Forum](https://forum.golangbridge.org/)
- [Gophers Slack](https://gophers.slack.com/)
- [r/golang](https://www.reddit.com/r/golang/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/go)

### Đóng góp

1. **Issues**: Báo cáo bugs tại [github.com/golang/go](https://github.com/golang/go/issues)
2. **Proposals**: Đề xuất tính năng mới
3. **Code**: Đóng góp patches
4. **Documentation**: Cải thiện docs

## Code Review Guidelines

### Đặt tên

```go
// Tốt
func ReadFile(name string) ([]byte, error)

// Không tốt
func rf(n string) ([]byte, error)
```

### Comments

```go
// Package math provides basic mathematical functions.
package math

// Add returns the sum of a and b.
func Add(a, b int) int {
    return a + b
}
```

### Error Handling

```go
// Tốt
if err != nil {
    return err
}

// Không tốt (swallowing errors)
if err != nil {
    log.Println(err)
}
```

---

*Xem Wiki đầy đủ tại [go.dev/wiki](https://go.dev/wiki)*
