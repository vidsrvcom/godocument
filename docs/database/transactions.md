# Thực thi Giao dịch (Transactions)

Một transaction là một chuỗi các câu lệnh SQL được thực thi như một đơn vị công việc duy nhất. Tất cả các câu lệnh trong transaction đều thành công hoặc tất cả đều thất bại, đảm bảo tính toàn vẹn dữ liệu.

## Tại sao sử dụng Transactions?

Transactions đảm bảo các thuộc tính ACID:

- **Atomicity** (Nguyên tử): Tất cả operations thành công hoặc không có gì thay đổi
- **Consistency** (Nhất quán): Database luôn ở trạng thái hợp lệ
- **Isolation** (Cô lập): Transactions không ảnh hưởng lẫn nhau
- **Durability** (Bền vững): Khi commit, thay đổi được lưu vĩnh viễn

## Tạo Transaction

Sử dụng `Begin` để bắt đầu transaction:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()  // Rollback nếu không commit

// Thực hiện các operations
_, err = tx.Exec("INSERT INTO accounts (name, balance) VALUES (?, ?)", "Alice", 1000)
if err != nil {
    return err  // tx.Rollback() sẽ được gọi bởi defer
}

_, err = tx.Exec("INSERT INTO accounts (name, balance) VALUES (?, ?)", "Bob", 500)
if err != nil {
    return err
}

// Commit transaction
if err = tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## BeginTx với Context và Options

Sử dụng `BeginTx` để có control tốt hơn:

```go
ctx := context.Background()

tx, err := db.BeginTx(ctx, &sql.TxOptions{
    Isolation: sql.LevelSerializable,
    ReadOnly:  false,
})
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

// ... thực hiện operations ...

if err = tx.Commit(); err != nil {
    log.Fatal(err)
}
```

### Isolation Levels

- `LevelDefault`: Mức mặc định của database
- `LevelReadUncommitted`: Có thể đọc uncommitted data
- `LevelReadCommitted`: Chỉ đọc committed data
- `LevelRepeatableRead`: Đọc nhất quán trong transaction
- `LevelSerializable`: Isolation cao nhất

## Ví dụ: Chuyển tiền

```go
func transferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // Trừ tiền từ tài khoản nguồn
    result, err := tx.Exec(
        "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
        amount, fromID, amount,
    )
    if err != nil {
        return err
    }

    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return err
    }
    if rowsAffected == 0 {
        return fmt.Errorf("insufficient balance or account not found")
    }

    // Cộng tiền vào tài khoản đích
    _, err = tx.Exec(
        "UPDATE accounts SET balance = balance + ? WHERE id = ?",
        amount, toID,
    )
    if err != nil {
        return err
    }

    // Commit nếu tất cả thành công
    return tx.Commit()
}
```

## Rollback và Commit

### Rollback

Hủy tất cả thay đổi trong transaction:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}

_, err = tx.Exec("INSERT INTO users (name) VALUES (?)", "John")
if err != nil {
    tx.Rollback()
    log.Fatal(err)
}

// Quyết định rollback
if someCondition {
    tx.Rollback()
    return errors.New("transaction cancelled")
}

tx.Commit()
```

### Commit

Lưu tất cả thay đổi vĩnh viễn:

```go
tx, err := db.Begin()
// ... operations ...

if err := tx.Commit(); err != nil {
    return fmt.Errorf("commit failed: %w", err)
}
```

## Pattern với defer

Pattern phổ biến để đảm bảo rollback:

```go
func doTransaction(db *sql.DB) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    
    // defer sẽ rollback nếu chưa commit
    defer tx.Rollback()
    
    // Thực hiện operations
    if err := doOperation1(tx); err != nil {
        return err  // Rollback tự động
    }
    
    if err := doOperation2(tx); err != nil {
        return err  // Rollback tự động
    }
    
    // Commit khi tất cả thành công
    // Rollback sau commit là no-op
    return tx.Commit()
}
```

## Query và Exec trong Transaction

Tất cả database operations có thể thực hiện trong transaction:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

// Query
rows, err := tx.Query("SELECT id, balance FROM accounts WHERE balance > ?", 100)
if err != nil {
    return err
}
defer rows.Close()

// QueryRow
var balance float64
err = tx.QueryRow("SELECT balance FROM accounts WHERE id = ?", 1).Scan(&balance)
if err != nil {
    return err
}

// Exec
_, err = tx.Exec("UPDATE accounts SET balance = ? WHERE id = ?", balance+100, 1)
if err != nil {
    return err
}

tx.Commit()
```

## Prepared Statements trong Transactions

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

stmt, err := tx.Prepare("INSERT INTO logs (user_id, action) VALUES (?, ?)")
if err != nil {
    return err
}
defer stmt.Close()

actions := []string{"login", "view_profile", "update_settings"}
for _, action := range actions {
    _, err := stmt.Exec(userID, action)
    if err != nil {
        return err
    }
}

tx.Commit()
```

## Nested Transactions

Go không hỗ trợ nested transactions trực tiếp. Sử dụng savepoints nếu database hỗ trợ:

```go
tx, _ := db.Begin()
defer tx.Rollback()

// Tạo savepoint
tx.Exec("SAVEPOINT sp1")

// Operations
_, err := tx.Exec("INSERT INTO users (name) VALUES (?)", "Alice")
if err != nil {
    // Rollback đến savepoint
    tx.Exec("ROLLBACK TO SAVEPOINT sp1")
} else {
    // Release savepoint
    tx.Exec("RELEASE SAVEPOINT sp1")
}

tx.Commit()
```

## Xử lý lỗi trong Transactions

```go
func processOrders(db *sql.DB, orders []Order) error {
    tx, err := db.Begin()
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }
    defer tx.Rollback()
    
    for i, order := range orders {
        _, err := tx.Exec(
            "INSERT INTO orders (user_id, product_id, quantity) VALUES (?, ?, ?)",
            order.UserID, order.ProductID, order.Quantity,
        )
        if err != nil {
            return fmt.Errorf("insert order %d: %w", i, err)
        }
    }
    
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("commit transaction: %w", err)
    }
    
    return nil
}
```

## Timeout cho Transactions

```go
func doTransactionWithTimeout(db *sql.DB) error {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()
    
    // Operations sẽ bị cancel sau 5 giây
    _, err = tx.ExecContext(ctx, "INSERT INTO logs (message) VALUES (?)", "test")
    if err != nil {
        return err
    }
    
    return tx.Commit()
}
```

## Thực hành tốt nhất

1. **Luôn sử dụng defer tx.Rollback()**: Đảm bảo rollback khi có lỗi
2. **Giữ transactions ngắn**: Giảm lock contention
3. **Xử lý lỗi cẩn thận**: Kiểm tra errors sau mỗi operation
4. **Sử dụng Context với timeout**: Tránh transactions chạy mãi
5. **Không thực hiện I/O chậm trong transaction**: Giữ transaction nhanh
6. **Commit một lần duy nhất**: Ở cuối khi tất cả operations thành công

## Lỗi phổ biến

```go
// ❌ Quên defer rollback
tx, _ := db.Begin()
tx.Exec("INSERT INTO users (name) VALUES (?)", "John")
// Nếu có lỗi ở đây, transaction bị treo

// ✅ Đúng
tx, _ := db.Begin()
defer tx.Rollback()
tx.Exec("INSERT INTO users (name) VALUES (?)", "John")
tx.Commit()

// ❌ Commit trong loop
for _, item := range items {
    tx, _ := db.Begin()
    tx.Exec("INSERT INTO items (name) VALUES (?)", item)
    tx.Commit()  // Quá nhiều commits
}

// ✅ Đúng
tx, _ := db.Begin()
defer tx.Rollback()
for _, item := range items {
    tx.Exec("INSERT INTO items (name) VALUES (?)", item)
}
tx.Commit()  // Một commit duy nhất
```

## Xem thêm

- [Thực thi câu lệnh SQL](executing-statements.md)
- [Sử dụng prepared statements](prepared-statements.md)
- [Xử lý lỗi](error-handling.md)
- [Quản lý kết nối](connection-management.md)
