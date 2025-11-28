# Private Modules

## Giới thiệu

Go hỗ trợ cấu hình các module riêng tư để sử dụng với các repositories nội bộ mà không cần đi qua public proxy và checksum database.

## GOPRIVATE

### Cấu hình cơ bản

```bash
# Set GOPRIVATE
go env -w GOPRIVATE=*.corp.example.com,github.com/myorg/*
```

### Patterns

| Pattern | Matches |
|---------|---------|
| `example.com` | Chính xác example.com |
| `*.example.com` | Subdomains của example.com |
| `example.com/*` | Paths dưới example.com |
| `example.com/org/*` | Paths dưới org |

### Ví dụ

```bash
# Single domain
go env -w GOPRIVATE=git.internal.company.com

# Multiple patterns
go env -w GOPRIVATE=*.corp.com,github.com/mycompany/*

# All of a GitHub org
go env -w GOPRIVATE=github.com/myorg/*
```

## GONOPROXY và GONOSUMDB

### Khi cần fine-grained control

```bash
# Không sử dụng proxy cho private modules
go env -w GONOPROXY=*.corp.example.com

# Không sử dụng checksum database
go env -w GONOSUMDB=*.corp.example.com
```

### GOPRIVATE tự động set cả hai

```bash
# Tương đương với:
go env -w GOPRIVATE=example.com
# = GONOPROXY=example.com + GONOSUMDB=example.com
```

## Git Configuration

### SSH Authentication

```bash
# ~/.gitconfig
[url "git@github.com:"]
    insteadOf = https://github.com/
```

### HTTPS với credentials

```bash
# ~/.netrc (Unix) hoặc %USERPROFILE%\_netrc (Windows)
machine github.com
login USERNAME
password TOKEN
```

### Git credential helper

```bash
git config --global credential.helper store
# hoặc
git config --global credential.helper cache --timeout=3600
```

## Private Proxy

### Tự host proxy

```bash
# Sử dụng private proxy trước public
go env -w GOPROXY=https://internal.proxy.example.com,https://proxy.golang.org,direct
```

### Athens proxy

```bash
# Cài đặt Athens
docker run -d \
    -p 3000:3000 \
    -e ATHENS_STORAGE_TYPE=disk \
    gomods/athens:latest

# Cấu hình Go
go env -w GOPROXY=http://localhost:3000,https://proxy.golang.org,direct
```

## GitLab Configuration

### Personal Access Token

```bash
# Tạo token với read_api scope
# ~/.netrc
machine gitlab.com
login __token__
password glpat-xxxxxxxxxxxx
```

### GitLab Groups

```bash
go env -w GOPRIVATE=gitlab.com/mygroup/*
```

## GitHub Configuration

### Personal Access Token

```bash
# Token với repo scope
# ~/.netrc
machine github.com
login USERNAME
password ghp_xxxxxxxxxxxx
```

### GitHub Enterprise

```bash
go env -w GOPRIVATE=github.mycompany.com/*
go env -w GOPROXY=https://proxy.mycompany.com,direct
```

## CI/CD Configuration

### GitHub Actions

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure Git for private modules
        run: |
          git config --global url."https://${{ secrets.GH_TOKEN }}@github.com/".insteadOf "https://github.com/"
      
      - name: Set GOPRIVATE
        run: go env -w GOPRIVATE=github.com/myorg/*
      
      - name: Build
        run: go build ./...
```

### GitLab CI

```yaml
build:
  script:
    - echo "machine gitlab.com login __token__ password ${CI_JOB_TOKEN}" > ~/.netrc
    - go env -w GOPRIVATE=gitlab.com/mygroup/*
    - go build ./...
```

### Docker

```dockerfile
FROM golang:1.21

ARG GITHUB_TOKEN
RUN git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"
RUN go env -w GOPRIVATE=github.com/myorg/*

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o main .
```

## Troubleshooting

### "unknown revision" error

```bash
# Kiểm tra Git access
git ls-remote https://github.com/myorg/private-repo

# Kiểm tra credentials
cat ~/.netrc
```

### "410 Gone" from proxy

```bash
# Đảm bảo GOPRIVATE đúng
go env GOPRIVATE

# Force direct access
go env -w GOPROXY=direct
go get github.com/myorg/private-repo
```

### SSH key issues

```bash
# Kiểm tra SSH
ssh -T git@github.com

# Thêm key vào agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

## Best Practices

1. **Sử dụng GOPRIVATE**: Thay vì cấu hình riêng GONOPROXY/GONOSUMDB
2. **Token management**: Sử dụng secrets trong CI/CD
3. **Minimal permissions**: Chỉ read access cho build
4. **Document configuration**: Cho team members
5. **Test locally**: Trước khi commit go.mod changes

---

*Xem thêm tại [go.dev/ref/mod#private-modules](https://go.dev/ref/mod#private-modules)*
