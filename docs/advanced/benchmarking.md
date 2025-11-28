# Đánh giá Hiệu suất (Benchmarking)

## Giới thiệu

Go cung cấp testing framework tích hợp cho benchmarks.

## Viết Benchmarks

### Cơ bản

```go
func BenchmarkFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Function()
    }
}
```

### Với setup

```go
func BenchmarkWithSetup(b *testing.B) {
    data := setupTestData()
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        Process(data)
    }
}
```

## Chạy Benchmarks

```bash
go test -bench=.
go test -bench=. -benchmem
go test -bench=. -count=5
```

## Output

```
BenchmarkFunction-8   1000000   1234 ns/op   64 B/op   2 allocs/op
```

- `8`: GOMAXPROCS
- `1000000`: iterations
- `1234 ns/op`: nanoseconds per operation
- `64 B/op`: bytes allocated
- `2 allocs/op`: allocations

## So sánh

```bash
go test -bench=. > old.txt
# Make changes
go test -bench=. > new.txt
benchstat old.txt new.txt
```

## Sub-benchmarks

```go
func BenchmarkSizes(b *testing.B) {
    for _, size := range []int{10, 100, 1000} {
        b.Run(fmt.Sprintf("size-%d", size), func(b *testing.B) {
            data := make([]int, size)
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                Process(data)
            }
        })
    }
}
```

---

*Xem thêm tại [go.dev/doc/testing](https://go.dev/doc/testing)*
