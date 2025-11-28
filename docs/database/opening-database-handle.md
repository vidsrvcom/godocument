# Mở một xử lý cơ sở dữ liệu

## Giới thiệu

Để truy cập cơ sở dữ liệu trong Go, bạn cần mở một database handle sử dụng package `database/sql`.

## Mở kết nối

### Sử dụng sql.Open

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    // DSN (Data Source Name) format: user:password@tcp(host:port)/dbname
    db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/testdb")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
}
```

### Lưu ý quan trọng

`sql.Open()` không tạo kết nối ngay lập tức. Nó chỉ:
- Validate DSN format
- Tạo `*sql.DB` object
- Khởi tạo connection pool

### Verify kết nối

```go
if err := db.Ping(); err != nil {
    log.Fatal("Cannot connect to database:", err)
}
fmt.Println("Connected successfully!")
```

## DSN Formats

### MySQL

```go
// Basic
"user:password@tcp(localhost:3306)/dbname"

// With parameters
"user:password@tcp(localhost:3306)/dbname?charset=utf8mb4&parseTime=True"
```

### PostgreSQL

```go
// Standard format
"postgres://user:password@localhost:5432/dbname?sslmode=disable"

// Key-value format
"host=localhost port=5432 user=postgres password=secret dbname=testdb sslmode=disable"
```

### SQLite

```go
// File database
"./database.db"

// In-memory database
":memory:"
```

## Quản lý lifecycle

### Singleton pattern

```go
var db *sql.DB

func InitDB(dsn string) error {
    var err error
    db, err = sql.Open("mysql", dsn)
    if err != nil {
        return err
    }
    return db.Ping()
}

func GetDB() *sql.DB {
    return db
}
```

### Với context

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

if err := db.PingContext(ctx); err != nil {
    log.Fatal("Database not responding:", err)
}
```

## Best Practices

1. **Chỉ mở một sql.DB**: Tái sử dụng cho toàn bộ ứng dụng
2. **Luôn defer Close()**: Đảm bảo cleanup
3. **Sử dụng Ping() để verify**: Kiểm tra kết nối thực sự
4. **Cấu hình connection pool**: Tối ưu cho workload

---

*Xem thêm tại [go.dev/doc/database/open-handle](https://go.dev/doc/database/open-handle)*
