# Mã nguồn Go

Tài liệu này cung cấp thông tin về cách truy cập, hiểu, và làm việc với mã nguồn của Go.

## Repositories Chính

### Go Main Repository

```bash
# Official (Gerrit)
git clone https://go.googlesource.com/go

# GitHub Mirror
git clone https://github.com/golang/go
```

Repository structure:

```
go/
├── src/             # Source code
│   ├── cmd/        # Go commands (go, gofmt, etc.)  
│   ├── runtime/    # Runtime system
│   ├── builtin/    # Built-in functions
│   └── */          # Standard library packages
├── lib/            # Libraries
├── test/           # Tests
├── doc/            # Documentation
├── misc/           # Miscellaneous tools
└── api/            # API definitions
```

### Sub-repositories

Common Go projects:

```bash
# Tools
git clone https://go.googlesource.com/tools

# Crypto
git clone https://go.googlesource.com/crypto

# Net
git clone https://go.googlesource.com/net

# Sys
git clone https://go.googlesource.com/sys

# Text
git clone https://go.googlesource.com/text
```

## Browsing Source Code

### Online

- **Main source**: https://go.googlesource.com/go
- **GitHub mirror**: https://github.com/golang/go
- **Go source browser**: https://cs.opensource.google/go

### Locally

```bash
# Clone repository
git clone https://go.googlesource.com/go
cd go

# Browse với editor
code .  # VS Code
vim .   # Vim

# Use gopls for navigation
```

## Understanding the Codebase

### Standard Library

Location: `src/` directory

```
src/
├── fmt/           # Formatting
├── io/            # I/O primitives
├── net/           # Networking
├── os/            # OS interface
├── sync/          # Synchronization
├── testing/       # Testing
├── time/          # Time operations
└── ...
```

Example: Reading `fmt` package

```bash
cd src/fmt
ls
# format.go  - Main formatting logic
# print.go   - Print functions
# scan.go    # Scan functions
# doc.go     - Package documentation
```

### Runtime

Location: `src/runtime/`

Key files:

```
runtime/
├── proc.go        # Goroutine scheduler
├── malloc.go      # Memory allocator
├── mgc.go         # Garbage collector
├── chan.go        # Channels
├── map.go         # Maps
├── slice.go       # Slices
└── asm_*.s        # Assembly code
```

### Compiler

Location: `src/cmd/compile/`

```
cmd/compile/
├── internal/
│   ├── gc/        # Main compiler
│   ├── ssa/       # SSA backend
│   ├── types/     # Type system
│   └── syntax/    # Parser
└── main.go
```

### Commands

Location: `src/cmd/`

```
cmd/
├── go/            # go command
├── gofmt/         # gofmt tool
├── vet/           # vet tool
├── compile/       # Compiler
├── link/          # Linker
└── ...
```

## Building from Source

### Quick Build

```bash
cd src
./all.bash  # Linux/macOS
all.bat     # Windows

# Result
../bin/go version
```

### Development Build

```bash
cd src

# Just compiler và standard library
./make.bash

# Full build với tests
./all.bash

# Clean build
./clean.bash
```

### Custom Build

```bash
# Build specific version
git checkout go1.21.0
cd src
./make.bash

# Build với custom flags
GOEXPERIMENT=boringcrypto ./make.bash
```

## Navigating Code

### Finding Definitions

```bash
# Using grep
grep -r "func Printf" src/

# Using git grep  
git grep "func Printf"

# Using go command
go doc fmt.Printf
```

### Understanding Dependencies

```bash
# List imports of package
go list -f '{{.Imports}}' fmt

# Reverse dependencies
go list -f '{{.ImportPath}} {{.Imports}}' std | grep fmt
```

### Call Graph

```bash
# Install callgraph tool
go install golang.org/x/tools/cmd/callgraph@latest

# Analyze code
callgraph -format digraph ./...
```

## Code Organization

### Package Structure

Standard Go package:

```
package/
├── doc.go         # Package documentation
├── file.go        # Main implementation
├── file_test.go   # Tests
├── example_test.go # Examples
└── internal/      # Internal subpackages
    └── helper.go
```

### Naming Conventions

```go
// Exported (public)
func ExportedFunction() {}
type ExportedType struct {}

// Unexported (private)
func unexportedFunction() {}
type unexportedType struct {}
```

### Internal Packages

```
pkg/
├── public.go
└── internal/      # Chỉ accessible từ pkg
    └── helper.go
```

## Tests

### Test Files

```bash
# All test files
find src -name "*_test.go"

# Run specific package tests
go test fmt

# Run all tests
cd src
./all.bash
```

### Test Organization

```go
// fmt/print_test.go
package fmt_test  // External testing

import "fmt"

func TestPrintf(t *testing.T) {
    // Test implementation
}

func ExamplePrintf() {
    fmt.Printf("Hello %s", "World")
    // Output: Hello World
}
```

### Benchmarks

```go
func BenchmarkSprintf(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fmt.Sprintf("hello %d", 42)
    }
}
```

## Assembly Code

### Assembly files

```
runtime/
├── asm.s          # Generic assembly
├── asm_amd64.s    # x86-64 assembly
├── asm_arm64.s    # ARM64 assembly
└── ...
```

### Reading Assembly

```assembly
// asm_amd64.s
TEXT runtime·add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    ADDQ b+8(FP), AX
    MOVQ AX, ret+16(FP)
    RET
```

## Build Tags

### Platform-specific Code

```go
// file_linux.go
//go:build linux

package mypackage

// Linux-specific implementation
```

```go
// file_windows.go
//go:build windows

package mypackage

// Windows-specific implementation
```

### Conditional Compilation

```bash
# Build với specific tags
go build -tags=debug

# Code
//go:build debug

package mypackage

func debugLog() {
    // Debug logging
}
```

## Documentation

### Package Documentation

```go
// Package fmt implements formatted I/O with functions analogous
// to C's printf and scanf. The format 'verbs' are derived from C's
// but are simpler.
package fmt
```

### Function Documentation

```go
// Printf formats according to a format specifier and writes to standard output.
// It returns the number of bytes written and any write error encountered.
func Printf(format string, a ...any) (n int, err error) {
    return Fprintf(os.Stdout, format, a...)
}
```

### Reading Documentation

```bash
# In terminal
go doc fmt.Printf

# Full package
go doc fmt

# Open trong browser
godoc -http=:6060
```

## Contributing to Source

### Making Changes

```bash
# Create branch
git checkout -b my-fix

# Make changes
vim src/fmt/print.go

# Test
cd src
./all.bash

# Commit
git add .
git commit -m "fmt: fix Printf bug"
```

### Code Review

```bash
# Install git-codereview
go install golang.org/x/review/git-codereview@latest

# Upload for review
git codereview mail
```

## Advanced Topics

### Compiler Internals

Study compiler:

```bash
# View SSA output
go build -gcflags="-S" main.go

# View optimization decisions
go build -gcflags="-m" main.go

# Full compiler trace
go build -gcflags="-d=ssa/check_bce/debug=1" main.go
```

### Runtime Internals

Debug runtime:

```bash
# Runtime statistics
GODEBUG=gctrace=1 go run main.go

# Scheduler trace
GODEBUG=schedtrace=1000 go run main.go
```

### Profiling

```bash
# CPU profile
go test -cpuprofile=cpu.prof

# Analyze
go tool pprof cpu.prof

# Visualize
go tool pprof -http=:8080 cpu.prof
```

## Resources

### Official Sources

- Main repo: https://go.googlesource.com/go
- Code search: https://cs.opensource.google/go
- Package docs: https://pkg.go.dev/std

### Learning Resources

- Effective Go: https://go.dev/doc/effective_go
- Go by Example: https://gobyexample.com
- Go source code studies: Various blogs

### Community

- golang-dev: Development discussions
- golang-nuts: General questions
- Gophers Slack: Realtime chat

## Example Workflows

### Study a Package

```bash
# 1. Clone repo
git clone https://go.googlesource.com/go
cd go

# 2. Navigate to package
cd src/encoding/json

# 3. Read documentation
cat doc.go

# 4. Study main files
ls -la
cat encode.go  # Study encoding
cat decode.go  # Study decoding

# 5. Read tests
cat encode_test.go

# 6. Run tests
go test -v
```

### Trace a Bug

```bash
# 1. Find relevant code
git grep "error message"

# 2. Check history
git log --all --grep="bug description"

# 3. View changes
git show <commit-hash>

# 4. Test fix locally
vim affected_file.go
go test
```

## Xem thêm

- [Development Process](process.md)
- [Đóng góp mã](contributing.md)
- [Wiki Contributing](wiki-contributing.md)
