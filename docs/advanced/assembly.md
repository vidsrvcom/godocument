# Assembly trong Go

Go cho phép bạn viết assembly code để tối ưu hóa hiệu suất hoặc truy cập các tính năng cấp thấp của phần cứng. Tài liệu này hướng dẫn cơ bản về việc sử dụng assembly trong Go.

## Khi nào nên sử dụng Assembly

Assembly code nên được sử dụng:

- ✅ Tối ưu hóa performance-critical code sau khi profile
- ✅ Truy cập CPU instructions cụ thể (SIMD, cryptography, etc.)
- ✅ Implement low-level system interfaces
- ❌ Không nên: premature optimization
- ❌ Không nên: code dễ viết bằng Go thuần

## Go Assembly Syntax

Go sử dụng một assembly syntax riêng (plan9 assembly), khác với AT&T và Intel syntax.

### File Structure

Assembly code được viết trong files `.s`:

```
mypackage/
├── math.go       # Go code với function declarations
└── math_amd64.s  # Assembly implementation
```

### Basic Syntax

```assembly
// func Add(a, b int64) int64
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX    // Load first argument
    MOVQ b+8(FP), BX    // Load second argument
    ADDQ BX, AX          // Add them
    MOVQ AX, ret+16(FP)  // Store result
    RET
```

## Registers

### General Purpose Registers (amd64)

- `AX`, `BX`, `CX`, `DX`: 64-bit general purpose
- `SI`, `DI`: Source/destination index
- `BP`: Base pointer
- `SP`: Stack pointer
- `R8-R15`: Additional registers

### Special Registers

- `PC`: Program counter
- `SB`: Static base (cho globals)
- `FP`: Frame pointer (cho arguments và locals)

## Function Declaration

### Go Code

```go
// math.go
package math

// Add adds two int64 numbers
// Implemented in assembly
func Add(a, b int64) int64
```

### Assembly Implementation

```assembly
// math_amd64.s
#include "textflag.h"

// func Add(a, b int64) int64
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    MOVQ b+8(FP), BX
    ADDQ BX, AX
    MOVQ AX, ret+16(FP)
    RET
```

### Frame Layout

```
+----------------+ ← FP + 24
|     ret        |  (8 bytes return value)
+----------------+ ← FP + 16
|      b         |  (8 bytes second arg)
+----------------+ ← FP + 8
|      a         |  (8 bytes first arg)
+----------------+ ← FP
```

## Calling Convention

### Arguments và Return Values

```go
// Go function
func Multiply(a, b int32) (int32, bool)
```

```assembly
// Assembly
TEXT ·Multiply(SB), NOSPLIT, $0-12
    // Arguments:
    // a: 0(FP)  - 4 bytes
    // b: 4(FP)  - 4 bytes
    // Return values:
    // result: 8(FP)  - 4 bytes
    // ok: 12(FP)     - 1 byte
    
    MOVL a+0(FP), AX
    MOVL b+4(FP), BX
    IMULL BX, AX
    MOVL AX, ret+8(FP)
    MOVB $1, ret1+12(FP)  // ok = true
    RET
```

## Stack Management

### Frame Size

```assembly
// $framesize-argsize
TEXT ·MyFunc(SB), NOSPLIT, $32-16
    // $32: local variables space
    // $16: arguments + return values size
```

### Local Variables

```assembly
TEXT ·Sum(SB), NOSPLIT, $16-24
    MOVQ a+0(FP), AX
    MOVQ b+8(FP), BX
    
    // Local variable at SP+0
    MOVQ $0, temp-8(SP)
    
    ADDQ BX, AX
    MOVQ AX, ret+16(FP)
    RET
```

## Architecture-Specific Code

### Platform Files

```
math_amd64.s    # x86-64
math_386.s      # x86
math_arm64.s    # ARM64
math_arm.s      # ARM
```

### Build Tags

```go
// math_amd64.go
//go:build amd64

package math

func Add(a, b int64) int64
```

## Inline Assembly

Go không hỗ trợ inline assembly. Sử dụng external assembly files.

## Common Patterns

### 1. Simple Math Operation

```assembly
// func AddInt64(a, b int64) int64
TEXT ·AddInt64(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    ADDQ b+8(FP), AX
    MOVQ AX, ret+16(FP)
    RET
```

### 2. Pointer Manipulation

```assembly
// func SwapInt64(a, b *int64)
TEXT ·SwapInt64(SB), NOSPLIT, $0-16
    MOVQ a+0(FP), DI    // DI = address of a
    MOVQ b+8(FP), SI    // SI = address of b
    
    MOVQ (DI), AX       // AX = *a
    MOVQ (SI), BX       // BX = *b
    
    MOVQ BX, (DI)       // *a = BX
    MOVQ AX, (SI)       // *b = AX
    RET
```

### 3. Loop

```assembly
// func Sum(data []int64) int64
TEXT ·Sum(SB), NOSPLIT, $0-32
    MOVQ data_base+0(FP), SI  // SI = slice base
    MOVQ data_len+8(FP), CX   // CX = slice length
    XORQ AX, AX               // AX = 0 (sum)
    
loop:
    TESTQ CX, CX              // if CX == 0
    JZ done                   // goto done
    
    ADDQ (SI), AX             // sum += *SI
    ADDQ $8, SI               // SI += 8 (next element)
    DECQ CX                   // CX--
    JMP loop
    
done:
    MOVQ AX, ret+24(FP)
    RET
```

### 4. SIMD Operations

```assembly
// func AddVectors(a, b []float32, result []float32)
TEXT ·AddVectors(SB), NOSPLIT, $0-72
    MOVQ a_base+0(FP), SI
    MOVQ b_base+24(FP), DI
    MOVQ result_base+48(FP), DX
    MOVQ a_len+8(FP), CX
    SHRQ $2, CX               // Divide by 4 (process 4 at a time)
    
loop:
    TESTQ CX, CX
    JZ done
    
    // Load 4 floats from a and b
    MOVUPS (SI), X0
    MOVUPS (DI), X1
    
    // Add vectors
    ADDPS X1, X0
    
    // Store result
    MOVUPS X0, (DX)
    
    ADDQ $16, SI
    ADDQ $16, DI
    ADDQ $16, DX
    DECQ CX
    JMP loop
    
done:
    RET
```

## Text Flags

Common flags trong TEXT directive:

```assembly
#include "textflag.h"

// NOSPLIT: không split stack
TEXT ·Func1(SB), NOSPLIT, $0

// NOPTR: frame không chứa pointers
TEXT ·Func2(SB), NOSPLIT|NOPTR, $32

// WRAPPER: function là wrapper
TEXT ·Func3(SB), WRAPPER, $0
```

## Data Declarations

### Global Variables

```assembly
// Data in readonly section
DATA ·pi+0(SB)/8, $3.14159265358979323846
GLOBL ·pi(SB), RODATA, $8

// Array/slice data
DATA ·values+0(SB)/4, $1
DATA ·values+4(SB)/4, $2
DATA ·values+8(SB)/4, $3
GLOBL ·values(SB), RODATA, $12
```

### Constants

```assembly
#define CONST1 0x1234
#define CONST2 0x5678

TEXT ·MyFunc(SB), NOSPLIT, $0
    MOVQ $CONST1, AX
    RET
```

## Debugging Assembly

### Generate Assembly from Go

```bash
# Generate assembly listing
go build -gcflags=-S main.go > output.s

# Dissasemble binary
go tool objdump -s main.main binary
```

### Check Assembly Syntax

```bash
# Compile and check
go tool asm -S file.s

# Build with assembly
go build -asmflags=-S
```

## Performance Optimization

### Benchmarking

```go
func BenchmarkGoAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = GoAdd(123, 456)
    }
}

func BenchmarkAsmAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = AsmAdd(123, 456)
    }
}
```

### Profiling

```bash
go test -bench=. -cpuprofile=cpu.prof
go tool pprof cpu.prof
```

## Best Practices

1. **Profile first**: Chỉ optimize khi cần thiết
2. **Write tests**: Verify correctness
3. **Benchmark**: Compare với Go implementation
4. **Comment thoroughly**: Assembly khó đọc
5. **Use build tags**: Support multiple architectures
6. **Keep it simple**: Complex logic nên để trong Go
7. **Avoid premature optimization**: Go compiler rất tốt

## Common Pitfalls

```assembly
// ❌ SAI - Forget to save caller-saved registers
TEXT ·BadFunc(SB), NOSPLIT, $0
    MOVQ AX, BX  // Clobbers BX without saving
    CALL ·other(SB)
    RET

// ✅ ĐÚNG - Save and restore
TEXT ·GoodFunc(SB), NOSPLIT, $8
    MOVQ BX, 0(SP)     // Save BX
    MOVQ AX, BX
    CALL ·other(SB)
    MOVQ 0(SP), BX     // Restore BX
    RET
```

## Ví dụ thực tế

### Fast Memory Copy

```assembly
// func MemCopy(dst, src []byte)
TEXT ·MemCopy(SB), NOSPLIT, $0-48
    MOVQ dst_base+0(FP), DI
    MOVQ src_base+24(FP), SI
    MOVQ src_len+32(FP), CX
    
    // Copy 8 bytes at a time
    SHRQ $3, CX
    REP; MOVSQ
    
    // Copy remaining bytes
    MOVQ src_len+32(FP), CX
    ANDQ $7, CX
    REP; MOVSB
    
    RET
```

## Resources

- Go Assembly Guide: https://go.dev/doc/asm
- Plan9 Assembly: https://9p.io/sys/doc/asm.html
- Go internal ABI: https://go.googlesource.com/go/+/refs/heads/master/src/cmd/compile/abi-internal.md

## Xem thêm

- [Benchmarking](benchmarking.md)
- [Race Detector](race-detector.md)
- [Code Coverage](code-coverage.md)
