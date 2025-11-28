# Sử dụng Prepared Statements

## Giới thiệu

Prepared statements cho phép bạn tạo một câu lệnh SQL một lần và thực thi nhiều lần với các tham số khác nhau, tăng hiệu suất và bảo mật.

## Tạo Prepared Statement

### Cơ bản

```go
stmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

// Thực thi nhiều lần
_, err = stmt.Exec("John", "john@example.com")
if err != nil {
    log.Fatal(err)
}

_, err = stmt.Exec("Jane", "jane@example.com")
if err != nil {
    log.Fatal(err)
}
```

### Với Query

```go
stmt, err := db.Prepare("SELECT id, name FROM users WHERE email = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

var id int
var name string
err = stmt.QueryRow("john@example.com").Scan(&id, &name)
if err != nil {
    log.Fatal(err)
}
```

## Batch Operations

### Inserting nhiều records

```go
func InsertUsers(db *sql.DB, users []User) error {
    stmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
    if err != nil {
        return err
    }
    defer stmt.Close()

    for _, u := range users {
        _, err := stmt.Exec(u.Name, u.Email)
        if err != nil {
            return err
        }
    }
    return nil
}
```

### Với Transaction

```go
func InsertUsersWithTx(db *sql.DB, users []User) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    stmt, err := tx.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
    if err != nil {
        return err
    }
    defer stmt.Close()

    for _, u := range users {
        _, err := stmt.Exec(u.Name, u.Email)
        if err != nil {
            return err
        }
    }

    return tx.Commit()
}
```

## Với Context

```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

stmt, err := db.PrepareContext(ctx, "SELECT id FROM users WHERE email = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

var id int
err = stmt.QueryRowContext(ctx, "john@example.com").Scan(&id)
```

## Named Statements (với sqlx)

```go
import "github.com/jmoiron/sqlx"

stmt, err := db.PrepareNamed("INSERT INTO users (name, email) VALUES (:name, :email)")
if err != nil {
    log.Fatal(err)
}

user := User{Name: "John", Email: "john@example.com"}
_, err = stmt.Exec(user)
```

## Khi nào sử dụng Prepared Statements

### Nên sử dụng khi:

- Thực thi cùng một query nhiều lần
- Batch insert/update
- Security là ưu tiên cao

### Không cần thiết khi:

- Query chỉ chạy một lần
- Sử dụng ORM (ORM tự handle)

## Performance Considerations

### Connection pooling

Prepared statements gắn với một connection. Với connection pool:

```go
// Mỗi lần stmt.Exec(), một connection được lấy từ pool
// Statement có thể được re-prepared trên connection khác
```

### Best practice

```go
// Sử dụng trong transaction để đảm bảo cùng connection
tx, _ := db.Begin()
stmt, _ := tx.Prepare("...")
// ... use stmt ...
tx.Commit()
```

## Best Practices

1. **Luôn defer stmt.Close()**: Tránh resource leak
2. **Sử dụng trong transactions**: Cho batch operations
3. **Không prepare statements quá sớm**: Chỉ khi cần
4. **Sử dụng Context**: Cho timeout và cancellation

---

*Xem thêm tại [go.dev/doc/database/prepared-statements](https://go.dev/doc/database/prepared-statements)*
