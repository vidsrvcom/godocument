# Cấu hình Fuzz

## Giới thiệu

Go 1.18+ cung cấp fuzzing tích hợp. Tài liệu này mô tả các cấu hình có thể đọc và giải quyết được bằng công cụ go.

## Fuzz Test cơ bản

```go
func FuzzReverse(f *testing.F) {
    // Seed corpus
    f.Add("hello")
    f.Add("world")
    
    // Fuzz function
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        if s != doubleRev {
            t.Errorf("Reverse(Reverse(%q)) = %q", s, doubleRev)
        }
    })
}
```

## Command Line Options

### Chạy fuzz tests

```bash
# Chạy một fuzz test cụ thể
go test -fuzz=FuzzReverse

# Chạy với thời gian giới hạn
go test -fuzz=FuzzReverse -fuzztime=30s

# Chạy với số iterations
go test -fuzz=FuzzReverse -fuzztime=1000x

# Parallel workers
go test -fuzz=FuzzReverse -parallel=4
```

### Options

| Flag | Description |
|------|-------------|
| `-fuzz=pattern` | Run fuzz tests matching pattern |
| `-fuzztime=duration` | Time limit (e.g., 30s, 1m) |
| `-fuzztime=Nx` | Iteration limit (e.g., 1000x) |
| `-parallel=n` | Number of parallel workers |
| `-fuzzminimizetime=duration` | Time to minimize failing input |

## Corpus Directory

### Cấu trúc

```
testdata/
└── fuzz/
    └── FuzzReverse/
        ├── seed1
        ├── seed2
        └── crashers/
            └── crash1
```

### Seed corpus

Đặt trong `testdata/fuzz/<FuzzName>/`:

```go
// testdata/fuzz/FuzzReverse/seed1
// Nội dung: go test fuzz v1
// string("hello")
```

### Format seed file

```
go test fuzz v1
<type>(<value>)
<type>(<value>)
...
```

### Ví dụ

```
go test fuzz v1
string("hello world")
int(42)
[]byte("binary data")
```

## Các kiểu dữ liệu được hỗ trợ

| Type | Ví dụ |
|------|-------|
| `string` | `string("hello")` |
| `[]byte` | `[]byte("data")` |
| `int`, `int8`, `int16`, `int32`, `int64` | `int(42)` |
| `uint`, `uint8`, `uint16`, `uint32`, `uint64` | `uint(42)` |
| `float32`, `float64` | `float64(3.14)` |
| `bool` | `bool(true)` |
| `rune` | `rune('a')` |

## Multiple Arguments

```go
func FuzzMultiple(f *testing.F) {
    f.Add("hello", 42, true)
    f.Add("world", 0, false)
    
    f.Fuzz(func(t *testing.T, s string, n int, b bool) {
        // Test with s, n, b
    })
}
```

Seed file format:

```
go test fuzz v1
string("hello")
int(42)
bool(true)
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GOFUZZCACHE` | Directory for cached fuzzing data |
| `GOTMPDIR` | Temporary directory for fuzz workers |

```bash
GOFUZZCACHE=/path/to/cache go test -fuzz=FuzzReverse
```

## Cấu hình trong go.mod

Không có cấu hình fuzz cụ thể trong go.mod, nhưng version Go ảnh hưởng:

```go
module example.com/myapp

go 1.21 // Fuzzing requires go 1.18+
```

## Crashers và Failing Inputs

### Khi fuzz tìm thấy bug

```
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse (0.00s)
        reverse_test.go:15: Reverse(Reverse("a\xf0")) = "a\xf0\x00\x00"
    
    Failing input written to testdata/fuzz/FuzzReverse/abc123
    To re-run:
    go test -run=FuzzReverse/abc123
```

### Re-run failing input

```bash
go test -run=FuzzReverse/abc123
```

## Integration với CI/CD

### GitHub Actions

```yaml
- name: Run fuzz tests
  run: go test -fuzz=. -fuzztime=30s ./...
```

### Continuous fuzzing

```bash
# Chạy indefinitely
go test -fuzz=FuzzReverse -fuzztime=0

# Với timeout
timeout 1h go test -fuzz=FuzzReverse
```

## Best Practices

1. **Seed với edge cases**: Empty strings, unicode, special chars
2. **Set fuzztime trong CI**: Giới hạn thời gian
3. **Check in crashers**: Commit failing inputs
4. **Use meaningful assertions**: Không chỉ kiểm tra crash
5. **Minimize dependencies**: Fuzz isolated functions

---

*Xem thêm tại [go.dev/doc/tutorial/fuzz](https://go.dev/doc/tutorial/fuzz)*
