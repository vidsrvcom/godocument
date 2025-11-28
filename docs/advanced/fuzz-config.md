# Cấu hình Fuzz Testing

Fuzzing trong Go có thể được cấu hình thông qua các flags của lệnh `go test` và các biến môi trường. Tài liệu này mô tả các tùy chọn cấu hình có sẵn.

## Fuzz Testing cơ bản

Fuzzing là kỹ thuật testing tự động cung cấp các input ngẫu nhiên cho code để tìm bugs, crashes, và các vấn đề bảo mật.

```go
func FuzzReverse(f *testing.F) {
    // Seed corpus
    f.Add("hello")
    f.Add("world")
    
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        if s != doubleRev {
            t.Errorf("Reverse(%q) = %q, want %q", s, rev, s)
        }
    })
}
```

## Chạy Fuzz Tests

### Chạy fuzzing

```bash
# Fuzz một function cụ thể
go test -fuzz=FuzzReverse

# Fuzz tất cả fuzz tests
go test -fuzz=.

# Fuzz với thời gian giới hạn
go test -fuzz=FuzzReverse -fuzztime=30s

# Fuzz với số lượng iterations
go test -fuzz=FuzzReverse -fuzztime=10000x
```

### Chạy với corpus

```bash
# Chạy corpus mà không fuzzing
go test

# Chỉ chạy seed inputs
go test -run=FuzzReverse
```

## Các Flags Cấu hình

### -fuzz

Xác định fuzz test nào sẽ chạy:

```bash
# Chạy fuzz test cụ thể
go test -fuzz=FuzzReverse

# Pattern matching
go test -fuzz=FuzzParse.*

# Chạy tất cả fuzz tests
go test -fuzz=.
```

### -fuzztime

Kiểm soát thời gian fuzzing:

```bash
# Chạy 30 giây
go test -fuzz=FuzzReverse -fuzztime=30s

# Chạy 5 phút
go test -fuzz=FuzzReverse -fuzztime=5m

# Chạy 10000 iterations
go test -fuzz=FuzzReverse -fuzztime=10000x
```

### -fuzzminimizetime

Thời gian tối thiểu để minimize failing input:

```bash
# Minimize trong 1 phút
go test -fuzz=FuzzReverse -fuzzminimizetime=1m

# Minimize với 100 iterations
go test -fuzz=FuzzReverse -fuzzminimizetime=100x
```

### -parallel

Số lượng fuzz processes chạy đồng thời:

```bash
# Chạy với 4 workers
go test -fuzz=FuzzReverse -parallel=4

# Sử dụng tất cả CPU cores
go test -fuzz=FuzzReverse -parallel=0
```

## Biến Môi trường

### GOCACHE

Vị trí lưu trữ fuzz corpus:

```bash
# Xem cache location
go env GOCACHE

# Set custom cache
export GOCACHE=/path/to/cache
```

### GODEBUG

Debug và diagnostic options:

```bash
# Enable fuzzing debug output
GODEBUG=fuzzdebug=1 go test -fuzz=FuzzReverse

# Show all GODEBUG options
go doc runtime
```

## Corpus Management

### Seed Corpus

Thêm inputs ban đầu trong code:

```go
func FuzzParse(f *testing.F) {
    // Add seed corpus
    f.Add("hello")
    f.Add("world")
    f.Add("123")
    f.Add("")
    
    f.Fuzz(func(t *testing.T, s string) {
        Parse(s)
    })
}
```

### File-based Corpus

Lưu trữ corpus trong thư mục `testdata/fuzz`:

```
mypackage/
├── mypackage.go
├── mypackage_test.go
└── testdata/
    └── fuzz/
        └── FuzzReverse/
            ├── seed1
            ├── seed2
            └── crash-1a2b3c4d5e6f
```

Thêm corpus file:

```bash
# File name format: any valid filename
# File content: encoded input values

echo -n 'go test fuzz v1\nstring("hello")' > testdata/fuzz/FuzzReverse/custom
```

### Corpus Format

```
go test fuzz v1
string("test input")
```

Với nhiều parameters:

```
go test fuzz v1
string("hello")
int(42)
[]byte("world")
```

## Cấu hình Fuzz Function

### Supported Types

Fuzz function hỗ trợ các types:

- `string`
- `[]byte`
- `int`, `int8`, `int16`, `int32`, `int64`
- `uint`, `uint8`, `uint16`, `uint32`, `uint64`
- `float32`, `float64`
- `bool`
- `rune`

```go
func FuzzMultipleTypes(f *testing.F) {
    f.Add("hello", 42, true, []byte("world"))
    
    f.Fuzz(func(t *testing.T, s string, n int, b bool, data []byte) {
        // Fuzz với multiple parameters
        Process(s, n, b, data)
    })
}
```

### Custom Corpus Loading

```go
func FuzzWithCustomCorpus(f *testing.F) {
    // Load corpus từ file
    corpus, err := loadCorpus("testdata/custom_corpus.txt")
    if err != nil {
        f.Fatal(err)
    }
    
    for _, item := range corpus {
        f.Add(item)
    }
    
    f.Fuzz(func(t *testing.T, s string) {
        Parse(s)
    })
}
```

## Minimization

Go tự động minimize failing inputs:

```go
// Original failing input
input := "aaaaaaaaaaaaaaaaaaaaa" // causes crash

// Minimized input
input := "aa" // shortest input that causes same crash
```

Cấu hình minimization:

```bash
# Tăng thời gian minimize
go test -fuzz=FuzzReverse -fuzzminimizetime=5m

# Skip minimization (để debug)
go test -fuzz=FuzzReverse -fuzzminimizetime=0
```

## Coverage-guided Fuzzing

Go fuzzer sử dụng coverage để guide input generation:

```bash
# Xem coverage trong khi fuzzing
go test -fuzz=FuzzReverse -coverprofile=coverage.out

# Analyze coverage
go tool cover -html=coverage.out
```

## Strategies và Best Practices

### 1. Seed Corpus Quality

```go
func FuzzJSONParse(f *testing.F) {
    // Good: diverse, edge cases
    f.Add(`{"key": "value"}`)
    f.Add(`{"nested": {"key": "value"}}`)
    f.Add(`[]`)
    f.Add(`null`)
    f.Add(`""`)
    f.Add(`123`)
    
    // Bad: too similar
    f.Add(`{"a": "1"}`)
    f.Add(`{"b": "2"}`)
    f.Add(`{"c": "3"}`)
}
```

### 2. Resource Limits

```go
func FuzzParse(f *testing.F) {
    f.Fuzz(func(t *testing.T, data []byte) {
        // Limit input size
        if len(data) > 10000 {
            return
        }
        
        // Set timeout
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()
        
        ParseWithContext(ctx, data)
    })
}
```

### 3. Deterministic Testing

```go
func FuzzHash(f *testing.F) {
    f.Fuzz(func(t *testing.T, data []byte) {
        // Ensure deterministic behavior
        h1 := Hash(data)
        h2 := Hash(data)
        
        if h1 != h2 {
            t.Errorf("Hash not deterministic")
        }
    })
}
```

## CI/CD Integration

### Continuous Fuzzing

```bash
#!/bin/bash
# fuzz.sh - Run in CI

# Fuzz mỗi test trong 10 giây
for test in $(go test -list=Fuzz); do
    echo "Fuzzing $test..."
    go test -fuzz=$test -fuzztime=10s || exit 1
done
```

### GitHub Actions

```yaml
name: Fuzz Testing

on: [push, pull_request]

jobs:
  fuzz:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Run Fuzz Tests
        run: |
          go test -fuzz=. -fuzztime=30s
```

## Debugging Fuzz Failures

### Reproduce Failure

```bash
# Corpus file được lưu khi tìm thấy failure
# testdata/fuzz/FuzzReverse/crash-abc123

# Reproduce bằng cách chạy test thông thường
go test -run=FuzzReverse
```

### Debug với Delve

```bash
# Run với debugger
dlv test -- -test.fuzz=FuzzReverse -test.fuzztime=10s
```

## Performance Tuning

```bash
# Tăng workers cho fuzzing nhanh hơn
go test -fuzz=FuzzReverse -parallel=8

# Giảm memory usage
GOMEMLIMIT=1GiB go test -fuzz=FuzzReverse

# Profile fuzzer
go test -fuzz=FuzzReverse -cpuprofile=cpu.prof -memprofile=mem.prof
```

## Ví dụ Configuration File

Tạo script để standardize fuzzing:

```bash
#!/bin/bash
# run_fuzz.sh

FUZZ_TIME="${FUZZ_TIME:-30s}"
PARALLEL="${PARALLEL:-4}"

go test \
    -fuzz="${1:-.}" \
    -fuzztime="$FUZZ_TIME" \
    -parallel="$PARALLEL" \
    -timeout=1h \
    "$@"
```

Usage:

```bash
# Fuzz với defaults
./run_fuzz.sh FuzzReverse

# Custom configuration
FUZZ_TIME=5m PARALLEL=8 ./run_fuzz.sh FuzzParse
```

## Thực hành tốt nhất

1. **Start với good seed corpus**: Bao gồm edge cases
2. **Set reasonable timeouts**: Tránh inputs quá lớn
3. **Run fuzzing regularly**: Trong CI/CD pipeline
4. **Keep corpus in version control**: Track discovered inputs
5. **Minimize failing inputs**: Dễ debug hơn
6. **Monitor coverage**: Đảm bảo fuzzer explore code paths mới
7. **Limit resource usage**: Memory, CPU, disk

## Xem thêm

- [Bắt đầu với fuzzing](../../tutorials/fuzzing.md)
- [Fuzzing](fuzzing.md)
- [Testdata Format](testdata-format.md)
- Go Fuzzing: https://go.dev/doc/fuzz/
