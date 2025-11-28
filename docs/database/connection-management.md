# Quản lý Kết nối (Connection Management)

Package `database/sql` trong Go tự động quản lý một connection pool, nhưng hiểu cách nó hoạt động và cấu hình đúng là rất quan trọng cho hiệu suất ứng dụng.

## Connection Pool

Khi bạn mở database handle với `sql.Open`, Go tạo một connection pool. Pool này:

- Tự động tạo kết nối khi cần
- Tái sử dụng kết nối idle
- Đóng kết nối quá cũ hoặc không dùng
- Giới hạn số lượng kết nối

```go
db, err := sql.Open("mysql", dsn)
if err != nil {
    log.Fatal(err)
}
defer db.Close()

// db là một connection pool, không phải một kết nối đơn lẻ
```

## Cấu hình Connection Pool

### SetMaxOpenConns

Đặt số lượng tối đa kết nối mở đến database:

```go
// Tối đa 25 kết nối mở
db.SetMaxOpenConns(25)

// 0 = không giới hạn (không khuyến nghị)
// Mặc định = 0 (unlimited)
```

**Khuyến nghị**: Đặt dựa trên capabilities của database và số lượng requests đồng thời dự kiến.

### SetMaxIdleConns

Đặt số lượng tối đa kết nối idle trong pool:

```go
// Tối đa 25 kết nối idle
db.SetMaxIdleConns(25)

// Mặc định = 2
```

**Khuyến nghị**: Nên bằng hoặc gần bằng `MaxOpenConns` để tránh việc mở đóng kết nối liên tục.

### SetConnMaxLifetime

Đặt thời gian tối đa một kết nối có thể được tái sử dụng:

```go
// Kết nối sẽ bị đóng sau 5 phút
db.SetConnMaxLifetime(5 * time.Minute)

// 0 = không có giới hạn thời gian
// Mặc định = 0
```

**Khuyến nghị**: Đặt thấp hơn timeout của database/load balancer để tránh connection errors.

### SetConnMaxIdleTime

Đặt thời gian tối đa một kết nối có thể idle trước khi bị đóng:

```go
// Đóng kết nối sau 10 phút idle
db.SetConnMaxIdleTime(10 * time.Minute)

// Mặc định = 0 (unlimited)
```

## Cấu hình tối ưu

```go
func setupDB(dsn string) (*sql.DB, error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }
    
    // Cấu hình connection pool
    db.SetMaxOpenConns(25)                 // Tối đa 25 kết nối mở
    db.SetMaxIdleConns(25)                 // Tối đa 25 kết nối idle
    db.SetConnMaxLifetime(5 * time.Minute) // Kết nối tồn tại tối đa 5 phút
    db.SetConnMaxIdleTime(5 * time.Minute) // Kết nối idle tối đa 5 phút
    
    // Verify kết nối
    if err = db.Ping(); err != nil {
        return nil, err
    }
    
    return db, nil
}
```

## Monitoring Connection Pool

Kiểm tra stats của connection pool:

```go
func printDBStats(db *sql.DB) {
    stats := db.Stats()
    
    fmt.Printf("Max Open Connections: %d\n", stats.MaxOpenConnections)
    fmt.Printf("Open Connections: %d\n", stats.OpenConnections)
    fmt.Printf("In Use: %d\n", stats.InUse)
    fmt.Printf("Idle: %d\n", stats.Idle)
    
    fmt.Printf("Wait Count: %d\n", stats.WaitCount)
    fmt.Printf("Wait Duration: %s\n", stats.WaitDuration)
    
    fmt.Printf("Max Idle Closed: %d\n", stats.MaxIdleClosed)
    fmt.Printf("Max Lifetime Closed: %d\n", stats.MaxLifetimeClosed)
}

// Sử dụng
printDBStats(db)
```

### Các metrics quan trọng

- **OpenConnections**: Tổng số kết nối hiện tại (in-use + idle)
- **InUse**: Số kết nối đang được sử dụng
- **Idle**: Số kết nối idle chờ được tái sử dụng
- **WaitCount**: Số lần phải chờ kết nối
- **WaitDuration**: Tổng thời gian chờ kết nối
- **MaxIdleClosed**: Số kết nối bị đóng do vượt quá MaxIdleConns
- **MaxLifetimeClosed**: Số kết nối bị đóng do vượt quá ConnMaxLifetime

## Connection Pooling Best Practices

### 1. Không tạo nhiều DB instances

```go
// ❌ SAI - Tạo nhiều pools
func getUser(id int) (*User, error) {
    db, _ := sql.Open("mysql", dsn)
    defer db.Close()
    // ...
}

// ✅ ĐÚNG - Sử dụng một pool duy nhất
var db *sql.DB

func init() {
    db, _ = sql.Open("mysql", dsn)
}

func getUser(id int) (*User, error) {
    // Sử dụng db global
    // ...
}
```

### 2. Đặt limits phù hợp

```go
// ❌ SAI - Unlimited connections
db.SetMaxOpenConns(0)  // Có thể overwhelm database

// ✅ ĐÚNG - Giới hạn hợp lý
db.SetMaxOpenConns(25)  // Dựa trên database capacity
```

### 3. MaxIdleConns gần với MaxOpenConns

```go
// ❌ SAI - Quá ít idle connections
db.SetMaxOpenConns(100)
db.SetMaxIdleConns(2)   // Sẽ mở đóng kết nối liên tục

// ✅ ĐÚNG - Cân bằng
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)  // Tái sử dụng tốt
```

### 4. Đặt ConnMaxLifetime

```go
// ✅ ĐÚNG - Tránh stale connections
db.SetConnMaxLifetime(5 * time.Minute)

// Load balancer timeout = 10 phút
// Nên đặt ConnMaxLifetime < 10 phút
```

## Xử lý Connection Errors

```go
func executeWithRetry(db *sql.DB, query string, args ...interface{}) error {
    maxRetries := 3
    
    for i := 0; i < maxRetries; i++ {
        _, err := db.Exec(query, args...)
        if err == nil {
            return nil
        }
        
        // Kiểm tra nếu là connection error
        if errors.Is(err, driver.ErrBadConn) {
            log.Printf("Bad connection, retrying... (%d/%d)", i+1, maxRetries)
            time.Sleep(time.Second * time.Duration(i+1))
            continue
        }
        
        // Lỗi khác, không retry
        return err
    }
    
    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

## Health Checks

Implement health check cho database:

```go
func healthCheck(db *sql.DB) error {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    // Ping với timeout
    if err := db.PingContext(ctx); err != nil {
        return fmt.Errorf("database ping failed: %w", err)
    }
    
    // Kiểm tra stats
    stats := db.Stats()
    if stats.OpenConnections == stats.MaxOpenConnections {
        log.Println("Warning: connection pool is full")
    }
    
    return nil
}

// Chạy định kỳ
func monitorDB(db *sql.DB) {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        if err := healthCheck(db); err != nil {
            log.Printf("Health check failed: %v", err)
        }
    }
}
```

## Connection Strings và Parameters

Cấu hình connection string tối ưu:

### MySQL

```go
dsn := "user:password@tcp(localhost:3306)/dbname?" +
    "charset=utf8mb4" +
    "&parseTime=True" +
    "&loc=Local" +
    "&timeout=10s" +           // Connection timeout
    "&readTimeout=30s" +       // Read timeout
    "&writeTimeout=30s" +      // Write timeout
    "&maxAllowedPacket=0" +    // Max packet size
    "&autocommit=true"         // Auto commit

db, err := sql.Open("mysql", dsn)
```

### PostgreSQL

```go
dsn := "host=localhost port=5432 user=myuser password=mypass dbname=mydb " +
    "sslmode=disable " +
    "connect_timeout=10 " +
    "statement_timeout=30000"  // 30 seconds

db, err := sql.Open("postgres", dsn)
```

## Graceful Shutdown

Đóng database gracefully:

```go
func main() {
    db, err := setupDB(dsn)
    if err != nil {
        log.Fatal(err)
    }
    
    // Setup graceful shutdown
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
    
    go func() {
        <-sigChan
        log.Println("Shutting down...")
        
        // Đóng database với timeout
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        
        if err := shutdownDB(ctx, db); err != nil {
            log.Printf("Error shutting down database: %v", err)
        }
        
        os.Exit(0)
    }()
    
    // Application code...
}

func shutdownDB(ctx context.Context, db *sql.DB) error {
    // Đợi tất cả connections idle
    ticker := time.NewTicker(100 * time.Millisecond)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            stats := db.Stats()
            if stats.InUse == 0 {
                return db.Close()
            }
            log.Printf("Waiting for %d connections to close...", stats.InUse)
        }
    }
}
```

## Ví dụ Configuration đầy đủ

```go
type DBConfig struct {
    MaxOpenConns    int
    MaxIdleConns    int
    ConnMaxLifetime time.Duration
    ConnMaxIdleTime time.Duration
}

func NewDB(dsn string, config DBConfig) (*sql.DB, error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, fmt.Errorf("failed to open database: %w", err)
    }
    
    // Apply configuration
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    db.SetConnMaxLifetime(config.ConnMaxLifetime)
    db.SetConnMaxIdleTime(config.ConnMaxIdleTime)
    
    // Verify connection
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := db.PingContext(ctx); err != nil {
        db.Close()
        return nil, fmt.Errorf("failed to ping database: %w", err)
    }
    
    return db, nil
}

// Usage
config := DBConfig{
    MaxOpenConns:    25,
    MaxIdleConns:    25,
    ConnMaxLifetime: 5 * time.Minute,
    ConnMaxIdleTime: 5 * time.Minute,
}

db, err := NewDB(dsn, config)
```

## Thực hành tốt nhất

1. **Sử dụng một db instance**: Cho toàn bộ ứng dụng
2. **Đặt MaxOpenConns**: Dựa trên database capacity
3. **MaxIdleConns gần MaxOpenConns**: Để tái sử dụng tốt
4. **Đặt ConnMaxLifetime**: Thấp hơn database/LB timeout
5. **Monitor connection pool**: Định kỳ kiểm tra stats
6. **Implement health checks**: Phát hiện vấn đề sớm
7. **Graceful shutdown**: Đóng connections đúng cách
8. **Test under load**: Verify configuration dưới tải cao

## Xem thêm

- [Mở database handle](opening-database.md)
- [Xử lý lỗi](error-handling.md)
- [Thực thi giao dịch](transactions.md)
- [Tổng quan về Truy cập Cơ sở Dữ liệu](overview.md)
