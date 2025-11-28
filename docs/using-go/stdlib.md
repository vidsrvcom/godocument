# Tài liệu Thư viện Chuẩn Go

## Giới thiệu

Thư viện chuẩn Go cung cấp một bộ sưu tập phong phú các package để thực hiện nhiều tác vụ lập trình phổ biến.

## Các Package Phổ biến

### fmt - Định dạng I/O

Package `fmt` cung cấp các hàm định dạng I/O tương tự như printf và scanf của C.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
    fmt.Printf("Value: %d\n", 42)
    
    var name string
    fmt.Scan(&name)
}
```

### io - I/O Primitives

Package `io` cung cấp các interface cơ bản cho I/O.

```go
import "io"

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

### os - Chức năng Hệ điều hành

```go
import "os"

// Đọc file
data, err := os.ReadFile("filename.txt")

// Ghi file
err := os.WriteFile("filename.txt", data, 0644)

// Biến môi trường
value := os.Getenv("HOME")
```

### strings - Xử lý chuỗi

```go
import "strings"

s := "Hello, World"
strings.Contains(s, "World")  // true
strings.ToUpper(s)            // "HELLO, WORLD"
strings.Split(s, ", ")        // ["Hello", "World"]
strings.Join([]string{"a", "b"}, "-")  // "a-b"
```

### strconv - Chuyển đổi chuỗi

```go
import "strconv"

i, err := strconv.Atoi("42")       // string to int
s := strconv.Itoa(42)              // int to string
f, err := strconv.ParseFloat("3.14", 64)
```

### time - Thời gian

```go
import "time"

now := time.Now()
fmt.Println(now.Format("2006-01-02 15:04:05"))

duration := 5 * time.Second
time.Sleep(duration)
```

### net/http - HTTP Client/Server

```go
import "net/http"

// HTTP Server
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
})
http.ListenAndServe(":8080", nil)

// HTTP Client
resp, err := http.Get("https://example.com")
```

### encoding/json - JSON

```go
import "encoding/json"

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

// Marshal
data, err := json.Marshal(person)

// Unmarshal
var p Person
err := json.Unmarshal(data, &p)
```

### log - Logging

```go
import "log"

log.Println("Info message")
log.Printf("Value: %d", 42)
log.Fatal("Fatal error")  // gọi os.Exit(1)
```

### errors - Xử lý lỗi

```go
import "errors"

err := errors.New("something went wrong")

// Go 1.13+
if errors.Is(err, os.ErrNotExist) {
    // ...
}
```

### context - Context

```go
import "context"

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case <-ctx.Done():
    fmt.Println("Timeout!")
}
```

### sync - Đồng bộ hóa

```go
import "sync"

var mu sync.Mutex
mu.Lock()
// critical section
mu.Unlock()

var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    // work
}()
wg.Wait()
```

### regexp - Biểu thức chính quy

```go
import "regexp"

re := regexp.MustCompile(`\d+`)
matches := re.FindAllString("abc123def456", -1)  // ["123", "456"]
```

### path/filepath - Đường dẫn file

```go
import "path/filepath"

fullPath := filepath.Join("dir", "subdir", "file.txt")
dir := filepath.Dir(fullPath)
ext := filepath.Ext("file.txt")  // ".txt"
```

### testing - Unit Testing

```go
import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}
```

## Các Package Khác

- `bufio`: Buffered I/O
- `bytes`: Thao tác byte slice
- `crypto/*`: Cryptography
- `database/sql`: SQL database
- `html/template`: HTML templates
- `math`: Hàm toán học
- `net`: Networking
- `reflect`: Runtime reflection
- `sort`: Sắp xếp
- `unicode`: Unicode utilities

---

*Xem tài liệu đầy đủ tại [pkg.go.dev/std](https://pkg.go.dev/std)*
