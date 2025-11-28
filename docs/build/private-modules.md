# Private Modules

Private modules là các Go modules không được publish công khai. Tài liệu này hướng dẫn cách cấu hình và làm việc với private modules trong Go.

## Tổng quan

Private modules có thể là:
- Internal company packages
- Proprietary code
- Code trong private repositories
- Modules yêu cầu authentication

## Cấu hình GOPRIVATE

### Environment Variable

`GOPRIVATE` chỉ định các module patterns là private:

```bash
# Single module
export GOPRIVATE=github.com/mycompany/private-repo

# Multiple modules
export GOPRIVATE=github.com/mycompany/*

# Multiple domains
export GOPRIVATE=github.com/mycompany/*,gitlab.company.com/*

# Permanent (add to ~/.bashrc or ~/.zshrc)
echo 'export GOPRIVATE=github.com/mycompany/*' >> ~/.bashrc
```

### Pattern Matching

```bash
# Exact match
GOPRIVATE=github.com/company/repo

# Wildcard for organization
GOPRIVATE=github.com/company/*

# All repos under domain
GOPRIVATE=gitlab.company.com/*

# Multiple patterns
GOPRIVATE=github.com/company/*,bitbucket.org/team/*
```

## GONOPROXY và GONOSUMDB

### GONOPROXY

Bypass module proxy cho private modules:

```bash
export GONOPROXY=github.com/mycompany/*
```

### GONOSUMDB

Skip checksum database cho private modules:

```bash
export GONOSUMDB=github.com/mycompany/*
```

### Simplified với GOPRIVATE

`GOPRIVATE` tự động set cả `GONOPROXY` và `GONOSUMDB`:

```bash
# Này:
export GOPRIVATE=github.com/company/*

# Tương đương:
export GONOPROXY=github.com/company/*
export GONOSUMDB=github.com/company/*
```

## Git Authentication

### HTTPS với Credentials

#### GitHub Personal Access Token

```bash
# Configure git credentials
git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"

# Or use .netrc
cat >> ~/.netrc << EOF
machine github.com
login ${GITHUB_USERNAME}
password ${GITHUB_TOKEN}
EOF

chmod 600 ~/.netrc
```

#### GitLab Personal Access Token

```bash
# .netrc
cat >> ~/.netrc << EOF
machine gitlab.com
login oauth2
password ${GITLAB_TOKEN}
EOF
```

### SSH Authentication

```bash
# Configure git to use SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# Add SSH key
ssh-add ~/.ssh/id_rsa

# Test
ssh -T git@github.com
```

### Git Credential Helper

```bash
# Configure credential helper
git config --global credential.helper store

# First time access will prompt for credentials
go get github.com/company/private-repo
```

## CI/CD Configuration

### GitHub Actions

```yaml
name: Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Configure private modules
        env:
          GITHUB_TOKEN: ${{ secrets.PRIVATE_REPO_TOKEN }}
        run: |
          git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"
          go env -w GOPRIVATE=github.com/mycompany/*
      
      - name: Build
        run: go build ./...
```

### GitLab CI

```yaml
build:
  image: golang:1.21
  before_script:
    - git config --global url."https://oauth2:${CI_JOB_TOKEN}@gitlab.com/".insteadOf "https://gitlab.com/"
    - go env -w GOPRIVATE=gitlab.com/mycompany/*
  script:
    - go build ./...
```

### Docker Build

```dockerfile
FROM golang:1.21 as builder

# Configure git credentials
ARG GITHUB_TOKEN
RUN git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"

# Set GOPRIVATE
ENV GOPRIVATE=github.com/mycompany/*

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o myapp

FROM debian:bullseye-slim
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

Build:

```bash
docker build --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} -t myapp .
```

## Module Proxies

### Self-hosted Proxy

#### Athens

```bash
# Run Athens proxy
docker run -d \
  -p 3000:3000 \
  -e ATHENS_STORAGE_TYPE=disk \
  -e ATHENS_DISK_STORAGE_ROOT=/var/lib/athens \
  gomods/athens:latest

# Configure Go to use Athens
go env -w GOPROXY=http://localhost:3000,https://proxy.golang.org,direct
go env -w GOPRIVATE=github.com/private/*
```

#### Artifactory

```bash
# Configure Artifactory as proxy
go env -w GOPROXY=https://artifactory.company.com/artifactory/api/go/go,direct

# With authentication
cat >> ~/.netrc << EOF
machine artifactory.company.com
login ${ARTIFACTORY_USER}
password ${ARTIFACTORY_PASSWORD}
EOF
```

### GoCenter / Nexus

```bash
# Nexus Repository Manager
go env -w GOPROXY=https://nexus.company.com/repository/go-proxy/,direct
```

## Working with Private Modules

### Import Private Module

```go
// go.mod
module github.com/mycompany/myapp

go 1.21

require (
    github.com/mycompany/private-lib v1.2.3
)
```

### Download Private Module

```bash
# Ensure GOPRIVATE is set
export GOPRIVATE=github.com/mycompany/*

# Download
go get github.com/mycompany/private-lib@v1.2.3

# Or
go mod download
```

### Replace với Local Path

```go
// go.mod
module myapp

require (
    github.com/company/private-lib v1.0.0
)

// Develop locally
replace github.com/company/private-lib => ../private-lib
```

## Versioning Private Modules

### Tagging

```bash
# Create version tag
git tag v1.0.0
git push origin v1.0.0

# Use in other projects
go get github.com/company/private-lib@v1.0.0
```

### Pseudo-versions

```bash
# Use commit hash
go get github.com/company/private-lib@abc123def

# Use branch
go get github.com/company/private-lib@main
```

## Troubleshooting

### Verification Errors

```bash
# Error: "verifying ... : reading https://sum.golang.org/lookup/..."

# Solution: Add to GONOSUMDB
export GOPRIVATE=github.com/company/*
# or
export GONOSUMDB=github.com/company/*
```

### Proxy Errors

```bash
# Error: "reading https://proxy.golang.org/..."

# Solution: Add to GONOPROXY
export GOPRIVATE=github.com/company/*
# or
export GONOPROXY=github.com/company/*
```

### Authentication Failed

```bash
# Check git config
git config --global --list | grep url

# Test authentication
git ls-remote https://github.com/company/private-repo

# Re-configure credentials
git config --global url."https://${TOKEN}@github.com/".insteadOf "https://github.com/"
```

### Module Not Found

```bash
# Clean module cache
go clean -modcache

# Re-download
go mod download

# Verify GOPRIVATE
go env GOPRIVATE
```

## Best Practices

1. **Set GOPRIVATE globally**: Trong shell rc file
   ```bash
   export GOPRIVATE=github.com/company/*
   ```

2. **Use SSH keys**: Thay vì tokens khi possible
   ```bash
   git config --global url."git@github.com:".insteadOf "https://github.com/"
   ```

3. **Secure credentials**: Không commit tokens
   ```bash
   # Use environment variables
   # Use secret management (Vault, etc.)
   ```

4. **Document requirements**: Trong README
   ```markdown
   # Setup
   export GOPRIVATE=github.com/company/*
   git config --global url."https://${TOKEN}@github.com/".insteadOf "https://github.com/"
   ```

5. **Use module proxy**: Cho team sharing
   ```bash
   # Athens, Artifactory, Nexus
   ```

## Security Considerations

### Token Management

```bash
# ❌ Never commit
# go.mod, .netrc, scripts

# ✅ Use environment variables
export GITHUB_TOKEN=xxx

# ✅ Use secret managers
# Hashicorp Vault, AWS Secrets Manager

# ✅ Rotate regularly
```

### Access Control

```bash
# Principle of least privilege
# Read-only tokens when possible
# Scope tokens to specific repos
```

### Audit

```bash
# Log module downloads
# Monitor access patterns
# Review dependency updates
```

## Example Configuration

### Complete Setup

```bash
#!/bin/bash
# setup-private-modules.sh

# Set private pattern
export GOPRIVATE="github.com/mycompany/*,gitlab.company.com/*"

# Configure git for HTTPS
git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"
git config --global url."https://oauth2:${GITLAB_TOKEN}@gitlab.com/".insteadOf "https://gitlab.com/"

# Verify
echo "GOPRIVATE: $(go env GOPRIVATE)"
echo "Git config:"
git config --global --get-regexp url

# Test
go get github.com/mycompany/private-repo
```

## Xem thêm

- [Module Mirror và Checksum Database](module-mirror.md)
- [Go Modules Reference](../using-go/modules-reference.md)
- [Reproducible Builds](reproducible-builds.md)
