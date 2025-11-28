# Thực thi các câu lệnh SQL không trả về dữ liệu

Khi bạn thực thi các câu lệnh cơ sở dữ liệu không trả về dữ liệu, sử dụng một method `Exec` hoặc `ExecContext` từ package `database/sql`. Các câu lệnh như vậy bao gồm `INSERT`, `DELETE`, và `UPDATE`.

## Method ExecContext

Khi bạn thực thi câu lệnh SQL có thể mất thời gian, hãy sử dụng `ExecContext`. Method này cho phép bạn chỉ định timeout hoặc deadline cho query, và hủy nó nếu cần thiết.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

result, err := db.ExecContext(ctx, "INSERT INTO users (name, email) VALUES (?, ?)", 
    "John Doe", "john@example.com")
if err != nil {
    log.Fatal(err)
}
```

## Method Exec

Sử dụng `Exec` khi bạn không cần timeout hoặc hủy query:

```go
result, err := db.Exec("UPDATE users SET email = ? WHERE id = ?", 
    "newemail@example.com", 123)
if err != nil {
    log.Fatal(err)
}
```

## Kiểm tra kết quả

Method `Exec` và `ExecContext` trả về `sql.Result` chứa thông tin về câu lệnh đã thực thi:

```go
result, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", 
    "Jane Smith", "jane@example.com")
if err != nil {
    log.Fatal(err)
}

// Lấy ID của row vừa insert (nếu database hỗ trợ)
id, err := result.LastInsertId()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Inserted record ID: %d\n", id)

// Lấy số lượng rows bị ảnh hưởng
rowsAffected, err := result.RowsAffected()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Rows affected: %d\n", rowsAffected)
```

## Các ví dụ

### INSERT

```go
// Insert một record
result, err := db.Exec(
    "INSERT INTO products (name, price, stock) VALUES (?, ?, ?)",
    "Laptop", 999.99, 10,
)
if err != nil {
    log.Fatal(err)
}

id, _ := result.LastInsertId()
fmt.Printf("Created product with ID: %d\n", id)
```

### UPDATE

```go
// Update một record
result, err := db.Exec(
    "UPDATE products SET price = ? WHERE id = ?",
    899.99, 123,
)
if err != nil {
    log.Fatal(err)
}

rowsAffected, _ := result.RowsAffected()
if rowsAffected == 0 {
    fmt.Println("No product found with that ID")
} else {
    fmt.Printf("Updated %d product(s)\n", rowsAffected)
}
```

### DELETE

```go
// Delete một record
result, err := db.Exec("DELETE FROM products WHERE id = ?", 123)
if err != nil {
    log.Fatal(err)
}

rowsAffected, _ := result.RowsAffected()
fmt.Printf("Deleted %d product(s)\n", rowsAffected)
```

## Sử dụng Parameters

Luôn sử dụng parameterized queries để tránh SQL injection:

```go
// ✅ ĐÚNG - Sử dụng parameters
db.Exec("INSERT INTO users (name) VALUES (?)", userName)

// ❌ SAI - Dễ bị SQL injection
db.Exec(fmt.Sprintf("INSERT INTO users (name) VALUES ('%s')", userName))
```

## Batch Operations

Khi cần thực thi nhiều câu lệnh, xem xét sử dụng transactions để cải thiện hiệu suất:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback() // Rollback nếu commit không thành công

for _, user := range users {
    _, err := tx.Exec("INSERT INTO users (name, email) VALUES (?, ?)", 
        user.Name, user.Email)
    if err != nil {
        return err
    }
}

if err = tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Xử lý lỗi

Các lỗi phổ biến khi thực thi statements:

- **Syntax errors**: Câu lệnh SQL không hợp lệ
- **Constraint violations**: Vi phạm ràng buộc dữ liệu (unique, foreign key, etc.)
- **Connection errors**: Mất kết nối với database
- **Permission errors**: Không đủ quyền thực thi câu lệnh

```go
result, err := db.Exec("INSERT INTO users (id, name) VALUES (?, ?)", 1, "John")
if err != nil {
    // Kiểm tra loại lỗi cụ thể
    if strings.Contains(err.Error(), "Duplicate entry") {
        fmt.Println("User with this ID already exists")
    } else {
        log.Fatal(err)
    }
}
```

## Thực hành tốt nhất

1. **Sử dụng ExecContext với timeout** cho các operations quan trọng
2. **Kiểm tra RowsAffected** để verify operation thành công
3. **Sử dụng transactions** cho multiple related operations
4. **Luôn sử dụng parameterized queries** để tránh SQL injection
5. **Xử lý lỗi một cách thích hợp** và cung cấp thông báo rõ ràng

## Xem thêm

- [Truy vấn dữ liệu](querying-data.md)
- [Sử dụng prepared statements](prepared-statements.md)
- [Thực thi giao dịch](transactions.md)
- [Tránh SQL injection](sql-injection.md)
