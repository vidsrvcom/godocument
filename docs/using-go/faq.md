# Câu hỏi Thường gặp (FAQ)

## Nguồn gốc

### Go đến từ đâu?

Go được thiết kế tại Google vào năm 2007 để cải thiện năng suất lập trình trong kỷ nguyên multicore, máy nối mạng và codebase lớn. Các nhà thiết kế muốn giải quyết những lời chỉ trích về các ngôn ngữ khác đang được sử dụng tại Google.

### Ai là người tạo ra Go?

Go được thiết kế bởi Robert Griesemer, Rob Pike và Ken Thompson tại Google.

### Biểu tượng Gopher đến từ đâu?

Gopher được thiết kế bởi Renee French. Nó dựa trên một thiết kế mà cô đã tạo cho WFMU, một đài phát thanh ở New Jersey.

### Ngôn ngữ được gọi là Go hay Golang?

Ngôn ngữ được gọi là Go. "Golang" xuất hiện vì tên miền là golang.org (vì go.com không khả dụng).

## Sử dụng

### Go có hỗ trợ lập trình hướng đối tượng không?

Có và không. Go có types và methods, nhưng không có class hierarchy. Interface trong Go cung cấp một cách khác để sử dụng, và Go support cho đa hình.

### Tại sao Go không có exceptions?

Go sử dụng multiple return values để báo lỗi. Cùng với các quy ước xử lý lỗi của Go, điều này giúp mã nguồn rõ ràng và dễ hiểu hơn.

### Tại sao Go không có generics? (Đã có từ Go 1.18)

Go 1.18 đã thêm support cho generics. Trước đó, thiếu generics là một quyết định có chủ đích để giữ ngôn ngữ đơn giản.

```go
func Print[T any](s []T) {
    for _, v := range s {
        fmt.Println(v)
    }
}
```

### Tại sao Go không có assertions?

Go không cung cấp assertions. Thay vào đó, người dùng được khuyến khích viết xử lý lỗi rõ ràng.

### Tại sao không có constructors?

Go không có constructors. Thay vào đó, sử dụng factory functions:

```go
func NewBook(title string) *Book {
    return &Book{Title: title}
}
```

## Kiểu (Types)

### Go có phải là ngôn ngữ object-oriented không?

Go hỗ trợ một số tính năng OOP nhưng không phải là ngôn ngữ OOP truyền thống:
- Có types và methods
- Không có class hierarchy
- Interfaces được satisfied implicitly

### Tại sao type T không thể implement interface khi nó có methods với pointer receiver?

```go
type T struct{}
func (t *T) Method() {}

var _ Interface = T{}  // Không compile!
var _ Interface = &T{} // OK
```

Giá trị của type T không có địa chỉ có thể lấy được trong mọi trường hợp.

### Tại sao maps, slices và channels là references?

Để tránh việc sao chép dữ liệu lớn không cần thiết và cho phép chia sẻ dữ liệu giữa các functions.

### Slice khác gì với array?

Array có kích thước cố định, là giá trị (value type).
Slice là một view vào array, có thể thay đổi kích thước.

```go
var a [5]int    // array
var s []int     // slice
s = a[1:4]      // slice từ array
```

## Đồng thời (Concurrency)

### Goroutine khác gì với thread?

Goroutines được multiplexed lên một số nhỏ OS threads. Chúng rất nhẹ (~2KB stack) và được quản lý bởi Go runtime, không phải OS.

### Tại sao map operations không atomic?

Để giữ cho map operations nhanh cho các trường hợp sử dụng phổ biến. Sử dụng `sync.Map` hoặc mutex cho concurrent access.

### Khi nào nên sử dụng channel vs mutex?

- **Channel**: Khi bạn muốn truyền ownership của data hoặc phân phối units of work.
- **Mutex**: Khi bạn cần bảo vệ internal state của một struct.

```go
// Channel - truyền data
ch := make(chan Result)
go func() { ch <- doWork() }()
result := <-ch

// Mutex - bảo vệ state
var mu sync.Mutex
mu.Lock()
counter++
mu.Unlock()
```

## Packages

### Tại sao không có dependency versioning trong early Go?

Go Modules (từ Go 1.11) giải quyết vấn đề này. Trước đó, Go sử dụng GOPATH.

### Làm sao để tổ chức code trong Go?

- Mỗi project là một module
- Mỗi directory là một package
- Package name = directory name
- Exported identifiers bắt đầu bằng chữ hoa

## Hiệu suất (Performance)

### Go nhanh như thế nào so với C/C++?

Go thường chậm hơn C/C++ một chút do garbage collection và bounds checking. Tuy nhiên, Go thường đủ nhanh và dễ viết, dễ maintain hơn.

### Tại sao garbage collection thay vì manual memory management?

GC giúp tránh memory leaks, use-after-free bugs, và làm cho code dễ viết hơn. Go's GC được tối ưu hóa cho low latency.

### Có thể tắt garbage collection không?

Không thể tắt hoàn toàn, nhưng có thể điều chỉnh với `GOGC` environment variable.

## Công cụ (Tools)

### Gofmt style là gì?

`gofmt` tự động format code Go. Một số quy tắc:
- Tabs thay vì spaces
- Không có trailing whitespace
- Braces trên cùng dòng

### Tại sao không có IDE chính thức?

Go được thiết kế để dễ hỗ trợ bởi nhiều editors. `gopls` (Go language server) cung cấp IDE features cho bất kỳ editor nào.

## Thay đổi ngôn ngữ

### Tại sao Go thay đổi ít?

Go coi trọng backward compatibility. Thay đổi ngôn ngữ được xem xét cẩn thận để không phá vỡ code hiện có.

### Go 2 là gì?

"Go 2" là thuật ngữ cho các thay đổi lớn hơn. Nhiều "Go 2" features đã được thêm vào Go 1.x (như generics trong Go 1.18).

---

*Xem FAQ đầy đủ tại [go.dev/doc/faq](https://go.dev/doc/faq)*
