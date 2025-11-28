# Tránh SQL Injection

SQL injection là một trong những lỗ hổng bảo mật nghiêm trọng nhất trong ứng dụng web. Tài liệu này hướng dẫn cách viết mã Go an toàn để tránh SQL injection khi làm việc với cơ sở dữ liệu.

## SQL Injection là gì?

SQL injection xảy ra khi kẻ tấn công có thể chèn hoặc "inject" mã SQL độc hại vào câu truy vấn của bạn, cho phép chúng:

- Đọc dữ liệu nhạy cảm
- Sửa đổi hoặc xóa dữ liệu
- Thực thi các lệnh quản trị
- Bypass authentication
- Trong một số trường hợp, thực thi lệnh trên operating system

## Ví dụ về SQL Injection

### ❌ Code không an toàn

```go
// NGUY HIỂM - Dễ bị SQL injection
func getUser(db *sql.DB, username string) error {
    query := fmt.Sprintf("SELECT * FROM users WHERE username = '%s'", username)
    rows, err := db.Query(query)
    // ...
}

// Nếu username = "admin' OR '1'='1"
// Query trở thành: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// Trả về tất cả users!
```

### ✅ Code an toàn

```go
// AN TOÀN - Sử dụng parameterized query
func getUser(db *sql.DB, username string) error {
    rows, err := db.Query("SELECT * FROM users WHERE username = ?", username)
    // ...
}
```

## Sử dụng Parameterized Queries

Cách chính xác để tránh SQL injection trong Go là luôn sử dụng parameterized queries (prepared statements):

### Query

```go
// ✅ ĐÚNG
func getUsers(db *sql.DB, minAge int) ([]*User, error) {
    rows, err := db.Query("SELECT id, name FROM users WHERE age > ?", minAge)
    // ...
}

// ❌ SAI
func getUsers(db *sql.DB, minAge int) ([]*User, error) {
    query := fmt.Sprintf("SELECT id, name FROM users WHERE age > %d", minAge)
    rows, err := db.Query(query)
    // ...
}
```

### QueryRow

```go
// ✅ ĐÚNG
func getUserByEmail(db *sql.DB, email string) (*User, error) {
    var user User
    err := db.QueryRow("SELECT id, name FROM users WHERE email = ?", email).
        Scan(&user.ID, &user.Name)
    // ...
}

// ❌ SAI
func getUserByEmail(db *sql.DB, email string) (*User, error) {
    query := "SELECT id, name FROM users WHERE email = '" + email + "'"
    // ...
}
```

### Exec

```go
// ✅ ĐÚNG
func createUser(db *sql.DB, name, email string) error {
    _, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", name, email)
    return err
}

// ❌ SAI
func createUser(db *sql.DB, name, email string) error {
    query := fmt.Sprintf("INSERT INTO users (name, email) VALUES ('%s', '%s')", name, email)
    _, err := db.Exec(query)
    return err
}
```

## Multiple Parameters

```go
// ✅ ĐÚNG - Multiple parameters
func searchUsers(db *sql.DB, name, city string, minAge, maxAge int) ([]*User, error) {
    rows, err := db.Query(`
        SELECT id, name, email 
        FROM users 
        WHERE name LIKE ? 
          AND city = ? 
          AND age BETWEEN ? AND ?
    `, "%"+name+"%", city, minAge, maxAge)
    // ...
}
```

## IN Clause

Xử lý IN clause cần cẩn thận:

```go
// ✅ ĐÚNG - Sử dụng parameterized query cho IN clause
func getUsersByIDs(db *sql.DB, ids []int) ([]*User, error) {
    if len(ids) == 0 {
        return []*User{}, nil
    }
    
    // Tạo placeholders: "?, ?, ?"
    placeholders := make([]string, len(ids))
    args := make([]interface{}, len(ids))
    for i, id := range ids {
        placeholders[i] = "?"
        args[i] = id
    }
    
    query := fmt.Sprintf("SELECT id, name FROM users WHERE id IN (%s)",
        strings.Join(placeholders, ", "))
    
    rows, err := db.Query(query, args...)
    // ...
}

// ❌ SAI
func getUsersByIDs(db *sql.DB, ids []int) ([]*User, error) {
    idsStr := strings.Trim(strings.Join(strings.Fields(fmt.Sprint(ids)), ", "), "[]")
    query := fmt.Sprintf("SELECT id, name FROM users WHERE id IN (%s)", idsStr)
    rows, err := db.Query(query)
    // ...
}
```

## Dynamic Table/Column Names

Table và column names KHÔNG THỂ sử dụng parameters. Cần whitelist:

```go
// ✅ ĐÚNG - Whitelist cho table/column names
func getRecords(db *sql.DB, tableName string) (*sql.Rows, error) {
    // Whitelist các table names hợp lệ
    validTables := map[string]bool{
        "users":    true,
        "products": true,
        "orders":   true,
    }
    
    if !validTables[tableName] {
        return nil, fmt.Errorf("invalid table name")
    }
    
    // An toàn sau khi validate
    query := fmt.Sprintf("SELECT * FROM %s", tableName)
    return db.Query(query)
}

func sortUsers(db *sql.DB, sortBy string) ([]*User, error) {
    // Whitelist các column names hợp lệ
    validColumns := map[string]bool{
        "name":       true,
        "email":      true,
        "created_at": true,
    }
    
    if !validColumns[sortBy] {
        sortBy = "created_at"  // Default
    }
    
    query := fmt.Sprintf("SELECT id, name FROM users ORDER BY %s", sortBy)
    rows, err := db.Query(query)
    // ...
}

// ❌ SAI - Không validate input
func getRecords(db *sql.DB, tableName string) (*sql.Rows, error) {
    query := fmt.Sprintf("SELECT * FROM %s", tableName)
    return db.Query(query)
}
```

## LIKE Queries

Cẩn thận với LIKE queries:

```go
// ✅ ĐÚNG - Escape special characters trong LIKE
func searchUsers(db *sql.DB, searchTerm string) ([]*User, error) {
    // Escape các special characters: %, _
    searchTerm = strings.ReplaceAll(searchTerm, "%", "\\%")
    searchTerm = strings.ReplaceAll(searchTerm, "_", "\\_")
    
    pattern := "%" + searchTerm + "%"
    
    rows, err := db.Query("SELECT id, name FROM users WHERE name LIKE ?", pattern)
    // ...
}
```

## Prepared Statements

Prepared statements cũng bảo vệ khỏi SQL injection:

```go
// ✅ ĐÚNG - Sử dụng prepared statements
func batchInsert(db *sql.DB, users []User) error {
    stmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
    if err != nil {
        return err
    }
    defer stmt.Close()
    
    for _, user := range users {
        _, err := stmt.Exec(user.Name, user.Email)
        if err != nil {
            return err
        }
    }
    return nil
}
```

## JSON và User Input

Cẩn thận khi xử lý JSON input:

```go
type SearchRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

// ✅ ĐÚNG
func searchUsersHandler(db *sql.DB, req SearchRequest) ([]*User, error) {
    rows, err := db.Query(`
        SELECT id, name, email 
        FROM users 
        WHERE name = ? AND email = ? AND age = ?
    `, req.Name, req.Email, req.Age)
    // ...
}

// ❌ SAI
func searchUsersHandler(db *sql.DB, req SearchRequest) ([]*User, error) {
    query := fmt.Sprintf(`
        SELECT id, name, email 
        FROM users 
        WHERE name = '%s' AND email = '%s' AND age = %d
    `, req.Name, req.Email, req.Age)
    rows, err := db.Query(query)
    // ...
}
```

## ORM và Query Builders

Sử dụng ORM hoặc query builders an toàn:

```go
// Ví dụ với GORM (ORM phổ biến)
import "gorm.io/gorm"

// ✅ ĐÚNG - GORM tự động escape
func getUser(db *gorm.DB, username string) (*User, error) {
    var user User
    result := db.Where("username = ?", username).First(&user)
    return &user, result.Error
}

// Ví dụ với squirrel (Query builder)
import sq "github.com/Masterminds/squirrel"

// ✅ ĐÚNG - Squirrel tạo parameterized queries
func getUsers(db *sql.DB, minAge int) ([]*User, error) {
    sql, args, err := sq.Select("id", "name").
        From("users").
        Where(sq.Gt{"age": minAge}).
        ToSql()
    
    if err != nil {
        return nil, err
    }
    
    rows, err := db.Query(sql, args...)
    // ...
}
```

## Testing cho SQL Injection

Test code của bạn với các input nguy hiểm:

```go
func TestSQLInjection(t *testing.T) {
    db := setupTestDB(t)
    
    dangerousInputs := []string{
        "admin' OR '1'='1",
        "'; DROP TABLE users; --",
        "admin'--",
        "' OR 1=1 --",
        "admin' /*",
    }
    
    for _, input := range dangerousInputs {
        _, err := getUser(db, input)
        // Không nên panic hoặc trả về tất cả users
        // Nên trả về "not found" hoặc error
    }
}
```

## Checklist An Toàn

- [ ] Luôn sử dụng parameterized queries (`?` placeholders)
- [ ] KHÔNG BAO GIỜ nối chuỗi để tạo SQL queries
- [ ] Validate và whitelist table/column names
- [ ] Escape special characters trong LIKE queries
- [ ] Sử dụng prepared statements cho batch operations
- [ ] Validate tất cả user input
- [ ] Sử dụng ORM hoặc query builders an toàn
- [ ] Test với malicious input
- [ ] Review code thường xuyên
- [ ] Sử dụng static analysis tools

## Tools để phát hiện SQL Injection

```bash
# Sử dụng gosec để scan code
go install github.com/securego/gosec/v2/cmd/gosec@latest
gosec ./...

# Tìm kiếm string concatenation trong SQL
grep -r "fmt.Sprintf.*SELECT" .
grep -r "\".*\" + .*" . | grep -i "select\|insert\|update\|delete"
```

## Thực hành tốt nhất

1. **Luôn sử dụng parameterized queries**: Đây là line of defense đầu tiên
2. **Validate input**: Kiểm tra type, length, format
3. **Principle of least privilege**: Database user chỉ có quyền cần thiết
4. **Use whitelist**: Cho table/column names động
5. **Escape special characters**: Trong LIKE queries
6. **Code review**: Có người khác review database code
7. **Security testing**: Test với malicious input
8. **Keep dependencies updated**: Cập nhật drivers và libraries

## Xem thêm

- [Thực thi câu lệnh SQL](executing-statements.md)
- [Sử dụng prepared statements](prepared-statements.md)
- [Xử lý lỗi](error-handling.md)
- [Best Practices về Bảo mật](../security/best-practices.md)
