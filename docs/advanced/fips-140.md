# Chế độ FIPS 140

## Giới thiệu

FIPS 140 (Federal Information Processing Standard 140) là tiêu chuẩn bảo mật của chính phủ Mỹ cho các module mật mã. Go 1.24+ hỗ trợ chế độ FIPS 140 compliance.

## FIPS 140 là gì?

FIPS 140 xác định các yêu cầu bảo mật cho các module mật mã được sử dụng trong:

- Các cơ quan chính phủ Mỹ
- Các contractors làm việc với chính phủ
- Các tổ chức yêu cầu tuân thủ quy định

## Kích hoạt FIPS Mode

### Build với FIPS

```bash
# Go 1.24+
GOFIPS140=1 go build ./...
```

### Environment variable

```bash
export GOFIPS140=1
go build ./...
```

### Trong runtime

```go
import "crypto/fips140"

func main() {
    if fips140.Enabled() {
        fmt.Println("FIPS mode is enabled")
    }
}
```

## Các thuật toán được hỗ trợ

### Approved trong FIPS mode

| Category | Algorithms |
|----------|------------|
| Hash | SHA-1*, SHA-256, SHA-384, SHA-512 |
| Symmetric | AES-128, AES-192, AES-256 |
| MAC | HMAC-SHA-256, HMAC-SHA-384, HMAC-SHA-512 |
| Asymmetric | RSA (2048+), ECDSA (P-256, P-384, P-521) |
| Key Exchange | ECDH (P-256, P-384, P-521) |
| Random | DRBG (Deterministic Random Bit Generator) |

*SHA-1 chỉ cho legacy applications

### Không được phép trong FIPS mode

- MD5
- DES/3DES (deprecated)
- RC4
- RSA keys < 2048 bits
- Non-approved curves

## Sử dụng Crypto trong FIPS Mode

### SHA-256

```go
import "crypto/sha256"

func HashData(data []byte) []byte {
    h := sha256.Sum256(data)
    return h[:]
}
```

### AES-GCM

```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
)

func Encrypt(key, plaintext []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return nil, err
    }
    
    return gcm.Seal(nonce, nonce, plaintext, nil), nil
}
```

### ECDSA Signing

```go
import (
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/sha256"
)

func SignData(data []byte) (*ecdsa.PrivateKey, []byte, error) {
    // P-256 is FIPS approved
    key, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        return nil, nil, err
    }
    
    hash := sha256.Sum256(data)
    sig, err := ecdsa.SignASN1(rand.Reader, key, hash[:])
    if err != nil {
        return nil, nil, err
    }
    
    return key, sig, nil
}
```

## TLS với FIPS

### FIPS-compliant TLS config

```go
import "crypto/tls"

func FIPSTLSConfig() *tls.Config {
    return &tls.Config{
        MinVersion: tls.VersionTLS12,
        MaxVersion: tls.VersionTLS13,
        CipherSuites: []uint16{
            tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
        },
        CurvePreferences: []tls.CurveID{
            tls.CurveP256,
            tls.CurveP384,
        },
    }
}
```

## Kiểm tra FIPS Compliance

### Runtime check

```go
import "crypto/fips140"

func RequireFIPS() error {
    if !fips140.Enabled() {
        return errors.New("FIPS mode is required but not enabled")
    }
    return nil
}
```

### Validate algorithm

```go
func ValidateFIPSKey(key *rsa.PrivateKey) error {
    if key.N.BitLen() < 2048 {
        return errors.New("RSA key must be at least 2048 bits for FIPS")
    }
    return nil
}
```

## Docker và Containers

### FIPS-enabled container

```dockerfile
FROM golang:1.24

ENV GOFIPS140=1

WORKDIR /app
COPY . .
RUN go build -o main .

CMD ["./main"]
```

## Testing FIPS Compliance

```go
func TestFIPSEnabled(t *testing.T) {
    if !fips140.Enabled() {
        t.Skip("FIPS mode not enabled")
    }
    
    // Test với FIPS-only features
}
```

## Troubleshooting

### Lỗi thường gặp

```
panic: crypto: use of non-FIPS algorithm in FIPS mode
```

**Giải pháp**: Sử dụng thuật toán FIPS-approved

```
panic: crypto: RSA key too small for FIPS mode
```

**Giải pháp**: Sử dụng RSA key ≥ 2048 bits

## Best Practices

1. **Enable FIPS sớm**: Trong development nếu cần compliance
2. **Test với FIPS mode**: CI/CD với GOFIPS140=1
3. **Audit third-party libraries**: Đảm bảo FIPS-compatible
4. **Document compliance**: Cho auditors
5. **Monitor runtime**: Log FIPS mode status
6. **Use strong key sizes**: RSA 2048+, P-256+

---

*Xem thêm tại [go.dev/doc/security/fips140](https://go.dev/doc/security/fips140)*
