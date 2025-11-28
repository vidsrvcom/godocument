# Tài liệu Go

Ngôn ngữ lập trình Go là một dự án mã nguồn mở nhằm giúp các lập trình viên làm việc hiệu quả hơn.

Go mang tính biểu đạt cao, súc tích, sạch sẽ và hiệu quả. Các cơ chế đồng thời của nó giúp dễ dàng viết các chương trình tận dụng tối đa các máy đa lõi và mạng, trong khi hệ thống kiểu mới lạ cho phép xây dựng chương trình linh hoạt và theo module. Go biên dịch nhanh thành mã máy nhưng có sự tiện lợi của garbage collection và sức mạnh của runtime reflection. Đây là một ngôn ngữ biên dịch tĩnh, có kiểu nhanh mà cảm giác như ngôn ngữ động, thông dịch.

### Tại sao chọn Go?

- **Đơn giản**: Cú pháp gọn gàng, dễ học, dễ đọc
- **Hiệu suất cao**: Biên dịch trực tiếp thành mã máy
- **Đồng thời mạnh mẽ**: Goroutines và channels
- **Thư viện chuẩn phong phú**: HTTP server, JSON, mã hóa, và nhiều hơn nữa
- **Công cụ tích hợp**: Testing, benchmarking, profiling, formatting
- **Cross-platform**: Biên dịch cho Windows, macOS, Linux, và nhiều nền tảng khác

---

## Bắt đầu

### Cài đặt Go

Hướng dẫn tải xuống và cài đặt Go.

**Yêu cầu hệ thống:**
- Windows 10 hoặc mới hơn, macOS 10.15 hoặc mới hơn, hoặc Linux
- 64-bit processor (amd64 hoặc arm64)
- Ít nhất 500MB dung lượng đĩa trống

**Các bước cài đặt:**

1. **Tải xuống Go**: Truy cập [go.dev/dl](https://go.dev/dl/) và tải phiên bản phù hợp với hệ điều hành của bạn.

2. **Cài đặt theo hệ điều hành:**

   **Linux/macOS:**
   ```bash
   # Xóa phiên bản cũ (nếu có)
   sudo rm -rf /usr/local/go
   
   # Giải nén vào /usr/local
   sudo tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz
   
   # Thêm vào PATH (thêm vào ~/.bashrc hoặc ~/.zshrc)
   export PATH=$PATH:/usr/local/go/bin
   ```

   **Windows:**
   - Chạy file MSI installer và làm theo hướng dẫn
   - Installer tự động thêm Go vào PATH

3. **Xác minh cài đặt:**
   ```bash
   go version
   # Output: go version go1.23.0 linux/amd64
   ```

4. **Cấu hình môi trường (tùy chọn):**
   ```bash
   # Xem cấu hình hiện tại
   go env
   
   # Đặt GOPATH (mặc định là $HOME/go)
   go env -w GOPATH=$HOME/go
   ```

### Hướng dẫn: Bắt đầu với Go

Một hướng dẫn giới thiệu ngắn gọn "Hello, World" để thiết lập và sử dụng Go.

**Bước 1: Tạo thư mục dự án**
```bash
mkdir hello
cd hello
```

**Bước 2: Khởi tạo module**
```bash
go mod init example/hello
```

**Bước 3: Tạo file main.go**
```go
package main

import "fmt"

func main() {
    fmt.Println("Xin chào, thế giới!")
}
```

**Bước 4: Chạy chương trình**
```bash
go run .
# Output: Xin chào, thế giới!
```

**Bước 5: Build thành file thực thi**
```bash
go build
./hello  # hoặc hello.exe trên Windows
```

### Hướng dẫn: Tạo một module Go

Một hướng dẫn giới thiệu ngắn về các hàm, xử lý lỗi, mảng, map, unit testing và biên dịch.

**Cấu trúc module cơ bản:**
```
mymodule/
├── go.mod
├── main.go
├── greetings/
│   ├── greetings.go
│   └── greetings_test.go
└── README.md
```

**Ví dụ hàm với xử lý lỗi:**
```go
package greetings

import (
    "errors"
    "fmt"
)

// Hello trả về lời chào cho người được chỉ định
func Hello(name string) (string, error) {
    if name == "" {
        return "", errors.New("tên không được để trống")
    }
    message := fmt.Sprintf("Xin chào, %v!", name)
    return message, nil
}
```

**Unit test:**
```go
package greetings

import "testing"

func TestHello(t *testing.T) {
    name := "Việt"
    want := "Xin chào, Việt!"
    
    got, err := Hello(name)
    if err != nil {
        t.Fatalf("Hello(%q) trả về lỗi: %v", name, err)
    }
    if got != want {
        t.Errorf("Hello(%q) = %q, muốn %q", name, got, want)
    }
}

func TestHelloEmpty(t *testing.T) {
    _, err := Hello("")
    if err == nil {
        t.Fatal("Hello(\"\") không trả về lỗi như mong đợi")
    }
}
```

**Chạy tests:**
```bash
go test -v ./...
```

### Hướng dẫn: Bắt đầu với multi-module workspaces

Giới thiệu về cơ bản của việc tạo và sử dụng multi-module workspaces trong Go. Multi-module workspaces hữu ích cho việc thực hiện các thay đổi trên nhiều module.

**Tạo workspace:**
```bash
mkdir workspace
cd workspace

# Tạo file go.work
go work init

# Thêm các module vào workspace
go work use ./module1
go work use ./module2
```

**Cấu trúc go.work:**
```go
go 1.23

use (
    ./module1
    ./module2
)
```

**Lợi ích của workspaces:**
- Phát triển nhiều module đồng thời
- Không cần publish module để test các thay đổi
- Dễ dàng debug và refactor code

### Hướng dẫn: Phát triển RESTful API với Go và Gin

Giới thiệu về viết RESTful web service API với Go và Gin Web Framework.

**Cài đặt Gin:**
```bash
go get -u github.com/gin-gonic/gin
```

**Ví dụ API cơ bản:**
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

var albums = []album{
    {ID: "1", Title: "Người lạ ơi", Artist: "Karik", Price: 15.99},
    {ID: "2", Title: "Đừng quên tên anh", Artist: "Hoa Vinh", Price: 12.99},
}

func main() {
    router := gin.Default()
    
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)
    
    router.Run("localhost:8080")
}

func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}

func getAlbumByID(c *gin.Context) {
    id := c.Param("id")
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "không tìm thấy album"})
}

func postAlbums(c *gin.Context) {
    var newAlbum album
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```

### Hướng dẫn: Bắt đầu với generics

Với generics, bạn có thể khai báo và sử dụng các hàm hoặc kiểu được viết để làm việc với bất kỳ tập hợp kiểu nào do mã gọi cung cấp.

**Hàm generic cơ bản:**
```go
// Hàm tìm index của phần tử trong slice
func Index[T comparable](s []T, x T) int {
    for i, v := range s {
        if v == x {
            return i
        }
    }
    return -1
}

// Sử dụng
func main() {
    ints := []int{10, 20, 30, 40}
    fmt.Println(Index(ints, 30))  // Output: 2
    
    strings := []string{"a", "b", "c"}
    fmt.Println(Index(strings, "b"))  // Output: 1
}
```

**Kiểu generic:**
```go
// Stack generic
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

**Type constraints:**
```go
// Constraint cho các kiểu số
type Number interface {
    int | int32 | int64 | float32 | float64
}

func Sum[T Number](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum
}
```

### Hướng dẫn: Bắt đầu với fuzzing

Fuzzing có thể tạo ra các đầu vào cho các bài test của bạn có thể bắt được các trường hợp biên mà bạn có thể đã bỏ lỡ.

**Viết fuzz test:**
```go
func FuzzReverse(f *testing.F) {
    // Thêm các test cases ban đầu
    testcases := []string{"Hello", "世界", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)
    }
    
    // Hàm fuzz
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Reverse 2 lần %q = %q, muốn %q", orig, doubleRev, orig)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse(%q) tạo ra chuỗi UTF-8 không hợp lệ: %q", orig, rev)
        }
    })
}
```

**Chạy fuzz test:**
```bash
# Chạy fuzz test trong 30 giây
go test -fuzz=FuzzReverse -fuzztime=30s

# Chạy với số worker cụ thể
go test -fuzz=FuzzReverse -fuzztime=1m -parallel=4
```

### Viết Ứng dụng Web

Xây dựng một ứng dụng web đơn giản.

**Ví dụ web server cơ bản:**
```go
package main

import (
    "fmt"
    "html/template"
    "log"
    "net/http"
)

type Page struct {
    Title string
    Body  []byte
}

var templates = template.Must(template.ParseFiles("edit.html", "view.html"))

func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}

func main() {
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/edit/", editHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Cách viết mã Go

Tài liệu này giải thích cách phát triển một tập hợp đơn giản các Go package bên trong một module, và cho thấy cách sử dụng lệnh go để build và test các package.

**Tổ chức mã nguồn:**
```
project/
├── go.mod
├── go.sum
├── main.go
├── cmd/
│   └── myapp/
│       └── main.go
├── internal/
│   ├── config/
│   └── database/
├── pkg/
│   └── utils/
└── tests/
    └── integration/
```

**Các lệnh go quan trọng:**
```bash
# Khởi tạo module
go mod init github.com/user/project

# Tải dependencies
go mod download

# Dọn dẹp dependencies không sử dụng
go mod tidy

# Build project
go build ./...

# Chạy tests
go test ./...

# Format code
go fmt ./...

# Kiểm tra lỗi tiềm ẩn
go vet ./...
```

### Hướng dẫn: Truy cập cơ sở dữ liệu quan hệ

Giới thiệu về cơ bản của việc truy cập cơ sở dữ liệu quan hệ với Go và package database/sql trong thư viện chuẩn.

**Ví dụ kết nối MySQL:**
```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    
    _ "github.com/go-sql-driver/mysql"
)

type Album struct {
    ID     int64
    Title  string
    Artist string
    Price  float64
}

var db *sql.DB

func main() {
    var err error
    db, err = sql.Open("mysql", "user:password@tcp(localhost:3306)/recordings")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Kiểm tra kết nối
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
    fmt.Println("Kết nối thành công!")
    
    // Truy vấn dữ liệu
    albums, err := albumsByArtist("Sơn Tùng MTP")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Albums: %v\n", albums)
}

func albumsByArtist(artist string) ([]Album, error) {
    var albums []Album
    rows, err := db.Query("SELECT * FROM album WHERE artist = ?", artist)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    for rows.Next() {
        var alb Album
        if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
            return nil, err
        }
        albums = append(albums, alb)
    }
    return albums, rows.Err()
}
```

### Hướng dẫn: Tìm và sửa các phụ thuộc có lỗ hổng bảo mật

Hướng dẫn sử dụng govulncheck để tìm và sửa các lỗ hổng bảo mật trong các phụ thuộc của bạn.

**Cài đặt govulncheck:**
```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
```

**Sử dụng govulncheck:**
```bash
# Kiểm tra lỗ hổng trong project hiện tại
govulncheck ./...

# Kiểm tra binary
govulncheck -mode=binary myapp

# Output chi tiết
govulncheck -show=verbose ./...
```

**Sửa lỗ hổng:**
```bash
# Cập nhật dependency có lỗ hổng
go get -u github.com/vulnerable/package@latest

# Dọn dẹp và cập nhật go.sum
go mod tidy
```

---

## Sử dụng và hiểu Go

### Effective Go

Một tài liệu cung cấp các mẹo để viết mã Go rõ ràng, theo phong cách chuẩn. Đây là tài liệu phải đọc cho bất kỳ lập trình viên Go mới nào. Nó bổ sung cho tour và đặc tả ngôn ngữ, cả hai đều nên đọc trước.

**Các nguyên tắc chính trong Effective Go:**

1. **Formatting (Định dạng)**
   - Sử dụng `gofmt` để format code tự động
   - Dùng tab cho indentation
   - Không giới hạn độ dài dòng, nhưng nên wrap hợp lý

2. **Naming (Đặt tên)**
   ```go
   // Package names: ngắn, lowercase, không underscore
   package httputil
   
   // Exported names: MixedCaps hoặc mixedCaps
   func ReadFile(filename string) ([]byte, error)
   
   // Acronyms: giữ nguyên case
   type HTTPClient struct{}  // không phải HttpClient
   var URL string            // không phải Url
   ```

3. **Control structures (Cấu trúc điều khiển)**
   ```go
   // If với short statement
   if err := doSomething(); err != nil {
       return err
   }
   
   // Switch không cần break
   switch x {
   case 1:
       fmt.Println("một")
   case 2:
       fmt.Println("hai")
   default:
       fmt.Println("khác")
   }
   ```

4. **Error handling (Xử lý lỗi)**
   ```go
   // Luôn kiểm tra lỗi
   f, err := os.Open(filename)
   if err != nil {
       return nil, fmt.Errorf("mở file %s: %w", filename, err)
   }
   defer f.Close()
   ```

### Đặc tả ngôn ngữ Go

Đặc tả chính thức của ngôn ngữ Go.

**Các khái niệm cơ bản:**

- **Kiểu dữ liệu cơ bản:**
  - Boolean: `bool`
  - Số nguyên: `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
  - Số thực: `float32`, `float64`
  - Số phức: `complex64`, `complex128`
  - Chuỗi: `string`
  - Byte và Rune: `byte` (alias cho `uint8`), `rune` (alias cho `int32`)

- **Kiểu dữ liệu phức hợp:**
  - Array: `[n]T`
  - Slice: `[]T`
  - Map: `map[K]V`
  - Struct: `struct { Field Type }`
  - Pointer: `*T`
  - Channel: `chan T`
  - Interface: `interface { Method() }`

### Tài liệu thư viện chuẩn

Tài liệu cho thư viện chuẩn Go.

**Các package quan trọng:**

| Package | Mô tả |
|---------|-------|
| `fmt` | Formatted I/O |
| `io` | I/O primitives |
| `os` | OS functions |
| `net/http` | HTTP client/server |
| `encoding/json` | JSON encoding/decoding |
| `database/sql` | SQL database interface |
| `sync` | Synchronization primitives |
| `context` | Context for cancellation |
| `testing` | Testing support |
| `time` | Time operations |
| `strings` | String manipulation |
| `strconv` | String conversions |
| `regexp` | Regular expressions |
| `crypto` | Cryptographic functions |

**Ví dụ sử dụng các package phổ biến:**

```go
// fmt - Formatted I/O
import "fmt"
fmt.Printf("Xin chào %s, bạn %d tuổi\n", name, age)

// encoding/json - JSON
import "encoding/json"
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
data, _ := json.Marshal(Person{"Việt", 25})
// {"name":"Việt","age":25}

// net/http - HTTP Server
import "net/http"
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Xin chào!")
})
http.ListenAndServe(":8080", nil)

// time - Time operations
import "time"
now := time.Now()
formatted := now.Format("02/01/2006 15:04:05")
```

### Go Modules Reference

Tham chiếu chi tiết cho hệ thống quản lý phụ thuộc của Go.

**Cấu trúc go.mod:**
```go
module github.com/myuser/myproject

go 1.23

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/go-sql-driver/mysql v1.7.1
)

require (
    // indirect dependencies
    github.com/some/indirect v1.0.0 // indirect
)

replace github.com/original/module => github.com/fork/module v1.0.0

exclude github.com/bad/module v1.0.0
```

**Các lệnh module quan trọng:**
```bash
# Tạo module mới
go mod init github.com/user/project

# Thêm dependency
go get github.com/gin-gonic/gin@latest
go get github.com/gin-gonic/gin@v1.9.0  # version cụ thể

# Cập nhật dependency
go get -u ./...                    # tất cả
go get -u github.com/gin-gonic/gin # một package

# Dọn dẹp dependencies không dùng
go mod tidy

# Vendor dependencies
go mod vendor

# Xác minh checksum
go mod verify

# Xem dependency graph
go mod graph
```

### go.mod file reference

Tham chiếu cho các chỉ thị có trong file go.mod.

| Chỉ thị | Mục đích |
|---------|----------|
| `module` | Định nghĩa module path |
| `go` | Phiên bản Go tối thiểu |
| `require` | Khai báo dependencies |
| `replace` | Thay thế module path |
| `exclude` | Loại trừ version cụ thể |
| `retract` | Đánh dấu version không nên sử dụng |

### Tham chiếu lệnh Go

Tài liệu cho công cụ Go.

**Các lệnh thường dùng:**
```bash
# Build và Run
go build            # Build package
go run main.go      # Build và chạy
go install          # Build và cài đặt

# Testing
go test             # Chạy tests
go test -v          # Verbose output
go test -cover      # Coverage report
go test -bench=.    # Chạy benchmarks
go test -race       # Race detector

# Dependencies
go get              # Thêm/cập nhật dependency
go mod tidy         # Dọn dẹp go.mod và go.sum
go mod download     # Tải dependencies

# Code quality
go fmt              # Format code
go vet              # Kiểm tra lỗi tiềm ẩn
go doc              # Xem documentation

# Tools
go generate         # Chạy code generators
go clean            # Xóa files được build
go env              # Xem environment variables
go version          # Xem version Go
go list             # List packages
```

### Tham chiếu Comment tài liệu

Viết các comment tài liệu cho godoc.

**Quy tắc viết comment:**
```go
// Package mypackage cung cấp các utility functions.
//
// Package này bao gồm các hàm hữu ích cho việc xử lý
// chuỗi và số trong ứng dụng Go.
package mypackage

// User đại diện cho một người dùng trong hệ thống.
//
// User chứa thông tin cơ bản như tên, email, và tuổi.
// Nó được sử dụng trong các tác vụ xác thực và
// quản lý người dùng.
type User struct {
    // Name là tên đầy đủ của người dùng.
    Name string
    
    // Email là địa chỉ email của người dùng.
    Email string
    
    // Age là tuổi của người dùng.
    Age int
}

// NewUser tạo một User mới với tên và email được cung cấp.
//
// Hàm này trả về một con trỏ đến User mới được tạo.
// Tuổi được khởi tạo bằng 0.
//
// Ví dụ:
//
//	user := NewUser("Việt", "viet@example.com")
//	fmt.Println(user.Name) // Output: Việt
func NewUser(name, email string) *User {
    return &User{Name: name, Email: email}
}
```

**Xem documentation:**
```bash
# Xem doc trong terminal
go doc fmt
go doc fmt.Println

# Chạy doc server
go doc -http=:6060
# Truy cập http://localhost:6060
```

### Một Tour về Go

Một tour tương tác về Go trong 4 phần. Phần đầu tiên bao gồm cú pháp cơ bản và cấu trúc dữ liệu; phần thứ hai thảo luận về các phương thức và interface; phần thứ ba giới thiệu về generics của Go; và phần thứ tư là về các nguyên thủy đồng thời của Go. Mỗi phần kết thúc với một vài bài tập để bạn có thể thực hành những gì bạn đã học. Bạn có thể tham gia tour trực tuyến hoặc cài đặt nó cục bộ với:

```bash
$ go install golang.org/x/website/tour@latest
```

Điều này sẽ đặt binary tour trong thư mục bin của GOPATH của bạn.

**Nội dung chính của Tour:**

1. **Basics (Cơ bản)**
   - Packages, imports, và exported names
   - Functions và multiple return values
   - Variables và short declarations
   - Basic types và type conversions
   - Constants và iota

2. **Flow control (Điều khiển luồng)**
   - For loops (Go chỉ có `for`, không có `while`)
   - If và switch statements
   - Defer, panic, và recover

3. **More types (Các kiểu khác)**
   - Pointers và structs
   - Arrays và slices
   - Maps và range
   - Function values và closures

4. **Methods và interfaces**
   - Methods với receivers
   - Interfaces và implicit implementation
   - Type assertions và type switches
   - Common interfaces (Stringer, error, Reader, Writer)

5. **Generics**
   - Type parameters
   - Type constraints
   - Generic types

6. **Concurrency (Đồng thời)**
   - Goroutines
   - Channels
   - Select statement
   - Mutexes

### Mô hình Bộ nhớ Go

Mô hình bộ nhớ Go xác định các điều kiện theo đó các hoạt động đọc một biến trong một goroutine có thể được đảm bảo quan sát các giá trị được tạo ra bởi các hoạt động ghi vào cùng một biến trong một goroutine khác.

**Các nguyên tắc chính:**

1. **Happens-before relationship**: Nếu event e1 happens-before event e2, thì e2 đảm bảo thấy được hiệu ứng của e1.

2. **Synchronization primitives:**
   ```go
   // Channel send/receive
   ch := make(chan int)
   go func() {
       x = 42     // Ghi
       ch <- 1    // Đồng bộ
   }()
   <-ch           // Đồng bộ
   fmt.Println(x) // Đọc - đảm bảo thấy 42
   
   // Mutex
   var mu sync.Mutex
   mu.Lock()
   x = 42
   mu.Unlock()
   
   mu.Lock()
   fmt.Println(x) // Đảm bảo thấy 42
   mu.Unlock()
   ```

3. **Atomic operations:**
   ```go
   import "sync/atomic"
   
   var counter int64
   atomic.AddInt64(&counter, 1)
   value := atomic.LoadInt64(&counter)
   ```

### Câu hỏi thường gặp (FAQ)

Câu trả lời cho các câu hỏi phổ biến về Go.

**Q: Go có hỗ trợ OOP không?**
A: Go không có classes, nhưng có structs với methods. Go sử dụng composition thay vì inheritance.

**Q: Tại sao Go không có generics từ đầu?**
A: Generics được thêm vào Go 1.18 (2022). Trước đó, team Go muốn đảm bảo thiết kế đơn giản và hiệu quả.

**Q: Go có garbage collection, có chậm không?**
A: Go GC được tối ưu hóa cho low latency. Trong Go 1.23, GC pause thường dưới 1ms.

**Q: Khi nào nên dùng pointer?**
A: Dùng pointer khi muốn modify giá trị gốc hoặc tránh copy data lớn.

**Q: Làm sao xử lý nil pointer?**
```go
if ptr != nil {
    // Sử dụng an toàn
    value := *ptr
}
```

**Q: Channel buffer nên dùng size bao nhiêu?**
A: Mặc định dùng unbuffered (0). Buffer chỉ dùng khi cần decouple sender/receiver.

---

## Tham chiếu

### Bản phát hành

Lịch sử phát hành Go.

**Các phiên bản quan trọng:**

| Phiên bản | Ngày phát hành | Tính năng nổi bật |
|-----------|----------------|-------------------|
| Go 1.0 | Tháng 3, 2012 | Phiên bản ổn định đầu tiên |
| Go 1.5 | Tháng 8, 2015 | Compiler viết bằng Go (self-hosting) |
| Go 1.11 | Tháng 8, 2018 | Go Modules (experimental) |
| Go 1.13 | Tháng 9, 2019 | Số với tiền tố (0b, 0o), error wrapping |
| Go 1.16 | Tháng 2, 2021 | embed package, io/fs |
| Go 1.18 | Tháng 3, 2022 | Generics, Fuzzing |
| Go 1.20 | Tháng 2, 2023 | PGO (Profile-Guided Optimization) |
| Go 1.21 | Tháng 8, 2023 | Built-in functions: min, max, clear |
| Go 1.22 | Tháng 2, 2024 | For-range over integers, enhanced routing |
| Go 1.23 | Tháng 8, 2024 | Iterators, timer changes |

**Chu kỳ phát hành:**
- Go phát hành phiên bản mới mỗi 6 tháng (tháng 2 và tháng 8)
- Mỗi phiên bản được hỗ trợ trong 1 năm
- Security fixes được backport cho 2 phiên bản gần nhất

**Kiểm tra và nâng cấp:**
```bash
# Kiểm tra version hiện tại
go version

# Cài đặt version mới (Linux/macOS)
go install golang.org/dl/go1.23.0@latest
go1.23.0 download
go1.23.0 version
```

### Cam kết Tương thích Go 1

Cam kết Go 1 về tính tương thích. Các chương trình Go viết theo đặc tả Go 1 sẽ tiếp tục biên dịch và chạy đúng, không thay đổi, trong suốt vòng đời của phiên bản 1 của Go.

**Những gì được đảm bảo:**
- ✅ API của standard library
- ✅ Cú pháp ngôn ngữ
- ✅ Đặc tả file go.mod

**Những gì KHÔNG được đảm bảo:**
- ❌ Hiệu suất cụ thể
- ❌ Thông báo lỗi cụ thể
- ❌ Behavior của bugs
- ❌ Internal packages

**Cách xử lý breaking changes:**
```go
// Sử dụng build constraints cho code khác nhau
//go:build go1.21

package mypackage

// Code chỉ build với Go 1.21+
```

### Go Wiki

Một wiki được duy trì bởi cộng đồng Go.

**Các trang wiki phổ biến:**
- [CodeReviewComments](https://go.dev/wiki/CodeReviewComments) - Các bình luận phổ biến trong code review
- [CommonMistakes](https://go.dev/wiki/CommonMistakes) - Các lỗi thường gặp
- [TableDrivenTests](https://go.dev/wiki/TableDrivenTests) - Hướng dẫn viết test
- [SliceTricks](https://go.dev/wiki/SliceTricks) - Các kỹ thuật với slice
- [GoIDEsEditors](https://go.dev/wiki/GoIDEsEditors) - IDE và editor hỗ trợ Go
- [Modules](https://go.dev/wiki/Modules) - Hướng dẫn chi tiết về modules

### Tài liệu Tham khảo về Môi trường

Các biến môi trường ảnh hưởng đến hành vi của các công cụ Go.

**Biến môi trường quan trọng:**

| Biến | Mô tả | Giá trị mặc định |
|------|-------|------------------|
| `GOPATH` | Workspace directory | `$HOME/go` |
| `GOROOT` | Go installation directory | Nơi cài đặt Go |
| `GOBIN` | Binary output directory | `$GOPATH/bin` |
| `GOPROXY` | Module proxy URL | `https://proxy.golang.org,direct` |
| `GOPRIVATE` | Private module patterns | (trống) |
| `GONOPROXY` | Modules không dùng proxy | (trống) |
| `GONOSUMDB` | Modules không verify checksum | (trống) |
| `GOOS` | Target operating system | Runtime OS |
| `GOARCH` | Target architecture | Runtime arch |
| `CGO_ENABLED` | Enable cgo | `1` (enabled) |
| `GOTOOLCHAIN` | Toolchain version | `local` |

**Xem và đặt biến môi trường:**
```bash
# Xem tất cả biến môi trường Go
go env

# Xem một biến cụ thể
go env GOPATH
go env GOPROXY

# Đặt biến môi trường
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOPRIVATE=github.com/mycompany/*

# Đặt biến môi trường tạm thời
GOOS=linux GOARCH=amd64 go build

# Cross-compilation
GOOS=windows GOARCH=amd64 go build -o app.exe
GOOS=darwin GOARCH=arm64 go build -o app-mac
GOOS=linux GOARCH=arm64 go build -o app-linux-arm
```

---

## Hỗ trợ biên tập viên và IDE

### Hỗ trợ plugin và IDE cho biên tập viên

Một tài liệu tóm tắt các plugin và IDE phổ biến được sử dụng với Go.

**IDE được đề xuất:**

| IDE/Editor | Plugin/Extension | Tính năng |
|------------|------------------|-----------|
| **VS Code** | Go (by Google) | Autocomplete, debugging, testing, refactoring |
| **GoLand** | Built-in | Full IDE với tất cả tính năng |
| **Vim/Neovim** | vim-go, nvim-lspconfig | Lightweight, powerful |
| **Emacs** | go-mode, lsp-mode | Customizable |
| **Sublime Text** | GoSublime, LSP | Fast, lightweight |

**Cài đặt VS Code với Go:**

1. Cài đặt VS Code
2. Cài đặt extension "Go" (by Google)
3. Mở Command Palette (Ctrl+Shift+P)
4. Chạy "Go: Install/Update Tools"
5. Chọn tất cả tools và cài đặt

**Các tools được cài đặt:**
```bash
gopls                 # Language server
dlv                   # Debugger
staticcheck           # Linter
goimports             # Import management
gomodifytags          # Struct tag management
impl                  # Interface implementation
goplay                # Playground
gotests               # Test generation
```

### Gopls

Gopls là Go language server cung cấp các tính năng như tự động hoàn thành, định dạng và chẩn đoán cho các IDE và biên tập văn bản.

**Tính năng của gopls:**
- ✅ Auto-completion (Tự động hoàn thành)
- ✅ Go to definition (Đi đến định nghĩa)
- ✅ Find references (Tìm tham chiếu)
- ✅ Rename (Đổi tên)
- ✅ Hover information (Thông tin khi di chuột)
- ✅ Code formatting (Định dạng code)
- ✅ Diagnostics (Chẩn đoán lỗi)
- ✅ Code actions (Hành động code)
- ✅ Inlay hints (Gợi ý inline)

**Cài đặt gopls:**
```bash
go install golang.org/x/tools/gopls@latest
```

**Cấu hình gopls (VS Code settings.json):**
```json
{
    "go.useLanguageServer": true,
    "gopls": {
        "formatting.gofumpt": true,
        "ui.completion.usePlaceholders": true,
        "ui.semanticTokens": true,
        "ui.diagnostic.analyses": {
            "unusedvariable": true,
            "shadow": true
        }
    }
}
```

**Cấu hình gopls (Neovim):**
```lua
require('lspconfig').gopls.setup{
    settings = {
        gopls = {
            analyses = {
                unusedparams = true,
            },
            staticcheck = true,
        },
    },
}
```

---

## Bảo mật

### Tổng quan về Bảo mật

Tổng quan về các tài nguyên bảo mật cho các nhà phát triển Go.

**Các khía cạnh bảo mật trong Go:**

1. **Ngôn ngữ an toàn theo mặc định**
   - Không có buffer overflow (trừ khi dùng unsafe)
   - Bounds checking cho arrays và slices
   - Garbage collection ngăn ngừa memory leaks

2. **Công cụ bảo mật tích hợp**
   - `go vet` - Phát hiện lỗi tiềm ẩn
   - `govulncheck` - Kiểm tra lỗ hổng dependencies
   - Race detector - Phát hiện data races

3. **Thực hành tốt nhất**
   - Luôn validate input từ người dùng
   - Sử dụng parameterized queries cho SQL
   - Không log thông tin nhạy cảm
   - Sử dụng HTTPS cho tất cả kết nối mạng

### Chính sách Bảo mật Go

Mô tả cách nhóm Go xử lý các lỗi bảo mật trong ngôn ngữ Go, thư viện, và các công cụ. Bao gồm hướng dẫn báo cáo các vấn đề bảo mật.

**Báo cáo lỗ hổng bảo mật:**
- Email: security@golang.org
- Sử dụng PGP key để mã hóa thông tin nhạy cảm
- Thời gian phản hồi: thường trong vòng 24 giờ

**Quy trình xử lý:**
1. Nhận báo cáo
2. Xác minh và đánh giá mức độ nghiêm trọng
3. Phát triển bản vá
4. Phát hành bản vá trong security release
5. Công bố CVE nếu cần

### Quản lý Lỗ hổng Bảo mật

Tổng quan về cách Go hỗ trợ phát hiện và giải quyết các lỗ hổng bảo mật trong các phụ thuộc của dự án Go.

**Công cụ quản lý lỗ hổng:**

```bash
# Cài đặt govulncheck
go install golang.org/x/vuln/cmd/govulncheck@latest

# Kiểm tra project
govulncheck ./...

# Kiểm tra với verbose mode
govulncheck -show=verbose ./...

# Kiểm tra binary
govulncheck -mode=binary ./myapp

# Xem format JSON
govulncheck -json ./...
```

**Ví dụ output:**
```
Scanning your code and 50 packages across 10 dependent modules for known vulnerabilities...

Vulnerability #1: GO-2023-1234
    A vulnerability in example.com/package allows...
    
    More info: https://pkg.go.dev/vuln/GO-2023-1234
    
    Module: example.com/package
    Found in: example.com/package@v1.0.0
    Fixed in: example.com/package@v1.0.1
    
    Example call stacks in your code:
    main.go:42: mypackage.Function calls example.com/package.VulnerableFunc
```

### Cơ sở dữ liệu Lỗ hổng Go

Cơ sở dữ liệu lỗ hổng Go là nguồn tổng hợp thông tin về các lỗ hổng bảo mật đã biết trong các module Go công khai.

**Truy cập database:**
- Web: [vuln.go.dev](https://vuln.go.dev)
- API: `https://vuln.go.dev/api`

**Cấu trúc vulnerability ID:**
- Format: `GO-YYYY-NNNN`
- Ví dụ: `GO-2023-2102`

**Tích hợp vào CI/CD:**
```yaml
# GitHub Actions
- name: Run govulncheck
  run: |
    go install golang.org/x/vuln/cmd/govulncheck@latest
    govulncheck ./...
```

### govulncheck

Govulncheck là công cụ dòng lệnh báo cáo các lỗ hổng bảo mật đã biết ảnh hưởng đến các package Go trong dự án của bạn. Nó phân tích cơ sở mã để chỉ để lộ các lỗ hổng thực sự ảnh hưởng đến mã của bạn.

**Ưu điểm của govulncheck:**
- Chỉ báo cáo lỗ hổng thực sự được sử dụng
- Phân tích call graph để giảm false positives
- Tích hợp với Go vulnerability database

**Sử dụng trong editor:**
- VS Code: Tự động chạy qua gopls
- GoLand: Tích hợp sẵn

### Các Thực hành Tốt nhất về Bảo mật

Hướng dẫn về các thực hành tốt nhất để viết mã Go an toàn, bao gồm xác thực đầu vào, xử lý lỗi, và sử dụng các package mật mã.

**1. Xác thực đầu vào:**
```go
import (
    "regexp"
    "strings"
    "unicode/utf8"
)

func validateEmail(email string) bool {
    // Kiểm tra độ dài
    if len(email) > 254 {
        return false
    }
    // Kiểm tra format
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    return emailRegex.MatchString(email)
}

func sanitizeInput(input string) string {
    // Loại bỏ ký tự nguy hiểm
    return strings.Map(func(r rune) rune {
        if r == '<' || r == '>' || r == '&' {
            return -1
        }
        return r
    }, input)
}
```

**2. SQL Injection Prevention:**
```go
// ❌ KHÔNG AN TOÀN - Dễ bị SQL injection
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", name)
db.Query(query)

// ✅ AN TOÀN - Sử dụng parameterized query
db.Query("SELECT * FROM users WHERE name = ?", name)

// ✅ AN TOÀN - Prepared statement
stmt, _ := db.Prepare("SELECT * FROM users WHERE name = ?")
stmt.Query(name)
```

**3. Mật mã an toàn:**
```go
import (
    "crypto/rand"
    "crypto/sha256"
    "encoding/hex"
    "golang.org/x/crypto/bcrypt"
)

// Hash password với bcrypt
func hashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

// Verify password
func checkPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}

// Tạo token ngẫu nhiên
func generateToken(length int) (string, error) {
    bytes := make([]byte, length)
    if _, err := rand.Read(bytes); err != nil {
        return "", err
    }
    return hex.EncodeToString(bytes), nil
}

// ❌ KHÔNG dùng crypto/md5 hoặc crypto/sha1 cho password
// ❌ KHÔNG dùng math/rand cho security
```

**4. Xử lý secrets:**
```go
import "os"

// ✅ Đọc secrets từ environment variables
apiKey := os.Getenv("API_KEY")
if apiKey == "" {
    log.Fatal("API_KEY environment variable is required")
}

// ❌ KHÔNG hardcode secrets trong code
// ❌ KHÔNG log secrets
```

**5. HTTPS và TLS:**
```go
import (
    "crypto/tls"
    "net/http"
)

// HTTP client với TLS cấu hình an toàn
client := &http.Client{
    Transport: &http.Transport{
        TLSClientConfig: &tls.Config{
            MinVersion: tls.VersionTLS12,
        },
    },
}

// HTTPS server
http.ListenAndServeTLS(":443", "cert.pem", "key.pem", nil)
```

---

## Công cụ chẩn đoán

### Tổng quan về công cụ chẩn đoán

Một tổng quan về các công cụ và phương pháp chẩn đoán các vấn đề trong chương trình Go.

**Các công cụ chẩn đoán chính:**

| Công cụ | Mục đích |
|---------|----------|
| `go vet` | Phát hiện lỗi code |
| `race detector` | Phát hiện data races |
| `pprof` | CPU và memory profiling |
| `trace` | Execution tracing |
| `deadlock detector` | Phát hiện deadlocks |
| `staticcheck` | Static analysis |

**Ví dụ sử dụng:**
```bash
# Kiểm tra lỗi tiềm ẩn
go vet ./...

# Chạy với race detector
go run -race main.go
go test -race ./...

# Profiling
go test -cpuprofile=cpu.prof -memprofile=mem.prof -bench=.

# Xem profile
go tool pprof cpu.prof
```

### Hướng dẫn Garbage Collector

Một hướng dẫn về cách Go quản lý bộ nhớ và cách tận dụng tối đa nó.

**Cách GC hoạt động:**
- Go sử dụng concurrent, tri-color mark-and-sweep GC
- GC pause thường dưới 1ms trong Go hiện đại
- GC tự động điều chỉnh dựa trên heap size

**Cấu hình GC:**
```go
import "runtime"

// Đặt GC percentage (mặc định 100)
// GOGC=50 sẽ trigger GC thường xuyên hơn
debug.SetGCPercent(50)

// Đặt memory limit
debug.SetMemoryLimit(1 << 30) // 1 GB

// Force GC
runtime.GC()

// Xem thống kê GC
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("Alloc: %d MB\n", m.Alloc/1024/1024)
fmt.Printf("TotalAlloc: %d MB\n", m.TotalAlloc/1024/1024)
fmt.Printf("Sys: %d MB\n", m.Sys/1024/1024)
fmt.Printf("NumGC: %d\n", m.NumGC)
```

**Biến môi trường GC:**
```bash
# Đặt GC target percentage
GOGC=100 ./myapp  # Mặc định

# Tắt GC (cẩn thận!)
GOGC=off ./myapp

# Memory limit
GOMEMLIMIT=1GiB ./myapp

# Debug GC
GODEBUG=gctrace=1 ./myapp
```

**Tối ưu bộ nhớ:**
```go
// Tái sử dụng buffers với sync.Pool
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 4096)
    },
}

func process() {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    // Sử dụng buf...
}

// Pre-allocate slices
data := make([]int, 0, 1000) // capacity = 1000

// Tránh string concatenation trong loop
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString("text")
}
result := builder.String()
```

### Hướng dẫn thu thập Profile

Một hướng dẫn về cách sử dụng profile hướng dẫn tối ưu hóa (PGO) trong Go.

**Profile-Guided Optimization (PGO):**

PGO là tính năng từ Go 1.20+ cho phép compiler tối ưu hóa dựa trên dữ liệu runtime thực tế.

**Quy trình sử dụng PGO:**

1. **Build và chạy để thu thập profile:**
```go
import (
    "os"
    "runtime/pprof"
)

func main() {
    // Thu thập CPU profile
    f, _ := os.Create("cpu.pprof")
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // Chạy ứng dụng...
}
```

2. **Build với profile:**
```bash
# Đổi tên profile thành default.pgo
mv cpu.pprof default.pgo

# Build với PGO (tự động phát hiện default.pgo)
go build -pgo=auto

# Hoặc chỉ định file profile
go build -pgo=cpu.pprof
```

**Thu thập profile từ HTTP server:**
```go
import (
    _ "net/http/pprof"
    "net/http"
)

func main() {
    // Endpoint pprof sẽ được thêm tự động
    http.ListenAndServe(":6060", nil)
}
```

```bash
# Thu thập profile
curl http://localhost:6060/debug/pprof/profile?seconds=30 > cpu.pprof

# Xem profile trong trình duyệt
go tool pprof -http=:8080 cpu.pprof
```

**Các loại profile:**
```bash
# CPU profile - xác định code chậm
go tool pprof http://localhost:6060/debug/pprof/profile

# Heap profile - xác định memory usage
go tool pprof http://localhost:6060/debug/pprof/heap

# Goroutine profile - xác định goroutine leaks
go tool pprof http://localhost:6060/debug/pprof/goroutine

# Block profile - xác định blocking operations
go tool pprof http://localhost:6060/debug/pprof/block

# Mutex profile - xác định mutex contention
go tool pprof http://localhost:6060/debug/pprof/mutex
```

**Phân tích profile với pprof:**
```bash
# Interactive mode
go tool pprof cpu.pprof
(pprof) top          # Top functions by CPU
(pprof) top -cum     # Top functions by cumulative time
(pprof) list funcName # Source code view
(pprof) web          # Generate graph (cần graphviz)

# Web UI mode
go tool pprof -http=:8080 cpu.pprof
```

**Benchmark với profiling:**
```go
func BenchmarkProcess(b *testing.B) {
    for i := 0; i < b.N; i++ {
        process()
    }
}
```

```bash
# Chạy benchmark với profile
go test -bench=. -cpuprofile=cpu.prof
go test -bench=. -memprofile=mem.prof
go test -bench=. -blockprofile=block.prof

# So sánh benchmarks
go test -bench=. -count=5 > old.txt
# Thay đổi code...
go test -bench=. -count=5 > new.txt
benchstat old.txt new.txt
```

---

## Các bài viết

### Wiki Go

Wiki Go, được duy trì bởi cộng đồng Go, bao gồm các bài viết về ngôn ngữ Go, công cụ và các chủ đề khác.

**Các bài viết Wiki quan trọng:**

| Chủ đề | Mô tả |
|--------|-------|
| [SliceTricks](https://go.dev/wiki/SliceTricks) | Các thao tác với slice |
| [TableDrivenTests](https://go.dev/wiki/TableDrivenTests) | Viết tests hiệu quả |
| [CodeReviewComments](https://go.dev/wiki/CodeReviewComments) | Best practices cho code review |
| [CommonMistakes](https://go.dev/wiki/CommonMistakes) | Các lỗi thường gặp |
| [MethodSets](https://go.dev/wiki/MethodSets) | Hiểu về method sets |
| [Range](https://go.dev/wiki/Range) | Chi tiết về range clause |
| [LockOSThread](https://go.dev/wiki/LockOSThread) | Thread locking |
| [LearnConcurrency](https://go.dev/wiki/LearnConcurrency) | Học về concurrency |
| [Modules](https://go.dev/wiki/Modules) | Go modules chi tiết |
| [ErrorValueFAQ](https://go.dev/wiki/ErrorValueFAQ) | FAQ về error handling |

**Ví dụ từ SliceTricks:**
```go
// Xóa phần tử tại index i
a = append(a[:i], a[i+1:]...)

// Insert phần tử tại index i
a = append(a[:i], append([]T{x}, a[i:]...)...)

// Pop phần tử cuối
x, a = a[len(a)-1], a[:len(a)-1]

// Đảo ngược slice
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Tài liệu không phải tiếng Anh

Xem trang Tài liệu không phải tiếng Anh tại Wiki Go để biết các bản địa hóa của tài liệu Go.

**Tài nguyên tiếng Việt:**
- [Go tiếng Việt](https://go.dev/wiki/NonEnglish#vietnamese) - Tài liệu tiếng Việt
- Diễn đàn Golang Vietnam
- Facebook group: Golang Vietnam

**Các ngôn ngữ khác:**
- Tiếng Trung: [Go 语言中文网](https://studygolang.com/)
- Tiếng Nhật: [Go 言語仕様書](https://go.dev/ref/spec)
- Tiếng Hàn: [한국어 Go 문서](https://github.com/golang-kr)
- Tiếng Tây Ban Nha: [Go en Español](https://go.dev/wiki/NonEnglish#spanish)

---

## Truy cập Cơ sở Dữ liệu

### Tổng quan về Truy cập Cơ sở Dữ liệu

Tổng quan về cách truy cập cơ sở dữ liệu từ Go.

**Package database/sql:**
- Interface chung cho các SQL databases
- Connection pooling tự động
- Prepared statements
- Transaction support

**Database drivers phổ biến:**

| Database | Driver |
|----------|--------|
| MySQL | `github.com/go-sql-driver/mysql` |
| PostgreSQL | `github.com/lib/pq` hoặc `github.com/jackc/pgx` |
| SQLite | `github.com/mattn/go-sqlite3` |
| SQL Server | `github.com/denisenkom/go-mssqldb` |
| Oracle | `github.com/godror/godror` |

### Mở một xử lý cơ sở dữ liệu

Cách kết nối với cơ sở dữ liệu sử dụng package database/sql trong Go.

```go
import (
    "database/sql"
    "log"
    
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    // Connection string format: user:password@tcp(host:port)/dbname
    dsn := "user:password@tcp(localhost:3306)/mydb?parseTime=true"
    
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Cấu hình connection pool
    db.SetMaxOpenConns(25)           // Max connections
    db.SetMaxIdleConns(5)            // Max idle connections
    db.SetConnMaxLifetime(5 * time.Minute) // Max lifetime
    
    // Kiểm tra kết nối
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
    
    log.Println("Kết nối thành công!")
}
```

**PostgreSQL example:**
```go
import (
    "database/sql"
    _ "github.com/lib/pq"
)

dsn := "host=localhost port=5432 user=myuser password=mypass dbname=mydb sslmode=disable"
db, err := sql.Open("postgres", dsn)
```

### Thực thi các câu lệnh SQL không trả về dữ liệu

Cách thực thi các câu lệnh INSERT, UPDATE, DELETE, và các câu lệnh SQL khác không trả về dữ liệu.

```go
// INSERT
result, err := db.Exec(
    "INSERT INTO users (name, email) VALUES (?, ?)",
    "Việt", "viet@example.com",
)
if err != nil {
    log.Fatal(err)
}

// Lấy ID của row vừa insert
id, err := result.LastInsertId()
fmt.Printf("Inserted user with ID: %d\n", id)

// UPDATE
result, err = db.Exec(
    "UPDATE users SET name = ? WHERE id = ?",
    "Việt Nam", 1,
)
rowsAffected, _ := result.RowsAffected()
fmt.Printf("Updated %d rows\n", rowsAffected)

// DELETE
result, err = db.Exec("DELETE FROM users WHERE id = ?", 1)
```

### Truy vấn dữ liệu

Cách truy vấn dữ liệu từ cơ sở dữ liệu trong Go.

```go
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

// Query một row
func getUserByID(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow(
        "SELECT id, name, email, created_at FROM users WHERE id = ?", id,
    ).Scan(&u.ID, &u.Name, &u.Email, &u.CreatedAt)
    
    if err == sql.ErrNoRows {
        return nil, fmt.Errorf("user %d không tồn tại", id)
    }
    if err != nil {
        return nil, err
    }
    return &u, nil
}

// Query nhiều rows
func getAllUsers(db *sql.DB) ([]User, error) {
    rows, err := db.Query("SELECT id, name, email, created_at FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []User
    for rows.Next() {
        var u User
        err := rows.Scan(&u.ID, &u.Name, &u.Email, &u.CreatedAt)
        if err != nil {
            return nil, err
        }
        users = append(users, u)
    }
    
    // Kiểm tra lỗi sau khi loop
    if err := rows.Err(); err != nil {
        return nil, err
    }
    
    return users, nil
}
```

### Sử dụng prepared statements

Cách sử dụng prepared statements để thực thi các câu lệnh SQL hiệu quả và an toàn hơn.

```go
// Tạo prepared statement
stmt, err := db.Prepare("SELECT id, name FROM users WHERE email = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

// Sử dụng nhiều lần
var user1 User
stmt.QueryRow("user1@example.com").Scan(&user1.ID, &user1.Name)

var user2 User
stmt.QueryRow("user2@example.com").Scan(&user2.ID, &user2.Name)

// Prepared statement cho INSERT
insertStmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
if err != nil {
    log.Fatal(err)
}
defer insertStmt.Close()

// Batch insert
users := []struct{ Name, Email string }{
    {"User 1", "user1@example.com"},
    {"User 2", "user2@example.com"},
    {"User 3", "user3@example.com"},
}

for _, u := range users {
    _, err := insertStmt.Exec(u.Name, u.Email)
    if err != nil {
        log.Printf("Error inserting %s: %v", u.Name, err)
    }
}
```

### Thực thi giao dịch

Cách thực thi các giao dịch cơ sở dữ liệu để đảm bảo tính toàn vẹn dữ liệu.

```go
func transferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    // Bắt đầu transaction
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    
    // Đảm bảo rollback nếu có lỗi
    defer func() {
        if err != nil {
            tx.Rollback()
        }
    }()
    
    // Trừ tiền từ tài khoản nguồn
    _, err = tx.Exec(
        "UPDATE accounts SET balance = balance - ? WHERE id = ?",
        amount, fromID,
    )
    if err != nil {
        return err
    }
    
    // Thêm tiền vào tài khoản đích
    _, err = tx.Exec(
        "UPDATE accounts SET balance = balance + ? WHERE id = ?",
        amount, toID,
    )
    if err != nil {
        return err
    }
    
    // Commit transaction
    return tx.Commit()
}

// Với context và timeout
func transferMoneyWithContext(ctx context.Context, db *sql.DB, fromID, toID int, amount float64) error {
    tx, err := db.BeginTx(ctx, &sql.TxOptions{
        Isolation: sql.LevelSerializable,
    })
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // ... thực hiện các operations ...
    
    return tx.Commit()
}
```

### Xử lý lỗi

Các thực hành tốt nhất cho việc xử lý lỗi khi làm việc với cơ sở dữ liệu trong Go.

```go
import (
    "database/sql"
    "errors"
    "github.com/go-sql-driver/mysql"
)

// Kiểm tra row không tồn tại
user, err := getUserByID(db, 999)
if errors.Is(err, sql.ErrNoRows) {
    log.Println("User không tồn tại")
}

// Kiểm tra connection đóng
if errors.Is(err, sql.ErrConnDone) {
    log.Println("Connection đã đóng")
}

// Kiểm tra lỗi MySQL cụ thể
var mysqlErr *mysql.MySQLError
if errors.As(err, &mysqlErr) {
    switch mysqlErr.Number {
    case 1062: // Duplicate entry
        log.Println("Dữ liệu đã tồn tại")
    case 1451: // Foreign key constraint
        log.Println("Không thể xóa do có dữ liệu liên quan")
    default:
        log.Printf("MySQL error: %d - %s", mysqlErr.Number, mysqlErr.Message)
    }
}

// Wrap errors với context
func getUser(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow("SELECT * FROM users WHERE id = ?", id).Scan(&u.ID, &u.Name)
    if err != nil {
        return nil, fmt.Errorf("getUser %d: %w", id, err)
    }
    return &u, nil
}
```

### Tránh các vấn đề SQL injection

Cách viết các truy vấn SQL an toàn để tránh các lỗ hổng SQL injection.

```go
// ❌ KHÔNG BAO GIỜ làm điều này - SQL Injection vulnerability
userInput := "'; DROP TABLE users; --"
query := "SELECT * FROM users WHERE name = '" + userInput + "'"
db.Query(query) // NGUY HIỂM!

// ✅ Luôn sử dụng parameterized queries
db.Query("SELECT * FROM users WHERE name = ?", userInput) // AN TOÀN

// ✅ Prepared statements
stmt, _ := db.Prepare("SELECT * FROM users WHERE name = ?")
stmt.Query(userInput) // AN TOÀN

// ✅ Với multiple parameters
db.Query(
    "SELECT * FROM users WHERE name = ? AND age > ? AND status = ?",
    name, age, status,
) // AN TOÀN

// ⚠️ Cẩn thận với ORDER BY và LIMIT
// Không thể parameterize column names hoặc SQL keywords
orderColumn := "name" // Validate this!
validColumns := map[string]bool{"name": true, "email": true, "created_at": true}
if !validColumns[orderColumn] {
    orderColumn = "id" // Default safe value
}
query := fmt.Sprintf("SELECT * FROM users ORDER BY %s LIMIT ?", orderColumn)
db.Query(query, limit)
```

### Quản lý kết nối

Cách quản lý và tối ưu hóa connection pool trong ứng dụng Go.

```go
import (
    "database/sql"
    "time"
)

func setupDB() *sql.DB {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatal(err)
    }
    
    // Số lượng connection tối đa
    // Phụ thuộc vào resources của database server
    db.SetMaxOpenConns(25)
    
    // Số lượng idle connections
    // Giúp giảm overhead tạo connection mới
    db.SetMaxIdleConns(10)
    
    // Thời gian sống tối đa của connection
    // Ngăn chặn connection stale
    db.SetConnMaxLifetime(5 * time.Minute)
    
    // Thời gian idle tối đa
    db.SetConnMaxIdleTime(1 * time.Minute)
    
    return db
}

// Monitoring connection pool
func monitorDBPool(db *sql.DB) {
    stats := db.Stats()
    log.Printf("Open connections: %d", stats.OpenConnections)
    log.Printf("In use: %d", stats.InUse)
    log.Printf("Idle: %d", stats.Idle)
    log.Printf("Wait count: %d", stats.WaitCount)
    log.Printf("Wait duration: %v", stats.WaitDuration)
}

// Health check
func healthCheck(db *sql.DB) error {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    return db.PingContext(ctx)
}
```

---

## Cgo

### Cgo

Liên kết mã Go với các package C bên ngoài.

**Cgo là gì?**
Cgo cho phép Go packages gọi code C. Điều này hữu ích khi:
- Sử dụng thư viện C hiện có
- Truy cập APIs hệ điều hành không có trong Go
- Tích hợp với code C legacy

**Ví dụ cơ bản:**
```go
package main

/*
#include <stdio.h>
#include <stdlib.h>

void greet(const char* name) {
    printf("Xin chào, %s!\n", name);
}
*/
import "C"

import (
    "unsafe"
)

func main() {
    name := C.CString("Việt")
    defer C.free(unsafe.Pointer(name))
    C.greet(name)
}
```

**Sử dụng thư viện C:**
```go
package main

/*
#cgo CFLAGS: -I/usr/local/include
#cgo LDFLAGS: -L/usr/local/lib -lmylib
#include <mylib.h>
*/
import "C"

func main() {
    result := C.my_function(10)
    println(result)
}
```

**Chuyển đổi kiểu dữ liệu:**
```go
// Go string -> C string
goStr := "Hello"
cStr := C.CString(goStr)
defer C.free(unsafe.Pointer(cStr))

// C string -> Go string
cStr2 := C.some_c_function()
goStr2 := C.GoString(cStr2)

// Số
var goInt int = 42
cInt := C.int(goInt)
goInt2 := int(C.some_int_function())

// Byte slice
goBytes := []byte{1, 2, 3, 4}
cPtr := (*C.char)(unsafe.Pointer(&goBytes[0]))
cLen := C.int(len(goBytes))
```

**Các cờ build quan trọng:**
```go
/*
// Đường dẫn include
#cgo CFLAGS: -I./include

// Đường dẫn thư viện
#cgo LDFLAGS: -L./lib -lmylib

// Flags theo platform
#cgo linux LDFLAGS: -lpthread
#cgo darwin LDFLAGS: -framework CoreFoundation

// Pkg-config
#cgo pkg-config: libpng
*/
import "C"
```

**Lưu ý khi dùng cgo:**
- ⚠️ Tăng thời gian build
- ⚠️ Không cross-compile dễ dàng
- ⚠️ Memory management phức tạp hơn
- ⚠️ CGO_ENABLED=1 là bắt buộc

```bash
# Build với cgo
CGO_ENABLED=1 go build

# Tắt cgo
CGO_ENABLED=0 go build
```

### Gọi Go từ C

Cách gọi hàm Go từ mã C.

**Export Go function sang C:**
```go
package main

import "C"

//export Add
func Add(a, b C.int) C.int {
    return a + b
}

//export Greet
func Greet(name *C.char) *C.char {
    goName := C.GoString(name)
    greeting := "Xin chào, " + goName + "!"
    return C.CString(greeting) // Caller phải free
}

// main() bắt buộc nhưng có thể trống
func main() {}
```

**Build thành shared library:**
```bash
# Build thành .so (Linux) / .dylib (macOS) / .dll (Windows)
go build -buildmode=c-shared -o libmylib.so mylib.go
```

**Sử dụng từ C:**
```c
#include "libmylib.h"
#include <stdio.h>

int main() {
    // Gọi hàm Go
    int result = Add(5, 3);
    printf("5 + 3 = %d\n", result);
    
    // Gọi hàm với string
    char* greeting = Greet("Việt");
    printf("%s\n", greeting);
    free(greeting);  // Phải free string từ Go
    
    return 0;
}
```

**Compile và link:**
```bash
gcc -o myapp main.c -L. -lmylib -Wl,-rpath,.
```

**Plugin mode:**
```go
// plugin.go
package main

import "C"

//export ProcessData
func ProcessData(data *C.char) *C.char {
    input := C.GoString(data)
    output := processInGo(input)
    return C.CString(output)
}

func processInGo(s string) string {
    return "Processed: " + s
}

func main() {}
```

```bash
go build -buildmode=c-shared -o plugin.so plugin.go
```

---

## Các chủ đề nâng cao

### Bảo hiểm Code

Mô tả cách sử dụng công cụ bảo hiểm code được tích hợp với go test.

**Chạy tests với coverage:**
```bash
# Coverage cơ bản
go test -cover ./...

# Output coverage ra file
go test -coverprofile=coverage.out ./...

# Xem coverage theo function
go tool cover -func=coverage.out

# Xem coverage trực quan trong browser
go tool cover -html=coverage.out

# Coverage cho tất cả packages (bao gồm packages không có tests)
go test -coverprofile=coverage.out -coverpkg=./... ./...
```

**Cấu hình coverage mode:**
```bash
# set: Mỗi statement được đánh dấu 0 hoặc 1
go test -covermode=set -coverprofile=coverage.out

# count: Đếm số lần mỗi statement được thực thi
go test -covermode=count -coverprofile=coverage.out

# atomic: Như count nhưng thread-safe (dùng với -race)
go test -covermode=atomic -coverprofile=coverage.out
```

**Ví dụ output:**
```
ok      mypackage    0.015s    coverage: 85.4% of statements
```

**Tích hợp CI/CD:**
```yaml
# GitHub Actions
- name: Run tests with coverage
  run: |
    go test -coverprofile=coverage.out -covermode=atomic ./...
    go tool cover -func=coverage.out | grep total | awk '{print $3}'
```

### Tham chiếu Format Testdata

Tham chiếu cho syntax của file "txtar" testdata.

**Txtar format:**
```
-- go.mod --
module example.com/mymod

go 1.23
-- main.go --
package main

func main() {
    println("Hello")
}
-- main_test.go --
package main

import "testing"

func TestMain(t *testing.T) {
    // test code
}
```

**Sử dụng trong tests:**
```go
import (
    "testing"
    "golang.org/x/tools/txtar"
)

func TestSomething(t *testing.T) {
    ar, err := txtar.ParseFile("testdata/example.txtar")
    if err != nil {
        t.Fatal(err)
    }
    
    for _, f := range ar.Files {
        t.Logf("File: %s, Content: %s", f.Name, f.Data)
    }
}
```

### Cấu hình Fuzz

Tham chiếu cho các cấu hình có thể đọc và giải quyết được bằng công cụ go.

**Cấu trúc fuzz test:**
```go
func FuzzMyFunc(f *testing.F) {
    // Thêm seed corpus
    f.Add("initial input")
    f.Add("another input")
    
    // Fuzz function
    f.Fuzz(func(t *testing.T, input string) {
        // Test logic
        result := MyFunc(input)
        if !isValid(result) {
            t.Errorf("Invalid result for input %q: %v", input, result)
        }
    })
}
```

**Chạy fuzz tests:**
```bash
# Chạy fuzz test
go test -fuzz=FuzzMyFunc

# Với thời gian giới hạn
go test -fuzz=FuzzMyFunc -fuzztime=30s

# Với số worker cụ thể
go test -fuzz=FuzzMyFunc -parallel=4

# Chạy với cache corpus cụ thể
go test -fuzz=FuzzMyFunc -fuzztime=1m -test.fuzzcachedir=./corpus
```

**Corpus directory structure:**
```
testdata/
└── fuzz/
    └── FuzzMyFunc/
        ├── corpus1
        ├── corpus2
        └── corpus3
```

### Quản lý phụ thuộc

Khi mã của bạn sử dụng các package bên ngoài, các package đó (được phân phối dưới dạng module) trở thành các phụ thuộc.

**Thêm dependencies:**
```bash
# Thêm dependency mới
go get github.com/gin-gonic/gin

# Thêm version cụ thể
go get github.com/gin-gonic/gin@v1.9.0

# Thêm từ branch
go get github.com/user/repo@branch-name

# Thêm từ commit
go get github.com/user/repo@abc1234
```

**Quản lý versions:**
```bash
# Cập nhật dependency
go get -u github.com/gin-gonic/gin

# Cập nhật tất cả
go get -u ./...

# Cập nhật patch versions only
go get -u=patch ./...

# Downgrade
go get github.com/gin-gonic/gin@v1.8.0
```

**Xem và phân tích dependencies:**
```bash
# List tất cả dependencies
go list -m all

# Xem dependency tree
go mod graph

# Tìm lý do require
go mod why github.com/some/package

# Xem versions có sẵn
go list -m -versions github.com/gin-gonic/gin
```

**Vendor dependencies:**
```bash
# Tạo vendor directory
go mod vendor

# Build với vendor
go build -mod=vendor

# Verify vendor
go mod verify
```

### Đánh giá hiệu suất

Mô tả cách viết và đọc benchmark tests trong Go.

**Viết benchmark:**
```go
func BenchmarkConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = "Hello" + " " + "World"
    }
}

func BenchmarkBuilder(b *testing.B) {
    for i := 0; i < b.N; i++ {
        var builder strings.Builder
        builder.WriteString("Hello")
        builder.WriteString(" ")
        builder.WriteString("World")
        _ = builder.String()
    }
}

// Benchmark với sub-benchmarks
func BenchmarkSort(b *testing.B) {
    sizes := []int{10, 100, 1000, 10000}
    for _, size := range sizes {
        b.Run(fmt.Sprintf("size-%d", size), func(b *testing.B) {
            data := generateData(size)
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                sort.Ints(data)
            }
        })
    }
}

// Benchmark memory allocations
func BenchmarkAllocs(b *testing.B) {
    b.ReportAllocs()
    for i := 0; i < b.N; i++ {
        _ = make([]byte, 1024)
    }
}
```

**Chạy benchmarks:**
```bash
# Chạy tất cả benchmarks
go test -bench=.

# Chạy benchmark cụ thể
go test -bench=BenchmarkConcat

# Với memory stats
go test -bench=. -benchmem

# Chạy nhiều lần để có kết quả ổn định
go test -bench=. -count=5

# Với CPU profile
go test -bench=. -cpuprofile=cpu.prof

# Với memory profile
go test -bench=. -memprofile=mem.prof
```

**Đọc kết quả:**
```
BenchmarkConcat-8        50000000    23.5 ns/op    16 B/op    1 allocs/op
BenchmarkBuilder-8       30000000    45.2 ns/op    64 B/op    2 allocs/op
```

**So sánh benchmarks:**
```bash
# Cài đặt benchstat
go install golang.org/x/perf/cmd/benchstat@latest

# So sánh
go test -bench=. -count=5 > old.txt
# Thay đổi code...
go test -bench=. -count=5 > new.txt
benchstat old.txt new.txt
```

### Fuzz Testing

Mô tả cách sử dụng fuzzer bản địa của Go.

```go
import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    // Seed corpus
    testcases := []string{"Hello", "世界", "", "!@#$%"}
    for _, tc := range testcases {
        f.Add(tc)
    }
    
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        
        // Property 1: Reverse hai lần phải bằng ban đầu
        if orig != doubleRev {
            t.Errorf("Reverse x2 failed: %q -> %q -> %q", orig, rev, doubleRev)
        }
        
        // Property 2: Độ dài phải giữ nguyên
        if len(rev) != len(orig) {
            t.Errorf("Length mismatch: %d vs %d", len(rev), len(orig))
        }
        
        // Property 3: UTF-8 validity
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Invalid UTF-8: %q", rev)
        }
    })
}
```

**Các kiểu input được hỗ trợ:**
- `string`, `[]byte`
- `int`, `int8`, `int16`, `int32`, `int64`
- `uint`, `uint8`, `uint16`, `uint32`, `uint64`
- `float32`, `float64`
- `bool`

### Assembly

Mô tả cách sử dụng hợp ngữ Go để thực hiện các tác vụ cấp thấp.

**Ví dụ assembly đơn giản:**

`add_amd64.s`:
```asm
// func Add(a, b int64) int64
TEXT ·Add(SB), NOSPLIT, $0-24
    MOVQ a+0(FP), AX
    ADDQ b+8(FP), AX
    MOVQ AX, ret+16(FP)
    RET
```

`add.go`:
```go
package math

func Add(a, b int64) int64
```

**Build với assembly:**
```bash
go build
```

**Khi nào dùng assembly:**
- Crypto operations yêu cầu constant-time
- SIMD instructions (SSE, AVX)
- Atomic operations đặc biệt
- Hardware-specific optimizations

### Race Detector

Mô tả cách sử dụng công cụ phát hiện data race.

**Sử dụng race detector:**
```bash
# Chạy với race detector
go run -race main.go

# Test với race detector
go test -race ./...

# Build với race detector
go build -race
```

**Ví dụ data race:**
```go
func main() {
    counter := 0
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            counter++ // DATA RACE!
            wg.Done()
        }()
    }
    wg.Wait()
    fmt.Println(counter)
}
```

**Sửa data race:**
```go
// Cách 1: Mutex
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}

// Cách 2: Atomic
var counter int64

func increment() {
    atomic.AddInt64(&counter, 1)
}

// Cách 3: Channel
func main() {
    counter := make(chan int, 1)
    counter <- 0
    
    go func() {
        val := <-counter
        val++
        counter <- val
    }()
}
```

### Làm việc với các đường dẫn file

Mô tả các vấn đề mà các đường dẫn file có thể gây ra và cách làm việc với chúng một cách an toàn trong Go.

```go
import (
    "path/filepath"
    "os"
)

// Dùng filepath cho cross-platform paths
path := filepath.Join("dir", "subdir", "file.txt")

// Lấy thư mục từ path
dir := filepath.Dir(path)

// Lấy tên file
base := filepath.Base(path)

// Lấy extension
ext := filepath.Ext(path)

// Absolute path
absPath, _ := filepath.Abs("./relative/path")

// Clean path (loại bỏ . và ..)
cleanPath := filepath.Clean("./dir/../other/./file.txt")

// Walk directory
filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
    if err != nil {
        return err
    }
    fmt.Println(path)
    return nil
})

// Match pattern
matched, _ := filepath.Match("*.go", "main.go")
```

### Chế độ FIPS 140

Mô tả cách kích hoạt chế độ tuân thủ FIPS 140 trong Go.

**FIPS 140 trong Go 1.24+:**
```bash
# Build với FIPS mode
GOFIPS=1 go build

# Kiểm tra FIPS mode
go run -ldflags="-X crypto/internal/fips.enabled=1" main.go
```

**Sử dụng crypto packages:**
```go
import (
    "crypto/aes"
    "crypto/sha256"
)

// Các operations tự động sử dụng FIPS-compliant implementations
// khi GOFIPS=1
hash := sha256.Sum256([]byte("data"))
block, _ := aes.NewCipher(key)
```

---

## Đo lường từ xa Go (Telemetry)

### Tổng quan về Đo lường từ xa

Đo lường từ xa là thu thập dữ liệu sử dụng và chẩn đoán ẩn danh từ chuỗi công cụ Go để cải thiện trải nghiệm nhà phát triển.

**Mục đích của telemetry:**
- Hiểu cách developers sử dụng Go
- Phát hiện và sửa lỗi trong toolchain
- Ưu tiên phát triển tính năng mới
- Cải thiện hiệu suất compiler và tools

**Nguyên tắc telemetry:**
- ✅ Opt-in mặc định
- ✅ Dữ liệu ẩn danh
- ✅ Không thu thập source code
- ✅ Không thu thập thông tin cá nhân
- ✅ Có thể tắt hoàn toàn

### Cấu hình Đo lường từ xa

Cách bật, tắt hoặc cấu hình đo lường từ xa trong cài đặt Go của bạn sử dụng lệnh `go telemetry`.

**Xem trạng thái hiện tại:**
```bash
go telemetry
# Output: local (hoặc on/off)
```

**Các chế độ telemetry:**
```bash
# Bật telemetry (gửi dữ liệu)
go telemetry on

# Tắt telemetry
go telemetry off

# Chế độ local (lưu local, không gửi)
go telemetry local
```

**Xem dữ liệu đã thu thập:**
```bash
# Liệt kê các files telemetry
ls $(go env GOTELEMETRYDIR)

# Xem nội dung
cat $(go env GOTELEMETRYDIR)/*.count
```

**Biến môi trường:**
```bash
# Thư mục lưu dữ liệu telemetry
go env GOTELEMETRYDIR
# Mặc định: $HOME/.config/go/telemetry (Linux/macOS)
#           %APPDATA%\go\telemetry (Windows)

# Upload URL
go env GOTELEMETRY
```

### Dữ liệu được thu thập

Các loại dữ liệu được thu thập bởi hệ thống đo lường từ xa Go và cách chúng được sử dụng để cải thiện Go.

**Loại dữ liệu:**

| Loại | Mô tả | Ví dụ |
|------|-------|-------|
| Counter | Số lần sự kiện xảy ra | Số lần compile thành công |
| Stack | Call stacks khi có lỗi | Stack trace của crashes |

**Dữ liệu cụ thể được thu thập:**
- Phiên bản Go
- Hệ điều hành và architecture
- Tần suất sử dụng các lệnh go
- Lỗi và warnings từ compiler
- Performance metrics của gopls
- Crashes trong Go tools

**Dữ liệu KHÔNG được thu thập:**
- ❌ Source code
- ❌ Tên file hoặc package paths
- ❌ IP address
- ❌ Thông tin cá nhân
- ❌ Secrets hoặc credentials

**Ví dụ counter file:**
```
# Format: counter_name count
go/build/success 42
go/test/run 15
go/test/fail 3
gopls/completion/latency_ms:bucket=100 27
```

### Quyền riêng tư và Đo lường từ xa

Thông tin về quyền riêng tư liên quan đến việc thu thập và xử lý dữ liệu đo lường từ xa.

**Cam kết quyền riêng tư:**
1. **Opt-in**: Telemetry chỉ gửi dữ liệu khi bạn cho phép
2. **Transparent**: Bạn có thể xem tất cả dữ liệu được thu thập
3. **Minimal**: Chỉ thu thập dữ liệu cần thiết
4. **Anonymous**: Không có identifiers cá nhân

**Kiểm soát dữ liệu:**
```bash
# Xem dữ liệu sẽ được upload
go telemetry view

# Xóa dữ liệu local
rm -rf $(go env GOTELEMETRYDIR)/*

# Tắt vĩnh viễn
go telemetry off
echo 'export GOTELEMETRY=off' >> ~/.bashrc
```

**Cho môi trường doanh nghiệp:**
```bash
# Tắt telemetry cho tất cả users
# /etc/profile.d/go-telemetry.sh
export GOTELEMETRY=off

# Hoặc trong CI/CD
env:
  GOTELEMETRY: off
```

---

## Xây dựng và phát hành

### Module Mirror và Checksum Database

Cách sử dụng module mirror và checksum database.

**Go Module Mirror:**
- URL: `https://proxy.golang.org`
- Cache các modules để tải xuống nhanh hơn
- Đảm bảo modules luôn available

**Checksum Database:**
- URL: `https://sum.golang.org`
- Xác minh tính toàn vẹn của modules
- Ngăn chặn supply chain attacks

**Cấu hình:**
```bash
# Xem cấu hình hiện tại
go env GOPROXY
go env GOSUMDB

# Mặc định
# GOPROXY=https://proxy.golang.org,direct
# GOSUMDB=sum.golang.org

# Sử dụng proxy khác (ví dụ Trung Quốc)
go env -w GOPROXY=https://goproxy.cn,direct

# Sử dụng proxy nội bộ
go env -w GOPROXY=https://goproxy.company.com,https://proxy.golang.org,direct
```

**Direct fallback:**
```bash
# Tắt proxy (tải trực tiếp từ source)
go env -w GOPROXY=direct

# Tắt checksum verification
go env -w GOSUMDB=off
```

### Private Module

Cách cấu hình các module riêng tư với go.

**GOPRIVATE:**
```bash
# Đặt private modules
go env -w GOPRIVATE=github.com/mycompany/*,gitlab.company.com/*

# Nhiều patterns
go env -w GOPRIVATE=*.corp.example.com,github.com/myorg/*
```

**GONOPROXY và GONOSUMDB:**
```bash
# Modules không qua proxy
go env -w GONOPROXY=github.com/mycompany/*

# Modules không verify checksum
go env -w GONOSUMDB=github.com/mycompany/*
```

**Xác thực private repos:**
```bash
# Git credentials
git config --global url."https://oauth2:${TOKEN}@github.com".insteadOf "https://github.com"

# SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# .netrc file
cat >> ~/.netrc << EOF
machine github.com
login oauth2
password ${GITHUB_TOKEN}
EOF
chmod 600 ~/.netrc
```

**Trong Docker:**
```dockerfile
FROM golang:1.23

ARG GITHUB_TOKEN

RUN git config --global url."https://oauth2:${GITHUB_TOKEN}@github.com".insteadOf "https://github.com/"
RUN go env -w GOPRIVATE=github.com/mycompany/*

COPY . .
RUN go build -o app
```

### Xây dựng lại có thể tái tạo

Cách xác minh rằng một bản build Go được tái tạo chính xác.

**Reproducible builds:**
Go hỗ trợ reproducible builds - cùng source code tạo ra binary giống hệt nhau.

**Các yếu tố ảnh hưởng:**
- Go version
- GOOS, GOARCH
- CGO_ENABLED
- Build flags
- Dependencies versions (go.sum)
- Timestamp (có thể loại bỏ với -trimpath)

**Build reproducible:**
```bash
# Loại bỏ paths từ binary
go build -trimpath -ldflags="-s -w" -o app

# Xác minh
sha256sum app
# Output: abc123... app

# Build lại trên máy khác
go build -trimpath -ldflags="-s -w" -o app
sha256sum app
# Output: abc123... app (giống hệt)
```

**CGO và reproducibility:**
```bash
# CGO có thể ảnh hưởng đến reproducibility
CGO_ENABLED=0 go build -trimpath -o app
```

**Cross-compilation:**
```bash
# Build cho nhiều platforms
GOOS=linux GOARCH=amd64 go build -trimpath -o app-linux-amd64
GOOS=darwin GOARCH=arm64 go build -trimpath -o app-darwin-arm64
GOOS=windows GOARCH=amd64 go build -trimpath -o app-windows-amd64.exe
```

**Release script mẫu:**
```bash
#!/bin/bash
set -e

VERSION=${1:-"dev"}
LDFLAGS="-s -w -X main.version=${VERSION}"

platforms=("linux/amd64" "linux/arm64" "darwin/amd64" "darwin/arm64" "windows/amd64")

for platform in "${platforms[@]}"; do
    IFS="/" read -r GOOS GOARCH <<< "$platform"
    output="dist/app-${GOOS}-${GOARCH}"
    [[ "$GOOS" == "windows" ]] && output+=".exe"
    
    echo "Building for ${GOOS}/${GOARCH}..."
    CGO_ENABLED=0 GOOS=$GOOS GOARCH=$GOARCH \
        go build -trimpath -ldflags="$LDFLAGS" -o "$output"
done

# Tạo checksums
cd dist && sha256sum * > checksums.txt
```

**Verify build:**
```bash
# So sánh hai binaries
diff <(xxd binary1) <(xxd binary2)

# Hoặc dùng sha256
[[ $(sha256sum binary1 | cut -d' ' -f1) == $(sha256sum binary2 | cut -d' ' -f1) ]]
```

---

## Phát triển Go

### Quá trình phát triển

Tham gia vào phát triển Go.

**Cách Go được phát triển:**
- Mã nguồn mở tại [go.googlesource.com/go](https://go.googlesource.com/go)
- Issues tại [github.com/golang/go/issues](https://github.com/golang/go/issues)
- Code review tại [go-review.googlesource.com](https://go-review.googlesource.com)
- Proposals tại [golang.org/s/proposal](https://golang.org/s/proposal)

**Chu kỳ phát hành:**
1. **Development phase** (~3 tháng): Thêm tính năng mới
2. **Freeze**: Không thêm tính năng mới
3. **Beta releases**: Testing rộng rãi
4. **Release candidates**: Final testing
5. **Release**: Phát hành chính thức

**Tham gia phát triển:**
```bash
# Clone Go source
git clone https://go.googlesource.com/go
cd go/src

# Build Go từ source
./all.bash

# Chạy tests
./run.bash
```

### Đóng góp mã

Quy trình để gửi các thay đổi cho dự án Go.

**Bước 1: Đăng ký CLA**
- Contributor License Agreement
- Truy cập [cla.developers.google.com](https://cla.developers.google.com)

**Bước 2: Clone và setup:**
```bash
git clone https://go.googlesource.com/go
cd go

# Cài đặt git-codereview
go install golang.org/x/review/git-codereview@latest

# Cấu hình
git config push.default nothing
git config alias.change 'codereview change'
git config alias.gofmt 'codereview gofmt'
git config alias.mail 'codereview mail'
git config alias.pending 'codereview pending'
git config alias.submit 'codereview submit'
git config alias.sync 'codereview sync'
```

**Bước 3: Tạo change:**
```bash
# Tạo branch mới
git checkout -b my-change

# Thực hiện thay đổi
# ...

# Format code
git gofmt

# Commit
git add .
git commit -m "pkg/subpkg: description of change

Detailed description of what and why.

Fixes #12345"
```

**Bước 4: Gửi để review:**
```bash
# Gửi lên Gerrit
git mail

# Cập nhật sau review feedback
git add .
git commit --amend
git mail
```

**Commit message format:**
```
package/name: short summary

Longer description of the change.
What the change does and why.

Fixes #12345
```

### Đóng góp vào Wiki

Quy trình để đóng góp vào Wiki Go.

**Wiki Go là gì?**
- Tài liệu cộng đồng bổ sung cho tài liệu chính thức
- Bất kỳ ai có GitHub account có thể chỉnh sửa

**Cách đóng góp:**
1. Truy cập [github.com/golang/go/wiki](https://github.com/golang/go/wiki)
2. Click vào trang muốn chỉnh sửa
3. Click nút "Edit"
4. Thực hiện thay đổi
5. Commit changes

**Guidelines:**
- Sử dụng Markdown format
- Kiểm tra links trước khi submit
- Giữ nội dung objective và accurate
- Thêm ví dụ code khi có thể

### Thiết lập và sử dụng Website Go cục bộ

Cách thiết lập website Go cục bộ để phát triển.

```bash
# Clone website source
git clone https://go.googlesource.com/website
cd website

# Chạy local server
go run . -http=:6060

# Truy cập http://localhost:6060
```

**Phát triển tài liệu:**
```bash
# Clone thêm Go source cho docs
git clone https://go.googlesource.com/go ../go

# Chạy với local Go source
go run . -http=:6060 -goroot=../go
```

### Source code

Xem mã nguồn Go.

**Repositories chính:**
| Repository | Mô tả | URL |
|------------|-------|-----|
| go | Ngôn ngữ và tools | go.googlesource.com/go |
| tools | gopls, goimports, etc. | go.googlesource.com/tools |
| website | go.dev website | go.googlesource.com/website |
| proposal | Đề xuất tính năng | go.googlesource.com/proposal |
| exp | Experimental packages | go.googlesource.com/exp |

**Xem source trực tuyến:**
- Thư viện chuẩn: [cs.opensource.google/go/go](https://cs.opensource.google/go/go)
- Packages: [pkg.go.dev](https://pkg.go.dev)

**Clone source:**
```bash
# Main Go repository
git clone https://go.googlesource.com/go

# Tools repository
git clone https://go.googlesource.com/tools

# Xem một package cụ thể
go doc -all fmt
```

---

## Cộng đồng

Cộng đồng Go rất sôi nổi và thân thiện. Để tham gia:

### Các kênh giao tiếp chính

| Kênh | Mô tả | Link |
|------|-------|------|
| **Go Forum** | Diễn đàn thảo luận chính thức | [forum.golangbridge.org](https://forum.golangbridge.org) |
| **Gophers Slack** | Chat realtime | [gophers.slack.com](https://invite.slack.golangbridge.org/) |
| **Reddit** | Subreddit Go | [reddit.com/r/golang](https://reddit.com/r/golang) |
| **Discord** | Discord server | [discord.gg/golang](https://discord.gg/golang) |
| **Stack Overflow** | Q&A | [stackoverflow.com/questions/tagged/go](https://stackoverflow.com/questions/tagged/go) |

### Sự kiện

- **Go Forum**: Diễn đàn thảo luận Go.
- **Gophers Slack**: Kênh chat cho các lập trình viên Go.
- **Go Meetups**: Các nhóm gặp mặt địa phương.
- **GopherCon**: Hội nghị chính thức về Go.

**Các hội nghị lớn:**

| Hội nghị | Địa điểm | Thời gian |
|----------|----------|-----------|
| GopherCon | Mỹ | Hàng năm |
| GopherCon Europe | Châu Âu | Hàng năm |
| GopherCon India | Ấn Độ | Hàng năm |
| GoLab | Italy | Hàng năm |
| dotGo | Paris | Hàng năm |

**Meetups Việt Nam:**
- Golang Vietnam (Facebook, Meetup)
- Ho Chi Minh City Go Meetup
- Hanoi Golang Meetup

### Code of Conduct

Cộng đồng Go tuân theo [Go Community Code of Conduct](https://go.dev/conduct):
- Tôn trọng lẫn nhau
- Xây dựng môi trường inclusive
- Hỗ trợ người mới
- Không quấy rối hoặc phân biệt đối xử

### Tài nguyên học tập

**Sách hay về Go:**
- "The Go Programming Language" - Donovan & Kernighan
- "Go in Action" - Kennedy, Ketelsen & Martin
- "Concurrency in Go" - Katherine Cox-Buday
- "Learning Go" - Jon Bodner

**Khóa học online:**
- [Go Tour](https://go.dev/tour/) - Miễn phí, chính thức
- [Exercism Go Track](https://exercism.org/tracks/go) - Miễn phí
- [Go by Example](https://gobyexample.com) - Miễn phí
- [Gophercises](https://gophercises.com) - Miễn phí

**YouTube channels:**
- [GopherCon](https://www.youtube.com/@GopherAcademy)
- [justforfunc](https://www.youtube.com/@JustForFunc)
- [Golang Dojo](https://www.youtube.com/@GolangDojo)

### Đóng góp cho cộng đồng

**Cách bạn có thể đóng góp:**
1. Trả lời câu hỏi trên Stack Overflow
2. Viết blog posts về Go
3. Đóng góp vào open source projects
4. Tổ chức hoặc tham gia meetups
5. Dịch tài liệu (như dự án này!)
6. Mentoring người mới

**Open source projects để đóng góp:**
- [Hugo](https://github.com/gohugoio/hugo) - Static site generator
- [Gin](https://github.com/gin-gonic/gin) - Web framework
- [Cobra](https://github.com/spf13/cobra) - CLI framework
- [Kubernetes](https://github.com/kubernetes/kubernetes) - Container orchestration
- [Docker](https://github.com/moby/moby) - Container platform

---

## Liên kết hữu ích

### Trang web chính thức
- [Go.dev](https://go.dev) - Trang chủ chính thức
- [Go Playground](https://go.dev/play/) - Chạy mã Go trực tuyến
- [Go Blog](https://go.dev/blog/) - Blog chính thức của Go
- [Go Packages](https://pkg.go.dev) - Tìm kiếm các package Go

### Tài liệu tham khảo
- [Go Documentation](https://go.dev/doc/) - Tài liệu chính thức
- [Go Tour](https://go.dev/tour/) - Tour học Go
- [Effective Go](https://go.dev/doc/effective_go) - Viết Go hiệu quả
- [Go Wiki](https://go.dev/wiki/) - Wiki cộng đồng

### Công cụ và Resources
- [Go Vulnerability Database](https://vuln.go.dev) - Cơ sở dữ liệu lỗ hổng
- [Go Code Search](https://cs.opensource.google/go) - Tìm kiếm mã nguồn Go
- [Go Issue Tracker](https://github.com/golang/go/issues) - Báo cáo bugs
- [Go Release Notes](https://go.dev/doc/devel/release) - Ghi chú phiên bản

### Công cụ phát triển
- [gopls](https://pkg.go.dev/golang.org/x/tools/gopls) - Language server
- [staticcheck](https://staticcheck.io/) - Linter nâng cao
- [golangci-lint](https://golangci-lint.run/) - Aggregated linters
- [delve](https://github.com/go-delve/delve) - Debugger

### Frameworks và Libraries phổ biến

| Loại | Package | Mô tả |
|------|---------|-------|
| Web Framework | [gin](https://github.com/gin-gonic/gin) | High performance |
| Web Framework | [echo](https://echo.labstack.com/) | Minimalist |
| Web Framework | [fiber](https://gofiber.io/) | Express-inspired |
| ORM | [gorm](https://gorm.io/) | Full-featured ORM |
| CLI | [cobra](https://github.com/spf13/cobra) | CLI framework |
| Config | [viper](https://github.com/spf13/viper) | Configuration |
| Logging | [zap](https://github.com/uber-go/zap) | Fast, structured |
| Testing | [testify](https://github.com/stretchr/testify) | Assertions |
| HTTP Client | [resty](https://github.com/go-resty/resty) | REST client |
| Validation | [validator](https://github.com/go-playground/validator) | Struct validation |

---

## Bảng thuật ngữ Go (Việt - Anh)

| Tiếng Việt | English | Giải thích |
|------------|---------|------------|
| Hàm | Function | Khối mã có thể tái sử dụng |
| Giao diện | Interface | Tập hợp các method signatures |
| Cấu trúc | Struct | Kiểu dữ liệu phức hợp |
| Mảng | Array | Tập hợp phần tử có kích thước cố định |
| Lát cắt | Slice | Dynamic array |
| Bản đồ | Map | Key-value store |
| Kênh | Channel | Giao tiếp giữa goroutines |
| Goroutine | Goroutine | Lightweight thread |
| Con trỏ | Pointer | Địa chỉ bộ nhớ |
| Phương thức | Method | Hàm gắn với kiểu |
| Gói | Package | Đơn vị tổ chức mã |
| Mô-đun | Module | Tập hợp packages |
| Biên dịch | Compile | Chuyển thành mã máy |
| Thu gom rác | Garbage Collection | Quản lý bộ nhớ tự động |
| Đồng thời | Concurrency | Xử lý nhiều tác vụ |
| Song song | Parallelism | Thực thi đồng thời |
| Phụ thuộc | Dependency | Package bên ngoài cần thiết |
| Nhúng | Embedding | Composition trong Go |
| Defer | Defer | Trì hoãn thực thi |
| Panic | Panic | Lỗi runtime nghiêm trọng |
| Recover | Recover | Phục hồi từ panic |

---

*Tài liệu này được dịch từ [go.dev/doc](https://go.dev/doc/) để phục vụ cộng đồng lập trình viên Việt Nam.*

*Đóng góp và phản hồi được hoan nghênh tại [GitHub repository](https://github.com/vidsrvcom/godocument).*

*Cập nhật lần cuối: Tháng 11, 2024*
