# Làm việc với các đường dẫn file

## Giới thiệu

Đường dẫn file có thể gây ra nhiều vấn đề, đặc biệt khi viết code cross-platform. Tài liệu này mô tả cách làm việc với chúng một cách an toàn trong Go.

## Các vấn đề thường gặp

### Path separators

```go
// ❌ KHÔNG LÀM NHƯ VẦY
path := "dir/subdir/file.txt"  // Không hoạt động trên Windows

// ✅ SỬ DỤNG filepath.Join
path := filepath.Join("dir", "subdir", "file.txt")
```

### Windows vs Unix paths

| Platform | Separator | Ví dụ |
|----------|-----------|-------|
| Unix/Mac | `/` | `/home/user/file.txt` |
| Windows | `\` | `C:\Users\user\file.txt` |

## Package filepath

### filepath.Join

```go
import "path/filepath"

// Tự động sử dụng separator phù hợp
path := filepath.Join("dir", "subdir", "file.txt")
// Unix: dir/subdir/file.txt
// Windows: dir\subdir\file.txt
```

### filepath.Clean

```go
// Normalize path
path := filepath.Clean("dir/../other/./file.txt")
// Result: other/file.txt
```

### filepath.Abs

```go
// Convert to absolute path
absPath, err := filepath.Abs("relative/path")
if err != nil {
    log.Fatal(err)
}
```

### filepath.Rel

```go
// Get relative path
rel, err := filepath.Rel("/home/user", "/home/user/dir/file.txt")
// Result: dir/file.txt
```

### filepath.Split

```go
dir, file := filepath.Split("/home/user/file.txt")
// dir: /home/user/
// file: file.txt
```

### filepath.Dir và filepath.Base

```go
dir := filepath.Dir("/home/user/file.txt")   // /home/user
base := filepath.Base("/home/user/file.txt") // file.txt
ext := filepath.Ext("/home/user/file.txt")   // .txt
```

## Package path (for URLs/URIs)

```go
import "path"

// Sử dụng path package cho URLs (luôn dùng /)
urlPath := path.Join("api", "users", "123")
// Result: api/users/123
```

## Xử lý Volume Names (Windows)

```go
// Windows volume
vol := filepath.VolumeName("C:\\Users\\file.txt")
// Result: C:

// Unix (trống)
vol := filepath.VolumeName("/home/user/file.txt")
// Result: ""
```

## Symlinks

### Đọc symlink

```go
target, err := os.Readlink("/path/to/symlink")
if err != nil {
    log.Fatal(err)
}
```

### Resolve symlink

```go
// filepath.EvalSymlinks resolves all symlinks
realPath, err := filepath.EvalSymlinks("/path/to/symlink")
if err != nil {
    log.Fatal(err)
}
```

## Path Traversal Security

### Kiểm tra path traversal

```go
func SafeJoin(basePath, userPath string) (string, error) {
    // Clean the user input
    cleanPath := filepath.Clean(userPath)
    
    // Check for path traversal attempts
    if strings.Contains(cleanPath, "..") {
        return "", errors.New("path traversal detected")
    }
    
    fullPath := filepath.Join(basePath, cleanPath)
    
    // Verify the result is under basePath
    if !strings.HasPrefix(fullPath, filepath.Clean(basePath)+string(os.PathSeparator)) {
        return "", errors.New("path escapes base directory")
    }
    
    return fullPath, nil
}
```

### Sử dụng

```go
safePath, err := SafeJoin("/var/www/files", userInput)
if err != nil {
    log.Printf("Invalid path: %v", err)
    return
}
```

## Glob Patterns

### filepath.Glob

```go
// Tìm tất cả .go files
matches, err := filepath.Glob("*.go")
if err != nil {
    log.Fatal(err)
}

// Recursive pattern
matches, err := filepath.Glob("**/*.go") // Không hoạt động!
```

### Recursive walking

```go
err := filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
    if err != nil {
        return err
    }
    if filepath.Ext(path) == ".go" {
        fmt.Println(path)
    }
    return nil
})
```

### filepath.WalkDir (Go 1.16+)

```go
err := filepath.WalkDir(".", func(path string, d fs.DirEntry, err error) error {
    if err != nil {
        return err
    }
    if !d.IsDir() && filepath.Ext(path) == ".go" {
        fmt.Println(path)
    }
    return nil
})
```

## Embedded Files

### Go 1.16+ embed

```go
import "embed"

//go:embed templates/*
var templates embed.FS

// Đọc file
data, err := templates.ReadFile("templates/index.html")
```

## Temporary Files

```go
// Tạo temp file
f, err := os.CreateTemp("", "prefix-*.txt")
if err != nil {
    log.Fatal(err)
}
defer os.Remove(f.Name()) // Cleanup

// Tạo temp directory
dir, err := os.MkdirTemp("", "prefix-*")
if err != nil {
    log.Fatal(err)
}
defer os.RemoveAll(dir)
```

## Best Practices

1. **Luôn sử dụng filepath.Join**: Không nối string paths
2. **Clean user input**: Trước khi join với base path
3. **Validate paths**: Kiểm tra path traversal
4. **Sử dụng filepath cho filesystem, path cho URLs**
5. **Handle symlinks cẩn thận**: Có thể gây security issues
6. **Sử dụng WalkDir thay vì Walk**: Hiệu quả hơn
7. **Test trên multiple platforms**: Windows và Unix

---

*Xem thêm tại [pkg.go.dev/path/filepath](https://pkg.go.dev/path/filepath)*
