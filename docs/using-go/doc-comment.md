# Tham chiếu Comment Tài liệu

## Giới thiệu

Go sử dụng comments để tạo documentation tự động thông qua công cụ `godoc`. Viết comments tốt giúp người khác hiểu và sử dụng code của bạn.

## Package Comments

Mỗi package nên có một package comment đặt ngay trước khai báo `package`.

```go
// Package fmt implements formatted I/O with functions analogous
// to C's printf and scanf. The format 'verbs' are derived from C's but
// are simpler.
package fmt
```

Đối với package lớn, có thể sử dụng file `doc.go`:

```go
// Package sort provides primitives for sorting slices and user-defined
// collections.
//
// # Example
//
// This example sorts a slice of strings:
//
//	sort.Strings([]string{"c", "a", "b"})
//
// After sorting, the slice will be ["a", "b", "c"].
package sort
```

## Function Comments

Comment cho exported functions bắt đầu với tên function:

```go
// Println formats using the default formats for its operands and writes to standard output.
// Spaces are always added between operands and a newline is appended.
// It returns the number of bytes written and any write error encountered.
func Println(a ...any) (n int, err error) {
    // ...
}
```

## Type Comments

Comment cho types bắt đầu với tên type:

```go
// A Reader reads and decodes JSON values from an input stream.
type Reader struct {
    r io.Reader
    // ...
}
```

## Constant và Variable Comments

```go
// MaxInt32 is the maximum value for a 32-bit signed integer.
const MaxInt32 = 1<<31 - 1

// DefaultClient is the default HTTP client.
var DefaultClient = &Client{Timeout: 30 * time.Second}
```

## Method Comments

```go
// Read reads up to len(p) bytes into p. It returns the number of bytes
// read (0 <= n <= len(p)) and any error encountered.
func (r *Reader) Read(p []byte) (n int, err error) {
    // ...
}
```

## Định dạng Comment

### Paragraphs

Đoạn văn được phân tách bởi dòng trống:

```go
// Foo does something important.
//
// This is the second paragraph with more details about
// what Foo does and how to use it.
func Foo() {
    // ...
}
```

### Headers (Go 1.19+)

Sử dụng `#` để tạo headers:

```go
// Package math provides basic constants and mathematical functions.
//
// # Trigonometric Functions
//
// This section describes the trigonometric functions.
//
// # Exponential Functions
//
// This section describes the exponential functions.
package math
```

### Code Blocks

Thụt lề để tạo code blocks:

```go
// Example usage:
//
//	result := Calculate(10, 20)
//	fmt.Println(result)
func Calculate(a, b int) int {
    return a + b
}
```

### Lists

Tạo lists với dấu `-` hoặc `*`:

```go
// Options for the function:
//   - Option1: Does something
//   - Option2: Does something else
//   - Option3: Another option
func DoSomething(options ...Option) {
    // ...
}
```

### Links

Liên kết URL tự động được nhận diện:

```go
// For more information, see https://example.com/docs
func Example() {
    // ...
}
```

### Doc Links (Go 1.19+)

Liên kết đến symbols khác:

```go
// Similar to [fmt.Println] but writes to stderr.
// See also [Writer.Write] for low-level writing.
func Eprintln(a ...any) (n int, err error) {
    // ...
}
```

## Examples

### Example Functions

Đặt trong file `*_test.go`:

```go
func ExamplePrintln() {
    fmt.Println("Hello, World!")
    // Output: Hello, World!
}

func ExampleReader_Read() {
    r := strings.NewReader("Hello")
    b := make([]byte, 5)
    r.Read(b)
    fmt.Println(string(b))
    // Output: Hello
}
```

### Example với method cụ thể

```go
// Example_suffix shows a specific use case.
func Example_suffix() {
    // ...
}

// ExampleType_Method shows how to use Method on Type.
func ExampleType_Method() {
    // ...
}
```

## Deprecated

Đánh dấu deprecated:

```go
// Deprecated: Use NewFunction instead.
func OldFunction() {
    // ...
}
```

## Best Practices

### 1. Bắt đầu với tên được document

```go
// Good: Printf formats according to a format specifier...
// Bad: This function formats...
```

### 2. Viết câu hoàn chỉnh

```go
// Good: Printf formats according to a format specifier and returns the number of bytes written.
// Bad: formats according to format specifier
```

### 3. Mô tả tham số và return values

```go
// Split slices s into all substrings separated by sep and returns a slice
// of the substrings between those separators.
// If sep is empty, Split splits after each UTF-8 sequence.
func Split(s, sep string) []string {
    // ...
}
```

### 4. Document các edge cases

```go
// Read reads up to len(p) bytes. It returns 0, io.EOF at end of file.
// If len(p) == 0, Read returns 0, nil.
```

## Xem Documentation

### Sử dụng go doc

```bash
go doc fmt
go doc fmt.Println
go doc -all fmt
```

### Sử dụng godoc server

```bash
go install golang.org/x/tools/cmd/godoc@latest
godoc -http=:6060
```

Sau đó mở `http://localhost:6060` trong browser.

---

*Xem tài liệu đầy đủ tại [go.dev/doc/comment](https://go.dev/doc/comment)*
