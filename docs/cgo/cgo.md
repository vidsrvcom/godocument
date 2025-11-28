# Cgo

## Giới thiệu

Cgo cho phép các package Go gọi code C. Đây là cầu nối giữa Go và các thư viện C hiện có.

## Cơ bản

### Import "C"

```go
package main

/*
#include <stdio.h>
#include <stdlib.h>

void hello() {
    printf("Hello from C!\n");
}
*/
import "C"

func main() {
    C.hello()
}
```

### Build

```bash
CGO_ENABLED=1 go build
```

## Kiểu dữ liệu

### Mapping Types

| Go | C |
|-----|-----|
| `C.int` | `int` |
| `C.uint` | `unsigned int` |
| `C.long` | `long` |
| `C.char` | `char` |
| `C.float` | `float` |
| `C.double` | `double` |

### Strings

```go
// Go string to C string
cstr := C.CString("hello")
defer C.free(unsafe.Pointer(cstr))

// C string to Go string
gostr := C.GoString(cstr)
```

### Arrays và Slices

```go
// Go slice to C array
goSlice := []C.int{1, 2, 3}
cArray := (*C.int)(unsafe.Pointer(&goSlice[0]))
```

## Gọi C từ Go

### Với inline C

```go
/*
#include <math.h>
*/
import "C"
import "fmt"

func main() {
    result := C.sqrt(C.double(4))
    fmt.Println(result) // 2
}
```

### Với external library

```go
/*
#cgo LDFLAGS: -lmylib
#include "mylib.h"
*/
import "C"
```

## CGO Flags

### CFLAGS

```go
// #cgo CFLAGS: -I/path/to/headers
```

### LDFLAGS

```go
// #cgo LDFLAGS: -L/path/to/libs -lmylib
```

### Platform-specific

```go
// #cgo linux LDFLAGS: -lm
// #cgo darwin LDFLAGS: -framework CoreFoundation
// #cgo windows LDFLAGS: -lws2_32
```

## Callbacks

### Go function as C callback

```go
/*
typedef void (*callback)(int);
void callCallback(callback f, int n) {
    f(n);
}
*/
import "C"

//export goCallback
func goCallback(n C.int) {
    println("Callback with:", int(n))
}

func main() {
    C.callCallback(C.callback(C.goCallback), 42)
}
```

## Memory Management

### Allocate

```go
// Allocate C memory
ptr := C.malloc(C.size_t(100))
defer C.free(ptr)
```

### Rules

1. **C memory**: Phải free thủ công
2. **Go memory**: GC quản lý
3. **Không giữ Go pointers trong C memory**

## Best Practices

### 1. Minimize CGO calls

CGO có overhead, batch operations khi có thể.

### 2. Handle errors

```go
result, err := C.someFunction()
if err != nil {
    // Handle error
}
```

### 3. Thread safety

C code có thể không thread-safe.

### 4. Build tags

```go
// +build cgo

package mypackage
```

## Performance

CGO có overhead:
- Function call overhead: ~100ns
- Memory copy overhead
- Scheduler complexity

## Debugging

```bash
# Verbose output
CGO_CFLAGS="-g" go build

# Use GDB
go build -gcflags=all="-N -l"
gdb ./myapp
```

## Alternatives

- Pure Go implementations
- syscall package
- FFI libraries

---

*Xem thêm tại [go.dev/cmd/cgo](https://go.dev/cmd/cgo)*
