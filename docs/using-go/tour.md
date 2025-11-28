# Một Tour về Go

## Giới thiệu

Tour Go là một giới thiệu tương tác về ngôn ngữ lập trình Go. Nó được chia thành 4 phần chính và bao gồm các bài tập để bạn thực hành.

## Cài đặt Tour cục bộ

```bash
go install golang.org/x/website/tour@latest
```

Sau đó chạy:

```bash
tour
```

Binary tour sẽ được đặt trong thư mục `bin` của `GOPATH`.

## Phần 1: Cơ bản

### Packages

Mỗi chương trình Go được tạo thành từ các packages.

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Println("My favorite number is", rand.Intn(10))
}
```

### Imports

```go
import (
    "fmt"
    "math"
)
```

### Exported names

Trong Go, một tên được exported nếu nó bắt đầu bằng chữ cái viết hoa.

```go
fmt.Println  // Exported
math.Pi      // Exported
math.pi      // Không exported
```

### Functions

```go
func add(x int, y int) int {
    return x + y
}

// Rút gọn
func add(x, y int) int {
    return x + y
}
```

### Multiple return values

```go
func swap(x, y string) (string, string) {
    return y, x
}

a, b := swap("hello", "world")
```

### Variables

```go
var i int
var c, python, java bool

// Với initializers
var i, j int = 1, 2

// Short variable declaration
k := 3
```

### Kiểu cơ bản

```
bool
string
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
byte // alias for uint8
rune // alias for int32, represents a Unicode code point
float32 float64
complex64 complex128
```

### Zero values

```go
var i int      // 0
var f float64  // 0.0
var b bool     // false
var s string   // ""
```

### Type conversions

```go
i := 42
f := float64(i)
u := uint(f)
```

### Constants

```go
const Pi = 3.14
const World = "世界"
```

## Phần 2: Flow Control

### For

Go chỉ có một loại vòng lặp: `for`.

```go
for i := 0; i < 10; i++ {
    sum += i
}

// Như while
for sum < 1000 {
    sum += sum
}

// Vô hạn
for {
    // ...
}
```

### If

```go
if x < 0 {
    return sqrt(-x) + "i"
}

// Với statement
if v := math.Pow(x, n); v < lim {
    return v
}
```

### Switch

```go
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X.")
case "linux":
    fmt.Println("Linux.")
default:
    fmt.Printf("%s.\n", os)
}

// Switch không có điều kiện
switch {
case t.Hour() < 12:
    fmt.Println("Good morning!")
case t.Hour() < 17:
    fmt.Println("Good afternoon.")
default:
    fmt.Println("Good evening.")
}
```

### Defer

```go
func main() {
    defer fmt.Println("world")
    fmt.Println("hello")
}
// Output: hello, world
```

## Phần 3: Kiểu phức tạp

### Pointers

```go
i := 42
p := &i         // point to i
fmt.Println(*p) // read i through the pointer
*p = 21         // set i through the pointer
```

### Structs

```go
type Vertex struct {
    X int
    Y int
}

v := Vertex{1, 2}
v.X = 4
```

### Arrays

```go
var a [2]string
a[0] = "Hello"
a[1] = "World"

primes := [6]int{2, 3, 5, 7, 11, 13}
```

### Slices

```go
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[1:4]  // [3, 5, 7]

// Slice literals
q := []int{2, 3, 5, 7, 11, 13}

// Length và capacity
len(s)
cap(s)

// make
a := make([]int, 5)    // len=5
b := make([]int, 0, 5) // len=0, cap=5

// append
s = append(s, 2, 3, 4)
```

### Range

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
}

// Bỏ qua index hoặc value
for _, v := range pow { }
for i := range pow { }
```

### Maps

```go
m := make(map[string]int)
m["Answer"] = 42

// Map literals
m := map[string]int{
    "Bell Labs": 1,
    "Google":    2,
}

// Delete
delete(m, "Answer")

// Test key
v, ok := m["Answer"]
```

### Function values

```go
hypot := func(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}

fmt.Println(hypot(5, 12))
```

### Closures

```go
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}
```

## Phần 4: Methods và Interfaces

### Methods

```go
type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

v := Vertex{3, 4}
fmt.Println(v.Abs())
```

### Pointer receivers

```go
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

### Interfaces

```go
type Abser interface {
    Abs() float64
}

var a Abser
a = Vertex{3, 4}
```

### Empty interface

```go
var i interface{}
i = 42
i = "hello"
```

### Type assertions

```go
t := i.(T)       // panic nếu không đúng type
t, ok := i.(T)   // ok = false nếu không đúng type
```

### Type switches

```go
switch v := i.(type) {
case int:
    fmt.Printf("Twice %v is %v\n", v, v*2)
case string:
    fmt.Printf("%q is %v bytes long\n", v, len(v))
default:
    fmt.Printf("I don't know about type %T!\n", v)
}
```

### Stringer

```go
type Stringer interface {
    String() string
}
```

### Error

```go
type error interface {
    Error() string
}
```

### Readers

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

## Phần 5: Generics (Go 1.18+)

### Type parameters

```go
func Index[T comparable](s []T, x T) int {
    for i, v := range s {
        if v == x {
            return i
        }
    }
    return -1
}
```

### Generic types

```go
type List[T any] struct {
    next *List[T]
    val  T
}
```

## Phần 6: Concurrency

### Goroutines

```go
go f(x, y, z)
```

### Channels

```go
ch := make(chan int)
ch <- v    // Send v to channel ch
v := <-ch  // Receive from ch

// Buffered channels
ch := make(chan int, 100)
```

### Range và close

```go
close(ch)

for i := range ch {
    fmt.Println(i)
}
```

### Select

```go
select {
case c <- x:
    // ...
case <-quit:
    return
default:
    // ...
}
```

### sync.Mutex

```go
type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    c.v[key]++
    c.mu.Unlock()
}
```

---

*Tham gia tour trực tuyến tại [tour.golang.org](https://tour.golang.org)*
