# Xử lý lỗi

## Giới thiệu

Xử lý lỗi đúng cách là rất quan trọng khi làm việc với cơ sở dữ liệu. Go cung cấp các patterns và errors chuẩn để xử lý các tình huống thường gặp.

## Các lỗi thường gặp

### sql.ErrNoRows

```go
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = ?", 999).Scan(&name)
if err != nil {
    if errors.Is(err, sql.ErrNoRows) {
        fmt.Println("User not found")
        return
    }
    log.Fatal("Query error:", err)
}
```

### sql.ErrConnDone

```go
// Connection đã được đóng hoặc trả về pool
if errors.Is(err, sql.ErrConnDone) {
    // Retry với connection mới
}
```

### sql.ErrTxDone

```go
// Transaction đã được commit hoặc rollback
if errors.Is(err, sql.ErrTxDone) {
    // Transaction không còn valid
}
```

## Error Wrapping

### Wrap errors cho context

```go
func GetUser(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow("SELECT id, name FROM users WHERE id = ?", id).
        Scan(&u.ID, &u.Name)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, fmt.Errorf("user %d not found: %w", id, err)
        }
        return nil, fmt.Errorf("failed to get user %d: %w", id, err)
    }
    return &u, nil
}
```

### Kiểm tra wrapped errors

```go
user, err := GetUser(db, 123)
if err != nil {
    if errors.Is(err, sql.ErrNoRows) {
        // Handle not found
    }
    // Handle other errors
}
```

## Database-specific Errors

### MySQL

```go
import "github.com/go-sql-driver/mysql"

_, err := db.Exec("INSERT INTO users (email) VALUES (?)", "duplicate@test.com")
if err != nil {
    var mysqlErr *mysql.MySQLError
    if errors.As(err, &mysqlErr) {
        if mysqlErr.Number == 1062 {
            fmt.Println("Duplicate entry")
            return
        }
    }
    log.Fatal(err)
}
```

### PostgreSQL

```go
import "github.com/lib/pq"

_, err := db.Exec("INSERT INTO users (email) VALUES ($1)", "duplicate@test.com")
if err != nil {
    var pqErr *pq.Error
    if errors.As(err, &pqErr) {
        if pqErr.Code == "23505" {
            fmt.Println("Unique violation")
            return
        }
    }
    log.Fatal(err)
}
```

## Retry Pattern

```go
func ExecuteWithRetry(db *sql.DB, query string, args ...interface{}) error {
    maxRetries := 3
    var err error
    
    for i := 0; i < maxRetries; i++ {
        _, err = db.Exec(query, args...)
        if err == nil {
            return nil
        }
        
        // Chỉ retry cho transient errors
        if isTransientError(err) {
            time.Sleep(time.Duration(i+1) * 100 * time.Millisecond)
            continue
        }
        return err
    }
    return fmt.Errorf("failed after %d retries: %w", maxRetries, err)
}

func isTransientError(err error) bool {
    // Check for connection errors, timeouts, etc.
    return strings.Contains(err.Error(), "connection refused") ||
           strings.Contains(err.Error(), "timeout")
}
```

## Error Handling trong Transactions

```go
func ExecuteInTransaction(db *sql.DB, fn func(tx *sql.Tx) error) (err error) {
    tx, err := db.Begin()
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }
    
    defer func() {
        if p := recover(); p != nil {
            tx.Rollback()
            panic(p) // Re-throw panic
        } else if err != nil {
            if rbErr := tx.Rollback(); rbErr != nil {
                err = fmt.Errorf("rollback failed: %v, original error: %w", rbErr, err)
            }
        }
    }()
    
    if err = fn(tx); err != nil {
        return err
    }
    
    if err = tx.Commit(); err != nil {
        return fmt.Errorf("commit transaction: %w", err)
    }
    return nil
}
```

## Logging Errors

```go
func LogDatabaseError(operation string, err error) {
    log.Printf("Database error [%s]: %v", operation, err)
    
    // Log thêm details nếu có
    var mysqlErr *mysql.MySQLError
    if errors.As(err, &mysqlErr) {
        log.Printf("MySQL Error %d: %s", mysqlErr.Number, mysqlErr.Message)
    }
}
```

## Best Practices

1. **Luôn kiểm tra errors**: Không bỏ qua bất kỳ error nào
2. **Sử dụng errors.Is() và errors.As()**: Cho error matching
3. **Wrap errors với context**: Thêm thông tin hữu ích
4. **Handle sql.ErrNoRows riêng biệt**: Đây là trường hợp đặc biệt
5. **Implement retry cho transient errors**: Connection issues, timeouts
6. **Log đầy đủ thông tin**: Cho debugging

---

*Xem thêm tại [go.dev/doc/database/handle-errors](https://go.dev/doc/database/handle-errors)*
