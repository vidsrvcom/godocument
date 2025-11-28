# Tổng quan về Công cụ Chẩn đoán

## Giới thiệu

Go cung cấp một bộ công cụ chẩn đoán phong phú để giúp developers tìm và sửa các vấn đề về hiệu suất, memory, và concurrency.

## Các Công cụ Chính

### 1. Profiling

#### CPU Profiling

```go
import "runtime/pprof"

f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Your code here
```

Hoặc với HTTP:

```go
import _ "net/http/pprof"

go http.ListenAndServe(":6060", nil)
```

Xem profile:

```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

#### Memory Profiling

```go
import "runtime/pprof"

f, _ := os.Create("mem.prof")
pprof.WriteHeapProfile(f)
```

Xem:

```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```

### 2. Tracing

```go
import "runtime/trace"

f, _ := os.Create("trace.out")
trace.Start(f)
defer trace.Stop()

// Your code here
```

Xem trace:

```bash
go tool trace trace.out
```

### 3. Race Detector

Phát hiện data races:

```bash
go test -race ./...
go build -race
go run -race main.go
```

### 4. go vet

Static analysis:

```bash
go vet ./...
```

## Pprof

### Các endpoints

| Endpoint | Mô tả |
|----------|-------|
| /debug/pprof/ | Index page |
| /debug/pprof/heap | Heap profile |
| /debug/pprof/goroutine | Goroutine stacks |
| /debug/pprof/profile | CPU profile |
| /debug/pprof/block | Block profile |
| /debug/pprof/mutex | Mutex profile |
| /debug/pprof/trace | Execution trace |

### Sử dụng go tool pprof

```bash
# Interactive mode
go tool pprof http://localhost:6060/debug/pprof/heap

# Các lệnh trong interactive mode
(pprof) top           # Top consumers
(pprof) list funcName # Source code
(pprof) web           # Open in browser
```

### Output formats

```bash
# SVG graph
go tool pprof -svg cpu.prof > cpu.svg

# Text report
go tool pprof -text cpu.prof

# Flamegraph (cần graphviz)
go tool pprof -http=:8080 cpu.prof
```

## Memory Analysis

### Heap Profile

```bash
go tool pprof -alloc_objects http://localhost:6060/debug/pprof/heap
go tool pprof -inuse_space http://localhost:6060/debug/pprof/heap
```

### Memory Stats

```go
import "runtime"

var m runtime.MemStats
runtime.ReadMemStats(&m)

fmt.Printf("Alloc = %v MiB", m.Alloc / 1024 / 1024)
fmt.Printf("TotalAlloc = %v MiB", m.TotalAlloc / 1024 / 1024)
fmt.Printf("Sys = %v MiB", m.Sys / 1024 / 1024)
fmt.Printf("NumGC = %v", m.NumGC)
```

### GC Tracing

```bash
GODEBUG=gctrace=1 ./myapp
```

Output:

```
gc 1 @0.001s 0%: 0.004+0.12+0.003 ms clock, 0.016+0/0.10/0.008+0.014 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
```

## Goroutine Analysis

### Goroutine Profile

```bash
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

### Goroutine Dump

```go
import "runtime/debug"

debug.PrintStack()
```

### Scheduler Tracing

```bash
GODEBUG=schedtrace=1000 ./myapp
```

## Block và Mutex Profiling

### Bật Block Profile

```go
runtime.SetBlockProfileRate(1)
```

### Bật Mutex Profile

```go
runtime.SetMutexProfileFraction(1)
```

## Execution Tracing

```bash
curl -o trace.out http://localhost:6060/debug/pprof/trace?seconds=5
go tool trace trace.out
```

Trace UI hiển thị:
- Goroutine execution
- System calls
- GC events
- Network blocking

## Benchmark Testing

```go
func BenchmarkFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Function()
    }
}
```

Chạy:

```bash
go test -bench=. -benchmem
go test -bench=. -cpuprofile=cpu.prof
go test -bench=. -memprofile=mem.prof
```

## Third-party Tools

### goleak

Detect goroutine leaks:

```go
import "go.uber.org/goleak"

func TestMain(m *testing.M) {
    goleak.VerifyTestMain(m)
}
```

### go-torch (deprecated)

Sử dụng `go tool pprof -http=:8080` thay thế.

### delve

Interactive debugger:

```bash
dlv debug main.go
```

## Best Practices

1. **Profile in production-like environment**
2. **Compare before/after optimization**
3. **Focus on hot paths**
4. **Use benchmarks to validate improvements**
5. **Monitor key metrics continuously**

---

*Xem thêm tại [go.dev/doc/diagnostics](https://go.dev/doc/diagnostics)*
