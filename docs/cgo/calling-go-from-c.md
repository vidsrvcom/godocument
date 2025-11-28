# Gọi Go từ C

## Giới thiệu

Go cho phép export functions để C code có thể gọi. Điều này hữu ích khi bạn muốn sử dụng Go như một thư viện trong ứng dụng C.

## Export Function

### Syntax

```go
//export FunctionName
func FunctionName(args) returnType {
    // implementation
}
```

### Ví dụ

```go
package main

import "C"

//export Add
func Add(a, b C.int) C.int {
    return a + b
}

//export Hello
func Hello(name *C.char) *C.char {
    goName := C.GoString(name)
    greeting := "Hello, " + goName
    return C.CString(greeting)
}

func main() {}
```

## Build Shared Library

### Linux (.so)

```bash
go build -buildmode=c-shared -o libmylib.so mylib.go
```

### macOS (.dylib)

```bash
go build -buildmode=c-shared -o libmylib.dylib mylib.go
```

### Windows (.dll)

```bash
go build -buildmode=c-shared -o mylib.dll mylib.go
```

## Generated Header

Build sẽ tạo header file (`libmylib.h`):

```c
// libmylib.h
#ifndef GO_CGO_PROLOGUE_H
#define GO_CGO_PROLOGUE_H

typedef signed char GoInt8;
typedef unsigned char GoUint8;
// ...

extern GoInt Add(GoInt a, GoInt b);
extern char* Hello(char* name);

#endif
```

## Sử dụng từ C

### C Code

```c
// main.c
#include <stdio.h>
#include "libmylib.h"

int main() {
    int result = Add(10, 20);
    printf("10 + 20 = %d\n", result);
    
    char* greeting = Hello("World");
    printf("%s\n", greeting);
    free(greeting);  // Phải free memory từ Go
    
    return 0;
}
```

### Compile

```bash
gcc -o main main.c -L. -lmylib
LD_LIBRARY_PATH=. ./main
```

## Kiểu dữ liệu

### Mapping

| Go | C (Generated) |
|----|---------------|
| `int` | `GoInt` |
| `int32` | `GoInt32` |
| `string` | `GoString` |
| `[]byte` | `GoSlice` |

### Strings

```go
//export ProcessString
func ProcessString(s *C.char) *C.char {
    goStr := C.GoString(s)
    result := strings.ToUpper(goStr)
    return C.CString(result)
}
```

**Lưu ý**: `C.CString` allocates memory phải được free từ C.

### Slices

```go
//export ProcessData
func ProcessData(data *C.char, length C.int) {
    goSlice := C.GoBytes(unsafe.Pointer(data), length)
    // Process goSlice
}
```

## Callbacks

### Go side

```go
//export RegisterCallback
func RegisterCallback(callback unsafe.Pointer) {
    // Store callback
    storedCallback = callback
}

//export TriggerCallback
func TriggerCallback() {
    if storedCallback != nil {
        C.callCallback(storedCallback)
    }
}
```

### C side

```c
void myCallback() {
    printf("Callback called!\n");
}

int main() {
    RegisterCallback((void*)myCallback);
    TriggerCallback();
    return 0;
}
```

## Error Handling

### Return error code

```go
//export DoSomething
func DoSomething() C.int {
    err := actualDoSomething()
    if err != nil {
        return -1  // Error code
    }
    return 0  // Success
}
```

### Return error message

```go
var lastError *C.char

//export GetLastError
func GetLastError() *C.char {
    return lastError
}

//export DoSomethingWithError
func DoSomethingWithError() C.int {
    err := actualDoSomething()
    if err != nil {
        lastError = C.CString(err.Error())
        return -1
    }
    return 0
}
```

## Concurrency

### Thread Safety

Go runtime quản lý goroutines riêng. Khi C gọi Go:
- Go function chạy trên Go runtime
- Có thể spawn goroutines
- Phải cẩn thận với blocking

### Ví dụ

```go
//export AsyncOperation
func AsyncOperation(callback unsafe.Pointer) {
    go func() {
        result := heavyComputation()
        C.invokeCallback(callback, C.int(result))
    }()
}
```

## Best Practices

1. **Minimize exported functions**: Overhead cao
2. **Document memory ownership**: Ai free memory?
3. **Handle panics**: Panic trong exported function có thể crash

```go
//export SafeFunction
func SafeFunction() (result C.int) {
    defer func() {
        if r := recover(); r != nil {
            result = -1
        }
    }()
    // ...
    return 0
}
```

4. **Use C types**: Trong exported functions

## Debugging

```bash
# Enable debug info
CGO_CFLAGS="-g" go build -buildmode=c-shared -o libmylib.so

# Debug with GDB
gdb ./c_application
```

---

*Xem thêm tại [go.dev/cmd/cgo](https://go.dev/cmd/cgo)*
