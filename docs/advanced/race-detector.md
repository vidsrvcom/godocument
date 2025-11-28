# Race Detector

## Giới thiệu

Go race detector phát hiện data races - khi hai goroutines truy cập cùng biến đồng thời và ít nhất một là write.

## Sử dụng

### Với go run

```bash
go run -race main.go
```

### Với go test

```bash
go test -race ./...
```

### Với go build

```bash
go build -race -o myapp
```

## Ví dụ Data Race

```go
package main

var counter int

func main() {
    go func() { counter++ }()
    go func() { counter++ }()
    time.Sleep(time.Second)
    println(counter)
}
```

Output khi chạy với `-race`:

```
WARNING: DATA RACE
Write at 0x... by goroutine 7:
  main.main.func1()
      main.go:7 +0x...

Previous write at 0x... by goroutine 6:
  main.main.func2()
      main.go:8 +0x...
```

## Sửa Data Race

### Sử dụng Mutex

```go
var (
    counter int
    mu      sync.Mutex
)

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}
```

### Sử dụng Atomic

```go
var counter atomic.Int64

func increment() {
    counter.Add(1)
}
```

### Sử dụng Channel

```go
func main() {
    ch := make(chan int)
    go func() { ch <- 1 }()
    go func() { ch <- 1 }()
    counter := <-ch + <-ch
    println(counter)
}
```

## Best Practices

1. **Chạy race detector trong CI/CD**
2. **Test với realistic load**
3. **Sửa TẤT CẢ races - không có "benign race"**

---

*Xem thêm tại [go.dev/doc/articles/race_detector](https://go.dev/doc/articles/race_detector)*
