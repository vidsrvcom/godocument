# Assembly trong Go

## Giới thiệu

Go cho phép viết hợp ngữ (assembly) cho các tác vụ cấp thấp, tối ưu hóa hiệu suất, hoặc truy cập các tính năng CPU đặc biệt.

## Go Assembler

Go sử dụng assembler riêng với syntax dựa trên Plan 9 assembly, khác với AT&T hoặc Intel syntax.

### Cú pháp cơ bản

```asm
// func Add(a, b int) int
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX    // Load first argument
    MOVQ b+8(FP), BX    // Load second argument
    ADDQ BX, AX         // Add them
    MOVQ AX, ret+16(FP) // Store result
    RET
```

## Cấu trúc File

### Naming convention

```
package/
├── math.go          # Go declarations
├── math_amd64.s     # AMD64 assembly
├── math_arm64.s     # ARM64 assembly
└── math_generic.go  # Generic Go implementation
```

### Go declaration

```go
// math.go
package math

// Add adds two integers.
// This function is implemented in assembly.
func Add(a, b int) int
```

### Assembly implementation

```asm
// math_amd64.s
#include "textflag.h"

// func Add(a, b int) int
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    MOVQ b+8(FP), BX
    ADDQ BX, AX
    MOVQ AX, ret+16(FP)
    RET
```

## Registers

### AMD64 (x86-64)

| Register | Purpose |
|----------|---------|
| AX, BX, CX, DX | General purpose |
| SI, DI | Source/Destination index |
| SP | Stack pointer (pseudo-register) |
| FP | Frame pointer (pseudo-register) |
| SB | Static base (pseudo-register) |
| PC | Program counter (pseudo-register) |
| R8-R15 | Additional general purpose |

### ARM64

| Register | Purpose |
|----------|---------|
| R0-R30 | General purpose |
| SP | Stack pointer |
| LR (R30) | Link register |
| FP (R29) | Frame pointer |

## Pseudo-registers

### FP (Frame Pointer)

```asm
// Arguments và return values
MOVQ arg1+0(FP), AX    // First argument
MOVQ arg2+8(FP), BX    // Second argument
MOVQ ret+16(FP), CX    // Return value location
```

### SB (Static Base)

```asm
// Global symbols
MOVQ ·globalVar(SB), AX  // Load global variable
```

### SP (Stack Pointer)

```asm
// Local variables
MOVQ -8(SP), AX   // Load from stack
```

## Flags

### TEXT flags

```asm
#include "textflag.h"

// NOSPLIT - không insert stack split check
TEXT ·Func(SB), NOSPLIT, $0-16

// NOFRAME - không allocate stack frame
TEXT ·Func(SB), NOSPLIT|NOFRAME, $0-16

// WRAPPER - wrapper function
TEXT ·Func(SB), WRAPPER, $0-16
```

### Frame size

```asm
// $framesize-argsize
TEXT ·Func(SB), NOSPLIT, $16-24
//                        ^^ local stack space
//                           ^^ argument + return size
```

## Ví dụ thực tế

### SIMD operations

```asm
// func AddVectors(a, b, result []float32)
TEXT ·AddVectors(SB), NOSPLIT, $0-72
    MOVQ a_data+0(FP), SI
    MOVQ b_data+24(FP), DI
    MOVQ result_data+48(FP), DX
    MOVQ a_len+8(FP), CX
    
    SHRQ $2, CX  // Divide by 4 (SIMD width)
    
loop:
    MOVUPS (SI), X0
    MOVUPS (DI), X1
    ADDPS X1, X0
    MOVUPS X0, (DX)
    
    ADDQ $16, SI
    ADDQ $16, DI
    ADDQ $16, DX
    DECQ CX
    JNZ loop
    
    RET
```

### Atomic operations

```asm
// func CompareAndSwap(addr *int64, old, new int64) bool
TEXT ·CompareAndSwap(SB), NOSPLIT, $0-25
    MOVQ addr+0(FP), BX
    MOVQ old+8(FP), AX
    MOVQ new+16(FP), CX
    LOCK
    CMPXCHGQ CX, (BX)
    SETEQ ret+24(FP)
    RET
```

## Build Tags

### Platform-specific

```go
//go:build amd64
// +build amd64

package math
```

### Fallback implementation

```go
//go:build !amd64
// +build !amd64

package math

func Add(a, b int) int {
    return a + b
}
```

## Debugging

### View generated assembly

```bash
go build -gcflags="-S" .
go tool compile -S file.go
```

### Disassemble binary

```bash
go tool objdump -s "main.Add" ./binary
```

## Best Practices

1. **Chỉ sử dụng khi cần thiết**: Go compiler khá tốt
2. **Luôn có Go fallback**: Cho các platforms không hỗ trợ
3. **Benchmark để verify**: Đo lường improvement thực sự
4. **Document thoroughly**: Assembly khó đọc
5. **Test extensively**: Trên tất cả target platforms
6. **Use NOSPLIT cẩn thận**: Có thể gây stack overflow

---

*Xem thêm tại [go.dev/doc/asm](https://go.dev/doc/asm)*
