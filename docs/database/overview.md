# Tổng quan về Truy cập Cơ sở Dữ liệu

## Giới thiệu

Go cung cấp package `database/sql` trong thư viện chuẩn để truy cập cơ sở dữ liệu quan hệ một cách nhất quán.

## Package database/sql

### Kiến trúc

```
Application
    ↓
database/sql (standard library)
    ↓
Driver (third-party)
    ↓
Database
```

### Các drivers phổ biến

| Database | Driver |
|----------|--------|
| MySQL | `github.com/go-sql-driver/mysql` |
| PostgreSQL | `github.com/lib/pq` |
| SQLite | `github.com/mattn/go-sqlite3` |
| SQL Server | `github.com/denisenkom/go-mssqldb` |

## Quick Start

### 1. Cài đặt driver

```bash
go get -u github.com/go-sql-driver/mysql
```

### 2. Import

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)
```

### 3. Kết nối

```go
db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/dbname")
if err != nil {
    log.Fatal(err)
}
defer db.Close()

// Verify connection
if err := db.Ping(); err != nil {
    log.Fatal(err)
}
```

### 4. Query

```go
rows, err := db.Query("SELECT id, name FROM users WHERE active = ?", true)
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var id int
    var name string
    if err := rows.Scan(&id, &name); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%d: %s\n", id, name)
}
```

### 5. Execute

```go
result, err := db.Exec("INSERT INTO users (name) VALUES (?)", "John")
if err != nil {
    log.Fatal(err)
}

id, _ := result.LastInsertId()
rows, _ := result.RowsAffected()
```

## Connection Pool

`sql.DB` quản lý connection pool tự động:

```go
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
```

## Transactions

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback() // Rollback if not committed

_, err = tx.Exec("INSERT INTO users (name) VALUES (?)", "Jane")
if err != nil {
    log.Fatal(err)
}

if err := tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Best Practices

1. **Always close resources**: `defer rows.Close()`, `defer tx.Rollback()`
2. **Use prepared statements**: Cho queries lặp lại
3. **Use parameterized queries**: Tránh SQL injection
4. **Check errors**: Luôn kiểm tra errors
5. **Configure connection pool**: Theo workload

## Các tài liệu liên quan

- [Mở một xử lý cơ sở dữ liệu](database-handle.md)
- [Thực thi các câu lệnh SQL](executing-statements.md)
- [Truy vấn dữ liệu](querying.md)
- [Sử dụng prepared statements](prepared-statements.md)
- [Thực thi giao dịch](transactions.md)
- [Xử lý lỗi](error-handling.md)
- [Tránh SQL injection](sql-injection.md)
- [Quản lý kết nối](connection-management.md)

---

*Xem thêm tại [go.dev/doc/database](https://go.dev/doc/database)*
