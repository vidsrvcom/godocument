# Cấu hình Đo lường từ xa

## Giới thiệu

Lệnh `go telemetry` cho phép bạn bật, tắt hoặc cấu hình đo lường từ xa trong cài đặt Go của bạn.

## Lệnh cơ bản

### Kiểm tra trạng thái hiện tại

```bash
go telemetry
```

Output:

```
mode: local
```

hoặc

```
mode: on
```

hoặc

```
mode: off
```

## Các chế độ

### on - Bật telemetry

```bash
go telemetry on
```

- Thu thập dữ liệu sử dụng
- Upload dữ liệu tổng hợp đến Go team
- Giúp cải thiện Go

### off - Tắt telemetry

```bash
go telemetry off
```

- Không thu thập dữ liệu
- Không upload bất cứ gì

### local - Chỉ lưu local

```bash
go telemetry local
```

- Thu thập dữ liệu locally
- Không upload đến server
- Có thể xem dữ liệu đã thu thập

## Xem dữ liệu local

### Hiển thị dữ liệu đã thu thập

```bash
go telemetry view
```

### Output mẫu

```json
{
  "Week": "2024-01-15",
  "Programs": [
    {
      "Program": "cmd/go",
      "Version": "go1.21.5",
      "GOOS": "linux",
      "GOARCH": "amd64",
      "Counters": {
        "go/invocations:build": 42,
        "go/invocations:test": 28
      }
    }
  ]
}
```

## Thư mục dữ liệu

### Vị trí mặc định

| Platform | Location |
|----------|----------|
| Linux | `~/.config/go/telemetry` |
| macOS | `~/Library/Application Support/go/telemetry` |
| Windows | `%APPDATA%\go\telemetry` |

### Cấu trúc thư mục

```
telemetry/
├── local/          # Dữ liệu local chưa upload
├── upload/         # Dữ liệu đã upload
├── mode            # File chứa mode hiện tại
└── weekends        # File chứa weekend dates
```

## Environment Variables

### GOTELEMETRY

```bash
# Override mode cho một command
GOTELEMETRY=off go build ./...

# Tương đương với running 'go telemetry off'
```

### GOTELEMETRYDIR

```bash
# Custom telemetry directory
GOTELEMETRYDIR=/custom/path go telemetry
```

## Cấu hình cho CI/CD

### GitHub Actions

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      GOTELEMETRY: off
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      - run: go build ./...
```

### Docker

```dockerfile
FROM golang:1.21

# Tắt telemetry trong container
RUN go telemetry off

# Hoặc sử dụng environment variable
ENV GOTELEMETRY=off
```

## Opt-in cho Organization

### Sử dụng scripts

```bash
#!/bin/bash
# setup-go-telemetry.sh
# Script để cấu hình telemetry cho team

if [ "$1" = "on" ]; then
    go telemetry on
    echo "Telemetry enabled"
elif [ "$1" = "off" ]; then
    go telemetry off
    echo "Telemetry disabled"
else
    echo "Usage: $0 [on|off]"
fi
```

### Dotfiles configuration

```bash
# ~/.bashrc hoặc ~/.zshrc
export GOTELEMETRY=local  # Chỉ local telemetry
```

## Kiểm tra programmatically

```go
import (
    "os"
    "os/exec"
)

func IsTelemetryEnabled() (bool, error) {
    cmd := exec.Command("go", "telemetry")
    output, err := cmd.Output()
    if err != nil {
        return false, err
    }
    return strings.Contains(string(output), "mode: on"), nil
}
```

## Best Practices

1. **Tôn trọng privacy của developers**: Cho phép opt-out
2. **Document chính sách telemetry**: Trong organization guidelines
3. **Disable trong CI/CD**: Tránh dữ liệu không cần thiết
4. **Review data locally**: Trước khi bật full telemetry
5. **Cân nhắc security requirements**: Một số organizations cấm telemetry

## Troubleshooting

### Không thể thay đổi mode

```bash
# Kiểm tra quyền
ls -la ~/.config/go/telemetry/

# Fix permissions
chmod 755 ~/.config/go/telemetry/
chmod 644 ~/.config/go/telemetry/mode
```

### Dữ liệu không xuất hiện

```bash
# Kiểm tra mode
go telemetry

# Kiểm tra directory
ls -la $(go env GOTELEMETRYDIR 2>/dev/null || echo ~/.config/go/telemetry)
```

---

*Xem thêm tại [go.dev/doc/telemetry](https://go.dev/doc/telemetry)*
