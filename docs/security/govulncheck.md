# govulncheck

## Giới thiệu

Govulncheck là công cụ dòng lệnh chính thức của Go để phát hiện các lỗ hổng bảo mật trong dependencies của dự án. Nó thông minh hơn các công cụ khác vì phân tích call graph để chỉ báo cáo các vulnerabilities thực sự ảnh hưởng đến code của bạn.

## Cài đặt

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
```

Đảm bảo `$GOPATH/bin` trong PATH.

## Sử dụng Cơ bản

### Kiểm tra Source Code

```bash
cd /path/to/your/project
govulncheck ./...
```

### Kiểm tra Binary

```bash
govulncheck -mode=binary /path/to/binary
```

### Kiểm tra Module

```bash
govulncheck -mode=source github.com/example/module@v1.0.0
```

## Output

### Không có Vulnerabilities

```
govulncheck ./...
No vulnerabilities found.
```

### Có Vulnerabilities

```
Vulnerability #1: GO-2023-1234
    A vulnerability in package allows remote code execution.
    
    More info: https://pkg.go.dev/vuln/GO-2023-1234
    
    Module: github.com/example/vulnerable-package
    Found in: github.com/example/vulnerable-package@v1.2.3
    Fixed in: github.com/example/vulnerable-package@v1.2.4
    
    Example traces found:
      #1: main.go:42: main.HandleRequest calls vulnerable.Function
```

## Options

### -json

Output JSON format:

```bash
govulncheck -json ./...
```

### -show

Hiển thị thêm thông tin:

```bash
# Hiển thị tất cả vulnerabilities (kể cả không gọi)
govulncheck -show verbose ./...
```

### -db

Chỉ định database URL:

```bash
govulncheck -db=https://vuln.go.dev ./...
```

### -tags

Build tags:

```bash
govulncheck -tags=integration ./...
```

### -test

Bao gồm test files:

```bash
govulncheck -test ./...
```

## Phân tích Call Graph

### Tại sao Call Graph quan trọng?

Govulncheck phân tích code để xác định:
1. Package vulnerable nào được import
2. Function vulnerable nào được gọi
3. Call path từ code của bạn đến vulnerable function

### Ví dụ

```go
// main.go
package main

import "github.com/example/pkg"

func main() {
    // Nếu chỉ gọi SafeFunction, không có vulnerability
    pkg.SafeFunction()
    
    // Nếu gọi VulnerableFunction, sẽ báo cáo
    // pkg.VulnerableFunction()
}
```

## Tích hợp CI/CD

### GitHub Actions

```yaml
name: Security
on: [push, pull_request]

jobs:
  govulncheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: stable
      - name: govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...
```

### GitLab CI

```yaml
govulncheck:
  image: golang:latest
  script:
    - go install golang.org/x/vuln/cmd/govulncheck@latest
    - govulncheck ./...
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

govulncheck ./...
if [ $? -ne 0 ]; then
    echo "Vulnerabilities found! Please fix before committing."
    exit 1
fi
```

## Xử lý Kết quả

### Cập nhật Dependencies

```bash
# Cập nhật package cụ thể
go get github.com/example/vulnerable-package@v1.2.4

# Cập nhật tất cả
go get -u ./...

# Cleanup
go mod tidy
```

### Nếu không thể Update

1. **Đánh giá risk**: Có thực sự bị ảnh hưởng không?
2. **Workaround**: Tránh gọi vulnerable function
3. **Fork và fix**: Nếu maintainer không responsive
4. **Document risk**: Ghi chép và track

### Ignore Vulnerability (Cẩn thận!)

Không có cách chính thức để ignore, nhưng có thể:

```bash
# Filter output
govulncheck ./... 2>&1 | grep -v "GO-2023-1234"
```

## So sánh với Tools Khác

| Feature | govulncheck | Snyk | Dependabot |
|---------|-------------|------|------------|
| Call graph analysis | ✓ | ✗ | ✗ |
| Official Go tool | ✓ | ✗ | ✗ |
| Binary scanning | ✓ | ✓ | ✗ |
| Automated PRs | ✗ | ✓ | ✓ |
| Free | ✓ | Limited | ✓ |

## Troubleshooting

### "no required module provides package"

```bash
go mod tidy
```

### Slow Performance

```bash
# Build cache
go build ./...
govulncheck ./...
```

### False Positives?

govulncheck đã giảm thiểu false positives bằng call graph analysis. Nếu vẫn thấy:

1. Kiểm tra xem code có thực sự gọi function không
2. Xem xét reflect/interface calls

## API Usage

```go
package main

import (
    "context"
    "fmt"
    "golang.org/x/vuln/scan"
)

func main() {
    ctx := context.Background()
    cmd := scan.Command(ctx, "./...")
    err := cmd.Start()
    if err != nil {
        fmt.Println(err)
    }
}
```

## Resources

- [Documentation](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
- [Source Code](https://cs.opensource.google/go/x/vuln)
- [Vulnerability Database](https://vuln.go.dev)

---

*Xem thêm tại [go.dev/doc/security/vuln/govulncheck](https://go.dev/doc/security/vuln/govulncheck)*
