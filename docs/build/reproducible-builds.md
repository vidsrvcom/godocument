# Xây dựng lại có thể tái tạo (Reproducible Builds)

## Giới thiệu

Reproducible builds đảm bảo rằng cùng một source code luôn tạo ra cùng một binary output, bất kể khi nào hoặc ở đâu được build.

## Tại sao Reproducible Builds quan trọng?

### Lợi ích

1. **Security**: Verify binary khớp với source
2. **Auditability**: Trace binary về source code
3. **Debugging**: Reproduce exact environment
4. **Trust**: Verify third-party binaries

## Go và Reproducible Builds

### Default behavior

Go được thiết kế cho reproducible builds. Mặc định:

- Cùng Go version + source → cùng binary
- `go.mod` và `go.sum` lock dependencies
- No timestamps in binaries

### Kiểm tra

```bash
# Build lần 1
go build -o binary1 .

# Build lần 2
go build -o binary2 .

# So sánh
diff binary1 binary2
# Nếu reproducible: no output
```

## Các yếu tố ảnh hưởng

### 1. Go version

```bash
# Specify Go version trong go.mod
go 1.21.5

# Build với specific version
go1.21.5 build .
```

### 2. Dependencies

```go.mod
module example.com/myapp

go 1.21

require (
    github.com/pkg/errors v0.9.1
)
```

```go.sum
github.com/pkg/errors v0.9.1 h1:...
github.com/pkg/errors v0.9.1/go.mod h1:...
```

### 3. Build flags

```bash
# Các flags quan trọng
go build \
    -trimpath \
    -ldflags="-buildid=" \
    .
```

## Ensuring Reproducibility

### trimpath

```bash
# Loại bỏ absolute paths
go build -trimpath .
```

Trước:

```
/home/user/project/main.go:10
```

Sau:

```
example.com/project/main.go:10
```

### buildid

```bash
# Loại bỏ build ID (có thể khác mỗi build)
go build -ldflags="-buildid=" .
```

### Complete reproducible build

```bash
go build \
    -trimpath \
    -ldflags="-buildid= -s -w" \
    -o binary \
    .
```

## CGO và Reproducibility

### Vấn đề với CGO

CGO có thể gây ra non-reproducibility:

```bash
# C compiler version
# System library versions
# Build environment
```

### Giải pháp

```bash
# Disable CGO nếu có thể
CGO_ENABLED=0 go build .
```

Hoặc sử dụng containerized builds.

## Docker Reproducible Builds

### Dockerfile

```dockerfile
FROM golang:1.21.5 AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 go build \
    -trimpath \
    -ldflags="-buildid= -s -w" \
    -o /app/binary \
    .

FROM scratch
COPY --from=builder /app/binary /binary
ENTRYPOINT ["/binary"]
```

### Pinning base image

```dockerfile
# Sử dụng SHA thay vì tag
FROM golang:1.21.5@sha256:xxx AS builder
```

## Verify Reproducibility

### Manual verification

```bash
#!/bin/bash

# Build 1
go build -trimpath -ldflags="-buildid=" -o bin1 .

# Clean và build 2
go clean -cache
go build -trimpath -ldflags="-buildid=" -o bin2 .

# Compare
if diff bin1 bin2 > /dev/null; then
    echo "Builds are reproducible!"
else
    echo "Builds differ"
    exit 1
fi
```

### Hash comparison

```bash
sha256sum binary1 binary2
```

### diffoscope

```bash
# Detailed binary diff
diffoscope binary1 binary2
```

## Build Metadata

### go version -m

```bash
go version -m ./binary
```

Output:

```
./binary: go1.21.5
    path    example.com/myapp
    mod     example.com/myapp   (devel)
    dep     github.com/pkg/errors   v0.9.1
```

### Embedding version info

```bash
go build -ldflags="-X main.version=1.0.0 -X main.commit=$(git rev-parse HEAD)" .
```

```go
package main

var (
    version string
    commit  string
)
```

## CI/CD Best Practices

### GitHub Actions

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21.5'
      
      - name: Build
        run: |
          CGO_ENABLED=0 go build \
            -trimpath \
            -ldflags="-buildid=" \
            -o binary .
      
      - name: Record hash
        run: sha256sum binary > binary.sha256
```

### Attestation

```bash
# Tạo SBOM (Software Bill of Materials)
go version -m ./binary > sbom.txt

# Sign với cosign
cosign sign-blob --bundle=binary.bundle binary
```

## Troubleshooting

### Binary differs

1. **Check Go version**: `go version`
2. **Check dependencies**: `go mod verify`
3. **Check build flags**: Đảm bảo `-trimpath`
4. **Check CGO**: `CGO_ENABLED=0`
5. **Check timestamps**: Một số tools embed timestamps

### Debug differences

```bash
# Xem build info
go tool buildid binary1
go tool buildid binary2

# Detailed comparison
go tool objdump -s main.main binary1 > dis1.txt
go tool objdump -s main.main binary2 > dis2.txt
diff dis1.txt dis2.txt
```

## Best Practices

1. **Lock dependencies**: `go mod tidy`, commit `go.sum`
2. **Use -trimpath**: Loại bỏ local paths
3. **Clear build ID**: `-ldflags="-buildid="`
4. **Pin Go version**: Trong go.mod và CI
5. **Disable CGO**: Nếu không cần
6. **Container builds**: Cho consistent environment
7. **Verify**: Regularly test reproducibility

---

*Xem thêm tại [go.dev/ref/mod#build-reproducibility](https://go.dev/ref/mod)*
