# Effective Go

## Giới thiệu

Effective Go là tài liệu hướng dẫn cách viết mã Go rõ ràng, theo phong cách chuẩn. Đây là tài liệu phải đọc cho bất kỳ lập trình viên Go mới nào.

## Định dạng

Go sử dụng `gofmt` để định dạng mã nguồn một cách tự động. Điều này giúp:
- Đảm bảo tính nhất quán trong toàn bộ codebase
- Giảm tranh cãi về phong cách code
- Dễ dàng đọc và bảo trì mã

```bash
gofmt -w yourfile.go
```

## Comment

Go hỗ trợ hai loại comment:
- `// comment dòng đơn`
- `/* comment nhiều dòng */`

Mỗi package nên có một package comment đặt trước khai báo package.

```go
// Package fmt implements formatted I/O with functions analogous
// to C's printf and scanf.
package fmt
```

## Đặt tên

### Package

- Tên package nên ngắn gọn, rõ ràng
- Sử dụng chữ thường, không dùng underscore hoặc mixedCaps
- Ví dụ: `bufio`, `http`, `json`

### Getter/Setter

Go không tự động cung cấp getter/setter. Nếu bạn có một trường `owner`, getter nên là `Owner()` (không phải `GetOwner()`), và setter là `SetOwner()`.

### Interface

Interface một phương thức thường đặt tên với hậu tố `-er`:
- `Reader`
- `Writer`
- `Formatter`

## Điều khiển luồng

### If

```go
if x > 0 {
    return y
}
```

### For

Go chỉ có một loại vòng lặp: `for`

```go
// Vòng lặp cơ bản
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While loop
for condition {
    // ...
}

// Vô hạn
for {
    // ...
}
```

### Switch

```go
switch c {
case ' ', '\t', '\n':
    return true
}
```

## Functions

### Nhiều giá trị trả về

```go
func nextInt(b []byte, i int) (int, int) {
    // ...
    return x, i
}
```

### Defer

Defer cho phép hoãn thực thi một hàm cho đến khi hàm bao quanh trả về.

```go
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()
    // ...
}
```

## Dữ liệu

### Cấp phát với new

`new(T)` cấp phát bộ nhớ zero cho kiểu T và trả về con trỏ.

```go
p := new(SyncedBuffer)
```

### Cấp phát với make

`make(T, args)` chỉ dùng cho slice, map và channel.

```go
v := make([]int, 100)
m := make(map[string]int)
```

### Slice

Slice là một wrapper xung quanh array, cung cấp interface linh hoạt hơn.

```go
slice := []int{1, 2, 3}
slice = append(slice, 4, 5)
```

### Map

```go
m := make(map[string]int)
m["key"] = 42
value, ok := m["key"]
```

## Đồng thời

### Goroutine

```go
go func() {
    // thực thi đồng thời
}()
```

### Channel

```go
ch := make(chan int)
ch <- v    // gửi v vào channel
v := <-ch  // nhận từ channel
```

## Xử lý lỗi

Go khuyến khích kiểm tra lỗi rõ ràng:

```go
f, err := os.Open(name)
if err != nil {
    return err
}
// sử dụng f
```

## Panic và Recover

- `panic` dừng chương trình
- `recover` bắt panic trong defer

```go
defer func() {
    if r := recover(); r != nil {
        log.Println("Recovered:", r)
    }
}()
```

---

*Xem tài liệu gốc tại [go.dev/doc/effective_go](https://go.dev/doc/effective_go)*
