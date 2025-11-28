# Đặc tả Ngôn ngữ Go

## Giới thiệu

Đây là tài liệu tham chiếu đặc tả chính thức cho ngôn ngữ lập trình Go. Đặc tả này định nghĩa cú pháp, ngữ nghĩa và các quy tắc của ngôn ngữ.

## Ký hiệu

Đặc tả sử dụng Extended Backus-Naur Form (EBNF) để mô tả cú pháp.

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

## Mã nguồn

Mã nguồn Go là Unicode text được mã hóa UTF-8.

## Từ khóa

Go có 25 từ khóa:

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## Toán tử và Dấu phân cách

```
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

## Kiểu dữ liệu

### Kiểu Boolean

```go
var b bool = true
```

### Kiểu số

- `int`, `int8`, `int16`, `int32`, `int64`
- `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
- `float32`, `float64`
- `complex64`, `complex128`

### Kiểu chuỗi

```go
var s string = "Hello, 世界"
```

### Kiểu Array

```go
var a [5]int
```

### Kiểu Slice

```go
var s []int = []int{1, 2, 3}
```

### Kiểu Map

```go
var m map[string]int = make(map[string]int)
```

### Kiểu Struct

```go
type Person struct {
    Name string
    Age  int
}
```

### Kiểu Interface

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

### Kiểu Channel

```go
var ch chan int
```

## Khai báo và Phạm vi

### Khai báo biến

```go
var x int
var x, y int = 1, 2
x := 1
```

### Khai báo hằng

```go
const Pi = 3.14159
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

### Khai báo kiểu

```go
type MyInt int
type Point struct {
    X, Y int
}
```

### Khai báo hàm

```go
func add(a, b int) int {
    return a + b
}
```

## Biểu thức

### Toán tử

- Toán tử số học: `+`, `-`, `*`, `/`, `%`
- Toán tử so sánh: `==`, `!=`, `<`, `<=`, `>`, `>=`
- Toán tử logic: `&&`, `||`, `!`

### Gọi hàm

```go
result := add(1, 2)
```

### Chuyển đổi kiểu

```go
i := int(f)
```

## Câu lệnh

### Câu lệnh If

```go
if x > 0 {
    return x
} else {
    return -x
}
```

### Câu lệnh Switch

```go
switch tag {
case 0, 1, 2:
    // ...
default:
    // ...
}
```

### Câu lệnh For

```go
for i := 0; i < 10; i++ {
    // ...
}
```

### Câu lệnh Go

```go
go func() {
    // thực thi đồng thời
}()
```

### Câu lệnh Select

```go
select {
case msg := <-ch:
    fmt.Println(msg)
default:
    // không chờ
}
```

## Package

Mỗi file Go thuộc về một package:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

## Chương trình

Một chương trình Go hoàn chỉnh được tạo bằng cách liên kết các package với nhau. Hàm `main` trong package `main` là điểm vào của chương trình.

---

*Xem đặc tả đầy đủ tại [go.dev/ref/spec](https://go.dev/ref/spec)*
