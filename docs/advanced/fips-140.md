# Chế độ FIPS 140

FIPS 140 (Federal Information Processing Standard 140) là một tiêu chuẩn bảo mật của chính phủ Mỹ cho các cryptographic modules. Go hỗ trợ chế độ FIPS 140 để đáp ứng các yêu cầu bảo mật nghiêm ngặt.

## FIPS 140 là gì?

FIPS 140-2 và FIPS 140-3 là các tiêu chuẩn xác định yêu cầu bảo mật cho cryptographic modules được sử dụng bởi:

- Chính phủ liên bang Mỹ
- Các tổ chức regulated (y tế, tài chính)
- Các hệ thống yêu cầu compliance nghiêm ngặt

## Go và FIPS 140

Go tiêu chuẩn **không** FIPS-certified by default. Để đạt FIPS compliance, bạn có các tùy chọn:

1. Sử dụng Go builds được FIPS-certified
2. Sử dụng BoringCrypto (Go's BoringSSL fork)
3. Link với FIPS-certified crypto libraries

## BoringCrypto

Google's BoringCrypto là một implementation của Go crypto sử dụng BoringSSL (FIPS 140-2 validated).

### Build với BoringCrypto

```bash
# Clone Go source
git clone https://go.googlesource.com/go
cd go/src

# Build với BoringCrypto
GOEXPERIMENT=boringcrypto ./make.bash

# Or từ Go 1.19+
go build -tags=boringcrypto
```

### Verify BoringCrypto Mode

```go
package main

import (
    "crypto/boring"
    "fmt"
)

func main() {
    if boring.Enabled() {
        fmt.Println("BoringCrypto is enabled")
    } else {
        fmt.Println("BoringCrypto is NOT enabled")
    }
}
```

## FIPS-Compliant Algorithms

### Supported Algorithms

Khi chạy trong FIPS mode, chỉ các algorithms được approved:

**Hash Functions:**
- SHA-256
- SHA-384
- SHA-512
- SHA-512/256

**Symmetric Encryption:**
- AES (128, 192, 256-bit)
- Triple-DES (deprecated nhưng vẫn allowed)

**Asymmetric Encryption:**
- RSA (2048-bit minimum)
- ECDSA (P-256, P-384, P-521)

**Key Exchange:**
- ECDH (P-256, P-384, P-521)
- RSA

### Non-FIPS Algorithms

Các algorithms sau **không** FIPS-approved:

- MD5
- SHA-1 (except for specific uses)
- RC4
- RSA < 2048 bits
- DES (single)

## Code Examples

### FIPS-Compliant Hashing

```go
import (
    "crypto/sha256"
    "fmt"
)

func hashDataFIPS(data []byte) []byte {
    // SHA-256 is FIPS-approved
    hash := sha256.Sum256(data)
    return hash[:]
}

// ❌ Non-FIPS
import "crypto/md5"
func hashDataNonFIPS(data []byte) []byte {
    hash := md5.Sum(data)  // MD5 không FIPS-approved
    return hash[:]
}
```

### FIPS-Compliant Encryption

```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
)

func encryptFIPS(plaintext, key []byte) ([]byte, error) {
    // AES is FIPS-approved
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    // GCM mode is FIPS-approved
    aesGCM, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonce := make([]byte, aesGCM.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return nil, err
    }
    
    ciphertext := aesGCM.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}
```

### FIPS-Compliant Key Generation

```go
import (
    "crypto/rand"
    "crypto/rsa"
)

func generateRSAKeyFIPS() (*rsa.PrivateKey, error) {
    // RSA 2048-bit minimum for FIPS
    return rsa.GenerateKey(rand.Reader, 2048)
}

// ❌ Non-FIPS
func generateRSAKeyNonFIPS() (*rsa.PrivateKey, error) {
    // 1024-bit không đủ cho FIPS
    return rsa.GenerateKey(rand.Reader, 1024)
}
```

### FIPS-Compliant TLS

```go
import (
    "crypto/tls"
)

func configureFIPSTLS() *tls.Config {
    return &tls.Config{
        MinVersion: tls.VersionTLS12,  // TLS 1.2 minimum
        CipherSuites: []uint16{
            // FIPS-approved cipher suites
            tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
        },
        CurvePreferences: []tls.CurveID{
            // FIPS-approved curves
            tls.CurveP256,
            tls.CurveP384,
            tls.CurveP521,
        },
    }
}
```

## Validation và Testing

### Runtime Checks

```go
package main

import (
    "crypto/boring"
    "log"
)

func enforceFIPS() {
    if !boring.Enabled() {
        log.Fatal("FIPS mode is required but not enabled")
    }
}

func main() {
    enforceFIPS()
    // Rest of application
}
```

### Algorithm Validation

```go
import (
    "crypto/tls"
    "fmt"
)

func validateFIPSCiphers(config *tls.Config) error {
    fipsApproved := map[uint16]bool{
        tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256:   true,
        tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384:   true,
        tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256: true,
        tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384: true,
    }
    
    for _, suite := range config.CipherSuites {
        if !fipsApproved[suite] {
            return fmt.Errorf("non-FIPS cipher suite: %x", suite)
        }
    }
    
    return nil
}
```

## Building FIPS-Compliant Applications

### Environment Setup

```bash
#!/bin/bash
# build-fips.sh

# Set FIPS mode
export GOEXPERIMENT=boringcrypto

# Build application
go build -tags=boringcrypto -o myapp

# Verify
./myapp -version
```

### Docker Image

```dockerfile
FROM golang:1.21 as builder

# Enable BoringCrypto
ENV GOEXPERIMENT=boringcrypto

WORKDIR /app
COPY . .

# Build with FIPS support
RUN go build -tags=boringcrypto -o myapp

FROM debian:bullseye-slim
COPY --from=builder /app/myapp /usr/local/bin/

CMD ["myapp"]
```

### Go.mod Configuration

```go
// go.mod
module myapp

go 1.21

require (
    // Dependencies
)
```

## Compliance Checklist

### Code Review

- [ ] Chỉ sử dụng FIPS-approved algorithms
- [ ] Minimum key sizes (RSA >= 2048, AES >= 128)
- [ ] TLS 1.2 or higher
- [ ] FIPS-approved cipher suites
- [ ] Proper random number generation

### Build Process

- [ ] Build với BoringCrypto enabled
- [ ] Verify FIPS mode tại runtime
- [ ] Document FIPS compliance
- [ ] Test với FIPS-validated libraries

### Runtime

- [ ] Reject non-FIPS algorithms
- [ ] Log FIPS mode status
- [ ] Monitor for compliance violations

## Limitations

### BoringCrypto Limitations

1. **Platform Support**: Chủ yếu Linux/amd64
2. **Performance**: Có thể chậm hơn native Go crypto
3. **Features**: Một số features có thể không available
4. **Updates**: Phụ thuộc vào BoringSSL updates

### FIPS Constraints

1. **Algorithm Restrictions**: Không thể dùng non-approved algorithms
2. **Key Length Requirements**: Minimum key sizes
3. **Mode Requirements**: Specific cipher modes
4. **Testing Overhead**: FIPS validation testing

## Alternatives

### Microsoft Go (FIPS for Windows)

```bash
# Microsoft's FIPS-certified Go build
# https://github.com/microsoft/go
```

### Red Hat Go

```bash
# Red Hat's FIPS-enabled Go
# Available in RHEL
```

### External Libraries

```go
// Link với FIPS-certified OpenSSL
// CGO-based approach
```

## Best Practices

1. **Verify at startup**: Check FIPS mode enabled
2. **Use approved algorithms only**: Follow FIPS requirements
3. **Document compliance**: Maintain compliance documentation
4. **Regular audits**: Review code for FIPS violations
5. **Test thoroughly**: Include FIPS-specific test cases
6. **Stay updated**: Keep crypto libraries current
7. **Monitor changes**: Track FIPS standard updates

## Testing

```go
func TestFIPSCompliance(t *testing.T) {
    if !boring.Enabled() {
        t.Skip("FIPS mode not enabled")
    }
    
    // Test FIPS-compliant operations
    key := make([]byte, 32)  // AES-256
    if _, err := rand.Read(key); err != nil {
        t.Fatal(err)
    }
    
    block, err := aes.NewCipher(key)
    if err != nil {
        t.Fatal(err)
    }
    
    // Verify it's using BoringCrypto
    // Implementation details...
}
```

## Resources

- BoringSSL: https://boringssl.googlesource.com/boringssl/
- FIPS 140-2: https://csrc.nist.gov/publications/detail/fips/140/2/final
- FIPS 140-3: https://csrc.nist.gov/publications/detail/fips/140/3/final
- Go Crypto: https://pkg.go.dev/crypto

## Xem thêm

- [Best Practices về Bảo mật](../security/best-practices.md)
- [Tổng quan về Bảo mật](../security/overview.md)
- [Package crypto](https://pkg.go.dev/crypto)
