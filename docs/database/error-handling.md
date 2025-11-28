# Xử lý Lỗi trong Database Operations

Xử lý lỗi đúng cách là rất quan trọng khi làm việc với cơ sở dữ liệu. Tài liệu này mô tả các thực hành tốt nhất và patterns phổ biến để xử lý lỗi trong Go database operations.

## Các loại lỗi phổ biến

### 1. Connection Errors

Lỗi kết nối với database:

```go
db, err := sql.Open("mysql", dsn)
if err != nil {
    return fmt.Errorf("failed to open database: %w", err)
}

err = db.Ping()
if err != nil {
    return fmt.Errorf("failed to connect to database: %w", err)
}
```

### 2. Query Errors

Lỗi khi thực thi query:

```go
rows, err := db.Query("SELECT id, name FROM users WHERE age > ?", age)
if err != nil {
    return fmt.Errorf("failed to query users: %w", err)
}
defer rows.Close()
```

### 3. No Rows Error

Xử lý khi không tìm thấy kết quả:

```go
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
if err != nil {
    if err == sql.ErrNoRows {
        return fmt.Errorf("user not found: %w", err)
    }
    return fmt.Errorf("failed to get user: %w", err)
}
```

### 4. Scan Errors

Lỗi khi scan giá trị:

```go
var id int
var name string
err := rows.Scan(&id, &name)
if err != nil {
    return fmt.Errorf("failed to scan row: %w", err)
}
```

### 5. Transaction Errors

Lỗi trong transactions:

```go
tx, err := db.Begin()
if err != nil {
    return fmt.Errorf("failed to begin transaction: %w", err)
}
defer tx.Rollback()

// ... operations ...

if err := tx.Commit(); err != nil {
    return fmt.Errorf("failed to commit transaction: %w", err)
}
```

## Error Wrapping

Sử dụng `fmt.Errorf` với `%w` để wrap errors và giữ nguyên error chain:

```go
func getUser(db *sql.DB, id int) (*User, error) {
    user := &User{}
    err := db.QueryRow(
        "SELECT id, name, email FROM users WHERE id = ?", 
        id,
    ).Scan(&user.ID, &user.Name, &user.Email)
    
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, fmt.Errorf("user %d not found: %w", id, err)
        }
        return nil, fmt.Errorf("failed to get user %d: %w", id, err)
    }
    
    return user, nil
}

// Sử dụng
user, err := getUser(db, 123)
if err != nil {
    if errors.Is(err, sql.ErrNoRows) {
        fmt.Println("User not found")
    } else {
        log.Printf("Error: %v", err)
    }
}
```

## Custom Error Types

Tạo các custom error types cho domain-specific errors:

```go
type UserError struct {
    UserID int
    Op     string
    Err    error
}

func (e *UserError) Error() string {
    return fmt.Sprintf("user %d: %s: %v", e.UserID, e.Op, e.Err)
}

func (e *UserError) Unwrap() error {
    return e.Err
}

func getUser(db *sql.DB, id int) (*User, error) {
    user := &User{}
    err := db.QueryRow("SELECT id, name FROM users WHERE id = ?", id).
        Scan(&user.ID, &user.Name)
    
    if err != nil {
        return nil, &UserError{
            UserID: id,
            Op:     "get",
            Err:    err,
        }
    }
    
    return user, nil
}
```

## Sentinel Errors

Package `database/sql` cung cấp các sentinel errors:

```go
// sql.ErrNoRows - Không tìm thấy rows
// sql.ErrConnDone - Kết nối đã đóng
// sql.ErrTxDone - Transaction đã kết thúc

func findUser(db *sql.DB, id int) (*User, error) {
    var user User
    err := db.QueryRow("SELECT id, name FROM users WHERE id = ?", id).
        Scan(&user.ID, &user.Name)
    
    switch {
    case err == sql.ErrNoRows:
        return nil, fmt.Errorf("user not found")
    case err != nil:
        return nil, fmt.Errorf("query error: %w", err)
    default:
        return &user, nil
    }
}
```

## Retry Logic

Implement retry logic cho transient errors:

```go
func queryWithRetry(db *sql.DB, query string, args ...interface{}) (*sql.Rows, error) {
    maxRetries := 3
    var rows *sql.Rows
    var err error
    
    for i := 0; i < maxRetries; i++ {
        rows, err = db.Query(query, args...)
        if err == nil {
            return rows, nil
        }
        
        // Kiểm tra nếu là temporary error
        if isTemporaryError(err) {
            time.Sleep(time.Duration(i+1) * time.Second)
            continue
        }
        
        // Non-temporary error, không retry
        break
    }
    
    return nil, fmt.Errorf("failed after %d retries: %w", maxRetries, err)
}

func isTemporaryError(err error) bool {
    // Kiểm tra các error codes cụ thể của database
    return strings.Contains(err.Error(), "connection refused") ||
           strings.Contains(err.Error(), "timeout")
}
```

## Context Cancellation

Xử lý context cancellation và timeouts:

```go
func getUserWithTimeout(db *sql.DB, id int) (*User, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    var user User
    err := db.QueryRowContext(ctx, "SELECT id, name FROM users WHERE id = ?", id).
        Scan(&user.ID, &user.Name)
    
    if err != nil {
        if ctx.Err() == context.DeadlineExceeded {
            return nil, fmt.Errorf("query timeout: %w", err)
        }
        if ctx.Err() == context.Canceled {
            return nil, fmt.Errorf("query cancelled: %w", err)
        }
        return nil, fmt.Errorf("query failed: %w", err)
    }
    
    return &user, nil
}
```

## Logging Errors

Best practices cho logging database errors:

```go
import "log/slog"

func getUser(db *sql.DB, id int) (*User, error) {
    start := time.Now()
    
    var user User
    err := db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id).
        Scan(&user.ID, &user.Name, &user.Email)
    
    duration := time.Since(start)
    
    if err != nil {
        slog.Error("failed to get user",
            "user_id", id,
            "error", err,
            "duration_ms", duration.Milliseconds(),
        )
        
        if err == sql.ErrNoRows {
            return nil, fmt.Errorf("user not found")
        }
        return nil, fmt.Errorf("database error: %w", err)
    }
    
    slog.Debug("got user",
        "user_id", id,
        "duration_ms", duration.Milliseconds(),
    )
    
    return &user, nil
}
```

## Validation Errors

Phân biệt validation errors và database errors:

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

func createUser(db *sql.DB, name, email string) error {
    // Validation
    if name == "" {
        return ValidationError{Field: "name", Message: "name is required"}
    }
    if !strings.Contains(email, "@") {
        return ValidationError{Field: "email", Message: "invalid email"}
    }
    
    // Database operation
    _, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", name, email)
    if err != nil {
        if strings.Contains(err.Error(), "Duplicate entry") {
            return ValidationError{Field: "email", Message: "email already exists"}
        }
        return fmt.Errorf("failed to create user: %w", err)
    }
    
    return nil
}

// Usage
err := createUser(db, "", "invalid")
if validErr, ok := err.(ValidationError); ok {
    fmt.Printf("Validation error in %s: %s\n", validErr.Field, validErr.Message)
} else if err != nil {
    log.Printf("Database error: %v", err)
}
```

## Panic Recovery

Sử dụng recover cho unexpected panics:

```go
func safeQuery(db *sql.DB) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic in database operation: %v", r)
            log.Printf("Panic recovered: %v\n%s", r, debug.Stack())
        }
    }()
    
    // Database operations
    rows, err := db.Query("SELECT * FROM users")
    if err != nil {
        return err
    }
    defer rows.Close()
    
    // Process rows...
    
    return nil
}
```

## Thực hành tốt nhất

1. **Luôn kiểm tra errors**: Không bao giờ ignore errors từ database operations
2. **Wrap errors với context**: Sử dụng `%w` để giữ error chain
3. **Log errors với context**: Include relevant information (IDs, parameters, etc.)
4. **Phân biệt error types**: Để xử lý khác nhau (validation vs database vs business logic)
5. **Sử dụng sentinel errors**: Cho các errors có thể recover
6. **Implement retry logic**: Cho transient errors
7. **Xử lý sql.ErrNoRows**: Riêng biệt với errors khác
8. **Close resources**: Ngay cả khi có errors

## Ví dụ đầy đủ

```go
package main

import (
    "context"
    "database/sql"
    "errors"
    "fmt"
    "log"
    "time"
)

type User struct {
    ID    int
    Name  string
    Email string
}

func getUser(ctx context.Context, db *sql.DB, id int) (*User, error) {
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    user := &User{}
    
    err := db.QueryRowContext(ctx,
        "SELECT id, name, email FROM users WHERE id = ?",
        id,
    ).Scan(&user.ID, &user.Name, &user.Email)
    
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, fmt.Errorf("user %d not found", id)
        }
        if ctx.Err() == context.DeadlineExceeded {
            return nil, fmt.Errorf("query timeout for user %d: %w", id, err)
        }
        return nil, fmt.Errorf("failed to get user %d: %w", id, err)
    }
    
    return user, nil
}

func main() {
    db, err := sql.Open("mysql", "user:pass@/dbname")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    user, err := getUser(context.Background(), db, 123)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            fmt.Println("User not found")
        } else {
            log.Printf("Error getting user: %v", err)
        }
        return
    }
    
    fmt.Printf("User: %+v\n", user)
}
```

## Xem thêm

- [Mở database handle](opening-database.md)
- [Truy vấn dữ liệu](querying-data.md)
- [Thực thi giao dịch](transactions.md)
- [Tổng quan về Truy cập Cơ sở Dữ liệu](overview.md)
