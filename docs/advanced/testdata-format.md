# Tham chiếu Format Testdata

## Giới thiệu

Go sử dụng format "txtar" cho các file testdata, đặc biệt trong các test liên quan đến filesystem và multiple files.

## Format Txtar

### Cấu trúc cơ bản

```txtar
comment or main content at the top

-- filename1.txt --
content of filename1

-- dir/filename2.go --
content of filename2
```

### Ví dụ thực tế

```txtar
This is a test case for parsing Go code.

-- go.mod --
module example.com/test

go 1.21

-- main.go --
package main

func main() {
    println("hello")
}

-- main_test.go --
package main

import "testing"

func TestMain(t *testing.T) {
    // test code
}
```

## Sử dụng trong Tests

### Package txtar

```go
import "golang.org/x/tools/txtar"

func TestParseTxtar(t *testing.T) {
    content := []byte(`
-- file1.txt --
hello

-- file2.txt --
world
`)
    
    archive := txtar.Parse(content)
    
    // archive.Comment chứa text trước file đầu tiên
    // archive.Files chứa slice of Files
    
    for _, f := range archive.Files {
        t.Logf("File: %s, Content: %s", f.Name, f.Data)
    }
}
```

### Đọc từ file

```go
func TestFromFile(t *testing.T) {
    archive, err := txtar.ParseFile("testdata/example.txtar")
    if err != nil {
        t.Fatal(err)
    }
    
    for _, f := range archive.Files {
        // Process each file
    }
}
```

## Tạo Archive

### Programmatically

```go
archive := &txtar.Archive{
    Comment: []byte("This is a test archive\n"),
    Files: []txtar.File{
        {Name: "hello.txt", Data: []byte("Hello, World!\n")},
        {Name: "goodbye.txt", Data: []byte("Goodbye!\n")},
    },
}

data := txtar.Format(archive)
```

### Từ directory

```go
func ArchiveDir(dir string) (*txtar.Archive, error) {
    var files []txtar.File
    
    err := filepath.Walk(dir, func(path string, info os.FileInfo, err error) error {
        if err != nil || info.IsDir() {
            return err
        }
        
        data, err := os.ReadFile(path)
        if err != nil {
            return err
        }
        
        relPath, _ := filepath.Rel(dir, path)
        files = append(files, txtar.File{
            Name: relPath,
            Data: data,
        })
        return nil
    })
    
    return &txtar.Archive{Files: files}, err
}
```

## Sử dụng trong Go Tool Tests

### go test với testdata

```
testdata/
├── case1.txtar
├── case2.txtar
└── case3.txtar
```

### Test runner

```go
func TestCases(t *testing.T) {
    files, err := filepath.Glob("testdata/*.txtar")
    if err != nil {
        t.Fatal(err)
    }
    
    for _, file := range files {
        t.Run(filepath.Base(file), func(t *testing.T) {
            archive, err := txtar.ParseFile(file)
            if err != nil {
                t.Fatal(err)
            }
            
            // Run test with archive
            runTest(t, archive)
        })
    }
}
```

## Script-style Tests (testscript)

### Package testscript

```go
import "github.com/rogpeppe/go-internal/testscript"

func TestScript(t *testing.T) {
    testscript.Run(t, testscript.Params{
        Dir: "testdata",
    })
}
```

### Testscript file format

```txtar
# This is a comment

# Run a command
exec go build ./...

# Check stdout
stdout 'expected output'

# Check stderr
! stderr 'error'

# Check file exists
exists output.txt

# Compare file content
cmp output.txt expected.txt

-- main.go --
package main

func main() {}

-- expected.txt --
expected content
```

## Commands thường dùng trong testscript

| Command | Description |
|---------|-------------|
| `exec cmd args` | Chạy command |
| `! exec cmd` | Expect command fail |
| `stdout pattern` | Check stdout matches |
| `stderr pattern` | Check stderr matches |
| `cmp file1 file2` | Compare files |
| `exists file` | Check file exists |
| `! exists file` | Check file not exists |
| `cp src dst` | Copy file |
| `rm file` | Remove file |
| `mkdir dir` | Create directory |
| `cd dir` | Change directory |
| `env VAR=value` | Set environment |

## Best Practices

1. **Đặt testdata trong thư mục testdata/**: Convention chuẩn
2. **Đặt tên file mô tả test case**: feature_test.txtar
3. **Include comment giải thích**: Ở đầu archive
4. **Sử dụng testscript cho integration tests**: Mạnh mẽ hơn
5. **Keep files nhỏ và focused**: Một test case per file

---

*Xem thêm tại [pkg.go.dev/golang.org/x/tools/txtar](https://pkg.go.dev/golang.org/x/tools/txtar)*
