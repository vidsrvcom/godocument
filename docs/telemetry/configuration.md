# Cấu hình Telemetry

Go telemetry thu thập dữ liệu sử dụng ẩn danh từ công cụ Go để cải thiện trải nghiệm nhà phát triển. Tài liệu này mô tả cách cấu hình telemetry.

## Tổng quan

Go telemetry là một tính năng opt-in để thu thập:
- Thông tin về cách các công cụ Go được sử dụng
- Crash reports và error diagnostics
- Performance metrics
- Feature usage statistics

**Quan trọng**: Tất cả dữ liệu được ẩn danh và tổng hợp.

## Lệnh `go telemetry`

### Kiểm tra trạng thái

```bash
# Xem trạng thái telemetry hiện tại
go telemetry

# Output:
# Telemetry uploading is on.
# Telemetry mode is local.
```

### Bật telemetry

```bash
# Bật uploading
go telemetry on

# Chỉ local collection (không upload)
go telemetry local
```

### Tắt telemetry

```bash
# Tắt hoàn toàn
go telemetry off
```

## Telemetry Modes

### Mode: on

Thu thập và upload dữ liệu:

```bash
go telemetry on
```

- Dữ liệu được thu thập locally
- Định kỳ upload lên Google servers
- Hoàn toàn ẩn danh
- Giúp cải thiện Go toolchain

### Mode: local

Chỉ thu thập local, không upload:

```bash
go telemetry local
```

- Dữ liệu được lưu locally
- Không upload lên server
- Có thể review dữ liệu
- Dùng cho testing/debugging

### Mode: off

Tắt hoàn toàn:

```bash
go telemetry off
```

- Không thu thập dữ liệu
- Không upload
- Default cho một số environments

## Configuration File

Telemetry config được lưu trong:

```bash
# Linux/macOS
~/.config/go/telemetry/mode

# Windows
%APPDATA%\go\telemetry\mode
```

### Xem config

```bash
# Linux/macOS
cat ~/.config/go/telemetry/mode

# Windows
type %APPDATA%\go\telemetry\mode
```

### Manual config

```bash
# Linux/macOS
echo "on" > ~/.config/go/telemetry/mode

# Windows
echo on > %APPDATA%\go\telemetry\mode
```

## Environment Variables

### GOTELEMETRY

Override telemetry mode:

```bash
# Tắt telemetry cho session
export GOTELEMETRY=off
go build

# Bật telemetry
export GOTELEMETRY=on
go test ./...

# Local mode
export GOTELEMETRY=local
```

### GOTELEMETRYDIR

Custom directory cho telemetry data:

```bash
export GOTELEMETRYDIR=/custom/path/telemetry
go telemetry on
```

## Viewing Telemetry Data

### Local data location

```bash
# Default location
# Linux/macOS: ~/.config/go/telemetry/
# Windows: %APPDATA%\go\telemetry\

# List collected data
ls ~/.config/go/telemetry/
```

### Reading telemetry data

```bash
# View collected counters
go telemetry view

# View specific date range
go telemetry view -start=2024-01-01 -end=2024-01-31
```

## CI/CD Configuration

### Disable trong CI

```bash
#!/bin/bash
# ci-build.sh

# Disable telemetry trong CI/CD
export GOTELEMETRY=off

# Run builds
go build ./...
go test ./...
```

### GitHub Actions

```yaml
name: Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
      
      - name: Build
        env:
          GOTELEMETRY: off
        run: go build ./...
```

### GitLab CI

```yaml
build:
  variables:
    GOTELEMETRY: "off"
  script:
    - go build ./...
```

### Docker

```dockerfile
FROM golang:1.21

# Disable telemetry
ENV GOTELEMETRY=off

WORKDIR /app
COPY . .

RUN go build -o myapp
```

## Enterprise Configuration

### Organizational Policy

Tạo shared config cho team:

```bash
#!/bin/bash
# setup-telemetry.sh

# Company policy: local only
go telemetry local

echo "Telemetry configured for company policy"
```

### Network Restrictions

Nếu organization block telemetry uploads:

```bash
# Sử dụng local mode
go telemetry local

# Or hoàn toàn disable
go telemetry off
```

## Privacy Controls

### Opt-in Model

Telemetry là hoàn toàn opt-in:

```bash
# Default: OFF
# User phải explicitly enable

# First time setup
go telemetry on
```

### Data Retention

```bash
# Local data được rotate automatically
# Chỉ giữ dữ liệu gần đây (typically 7 days)

# Manual cleanup
rm -rf ~/.config/go/telemetry/local/
```

### Opting Out

```bash
# Luôn có thể opt-out
go telemetry off

# Xóa dữ liệu đã thu thập
rm -rf ~/.config/go/telemetry/
```

## Programmatic Access

### Check telemetry status in Go

```go
package main

import (
    "fmt"
    "os"
)

func checkTelemetry() {
    mode := os.Getenv("GOTELEMETRY")
    
    switch mode {
    case "on":
        fmt.Println("Telemetry uploading enabled")
    case "local":
        fmt.Println("Local telemetry only")
    case "off", "":
        fmt.Println("Telemetry disabled")
    }
}
```

### Disable in code

```go
// Trong init() hoặc main()
func init() {
    // Disable telemetry programmatically
    os.Setenv("GOTELEMETRY", "off")
}
```

## Troubleshooting

### Verify configuration

```bash
# Check current mode
go telemetry

# Check environment
echo $GOTELEMETRY

# Check config file
cat ~/.config/go/telemetry/mode
```

### Reset configuration

```bash
# Remove config
rm ~/.config/go/telemetry/mode

# Reconfigure
go telemetry on
```

### Upload issues

```bash
# Switch to local mode nếu có network issues
go telemetry local

# Check later
go telemetry on
```

## Best Practices

1. **Respect user choice**: Không force enable telemetry
2. **Document in README**: Thông báo nếu project enable telemetry
3. **Disable in CI**: Không cần telemetry trong automated builds
4. **Use local mode for testing**: Khi develop telemetry-aware tools
5. **Regular review**: Periodically review telemetry settings

## Command Reference

```bash
# Status
go telemetry              # Show current mode

# Enable
go telemetry on          # Enable collection và upload
go telemetry local       # Local collection only

# Disable
go telemetry off         # Disable completely

# View data
go telemetry view        # View collected data
```

## Integration Examples

### Makefile

```makefile
.PHONY: build test

build:
    GOTELEMETRY=off go build ./...

test:
    GOTELEMETRY=off go test ./...

# Development build với telemetry
dev:
    GOTELEMETRY=local go build ./...
```

### Shell Script

```bash
#!/bin/bash
# build.sh

set -e

# Disable telemetry
export GOTELEMETRY=off

# Build
echo "Building..."
go build -o myapp

echo "Build complete"
```

## FAQs

**Q: Telemetry có được bật mặc định không?**  
A: Không. Telemetry là opt-in và mặc định OFF.

**Q: Dữ liệu nào được thu thập?**  
A: Xem [Data Collected](data-collected.md) để biết chi tiết.

**Q: Có thể disable hoàn toàn không?**  
A: Có. `go telemetry off` sẽ disable hoàn toàn.

**Q: Dữ liệu có được ẩn danh không?**  
A: Có. Xem [Privacy](privacy.md) để biết thêm.

**Q: Có thể review dữ liệu trước khi upload không?**  
A: Có. Sử dụng `go telemetry local` và `gotelemetry view`.

## Xem thêm

- [Tổng quan về Telemetry](overview.md)
- [Dữ liệu được thu thập](data-collected.md)
- [Quyền riêng tư và Telemetry](privacy.md)
