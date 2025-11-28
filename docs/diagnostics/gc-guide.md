# Hướng dẫn Garbage Collector

## Giới thiệu

Go sử dụng garbage collector (GC) tự động để quản lý bộ nhớ. Tài liệu này giải thích cách GC hoạt động và cách tối ưu hóa ứng dụng của bạn.

## Cách GC Hoạt động

### Concurrent Mark-and-Sweep

Go GC sử dụng thuật toán concurrent tricolor mark-and-sweep:

1. **Mark Phase**: Đánh dấu các objects có thể truy cập
2. **Sweep Phase**: Thu hồi memory từ objects không được đánh dấu

### Tricolor Marking

- **White**: Chưa được xem xét
- **Grey**: Được đánh dấu nhưng chưa scan children
- **Black**: Đã scan hoàn toàn

## GOGC

### Thiết lập

```bash
GOGC=100 ./myapp  # Mặc định
GOGC=50 ./myapp   # GC thường xuyên hơn
GOGC=200 ./myapp  # GC ít hơn
GOGC=off ./myapp  # Tắt GC (cẩn thận!)
```

### Trong Code

```go
import "runtime/debug"

debug.SetGCPercent(50)
```

### Công thức

```
Trigger GC when: live memory > GOGC% * last_GC_live_memory
```

Với `GOGC=100`:
- Nếu live memory sau GC = 100MB
- GC tiếp theo khi live memory đạt 200MB

## GOMEMLIMIT (Go 1.19+)

### Thiết lập

```bash
GOMEMLIMIT=1GiB ./myapp
```

### Trong Code

```go
debug.SetMemoryLimit(1 << 30) // 1 GiB
```

### Khi nào sử dụng

- Container với memory limit
- Muốn tránh OOM
- Kiểm soát memory usage tốt hơn

## GC Pauses

### Latency Goals

Go GC được thiết kế cho low-latency:
- Target: < 1ms pause times
- Thực tế: Microseconds đến vài ms

### Giảm GC Pause

1. **Giảm allocations**
2. **Sử dụng object pools**
3. **Preallocate slices và maps**
4. **Tránh large heap**

## Giám sát GC

### GC Trace

```bash
GODEBUG=gctrace=1 ./myapp
```

Output:

```
gc 1 @0.001s 2%: 0.013+0.12+0.004 ms clock, 0.052+0/0.10/0.008+0.016 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
```

Giải thích:
- `gc 1`: GC cycle thứ 1
- `@0.001s`: 0.001s từ khi start
- `2%`: 2% CPU cho GC
- `4->4->0 MB`: Heap size before->after->live
- `5 MB goal`: Target heap size
- `4 P`: Số processors

### Memory Stats

```go
var m runtime.MemStats
runtime.ReadMemStats(&m)

fmt.Printf("Alloc = %v MiB\n", m.Alloc/1024/1024)
fmt.Printf("TotalAlloc = %v MiB\n", m.TotalAlloc/1024/1024)
fmt.Printf("Sys = %v MiB\n", m.Sys/1024/1024)
fmt.Printf("NumGC = %v\n", m.NumGC)
fmt.Printf("PauseTotalNs = %v\n", m.PauseTotalNs)
```

## Tối ưu hóa Memory

### 1. Giảm Allocations

```go
// ❌ Allocates mỗi lần gọi
func process() []byte {
    return make([]byte, 1024)
}

// ✅ Sử dụng sync.Pool
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func process() []byte {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    // use buf
    return buf
}
```

### 2. Preallocate

```go
// ❌ Grows dynamically
var result []int
for i := 0; i < 1000; i++ {
    result = append(result, i)
}

// ✅ Preallocate
result := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    result = append(result, i)
}
```

### 3. Avoid Pointers khi có thể

```go
// Pointers require GC scanning
type Bad struct {
    data *[1024]byte
}

// Value type - less GC work
type Good struct {
    data [1024]byte
}
```

### 4. Sử dụng Strings Builder

```go
// ❌ Multiple allocations
s := ""
for i := 0; i < 1000; i++ {
    s += "x"
}

// ✅ Single allocation
var sb strings.Builder
for i := 0; i < 1000; i++ {
    sb.WriteString("x")
}
s := sb.String()
```

## sync.Pool

### Cách sử dụng

```go
var pool = sync.Pool{
    New: func() interface{} {
        return &bytes.Buffer{}
    },
}

func process() {
    buf := pool.Get().(*bytes.Buffer)
    defer pool.Put(buf)
    
    buf.Reset()
    // use buf
}
```

### Lưu ý

- Pool có thể bị clear bất kỳ lúc nào
- Không dùng cho limited resources (connections)
- Tốt cho temporary objects

## Stack vs Heap

### Escape Analysis

```bash
go build -gcflags="-m" main.go
```

### Khi variable escapes to heap

```go
// Escapes - returned
func create() *int {
    x := 42
    return &x  // escapes
}

// Stays on stack
func process() {
    x := 42
    use(&x)  // may stay on stack
}
```

## Profiling Memory

### Heap Profile

```bash
go tool pprof -alloc_objects http://localhost:6060/debug/pprof/heap
go tool pprof -inuse_space http://localhost:6060/debug/pprof/heap
```

### In Tests

```bash
go test -memprofile=mem.prof -bench=.
go tool pprof mem.prof
```

## GC Tuning Recommendations

### High Throughput

```bash
GOGC=200  # Less frequent GC
```

### Low Latency

```bash
GOGC=50   # More frequent, smaller pauses
```

### Memory Constrained

```bash
GOMEMLIMIT=500MiB
GOGC=50
```

### Disable GC (Dangerous!)

```go
debug.SetGCPercent(-1)  // Disable
// Do work
runtime.GC()            // Force GC
debug.SetGCPercent(100) // Re-enable
```

---

*Xem thêm tại [go.dev/doc/gc-guide](https://go.dev/doc/gc-guide)*
