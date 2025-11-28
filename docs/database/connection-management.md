# Quản lý kết nối

## Giới thiệu

`sql.DB` quản lý một connection pool tự động. Hiểu và cấu hình đúng connection pool là quan trọng cho performance.

## Connection Pool

### Cách hoạt động

```
Application
    ↓
sql.DB (Connection Pool)
    ↓  ↓  ↓  ↓
   [c1][c2][c3][c4]  ← Pool of connections
    ↓  ↓  ↓  ↓
Database Server
```

### Mặc định

- Unlimited max open connections
- 2 max idle connections
- No max lifetime
- No max idle time

## Cấu hình Connection Pool

### SetMaxOpenConns

```go
// Giới hạn số connections đồng thời tối đa
db.SetMaxOpenConns(25)
```

- Ngăn quá tải database
- Requests sẽ block nếu đạt limit

### SetMaxIdleConns

```go
// Số connections idle giữ trong pool
db.SetMaxIdleConns(25)
```

- Tái sử dụng connections nhanh hơn
- Tốn memory nếu quá cao

### SetConnMaxLifetime

```go
// Thời gian sống tối đa của connection
db.SetConnMaxLifetime(5 * time.Minute)
```

- Ngăn sử dụng stale connections
- Hữu ích khi database có timeout

### SetConnMaxIdleTime

```go
// Thời gian idle tối đa trước khi đóng
db.SetConnMaxIdleTime(10 * time.Minute)
```

- Giải phóng resources không sử dụng

## Cấu hình theo môi trường

### Development

```go
db.SetMaxOpenConns(5)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(30 * time.Minute)
```

### Production

```go
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
db.SetConnMaxIdleTime(10 * time.Minute)
```

### High-traffic application

```go
db.SetMaxOpenConns(100)
db.SetMaxIdleConns(50)
db.SetConnMaxLifetime(3 * time.Minute)
```

## Monitoring Connection Pool

### Stats

```go
stats := db.Stats()

fmt.Printf("Open connections: %d\n", stats.OpenConnections)
fmt.Printf("In use: %d\n", stats.InUse)
fmt.Printf("Idle: %d\n", stats.Idle)
fmt.Printf("Wait count: %d\n", stats.WaitCount)
fmt.Printf("Wait duration: %v\n", stats.WaitDuration)
fmt.Printf("Max idle closed: %d\n", stats.MaxIdleClosed)
fmt.Printf("Max lifetime closed: %d\n", stats.MaxLifetimeClosed)
```

### Prometheus metrics

```go
import "github.com/prometheus/client_golang/prometheus"

func collectDBStats(db *sql.DB) {
    prometheus.MustRegister(prometheus.NewGaugeFunc(
        prometheus.GaugeOpts{
            Name: "db_open_connections",
            Help: "Number of open connections",
        },
        func() float64 {
            return float64(db.Stats().OpenConnections)
        },
    ))
}
```

## Connection Health

### Ping

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

if err := db.PingContext(ctx); err != nil {
    log.Printf("Database is unreachable: %v", err)
}
```

### Health check endpoint

```go
func HealthHandler(db *sql.DB) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
        defer cancel()
        
        if err := db.PingContext(ctx); err != nil {
            http.Error(w, "Database unavailable", http.StatusServiceUnavailable)
            return
        }
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    }
}
```

## Graceful Shutdown

```go
func main() {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatal(err)
    }
    
    // Start server...
    
    // Wait for shutdown signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    // Graceful shutdown
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Close database connections
    if err := db.Close(); err != nil {
        log.Printf("Error closing database: %v", err)
    }
}
```

## Troubleshooting

### "too many connections"

```go
// Giảm max open connections
db.SetMaxOpenConns(25)

// Kiểm tra connection leaks
stats := db.Stats()
if stats.InUse > stats.MaxOpenConnections-5 {
    log.Warning("Running low on connections")
}
```

### Connection leaks

```go
// Luôn close resources
rows, err := db.Query("SELECT * FROM users")
if err != nil {
    return err
}
defer rows.Close() // ← QUAN TRỌNG
```

### Stale connections

```go
// Set lifetime để refresh connections
db.SetConnMaxLifetime(5 * time.Minute)
```

## Best Practices

1. **Cấu hình phù hợp workload**: Không để mặc định
2. **Monitor connection pool**: Sử dụng db.Stats()
3. **Set max lifetime**: Tránh stale connections
4. **Match MaxIdle với MaxOpen**: Tránh churn
5. **Graceful shutdown**: Đóng connections cleanly
6. **Health checks**: Verify database reachable

---

*Xem thêm tại [go.dev/doc/database/manage-connections](https://go.dev/doc/database/manage-connections)*
