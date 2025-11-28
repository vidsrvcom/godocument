# Làm việc với Đường dẫn File

Làm việc với file paths trong Go có thể gặp nhiều vấn đề liên quan đến platform differences, security, và edge cases. Tài liệu này mô tả các thực hành tốt nhất và các vấn đề cần tránh.

## Platform Differences

### Path Separators

Hệ điều hành khác nhau sử dụng separators khác nhau:

```go
// Windows: C:\Users\name\file.txt
// Unix/Linux/macOS: /home/name/file.txt

// ❌ SAI - Hard-coded separator
path := "data" + "/" + "file.txt"  // Không hoạt động trên Windows

// ✅ ĐÚNG - Sử dụng filepath package
path := filepath.Join("data", "file.txt")  // Hoạt động mọi platform
```

### Root Paths

```go
import "path/filepath"

// Windows: C:\, D:\, \\server\share
// Unix: /

func isAbsolute(path string) bool {
    return filepath.IsAbs(path)
}

fmt.Println(filepath.IsAbs("C:\\Users"))      // true (Windows)
fmt.Println(filepath.IsAbs("/home/user"))     // true (Unix)
fmt.Println(filepath.IsAbs("relative/path"))  // false
```

## filepath vs path

### filepath package

Dùng cho file system paths (platform-dependent):

```go
import "path/filepath"

// Automatically uses correct separator
path := filepath.Join("dir", "subdir", "file.txt")
// Windows: dir\subdir\file.txt
// Unix: dir/subdir/file.txt

// Clean path
cleaned := filepath.Clean("dir//subdir/../file.txt")
// Result: dir/file.txt

// Split path
dir, file := filepath.Split("/path/to/file.txt")
// dir: /path/to/
// file: file.txt

// Get extension
ext := filepath.Ext("file.txt")  // .txt
```

### path package

Dùng cho URL paths (luôn sử dụng /):

```go
import "path"

// Always uses forward slash
urlPath := path.Join("api", "v1", "users")
// Result: api/v1/users (trên mọi platform)

// For URLs
url := "https://example.com/" + path.Join("api", "users", "123")
```

## Path Security

### Path Traversal

Ngăn chặn path traversal attacks:

```go
import (
    "path/filepath"
    "strings"
)

// ❌ NGUY HIỂM - Path traversal
func unsafeReadFile(baseDir, filename string) ([]byte, error) {
    path := filepath.Join(baseDir, filename)
    return os.ReadFile(path)
}
// Attacker: "../../../etc/passwd"

// ✅ AN TOÀN - Validate path
func safeReadFile(baseDir, filename string) ([]byte, error) {
    // Clean và resolve path
    baseDir, err := filepath.Abs(baseDir)
    if err != nil {
        return nil, err
    }
    
    path := filepath.Join(baseDir, filepath.Clean(filename))
    path, err = filepath.Abs(path)
    if err != nil {
        return nil, err
    }
    
    // Verify path is within baseDir
    if !strings.HasPrefix(path, baseDir+string(filepath.Separator)) {
        return nil, fmt.Errorf("invalid path: outside base directory")
    }
    
    return os.ReadFile(path)
}
```

### Path Validation

```go
func validatePath(basePath, userPath string) (string, error) {
    // Clean user input
    userPath = filepath.Clean(userPath)
    
    // Reject absolute paths
    if filepath.IsAbs(userPath) {
        return "", fmt.Errorf("absolute paths not allowed")
    }
    
    // Join with base
    fullPath := filepath.Join(basePath, userPath)
    
    // Get absolute paths
    absBase, err := filepath.Abs(basePath)
    if err != nil {
        return "", err
    }
    
    absFull, err := filepath.Abs(fullPath)
    if err != nil {
        return "", err
    }
    
    // Verify within base directory
    rel, err := filepath.Rel(absBase, absFull)
    if err != nil {
        return "", err
    }
    
    if strings.HasPrefix(rel, ".."+string(filepath.Separator)) {
        return "", fmt.Errorf("path escapes base directory")
    }
    
    return fullPath, nil
}
```

## Path Manipulation

### Join Paths

```go
// ✅ ĐÚNG
path := filepath.Join("dir", "subdir", "file.txt")

// ❌ SAI
path := "dir" + "/" + "subdir" + "/" + "file.txt"
```

### Clean Paths

```go
// Remove redundant elements
cleaned := filepath.Clean("dir/./subdir/../file.txt")
// Result: dir/file.txt

// Clean handles various cases
examples := []string{
    "a/b/c",           // a/b/c
    "a/b/c/./../../g", // a/g
    "a/../../b/c",     // ../b/c
    ".",               // .
    "",                // .
}

for _, ex := range examples {
    fmt.Printf("%q => %q\n", ex, filepath.Clean(ex))
}
```

### Absolute Paths

```go
// Get absolute path
absPath, err := filepath.Abs("relative/path")
if err != nil {
    log.Fatal(err)
}

// Check if path is absolute
if filepath.IsAbs(path) {
    fmt.Println("Absolute path")
}
```

### Relative Paths

```go
// Get relative path from base to target
rel, err := filepath.Rel("/a/b", "/a/b/c/d")
// rel: c/d

rel, err = filepath.Rel("/a/b", "/a/x/y")
// rel: ../x/y
```

## Path Components

### Split Components

```go
// Split into directory and file
dir, file := filepath.Split("/path/to/file.txt")
// dir: /path/to/
// file: file.txt

// Get directory
dir := filepath.Dir("/path/to/file.txt")
// dir: /path/to

// Get filename
file := filepath.Base("/path/to/file.txt")
// file: file.txt
```

### Extension

```go
// Get extension
ext := filepath.Ext("file.txt")      // .txt
ext = filepath.Ext("file.tar.gz")    // .gz
ext = filepath.Ext("file")           // ""

// Check extension
if filepath.Ext(filename) == ".txt" {
    // Process text file
}

// Change extension
func changeExt(path, newExt string) string {
    return path[:len(path)-len(filepath.Ext(path))] + newExt
}

newPath := changeExt("file.txt", ".md")  // file.md
```

### Volume Name (Windows)

```go
// Get volume (drive letter on Windows)
vol := filepath.VolumeName("C:\\path\\file")  // C:
vol = filepath.VolumeName("/path/file")       // "" (Unix)
```

## Walking Directories

### filepath.Walk

```go
func walkDir(root string) error {
    return filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        // Skip directories
        if info.IsDir() {
            return nil
        }
        
        // Process files
        fmt.Println(path)
        return nil
    })
}
```

### filepath.WalkDir (Go 1.16+)

Hiệu quả hơn Walk:

```go
func walkDirFast(root string) error {
    return filepath.WalkDir(root, func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        if d.IsDir() {
            return nil
        }
        
        fmt.Println(path)
        return nil
    })
}
```

### Skip Directories

```go
func walkSkipHidden(root string) error {
    return filepath.WalkDir(root, func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        // Skip hidden directories
        if d.IsDir() && strings.HasPrefix(d.Name(), ".") {
            return filepath.SkipDir
        }
        
        fmt.Println(path)
        return nil
    })
}
```

## Glob Patterns

### Match Patterns

```go
// Find all .txt files
matches, err := filepath.Glob("*.txt")

// Find in subdirectories (not recursive)
matches, err = filepath.Glob("data/*.txt")

// Match patterns
matched, err := filepath.Match("*.txt", "file.txt")  // true
matched, err = filepath.Match("file?.txt", "file1.txt")  // true
```

### Recursive Glob

```go
func recursiveGlob(root, pattern string) ([]string, error) {
    var matches []string
    
    err := filepath.WalkDir(root, func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        if d.IsDir() {
            return nil
        }
        
        matched, err := filepath.Match(pattern, filepath.Base(path))
        if err != nil {
            return err
        }
        
        if matched {
            matches = append(matches, path)
        }
        
        return nil
    })
    
    return matches, err
}

// Usage
txtFiles, err := recursiveGlob("./data", "*.txt")
```

## Symbolic Links

### Resolve Symlinks

```go
// Follow symlink to target
target, err := filepath.EvalSymlinks("/path/to/symlink")
if err != nil {
    log.Fatal(err)
}

// Check if path is symlink
info, err := os.Lstat(path)
if err != nil {
    log.Fatal(err)
}

if info.Mode()&os.ModeSymlink != 0 {
    fmt.Println("Is a symlink")
}
```

## Temporary Files và Directories

### Temp Paths

```go
// Get temp directory
tempDir := os.TempDir()

// Create safe temp path
tempPath := filepath.Join(tempDir, "myapp", "temp.txt")

// Ensure parent directory exists
if err := os.MkdirAll(filepath.Dir(tempPath), 0755); err != nil {
    log.Fatal(err)
}
```

## Edge Cases

### Empty Paths

```go
// Clean handles empty string
filepath.Clean("")  // "."

// Join with empty
filepath.Join("a", "", "b")  // "a/b"
```

### Dots

```go
// Current directory
filepath.Clean(".")   // "."
filepath.Clean("./")  // "."

// Parent directory
filepath.Clean("..")   // ".."
filepath.Clean("../")  // ".."
```

### Trailing Slashes

```go
// Cleaned paths don't have trailing slash
filepath.Clean("dir/")  // "dir"

// Exception: root
filepath.Clean("/")  // "/"
```

## Best Practices

1. **Luôn sử dụng filepath.Join**: Không concatenate strings
2. **Clean user input**: Sử dụng filepath.Clean
3. **Validate paths**: Kiểm tra path traversal
4. **Use Abs paths**: Cho security-sensitive operations
5. **Handle errors**: Mọi filepath operation có thể fail
6. **Test on multiple platforms**: Windows và Unix khác nhau
7. **Use path for URLs**: Không dùng filepath cho URL paths

## Common Pitfalls

```go
// ❌ Hard-coded separator
path := "dir" + "/" + "file.txt"

// ✅ Platform-independent
path := filepath.Join("dir", "file.txt")

// ❌ Not validating user input
userPath := r.URL.Query().Get("file")
os.ReadFile(userPath)  // Path traversal!

// ✅ Validate and sanitize
safeReadFile(baseDir, userPath)

// ❌ Assuming Unix paths on Windows
path := "/home/user/file.txt"  // Không tồn tại trên Windows

// ✅ Use platform-appropriate paths
home, _ := os.UserHomeDir()
path := filepath.Join(home, "file.txt")
```

## Testing Paths

```go
func TestPathHandling(t *testing.T) {
    testCases := []struct {
        input    string
        expected string
    }{
        {"dir/./file.txt", "dir/file.txt"},
        {"dir/../file.txt", "file.txt"},
        {"./file.txt", "file.txt"},
    }
    
    for _, tc := range testCases {
        got := filepath.Clean(tc.input)
        if got != tc.expected {
            t.Errorf("Clean(%q) = %q, want %q", tc.input, got, tc.expected)
        }
    }
}
```

## Xem thêm

- [Package filepath](https://pkg.go.dev/path/filepath)
- [Package path](https://pkg.go.dev/path)
- [Bảo mật](../security/best-practices.md)
