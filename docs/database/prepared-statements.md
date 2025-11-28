# Sử dụng Prepared Statements

Prepared statements là các câu lệnh SQL được biên dịch trước và có thể được thực thi nhiều lần với các parameters khác nhau. Chúng cung cấp:

- **Hiệu suất tốt hơn**: Câu lệnh được parse một lần, thực thi nhiều lần
- **Bảo mật cao hơn**: Tự động escape parameters, ngăn chặn SQL injection
- **Tái sử dụng**: Cùng một câu lệnh có thể dùng với parameters khác nhau

## Tạo Prepared Statement

Sử dụng method `Prepare` để tạo prepared statement:

```go
stmt, err := db.Prepare("SELECT id, name, email FROM users WHERE age > ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

// Sử dụng statement nhiều lần
rows, err := stmt.Query(18)
// ... xử lý rows ...

rows, err = stmt.Query(25)
// ... xử lý rows ...
```

## Prepare với Context

Sử dụng `PrepareContext` để có thể cancel hoặc timeout:

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

stmt, err := db.PrepareContext(ctx, "INSERT INTO users (name, email, age) VALUES (?, ?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()
```

## Thực thi Prepared Statements

### Query cho nhiều rows

```go
stmt, err := db.Prepare("SELECT id, name, email FROM users WHERE city = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

rows, err := stmt.Query("Hanoi")
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
```

### QueryRow cho một row

```go
stmt, err := db.Prepare("SELECT name, email FROM users WHERE id = ?")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

var name, email string
err = stmt.QueryRow(123).Scan(&name, &email)
if err != nil {
    if err == sql.ErrNoRows {
        fmt.Println("No user found")
    } else {
        log.Fatal(err)
    }
}
```

### Exec cho INSERT, UPDATE, DELETE

```go
stmt, err := db.Prepare("INSERT INTO users (name, email, age) VALUES (?, ?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

result, err := stmt.Exec("John Doe", "john@example.com", 30)
if err != nil {
    log.Fatal(err)
}

id, _ := result.LastInsertId()
fmt.Printf("Inserted user with ID: %d\n", id)
```

## Batch Insert với Prepared Statement

Prepared statements đặc biệt hiệu quả cho batch operations:

```go
stmt, err := db.Prepare("INSERT INTO products (name, price, stock) VALUES (?, ?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

products := []struct {
    Name  string
    Price float64
    Stock int
}{
    {"Laptop", 999.99, 10},
    {"Mouse", 29.99, 50},
    {"Keyboard", 79.99, 30},
}

for _, p := range products {
    _, err := stmt.Exec(p.Name, p.Price, p.Stock)
    if err != nil {
        log.Fatal(err)
    }
}
fmt.Printf("Inserted %d products\n", len(products))
```

## Prepared Statements trong Transactions

Kết hợp prepared statements với transactions để có hiệu suất và tính toàn vẹn dữ liệu tốt nhất:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()

stmt, err := tx.Prepare("INSERT INTO orders (user_id, product_id, quantity) VALUES (?, ?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

orders := []struct {
    UserID    int
    ProductID int
    Quantity  int
}{
    {1, 101, 2},
    {1, 102, 1},
    {2, 101, 3},
}

for _, order := range orders {
    _, err := stmt.Exec(order.UserID, order.ProductID, order.Quantity)
    if err != nil {
        return err
    }
}

if err = tx.Commit(); err != nil {
    log.Fatal(err)
}
```

## Statement từ Transaction

Một statement được prepare từ transaction chỉ có thể được sử dụng trong transaction đó:

```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}

// Statement này chỉ hoạt động trong tx
stmt, err := tx.Prepare("INSERT INTO logs (message) VALUES (?)")
if err != nil {
    tx.Rollback()
    log.Fatal(err)
}
defer stmt.Close()

_, err = stmt.Exec("Transaction started")
if err != nil {
    tx.Rollback()
    log.Fatal(err)
}

tx.Commit()
```

## Khi nào nên sử dụng Prepared Statements

### ✅ Nên sử dụng khi:

1. **Thực thi cùng query nhiều lần** với parameters khác nhau
2. **Batch operations**: Insert/update nhiều records
3. **Performance critical code**: Giảm overhead của parsing
4. **Bảo mật cao**: Tự động escape parameters

### ❌ Không cần thiết khi:

1. **Query chỉ chạy một lần**: Overhead của prepare không đáng kể
2. **Simple queries**: `db.Query()` đã đủ an toàn với parameters

## Ví dụ so sánh

### Không dùng Prepared Statement

```go
// Không hiệu quả cho nhiều inserts
for _, user := range users {
    db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", 
        user.Name, user.Email)
}
```

### Dùng Prepared Statement

```go
// Hiệu quả hơn
stmt, _ := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
defer stmt.Close()

for _, user := range users {
    stmt.Exec(user.Name, user.Email)
}
```

## Quản lý vòng đời Statement

Luôn close statements khi không dùng nữa:

```go
func insertUser(db *sql.DB, name, email string) error {
    stmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
    if err != nil {
        return err
    }
    defer stmt.Close()  // Quan trọng!
    
    _, err = stmt.Exec(name, email)
    return err
}
```

## Lỗi phổ biến

```go
// ❌ Quên close statement
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = ?")
stmt.Query(1)  // Memory leak!

// ✅ Đúng
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = ?")
defer stmt.Close()
stmt.Query(1)

// ❌ Prepare trong loop
for _, id := range ids {
    stmt, _ := db.Prepare("SELECT * FROM users WHERE id = ?")
    stmt.QueryRow(id).Scan(&name)
    stmt.Close()
}

// ✅ Đúng - Prepare một lần
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = ?")
defer stmt.Close()
for _, id := range ids {
    stmt.QueryRow(id).Scan(&name)
}
```

## Thực hành tốt nhất

1. **Luôn close statements**: Sử dụng `defer stmt.Close()`
2. **Prepare bên ngoài loops**: Tái sử dụng statement
3. **Kết hợp với transactions**: Cho batch operations
4. **Sử dụng Context**: Cho timeout và cancellation
5. **Xử lý lỗi đúng cách**: Kiểm tra errors khi prepare và execute

## Xem thêm

- [Thực thi câu lệnh SQL](executing-statements.md)
- [Truy vấn dữ liệu](querying-data.md)
- [Thực thi giao dịch](transactions.md)
- [Tránh SQL injection](sql-injection.md)
