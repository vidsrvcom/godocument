# Truy vấn dữ liệu

## Giới thiệu

Go cung cấp hai phương thức chính để truy vấn dữ liệu: `db.Query()` cho nhiều rows và `db.QueryRow()` cho một row.

## Query nhiều rows

### Sử dụng db.Query()

```go
rows, err := db.Query("SELECT id, name, email FROM users WHERE active = ?", true)
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var id int
    var name, email string
    if err := rows.Scan(&id, &name, &email); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%d: %s <%s>\n", id, name, email)
}

// Kiểm tra lỗi sau khi iteration
if err := rows.Err(); err != nil {
    log.Fatal(err)
}
```

### Với struct

```go
type User struct {
    ID    int
    Name  string
    Email string
}

func GetUsers(db *sql.DB) ([]User, error) {
    rows, err := db.Query("SELECT id, name, email FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var u User
        if err := rows.Scan(&u.ID, &u.Name, &u.Email); err != nil {
            return nil, err
        }
        users = append(users, u)
    }
    return users, rows.Err()
}
```

## Query một row

### Sử dụng db.QueryRow()

```go
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = ?", 1).Scan(&name)
if err != nil {
    if err == sql.ErrNoRows {
        fmt.Println("No user found")
    } else {
        log.Fatal(err)
    }
}
fmt.Printf("User name: %s\n", name)
```

### Với struct

```go
func GetUserByID(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id).
        Scan(&u.ID, &u.Name, &u.Email)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, nil // User not found
        }
        return nil, err
    }
    return &u, nil
}
```

## Xử lý NULL values

### Sử dụng sql.NullString, sql.NullInt64

```go
var name sql.NullString
err := db.QueryRow("SELECT nickname FROM users WHERE id = ?", 1).Scan(&name)
if err != nil {
    log.Fatal(err)
}

if name.Valid {
    fmt.Printf("Nickname: %s\n", name.String)
} else {
    fmt.Println("Nickname is NULL")
}
```

### Sử dụng pointers

```go
var nickname *string
err := db.QueryRow("SELECT nickname FROM users WHERE id = ?", 1).Scan(&nickname)
if err != nil {
    log.Fatal(err)
}

if nickname != nil {
    fmt.Printf("Nickname: %s\n", *nickname)
}
```

## Với Context

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()
```

## Column metadata

```go
rows, err := db.Query("SELECT * FROM users")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

columns, err := rows.Columns()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Columns: %v\n", columns)
```

## Best Practices

1. **Luôn defer rows.Close()**: Tránh connection leak
2. **Kiểm tra rows.Err()**: Sau khi iteration
3. **Sử dụng sql.ErrNoRows**: Cho QueryRow()
4. **Xử lý NULL values**: Với sql.Null* types hoặc pointers
5. **Sử dụng Context**: Cho timeout

---

*Xem thêm tại [go.dev/doc/database/querying](https://go.dev/doc/database/querying)*
