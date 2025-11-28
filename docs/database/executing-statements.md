# Thực thi các câu lệnh SQL không trả về dữ liệu

## Giới thiệu

Sử dụng `db.Exec()` để thực thi các câu lệnh INSERT, UPDATE, DELETE và các câu lệnh SQL khác không trả về dữ liệu.

## Sử dụng db.Exec()

### INSERT

```go
result, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", "John", "john@example.com")
if err != nil {
    log.Fatal(err)
}

// Lấy ID của record mới
id, err := result.LastInsertId()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("New user ID: %d\n", id)
```

### UPDATE

```go
result, err := db.Exec("UPDATE users SET name = ? WHERE id = ?", "Jane", 1)
if err != nil {
    log.Fatal(err)
}

// Số rows bị ảnh hưởng
rows, err := result.RowsAffected()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Updated %d rows\n", rows)
```

### DELETE

```go
result, err := db.Exec("DELETE FROM users WHERE id = ?", 1)
if err != nil {
    log.Fatal(err)
}

rows, _ := result.RowsAffected()
fmt.Printf("Deleted %d rows\n", rows)
```

## Với Context

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

result, err := db.ExecContext(ctx, "INSERT INTO users (name) VALUES (?)", "John")
if err != nil {
    log.Fatal(err)
}
```

## sql.Result interface

```go
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```

### Lưu ý về LastInsertId()

- Không phải tất cả drivers đều hỗ trợ
- PostgreSQL yêu cầu sử dụng RETURNING clause

```go
// PostgreSQL - sử dụng QueryRow thay vì Exec
var id int64
err := db.QueryRow("INSERT INTO users (name) VALUES ($1) RETURNING id", "John").Scan(&id)
```

## Batch Operations

### Nhiều INSERTs

```go
users := []string{"Alice", "Bob", "Charlie"}

for _, name := range users {
    _, err := db.Exec("INSERT INTO users (name) VALUES (?)", name)
    if err != nil {
        log.Printf("Error inserting %s: %v", name, err)
    }
}
```

### Với transaction (recommended)

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

for _, name := range users {
    _, err := tx.Exec("INSERT INTO users (name) VALUES (?)", name)
    if err != nil {
        log.Fatal(err)
    }
}

if err := tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Best Practices

1. **Luôn kiểm tra error**: Ngay cả với Exec()
2. **Sử dụng parameterized queries**: Tránh SQL injection
3. **Sử dụng transactions cho batch**: Tốt hơn cho performance
4. **Sử dụng Context**: Cho timeout và cancellation

---

*Xem thêm tại [go.dev/doc/database/change-data](https://go.dev/doc/database/change-data)*
