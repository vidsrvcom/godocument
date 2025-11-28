# Thực thi Giao dịch

## Giới thiệu

Transactions đảm bảo tính toàn vẹn dữ liệu bằng cách thực thi một nhóm các câu lệnh SQL như một đơn vị atomic - tất cả thành công hoặc tất cả thất bại.

## Cơ bản về Transactions

### Pattern cơ bản

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback() // Rollback nếu không commit

// Thực thi các operations
_, err = tx.Exec("INSERT INTO users (name) VALUES (?)", "John")
if err != nil {
    log.Fatal(err)
}

_, err = tx.Exec("INSERT INTO profiles (user_id, bio) VALUES (?, ?)", 1, "Developer")
if err != nil {
    log.Fatal(err)
}

// Commit transaction
if err := tx.Commit(); err != nil {
    log.Fatal(err)
}
```

### Với error handling hoàn chỉnh

```go
func TransferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // Trừ tiền từ tài khoản nguồn
    _, err = tx.Exec("UPDATE accounts SET balance = balance - ? WHERE id = ?", amount, fromID)
    if err != nil {
        return fmt.Errorf("debit failed: %w", err)
    }

    // Cộng tiền vào tài khoản đích
    _, err = tx.Exec("UPDATE accounts SET balance = balance + ? WHERE id = ?", amount, toID)
    if err != nil {
        return fmt.Errorf("credit failed: %w", err)
    }

    return tx.Commit()
}
```

## Với Context

```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

tx, err := db.BeginTx(ctx, nil)
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

_, err = tx.ExecContext(ctx, "INSERT INTO users (name) VALUES (?)", "John")
if err != nil {
    log.Fatal(err)
}

if err := tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Transaction Options

### Isolation Levels

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{
    Isolation: sql.LevelSerializable,
    ReadOnly:  false,
})
```

### Các Isolation Levels

| Level | Description |
|-------|-------------|
| `sql.LevelDefault` | Driver default |
| `sql.LevelReadUncommitted` | Cho phép dirty reads |
| `sql.LevelReadCommitted` | Chỉ đọc committed data |
| `sql.LevelRepeatableRead` | Consistent reads trong transaction |
| `sql.LevelSerializable` | Full isolation |

### Read-only Transaction

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{
    ReadOnly: true,
})
```

## Prepared Statements trong Transaction

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

stmt, err := tx.Prepare("INSERT INTO items (name, price) VALUES (?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

items := []Item{
    {Name: "Item 1", Price: 10.00},
    {Name: "Item 2", Price: 20.00},
}

for _, item := range items {
    _, err = stmt.Exec(item.Name, item.Price)
    if err != nil {
        log.Fatal(err)
    }
}

if err := tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Helper Function

```go
func WithTransaction(db *sql.DB, fn func(tx *sql.Tx) error) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    if err := fn(tx); err != nil {
        return err
    }

    return tx.Commit()
}

// Sử dụng
err := WithTransaction(db, func(tx *sql.Tx) error {
    _, err := tx.Exec("INSERT INTO users (name) VALUES (?)", "John")
    if err != nil {
        return err
    }
    _, err = tx.Exec("INSERT INTO profiles (user_id) VALUES (?)", 1)
    return err
})
```

## Best Practices

1. **Luôn defer Rollback()**: Đảm bảo cleanup
2. **Giữ transactions ngắn**: Tránh lock conflicts
3. **Sử dụng Context**: Cho timeout
4. **Chọn Isolation Level phù hợp**: Balance performance và consistency
5. **Handle errors đầy đủ**: Mỗi operation trong transaction

---

*Xem thêm tại [go.dev/doc/database/execute-transactions](https://go.dev/doc/database/execute-transactions)*
