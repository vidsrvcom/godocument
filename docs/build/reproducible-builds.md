# Reproducible Builds (Xây dựng có thể Tái tạo)

Reproducible builds đảm bảo rằng cùng một source code luôn tạo ra cùng một binary, bất kể nơi hay khi nào build được thực hiện. Điều này quan trọng cho security và verification.

## Tại sao Reproducible Builds?

### Security

- Verify rằng binary match source code
- Detect tampering trong build process
- Trust but verify distributed binaries

### Debugging

- Reproduce exact binary từ past releases
- Debug issues với identical binaries
- Consistent behavior across environments

### Compliance

- Audit trail cho builds
- Regulatory requirements
- Software supply chain security

## Go và Reproducibility

Go compiler được designed cho reproducible builds:

- Deterministic compilation
- No timestamps trong binaries (by default)
- Consistent linking order
- Stable module versioning

## Building Reproducibly

### Basic Build

```bash
# Standard build
go build -o myapp

# Check reproducibility
sha256sum myapp
# Build again
go build -o myapp2
sha256sum myapp2
# Should match!
```

### Trimpath Flag

Remove absolute paths trong debug info:

```bash
# Without trimpath (không reproducible)
go build -o myapp
# Binary chứa: /home/user/project/file.go

# With trimpath (reproducible)
go build -trimpath -o myapp
# Binary chứa: file.go (relative path)
```

### Build Flags

```bash
# Full reproducible build
go build \
  -trimpath \
  -ldflags="-w -s" \
  -o myapp

# -trimpath: Remove absolute paths
# -w: Omit DWARF symbol table
# -s: Omit symbol table và debug info
```

## Verifying Reproducibility

### Hash Comparison

```bash
#!/bin/bash
# verify-reproducible.sh

# Build 1
go build -trimpath -o build1/myapp
HASH1=$(sha256sum build1/myapp | cut -d' ' -f1)

# Clean
go clean

# Build 2
go build -trimpath -o build2/myapp
HASH2=$(sha256sum build2/myapp | cut -d' ' -f1)

# Compare
if [ "$HASH1" = "$HASH2" ]; then
    echo "✅ Build is reproducible!"
    echo "Hash: $HASH1"
else
    echo "❌ Build is NOT reproducible"
    echo "Hash 1: $HASH1"
    echo "Hash 2: $HASH2"
fi
```

### Diff Binaries

```bash
# Compare binaries
cmp build1/myapp build2/myapp && echo "Identical" || echo "Different"

# Detailed diff
diff <(hexdump -C build1/myapp) <(hexdump -C build2/myapp)
```

## Module Versioning

### go.mod và go.sum

Ensure deterministic dependencies:

```go
// go.mod
module myapp

go 1.21

require (
    github.com/pkg/errors v0.9.1
    golang.org/x/sync v0.3.0
)
```

```
# go.sum contains cryptographic hashes
github.com/pkg/errors v0.9.1 h1:abc123...
github.com/pkg/errors v0.9.1/go.mod h1:def456...
```

### Vendor Dependencies

Pin exact versions:

```bash
# Vendor dependencies
go mod vendor

# Build với vendor
go build -mod=vendor -trimpath -o myapp
```

### Lock File

go.sum acts as lock file:

```bash
# Commit go.sum to version control
git add go.sum
git commit -m "Lock dependencies"
```

## Build Environment

### Go Version

Use exact Go version:

```bash
# Check version
go version
# go version go1.21.0 linux/amd64

# Install specific version
go install golang.org/dl/go1.21.0@latest
go1.21.0 download
```

### Environment Variables

Set consistently:

```bash
# Consistent environment
export CGO_ENABLED=0
export GOOS=linux
export GOARCH=amd64

go build -trimpath -o myapp
```

### Docker Builder

```dockerfile
# Use specific Go version
FROM golang:1.21.0 AS builder

# Set build env
ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GOARCH=amd64

WORKDIR /build

# Copy dependencies first
COPY go.mod go.sum ./
RUN go mod download

# Copy source
COPY . .

# Reproducible build
RUN go build -trimpath -ldflags="-w -s" -o myapp

# Verify
RUN sha256sum myapp > myapp.sha256

FROM scratch
COPY --from=builder /build/myapp /myapp
COPY --from=builder /build/myapp.sha256 /myapp.sha256
ENTRYPOINT ["/myapp"]
```

## LDFlags và Build Info

### Remove Build Info

```bash
# Build với minimal info
go build \
  -trimpath \
  -ldflags="-w -s -buildid=" \
  -o myapp
```

### Controlled Build Info

```bash
# Inject controlled version info
VERSION="v1.0.0"
COMMIT=$(git rev-parse HEAD)
BUILD_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)

go build \
  -trimpath \
  -ldflags="-X main.Version=${VERSION} -X main.Commit=${COMMIT} -X main.BuildTime=${BUILD_TIME}" \
  -o myapp
```

Code:

```go
package main

var (
    Version   = "dev"
    Commit    = "unknown"
    BuildTime = "unknown"
)

func main() {
    fmt.Printf("Version: %s\n", Version)
    fmt.Printf("Commit: %s\n", Commit)
    fmt.Printf("Built: %s\n", BuildTime)
}
```

## CGO và Reproducibility

### Disable CGO

CGO builds often không reproducible:

```bash
# Disable CGO
CGO_ENABLED=0 go build -trimpath -o myapp
```

### CGO với Docker

Nếu cần CGO, use containerized build:

```dockerfile
FROM golang:1.21.0

# Install specific compiler versions
RUN apt-get update && apt-get install -y \
    gcc=4:10.2.1-1 \
    && rm -rf /var/lib/apt/lists/*

ENV CGO_ENABLED=1
CMD ["go", "build", "-trimpath"]
```

## CI/CD Reproducibility

### GitHub Actions

```yaml
name: Reproducible Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21.0'  # Exact version
      
      - name: Build
        run: |
          go build -trimpath -ldflags="-w -s" -o myapp
          sha256sum myapp > myapp.sha256
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: binary
          path: |
            myapp
            myapp.sha256
```

### Verification Workflow

```yaml
verify:
  runs-on: ubuntu-latest
  needs: build
  steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-go@v4
      with:
        go-version: '1.21.0'
    
    - name: Download artifact
      uses: actions/download-artifact@v3
      with:
        name: binary
    
    - name: Rebuild
      run: go build -trimpath -ldflags="-w -s" -o myapp-verify
    
    - name: Verify
      run: |
        EXPECTED=$(cat myapp.sha256 | cut -d' ' -f1)
        ACTUAL=$(sha256sum myapp-verify | cut -d' ' -f1)
        
        if [ "$EXPECTED" = "$ACTUAL" ]; then
          echo "✅ Build is reproducible!"
        else
          echo "❌ Build verification failed"
          exit 1
        fi
```

## Signing Binaries

### Cosign

```bash
# Build
go build -trimpath -o myapp

# Sign
cosign sign-blob --key cosign.key myapp > myapp.sig

# Verify
cosign verify-blob --key cosign.pub --signature myapp.sig myapp
```

### GPG

```bash
# Build
go build -trimpath -o myapp

# Sign
gpg --detach-sign --armor myapp

# Verify
gpg --verify myapp.asc myapp
```

## SBOM (Software Bill of Materials)

### Generate SBOM

```bash
# Using syft
syft myapp -o spdx-json > myapp-sbom.json

# Or cyclonedx
cyclonedx-gomod app -json -output myapp-sbom.json
```

### Include Dependencies

```bash
# List all dependencies
go list -m all > dependencies.txt

# With versions
go list -m -json all > dependencies.json
```

## Best Practices

1. **Use -trimpath**: Always
   ```bash
   go build -trimpath
   ```

2. **Pin Go version**: Exact version
   ```
   go 1.21.0 (not 1.21)
   ```

3. **Commit go.sum**: Version control
   ```bash
   git add go.sum
   ```

4. **Disable CGO**: When possible
   ```bash
   CGO_ENABLED=0
   ```

5. **Document build**: Include instructions
   ```markdown
   # Build
   Go version: 1.21.0
   Command: go build -trimpath -ldflags="-w -s"
   ```

6. **Verify regularly**: Automated checks
   ```bash
   # In CI/CD
   ```

7. **Sign releases**: Cryptographic signatures
   ```bash
   cosign sign-blob
   ```

## Troubleshooting

### Timestamps trong Binary

```bash
# Check for timestamps
strings myapp | grep -E '[0-9]{4}-[0-9]{2}-[0-9]{2}'

# Fix: Use -ldflags to set or remove
```

### Absolute Paths

```bash
# Check for paths
strings myapp | grep "/home/"

# Fix: Use -trimpath
go build -trimpath
```

### Build ID Mismatch

```bash
# Remove build ID
go build -ldflags="-buildid="

# Or consistent build ID
go build -ldflags="-buildid=<fixed-value>"
```

## Verification Tools

### diffoscope

```bash
# Install
apt-get install diffoscope

# Compare builds
diffoscope build1/myapp build2/myapp
```

### reproducible-builds.org

Tools và resources:
- https://reproducible-builds.org/
- Testing tools
- Documentation

## Example: Complete Reproducible Build

```bash
#!/bin/bash
# reproducible-build.sh

set -e

# Configuration
GO_VERSION="1.21.0"
OUTPUT="myapp"

# Verify Go version
CURRENT_VERSION=$(go version | grep -oP 'go\K[0-9.]+')
if [ "$CURRENT_VERSION" != "$GO_VERSION" ]; then
    echo "Error: Required Go $GO_VERSION, found $CURRENT_VERSION"
    exit 1
fi

# Set environment
export CGO_ENABLED=0
export GOOS=linux
export GOARCH=amd64

# Get version info
VERSION=$(git describe --tags --always)
COMMIT=$(git rev-parse HEAD)
BUILD_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Build
go build \
  -trimpath \
  -ldflags="-w -s -X main.Version=${VERSION} -X main.Commit=${COMMIT} -X main.BuildTime=${BUILD_TIME}" \
  -o "${OUTPUT}"

# Generate checksums
sha256sum "${OUTPUT}" > "${OUTPUT}.sha256"
sha512sum "${OUTPUT}" > "${OUTPUT}.sha512"

# Print info
echo "Build complete:"
echo "  Output: ${OUTPUT}"
echo "  Version: ${VERSION}"
echo "  Commit: ${COMMIT}"
echo "  Built: ${BUILD_TIME}"
echo "  SHA256: $(cat ${OUTPUT}.sha256)"

echo "✅ Reproducible build successful!"
```

## Xem thêm

- [Module Mirror và Checksum Database](module-mirror.md)
- [Private Modules](private-modules.md)
- [Go Modules Reference](../using-go/modules-reference.md)
- Reproducible Builds: https://reproducible-builds.org/
