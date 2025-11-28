# Hướng dẫn Profile-Guided Optimization (PGO)

## Giới thiệu

Profile-Guided Optimization (PGO) là kỹ thuật tối ưu hóa sử dụng production profile data để cải thiện hiệu suất. Go 1.20 giới thiệu PGO và Go 1.21 mặc định bật tính năng này.

## Cách hoạt động

1. **Collect**: Thu thập CPU profile từ production
2. **Build**: Build với profile data
3. **Deploy**: Deploy binary được tối ưu

## Quick Start

### 1. Thu thập Profile

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go http.ListenAndServe(":6060", nil)
    // Your application
}
```

Thu thập:

```bash
curl -o cpu.pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

### 2. Build với Profile

```bash
# Đặt profile ở thư mục gốc
mv cpu.pprof default.pgo

# Build
go build -pgo=auto .
# hoặc
go build -pgo=./default.pgo .
```

### 3. Verify

```bash
go version -m ./myapp | grep pgo
# Build info: ...
# -pgo=/path/to/default.pgo
```

## Thu thập Profile

### Từ HTTP Server

```bash
# 30 giây profile
curl -o cpu.pprof "http://localhost:6060/debug/pprof/profile?seconds=30"

# 60 giây cho comprehensive profile
curl -o cpu.pprof "http://localhost:6060/debug/pprof/profile?seconds=60"
```

### Từ Test

```bash
go test -cpuprofile=cpu.pprof -bench=.
```

### Từ Running Program

```go
import (
    "os"
    "runtime/pprof"
)

f, _ := os.Create("cpu.pprof")
pprof.StartCPUProfile(f)

// Run workload

pprof.StopCPUProfile()
f.Close()
```

## Naming Convention

### default.pgo

Khi file `default.pgo` tồn tại trong thư mục main package:

```bash
./
├── main.go
└── default.pgo  # Tự động được sử dụng
```

### Custom name

```bash
go build -pgo=./my-profile.pprof .
```

## Merge Profiles

Kết hợp nhiều profiles:

```bash
go tool pprof -proto cpu1.pprof cpu2.pprof cpu3.pprof > merged.pgo
```

## PGO Improvements

### Các tối ưu

| Optimization | Mô tả |
|--------------|-------|
| Inlining | Inline hot functions |
| Devirtualization | Direct calls thay vì interface calls |
| Block layout | Sắp xếp hot blocks |

### Typical Improvement

- 2-7% CPU reduction
- Tùy thuộc vào workload

## Best Practices

### 1. Representative Profile

```bash
# Profile trong production
# hoặc với realistic load test
```

### 2. Refresh Periodically

```bash
# Update profile khi code thay đổi đáng kể
```

### 3. Merge Multiple Profiles

```bash
# Profiles từ nhiều servers/times
go tool pprof -proto profile1.pprof profile2.pprof > default.pgo
```

### 4. Verify Improvements

```bash
# Build without PGO
go build -pgo=off -o app-nopgo .

# Build with PGO
go build -pgo=auto -o app-pgo .

# Compare benchmarks
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Build with PGO
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      
      - name: Download production profile
        run: |
          # Download from storage
          aws s3 cp s3://bucket/default.pgo ./default.pgo
      
      - name: Build with PGO
        run: go build -pgo=auto .
```

### Profile Collection Workflow

```yaml
# Separate workflow để collect profiles
name: Collect Profiles
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly

jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: Collect from production
        run: |
          curl -o cpu.pprof "http://prod-server:6060/debug/pprof/profile?seconds=60"
          aws s3 cp cpu.pprof s3://bucket/default.pgo
```

## Troubleshooting

### Profile không được sử dụng

```bash
# Kiểm tra build info
go version -m ./myapp

# Nếu không có "-pgo=..."
# - Đảm bảo profile file tồn tại
# - Kiểm tra Go version >= 1.20
```

### Profile quá cũ

- Code đã thay đổi nhiều
- Workload patterns khác
- Thu thập profile mới

### No improvement

- Workload không CPU-bound
- Profile không representative
- Bottleneck ở IO/Network

## Monitoring PGO Effectiveness

### Benchmark Comparison

```go
func BenchmarkWithProfile(b *testing.B) {
    for i := 0; i < b.N; i++ {
        HotFunction()
    }
}
```

```bash
# Without PGO
go test -pgo=off -bench=. > baseline.txt

# With PGO
go test -pgo=auto -bench=. > pgo.txt

# Compare
benchstat baseline.txt pgo.txt
```

## Advanced Topics

### Partial Profile

PGO hoạt động ngay cả với partial profile coverage.

### Multi-binary Projects

Mỗi binary cần profile riêng:

```bash
./cmd/server/default.pgo
./cmd/worker/default.pgo
```

### Debug Build

```bash
go build -pgo=auto -gcflags="all=-N -l" .
```

---

*Xem thêm tại [go.dev/doc/pgo](https://go.dev/doc/pgo)*
