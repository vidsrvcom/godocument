# Tham chiếu Format Testdata

Testdata trong Go thường sử dụng format "txtar" (text archive), một format đơn giản để lưu trữ nhiều file trong một file text duy nhất. Format này đặc biệt hữu ích cho testing và examples.

## Format Txtar

Txtar là một format archive đơn giản dựa trên text để đại diện cho một tập hợp các files. Nó được sử dụng rộng rãi trong Go toolchain, đặc biệt trong testing.

### Cú pháp cơ bản

```
comment text
-- filename1 --
content of filename1
-- filename2 --
content of filename2
```

### Cấu trúc

1. **Comment section** (tùy chọn): Các dòng ở đầu file trước marker đầu tiên
2. **File markers**: Dòng có dạng `-- filename --`
3. **File content**: Nội dung giữa các markers

## Ví dụ Txtar

```txtar
This is a test archive containing multiple files.
It demonstrates the txtar format.

-- hello.txt --
Hello, World!
This is the content of hello.txt

-- config.json --
{
  "name": "example",
  "version": "1.0.0"
}

-- main.go --
package main

import "fmt"

func main() {
    fmt.Println("Hello from main.go")
}
```

## Sử dụng trong Go Tests

### Package txtar

Go cung cấp package `golang.org/x/tools/txtar` để làm việc với txtar files:

```go
import "golang.org/x/tools/txtar"

// Đọc txtar file
func readArchive(filename string) (*txtar.Archive, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    return txtar.Parse(data), nil
}

// Tạo txtar archive
func createArchive() *txtar.Archive {
    return &txtar.Archive{
        Comment: []byte("This is a test archive\n"),
        Files: []txtar.File{
            {
                Name: "hello.txt",
                Data: []byte("Hello, World!\n"),
            },
            {
                Name: "config.json",
                Data: []byte(`{"version": "1.0"}`),
            },
        },
    }
}

// Ghi archive ra file
func writeArchive(arch *txtar.Archive, filename string) error {
    data := txtar.Format(arch)
    return os.WriteFile(filename, data, 0644)
}
```

### Ví dụ trong Test

```go
func TestWithTxtar(t *testing.T) {
    // Txtar content
    input := `
Test case for processing files
-- input.txt --
line 1
line 2
line 3
-- expected.txt --
LINE 1
LINE 2
LINE 3
`
    
    archive := txtar.Parse([]byte(input))
    
    // Extract files
    var inputFile, expectedFile []byte
    for _, f := range archive.Files {
        switch f.Name {
        case "input.txt":
            inputFile = f.Data
        case "expected.txt":
            expectedFile = f.Data
        }
    }
    
    // Process and compare
    result := processInput(inputFile)
    if !bytes.Equal(result, expectedFile) {
        t.Errorf("got %s, want %s", result, expectedFile)
    }
}
```

## Testdata trong Go Projects

### Cấu trúc thư mục

```
mypackage/
├── mypackage.go
├── mypackage_test.go
└── testdata/
    ├── test1.txtar
    ├── test2.txtar
    └── golden/
        ├── output1.txt
        └── output2.txt
```

### Đọc testdata files

```go
func loadTestdata(t *testing.T, filename string) []byte {
    t.Helper()
    
    path := filepath.Join("testdata", filename)
    data, err := os.ReadFile(path)
    if err != nil {
        t.Fatalf("failed to read testdata: %v", err)
    }
    
    return data
}

func TestWithTestdata(t *testing.T) {
    data := loadTestdata(t, "test1.txtar")
    archive := txtar.Parse(data)
    
    // Use archive in tests
    for _, file := range archive.Files {
        t.Run(file.Name, func(t *testing.T) {
            // Test each file
        })
    }
}
```

## Table-driven Tests với Txtar

```go
func TestProcessor(t *testing.T) {
    tests := []struct {
        name    string
        txtar   string
        wantErr bool
    }{
        {
            name: "basic",
            txtar: `
-- input.go --
package main

func main() {}
-- want.txt --
success
`,
        },
        {
            name: "with_error",
            txtar: `
-- input.go --
invalid go code
-- want.txt --
error: syntax error
`,
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            archive := txtar.Parse([]byte(tt.txtar))
            
            var input, want []byte
            for _, f := range archive.Files {
                switch f.Name {
                case "input.go":
                    input = f.Data
                case "want.txt":
                    want = f.Data
                }
            }
            
            got, err := processInput(input)
            if (err != nil) != tt.wantErr {
                t.Errorf("processInput() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            
            if !bytes.Equal(got, want) {
                t.Errorf("got %s, want %s", got, want)
            }
        })
    }
}
```

## Golden Files

Sử dụng txtar cho golden file testing:

```go
var update = flag.Bool("update", false, "update golden files")

func TestGolden(t *testing.T) {
    input := "test input"
    got := processInput(input)
    
    goldenFile := "testdata/golden.txtar"
    
    if *update {
        // Update golden file
        arch := &txtar.Archive{
            Comment: []byte("Golden test data\n"),
            Files: []txtar.File{
                {Name: "output.txt", Data: got},
            },
        }
        data := txtar.Format(arch)
        if err := os.WriteFile(goldenFile, data, 0644); err != nil {
            t.Fatal(err)
        }
    }
    
    // Compare with golden file
    data, err := os.ReadFile(goldenFile)
    if err != nil {
        t.Fatal(err)
    }
    
    arch := txtar.Parse(data)
    want := arch.Files[0].Data
    
    if !bytes.Equal(got, want) {
        t.Errorf("got:\n%s\nwant:\n%s", got, want)
    }
}

// Run with: go test -update
```

## Multi-file Test Scenarios

```go
func TestMultiFileScenario(t *testing.T) {
    scenario := `
Test scenario: module with multiple files

-- go.mod --
module example.com/test

go 1.21

-- main.go --
package main

import "example.com/test/util"

func main() {
    util.PrintHello()
}

-- util/util.go --
package util

import "fmt"

func PrintHello() {
    fmt.Println("Hello!")
}

-- want.txt --
Hello!
`
    
    archive := txtar.Parse([]byte(scenario))
    
    // Create temporary directory with files
    tmpDir := t.TempDir()
    
    for _, f := range archive.Files {
        if f.Name == "want.txt" {
            continue
        }
        
        path := filepath.Join(tmpDir, f.Name)
        dir := filepath.Dir(path)
        
        if err := os.MkdirAll(dir, 0755); err != nil {
            t.Fatal(err)
        }
        
        if err := os.WriteFile(path, f.Data, 0644); err != nil {
            t.Fatal(err)
        }
    }
    
    // Run test with created files
    // ...
}
```

## Best Practices

1. **Sử dụng comment section**: Mô tả mục đích của test case
2. **Tên file rõ ràng**: Sử dụng tên file có ý nghĩa
3. **Organize testdata**: Nhóm các test cases liên quan
4. **Version control**: Commit testdata files vào git
5. **Update mechanism**: Cung cấp cách update golden files

## Ví dụ thực tế

```go
// testdata/example.txtar
/*
Example from Go standard library
This demonstrates code transformation

-- input.go --
package p

var x = 1 + 2

-- output.go --
package p

var x = 3
*/

func TestCodeTransform(t *testing.T) {
    data, err := os.ReadFile("testdata/example.txtar")
    if err != nil {
        t.Fatal(err)
    }
    
    arch := txtar.Parse(data)
    
    var input, want []byte
    for _, f := range arch.Files {
        switch f.Name {
        case "input.go":
            input = f.Data
        case "output.go":
            want = f.Data
        }
    }
    
    got := transformCode(input)
    
    if !bytes.Equal(got, want) {
        t.Errorf("transformation failed\ngot:\n%s\nwant:\n%s", got, want)
    }
}
```

## Xem thêm

- [Fuzz Configuration](fuzz-config.md)
- [Benchmarking](benchmarking.md)
- [Code Coverage](code-coverage.md)
- Package txtar: https://pkg.go.dev/golang.org/x/tools/txtar
