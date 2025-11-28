# Truy vấn dữ liệu

Khi thực thi câu lệnh SQL trả về dữ liệu, sử dụng một trong các query method được cung cấp bởi package `database/sql`. Mỗi method được thiết kế cho các trường hợp sử dụng khác nhau.

## Các method truy vấn

Package `database/sql` cung cấp hai method chính để truy vấn:

- **`Query`/`QueryContext`**: Để truy vấn trả về nhiều rows
- **`QueryRow`/`QueryRowContext`**: Để truy vấn trả về tối đa một row

## Query cho nhiều rows

Sử dụng `Query` hoặc `QueryContext` khi truy vấn của bạn có thể trả về không hoặc nhiều rows:

```go
rows, err := db.Query("SELECT id, name, email FROM users WHERE age > ?", 18)
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
    fmt.Printf("ID: %d, Name: %s, Email: %s\n", id, name, email)
}

// Kiểm tra lỗi sau khi iteration
if err = rows.Err(); err != nil {
    log.Fatal(err)
}
```

### Quan trọng về Rows

- **Luôn close `rows`**: Sử dụng `defer rows.Close()` để đảm bảo giải phóng tài nguyên
- **Kiểm tra `rows.Err()`**: Sau khi iteration, kiểm tra lỗi có thể xảy ra trong quá trình
- **Scan theo thứ tự**: Các biến trong `Scan()` phải khớp với thứ tự columns trong SELECT

## QueryRow cho một row

Sử dụng `QueryRow` hoặc `QueryRowContext` khi bạn mong đợi truy vấn trả về tối đa một row:

```go
var name, email string
var age int

err := db.QueryRow("SELECT name, email, age FROM users WHERE id = ?", 123).
    Scan(&name, &email, &age)

if err != nil {
    if err == sql.ErrNoRows {
        fmt.Println("No user found with that ID")
    } else {
        log.Fatal(err)
    }
}

fmt.Printf("User: %s (%s), Age: %d\n", name, email, age)
```

### Xử lý không tìm thấy kết quả

`QueryRow` trả về `sql.ErrNoRows` nếu không tìm thấy row nào:

```go
err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
switch {
case err == sql.ErrNoRows:
    fmt.Println("No user with that ID")
case err != nil:
    log.Fatal(err)
default:
    fmt.Printf("Username is %s\n", name)
}
```

## Sử dụng Context

Sử dụng các method với Context để có control tốt hơn:

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var id int
    var name string
    if err := rows.Scan(&id, &name); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("ID: %d, Name: %s\n", id, name)
}
```

## Scan vào Structs

Một pattern phổ biến là scan vào structs:

```go
type User struct {
    ID    int
    Name  string
    Email string
    Age   int
}

func getUser(db *sql.DB, id int) (*User, error) {
    user := &User{}
    
    err := db.QueryRow(
        "SELECT id, name, email, age FROM users WHERE id = ?", 
        id,
    ).Scan(&user.ID, &user.Name, &user.Email, &user.Age)
    
    if err != nil {
        return nil, err
    }
    return user, nil
}

func getAllUsers(db *sql.DB) ([]*User, error) {
    rows, err := db.Query("SELECT id, name, email, age FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []*User
    for rows.Next() {
        user := &User{}
        if err := rows.Scan(&user.ID, &user.Name, &user.Email, &user.Age); err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    
    if err = rows.Err(); err != nil {
        return nil, err
    }
    
    return users, nil
}
```

## Xử lý NULL values

Sử dụng các kiểu `sql.Null*` để xử lý NULL values:

```go
var name string
var email sql.NullString  // Có thể là NULL

err := db.QueryRow("SELECT name, email FROM users WHERE id = ?", id).
    Scan(&name, &email)

if err != nil {
    log.Fatal(err)
}

if email.Valid {
    fmt.Printf("Email: %s\n", email.String)
} else {
    fmt.Println("Email is NULL")
}
```

Các kiểu `sql.Null*` có sẵn:
- `sql.NullString`
- `sql.NullBool`
- `sql.NullInt64`
- `sql.NullInt32`
- `sql.NullFloat64`
- `sql.NullTime`

## Truy vấn với nhiều parameters

```go
rows, err := db.Query(`
    SELECT id, name, email 
    FROM users 
    WHERE age > ? AND city = ? AND active = ?
    ORDER BY name
    LIMIT ?
`, 18, "Hanoi", true, 10)
```

## Join queries

```go
rows, err := db.Query(`
    SELECT u.id, u.name, o.order_id, o.total
    FROM users u
    INNER JOIN orders o ON u.id = o.user_id
    WHERE u.age > ?
`, 18)
if err != nil {
    log.Fatal(err)
}
defer rows.Close()

for rows.Next() {
    var userID int
    var userName string
    var orderID int
    var total float64
    
    if err := rows.Scan(&userID, &userName, &orderID, &total); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User %s (ID: %d) - Order #%d: $%.2f\n", 
        userName, userID, orderID, total)
}
```

## Thực hành tốt nhất

1. **Luôn close rows**: Sử dụng `defer rows.Close()` ngay sau khi mở
2. **Kiểm tra rows.Err()**: Sau khi loop qua tất cả rows
3. **Sử dụng QueryRow cho single row**: Đơn giản hơn và hiệu quả hơn
4. **Sử dụng Context với timeout**: Cho các queries có thể chạy lâu
5. **Xử lý NULL values đúng cách**: Sử dụng `sql.Null*` types
6. **Sử dụng parameterized queries**: Tránh SQL injection

## Lỗi phổ biến

```go
// ❌ Quên close rows
rows, _ := db.Query("SELECT * FROM users")
for rows.Next() { /* ... */ }  // Memory leak!

// ✅ Đúng
rows, _ := db.Query("SELECT * FROM users")
defer rows.Close()
for rows.Next() { /* ... */ }

// ❌ Không kiểm tra rows.Err()
for rows.Next() { /* ... */ }
// Có thể có lỗi không được phát hiện

// ✅ Đúng
for rows.Next() { /* ... */ }
if err := rows.Err(); err != nil {
    log.Fatal(err)
}
```

## Xem thêm

- [Thực thi câu lệnh SQL](executing-statements.md)
- [Sử dụng prepared statements](prepared-statements.md)
- [Xử lý lỗi](error-handling.md)
- [Tránh SQL injection](sql-injection.md)
