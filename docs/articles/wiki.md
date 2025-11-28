# Wiki Go - Các bài viết

## Giới thiệu

Wiki Go chứa nhiều bài viết và hướng dẫn do cộng đồng đóng góp về ngôn ngữ Go, công cụ và các chủ đề liên quan.

## Các chủ đề phổ biến

### Learning Go

- **[Go Tour](https://tour.golang.org/)**: Tour tương tác cơ bản
- **[Effective Go](https://go.dev/doc/effective_go)**: Best practices
- **[How to Write Go Code](https://go.dev/doc/code)**: Tổ chức code

### Best Practices

- Error handling patterns
- Interface design
- Package organization
- Testing strategies

### Concurrency

- Goroutine patterns
- Channel usage
- sync package
- Context handling

### Performance

- Profiling
- Benchmarking
- Memory optimization
- CPU optimization

## Các bài viết nổi bật

### CodeReviewComments

Các comment thường gặp trong code review:
- Gofmt
- Comment sentences
- Error handling
- Package comments

### CommonMistakes

Các lỗi phổ biến:
- Not checking errors
- Using default HTTP client
- Improper slice usage
- Race conditions

### TableDrivenTests

Cách viết table-driven tests:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", 
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

## Đóng góp

Wiki được duy trì bởi cộng đồng. Bất kỳ ai cũng có thể đóng góp.

---

*Xem Wiki đầy đủ tại [go.dev/wiki](https://go.dev/wiki)*
