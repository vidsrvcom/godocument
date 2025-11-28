# Các Thực hành Tốt nhất về Bảo mật

## Giới thiệu

Tài liệu này hướng dẫn các thực hành tốt nhất để viết mã Go an toàn, bao gồm xác thực đầu vào, xử lý lỗi, và sử dụng các package mật mã.

## Input Validation

### Validate tất cả User Input

```go
func validateUsername(username string) error {
    if len(username) < 3 || len(username) > 50 {
        return errors.New("username must be 3-50 characters")
    }
    
    // Only allow alphanumeric and underscore
    matched, _ := regexp.MatchString(`^[a-zA-Z0-9_]+$`, username)
    if !matched {
        return errors.New("username contains invalid characters")
    }
    
    return nil
}
```

### Sanitize HTML

```go
import "html"

func sanitizeInput(input string) string {
    return html.EscapeString(input)
}

// Output: &lt;script&gt;alert(1)&lt;/script&gt;
```

### Validate Numbers

```go
func validateAge(age int) error {
    if age < 0 || age > 150 {
        return errors.New("invalid age")
    }
    return nil
}
```

### File Path Validation

```go
import "path/filepath"

func validatePath(userPath string) (string, error) {
    // Clean the path
    cleaned := filepath.Clean(userPath)
    
    // Ensure it doesn't escape base directory
    if !strings.HasPrefix(cleaned, "/safe/directory/") {
        return "", errors.New("path traversal detected")
    }
    
    return cleaned, nil
}
```

## SQL Injection Prevention

### Sử dụng Parameterized Queries

```go
// ✅ ĐÚNG: Parameterized query
db.Query("SELECT * FROM users WHERE id = ?", userID)

// ❌ SAI: String concatenation
db.Query("SELECT * FROM users WHERE id = " + userID)
```

### Với Named Parameters

```go
db.NamedQuery(`SELECT * FROM users WHERE name = :name`, 
    map[string]interface{}{"name": userName})
```

### ORM (GORM example)

```go
// ✅ ĐÚNG
db.Where("name = ?", userName).Find(&users)

// ❌ SAI
db.Where("name = '" + userName + "'").Find(&users)
```

## Password Handling

### Hashing với bcrypt

```go
import "golang.org/x/crypto/bcrypt"

func hashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

func checkPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
```

### Hashing với Argon2 (Recommended)

```go
import "golang.org/x/crypto/argon2"

func hashPasswordArgon2(password string) (string, error) {
    salt := make([]byte, 16)
    rand.Read(salt)
    
    hash := argon2.IDKey([]byte(password), salt, 1, 64*1024, 4, 32)
    
    // Encode salt + hash
    return base64.StdEncoding.EncodeToString(append(salt, hash...)), nil
}
```

### Password Requirements

```go
func validatePassword(password string) error {
    if len(password) < 12 {
        return errors.New("password must be at least 12 characters")
    }
    
    hasUpper := regexp.MustCompile(`[A-Z]`).MatchString(password)
    hasLower := regexp.MustCompile(`[a-z]`).MatchString(password)
    hasDigit := regexp.MustCompile(`[0-9]`).MatchString(password)
    hasSpecial := regexp.MustCompile(`[!@#$%^&*]`).MatchString(password)
    
    if !hasUpper || !hasLower || !hasDigit || !hasSpecial {
        return errors.New("password must contain upper, lower, digit, and special char")
    }
    
    return nil
}
```

## Cryptography

### Secure Random

```go
import "crypto/rand"

// ✅ ĐÚNG: crypto/rand for security
func generateToken() (string, error) {
    b := make([]byte, 32)
    _, err := rand.Read(b)
    if err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(b), nil
}

// ❌ SAI: math/rand is not cryptographically secure
// import "math/rand"
// rand.Read(b)
```

### Encryption (AES-GCM)

```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
)

func encrypt(plaintext, key []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    rand.Read(nonce)
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

func decrypt(ciphertext, key []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonceSize := gcm.NonceSize()
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    
    return gcm.Open(nil, nonce, ciphertext, nil)
}
```

## TLS/HTTPS

### HTTP Server với TLS

```go
import "crypto/tls"

tlsConfig := &tls.Config{
    MinVersion: tls.VersionTLS12,
    CipherSuites: []uint16{
        tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
        tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
    },
}

server := &http.Server{
    Addr:      ":443",
    TLSConfig: tlsConfig,
}
server.ListenAndServeTLS("cert.pem", "key.pem")
```

### HTTP Client với TLS Verification

```go
// ✅ ĐÚNG: Verify TLS (default)
client := &http.Client{}

// ❌ SAI: Skip TLS verification
// client := &http.Client{
//     Transport: &http.Transport{
//         TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
//     },
// }
```

## Error Handling

### Không Leak Thông tin Nhạy cảm

```go
// ✅ ĐÚNG: Generic error to user
func login(username, password string) error {
    user, err := findUser(username)
    if err != nil {
        log.Printf("Login failed for %s: %v", username, err)
        return errors.New("invalid credentials") // Generic message
    }
    
    if !checkPassword(password, user.PasswordHash) {
        log.Printf("Wrong password for %s", username)
        return errors.New("invalid credentials") // Same message
    }
    
    return nil
}
```

### Log Security Events

```go
import "log/slog"

func auditLog(event string, details map[string]any) {
    slog.Info("SECURITY_EVENT",
        "event", event,
        "details", details,
        "timestamp", time.Now().UTC(),
    )
}
```

## Authentication

### JWT Best Practices

```go
import "github.com/golang-jwt/jwt/v5"

func createToken(userID string, secret []byte) (string, error) {
    claims := jwt.MapClaims{
        "sub": userID,
        "exp": time.Now().Add(time.Hour).Unix(),
        "iat": time.Now().Unix(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}

func verifyToken(tokenString string, secret []byte) (*jwt.Token, error) {
    return jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, errors.New("unexpected signing method")
        }
        return secret, nil
    })
}
```

### Session Management

```go
func createSession(userID string) (string, error) {
    sessionID, _ := generateToken()
    
    // Store in Redis with expiration
    redisClient.Set(ctx, "session:"+sessionID, userID, 24*time.Hour)
    
    return sessionID, nil
}

func destroySession(sessionID string) {
    redisClient.Del(ctx, "session:"+sessionID)
}
```

## Rate Limiting

```go
import "golang.org/x/time/rate"

var limiter = rate.NewLimiter(rate.Every(time.Second), 10) // 10 req/sec

func rateLimitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

## Security Headers

```go
func securityHeaders(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        w.Header().Set("Content-Security-Policy", "default-src 'self'")
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        next.ServeHTTP(w, r)
    })
}
```

## Checklist

- [ ] Validate all user input
- [ ] Use parameterized SQL queries
- [ ] Hash passwords with bcrypt/argon2
- [ ] Use crypto/rand for random numbers
- [ ] Enable TLS 1.2+
- [ ] Set security headers
- [ ] Implement rate limiting
- [ ] Log security events
- [ ] Regular dependency updates
- [ ] Run govulncheck

---

*Xem thêm tài nguyên bảo mật tại [go.dev/security](https://go.dev/security)*
