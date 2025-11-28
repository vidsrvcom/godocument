# Tránh các vấn đề SQL Injection

## Giới thiệu

SQL injection là một trong những lỗ hổng bảo mật phổ biến nhất. Go cung cấp các công cụ để viết các truy vấn SQL an toàn.

## Vấn đề SQL Injection

### Ví dụ về mã không an toàn

```go
// ❌ KHÔNG BAO GIỜ LÀM NHƯ VẦY
func GetUserUnsafe(db *sql.DB, username string) (*User, error) {
    query := fmt.Sprintf("SELECT * FROM users WHERE username = '%s'", username)
    row := db.QueryRow(query)
    // ...
}

// Attacker có thể input: ' OR '1'='1
// Query trở thành: SELECT * FROM users WHERE username = '' OR '1'='1'
```

## Sử dụng Parameterized Queries

### Cách an toàn

```go
// ✅ SỬ DỤNG PARAMETERIZED QUERIES
func GetUserSafe(db *sql.DB, username string) (*User, error) {
    var u User
    err := db.QueryRow("SELECT id, username, email FROM users WHERE username = ?", username).
        Scan(&u.ID, &u.Username, &u.Email)
    return &u, err
}
```

### Với nhiều parameters

```go
rows, err := db.Query(
    "SELECT * FROM products WHERE category = ? AND price < ? AND active = ?",
    category, maxPrice, true,
)
```

## Placeholder Syntax

### MySQL, SQLite

```go
// Sử dụng ?
db.Query("SELECT * FROM users WHERE id = ?", id)
```

### PostgreSQL

```go
// Sử dụng $1, $2, $3...
db.Query("SELECT * FROM users WHERE id = $1 AND status = $2", id, status)
```

### SQL Server

```go
// Sử dụng @p1, @p2... hoặc ?
db.Query("SELECT * FROM users WHERE id = @p1", id)
```

## Prepared Statements

```go
stmt, err := db.Prepare("INSERT INTO users (username, email) VALUES (?, ?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close()

// Mỗi lần thực thi đều an toàn
stmt.Exec(username1, email1)
stmt.Exec(username2, email2)
```

## IN Clauses

### Vấn đề với IN

```go
// ❌ KHÔNG LÀM NHƯ VẦY
ids := []int{1, 2, 3}
query := fmt.Sprintf("SELECT * FROM users WHERE id IN (%s)", strings.Join(ids, ","))
```

### Giải pháp an toàn

```go
// ✅ BUILD PLACEHOLDERS ĐỘNG
func GetUsersByIDs(db *sql.DB, ids []int) ([]User, error) {
    if len(ids) == 0 {
        return nil, nil
    }
    
    // Tạo placeholders: ?, ?, ?
    placeholders := make([]string, len(ids))
    args := make([]interface{}, len(ids))
    for i, id := range ids {
        placeholders[i] = "?"
        args[i] = id
    }
    
    query := fmt.Sprintf("SELECT * FROM users WHERE id IN (%s)", 
        strings.Join(placeholders, ","))
    
    rows, err := db.Query(query, args...)
    // ...
}
```

### Với sqlx

```go
import "github.com/jmoiron/sqlx"

ids := []int{1, 2, 3}
query, args, err := sqlx.In("SELECT * FROM users WHERE id IN (?)", ids)
if err != nil {
    log.Fatal(err)
}
rows, err := db.Query(query, args...)
```

## LIKE Clauses

```go
// ✅ AN TOÀN - wildcard trong parameter
searchTerm := "%" + userInput + "%"
rows, err := db.Query("SELECT * FROM users WHERE name LIKE ?", searchTerm)

// Escape special characters nếu cần
func EscapeLike(s string) string {
    s = strings.ReplaceAll(s, "\\", "\\\\")
    s = strings.ReplaceAll(s, "%", "\\%")
    s = strings.ReplaceAll(s, "_", "\\_")
    return s
}
```

## ORDER BY và Column Names

```go
// ❌ CẨN THẬN - không thể parameterize column names
func GetUsers(db *sql.DB, orderBy string) ([]User, error) {
    // orderBy không thể dùng ?
    query := fmt.Sprintf("SELECT * FROM users ORDER BY %s", orderBy)
    // Attacker có thể inject: "id; DROP TABLE users;--"
}

// ✅ SỬ DỤNG WHITELIST
func GetUsersSafe(db *sql.DB, orderBy string) ([]User, error) {
    allowedColumns := map[string]bool{
        "id": true, "name": true, "created_at": true,
    }
    
    if !allowedColumns[orderBy] {
        orderBy = "id" // default
    }
    
    query := fmt.Sprintf("SELECT * FROM users ORDER BY %s", orderBy)
    return db.Query(query)
}
```

## Best Practices

1. **Luôn sử dụng parameterized queries**: Không bao giờ nối string
2. **Sử dụng prepared statements**: Cho queries lặp lại
3. **Whitelist column names**: Cho ORDER BY, GROUP BY
4. **Escape LIKE wildcards**: Nếu cần tìm kiếm literal characters
5. **Validate input**: Thêm layer validation
6. **Sử dụng ORM**: Nhiều ORMs handle escaping tự động

---

*Xem thêm tại [go.dev/doc/database/sql-injection](https://go.dev/doc/database/sql-injection)*
