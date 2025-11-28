# Dữ liệu được thu thập bởi Telemetry

Go telemetry thu thập dữ liệu sử dụng ẩn danh để cải thiện công cụ Go. Tài liệu này mô tả chính xác dữ liệu nào được thu thập và cách nó được sử dụng.

## Nguyên tắc Thu thập Dữ liệu

1. **Hoàn toàn ẩn danh**: Không có thông tin định danh cá nhân
2. **Opt-in**: Phải explicitly enable
3. **Transparent**: Có thể review dữ liệu trước khi upload
4. **Minimal**: Chỉ thu thập dữ liệu cần thiết
5. **Aggregated**: Dữ liệu được tổng hợp, không theo dõi cá nhân

## Loại Dữ liệu  được Thu thập

### 1. Command Usage

Các lệnh Go nào được sử dụng và tần suất:

```
go build     - 1234 times
go test      - 567 times
go mod tidy  - 89 times
gopls        - 3456 times
```

**Không** thu thập:
- File names
- Package paths
- Project names
- Source code

### 2. Command Flags

Flags nào được sử dụng phổ biến:

```
go build -race           - 45 times
go test -cover           - 123 times
go build -tags=boringcrypto - 12 times
```

**Không** thu thập:
- Values của flags
- Custom build tags
- Environment-specific information

### 3. Go Version

Version của Go toolchain được sử dụng:

```
go1.21.0 - 1000 users
go1.21.1 - 500 users
go1.20.5 - 200 users
```

### 4. Platform Information

Operating system và architecture:

```
linux/amd64    - 5000 users
darwin/arm64   - 2000 users
windows/amd64  - 1500 users
```

**Không** thu thập:
- OS version cụ thể
- Hostname
- IP address
- Location

### 5. Crash Reports

Khi Go tools crash, stack trace được thu thập:

```
panic: runtime error: invalid memory address
goroutine 1 [running]:
main.processFile(...)
    /path/to/file.go:123
```

**Được sanitize**:
- Personal paths được remove
- Chỉ giữ stack trace
- Không có source code

### 6. Compiler Diagnostics

Thống kê về compilation:

```
Build time: 10s (median)
Package count: 50 (median)
File count: 200 (median)
```

**Không** thu thập:
- Specific package names
- Import paths
- File names

### 7. Language Server (gopls) Metrics

Gopls performance và usage:

```
Initialization time: 3s
Memory usage: 500MB
Completion latency: 50ms
```

**Không** thu thập:
- Code content
- Edited files
- Variable/function names

### 8. Module Usage

Thống kê về Go modules:

```
Module-enabled projects: 95%
Average module count: 20
Go version in go.mod: 1.21
```

**Không** thu thập:
- Module names
- Module paths
- Dependencies

## Data Format

### Counter Format

Dữ liệu được lưu dưới dạng counters:

```json
{
  "command:build": 1234,
  "command:test": 567,
  "flag:build:-race": 45,
  "platform:linux/amd64": 1,
  "version:go1.21.0": 1
}
```

### Aggregation

Dữ liệu được aggregate trước khi upload:

```
Local collection:
  Day 1: go build (10 times)
  Day 2: go build (15 times)

Aggregated upload:
  Week 1: go build (25 times)
```

## Dữ liệu KHÔNG được Thu thập

### ❌ Personal Information

- Tên
- Email
- Company name
- IP address
- Location

### ❌ Source Code

- File contents
- Function names
- Variable names
- Comments
- Package names (cụ thể)

### ❌ Project Details

- Project names
- Repository URLs
- File paths (absolute)
- Dependencies (cụ thể)

### ❌ Environment

- Environment variables
- System configuration
- Network information
- User accounts

## Viewing Local Data

### Review trước khi Upload

```bash
# View collected data
go telemetry view

# Output example:
# command:build: 10
# command:test: 5
# platform:linux/amd64: 1
# version:go1.21.0: 1
```

### Export Data

```bash
# Export to file
go telemetry view > telemetry-data.txt

# Review file
cat telemetry-data.txt
```

## Upload Process

### When Data is Uploaded

- Định kỳ (daily or weekly)
- Chỉ khi mode = "on"
- Qua HTTPS encrypted connection
- Batch upload, không realtime

### Upload Destination

```
Endpoint: telemetry.go.dev
Protocol: HTTPS
Method: POST
```

### Upload Content

```json
{
  "version": "v1",
  "counters": {
    "command:build": 100,
    "command:test": 50
  },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

## Using Collected Data

### Goal 1: Improve Tooling

```
Data shows: 
  "go build -race" used frequently
  
Action:
  Optimize race detector performance
  Improve documentation for race detection
```

### Goal 2: Prioritize Features

```
Data shows:
  90% developers use modules
  10% use GOPATH
  
Action:
  Focus development on modules
  Maintain but don't enhance GOPATH
```

### Goal 3: Fix Bugs

```
Data shows:
  Crash in "go test -cover" on certain inputs
  
Action:
  Investigate and fix crash
  Add regression test
```

### Goal 4: Performance

```
Data shows:
  "gopls" initialization takes 5s median
  
Action:
  Profile and optimize startup
  Reduce memory usage
```

## Transparency

### Public Reports

Go team publishes aggregated data:

- Blog posts về telemetry insights
- Dashboards (planned)
- Release notes citing telemetry

### Example Report

```
Q1 2024 Telemetry Report:
- Go 1.21 adoption: 75%
- Module usage: 95%
- Most used commands: build, test, mod
- Average build time improved 10%
```

## Opting Out

### Complete Opt-out

```bash
# Disable all telemetry
go telemetry off

# Delete collected data
rm -rf ~/.config/go/telemetry/
```

### Local-only Mode

```bash
# Collect but don't upload
go telemetry local

# Review data
go telemetry view
```

## Security

### Data Transportation

- HTTPS encryption
- Certificate pinning
- No sensitive data

### Data Storage

ở Client:
```
~/.config/go/telemetry/
  - mode           (config)
  - local/         (counters)
  - upload/        (pending uploads)
```

Trên Server:
- Aggregated counters only
- No user identification
- Automatic retention limits

## Best Practices để Review

### Before Enabling

```bash
# 1. Set to local mode
go telemetry local

# 2. Use Go tools normally for một ngày

# 3. Review collected data
go telemetry view

# 4. Nếu comfortable, enable uploading
go telemetry on
```

### Regular Review

```bash
# Monthly review
crontab -e

# Add:
# 0 0 1 * * go telemetry view > /tmp/telemetry-review.txt
```

## FAQs

**Q: Có thể trace dữ liệu về cá nhân tôi không?**  
A: Không. Dữ liệu được aggregate và anonymous.

**Q: Source code có được thu thập không?**  
A: Không bao giờ. Chỉ counters và metrics.

**Q: Có thể xóa dữ liệu đã upload không?**  
A: Dữ liệu đã upload là aggregated, không có dữ liệu cá nhân để xóa.

**Q: Crash reports có chứa sensitive info không?**  
A: Stack traces được sanitize, personal paths removed.

**Q: Có thể see exactly dữ liệu sẽ được upload không?**  
A: Có. `go telemetry local` + `go telemetry view`.

## Example Data

### Typical Weekly Collection

```
command:build: 50
command:test: 30
command:mod:tidy: 5
flag:build:-race: 10
flag:test:-cover: 15
platform:linux/amd64: 1
version:go1.21.0: 1
crash:builtin:panic: 0
```

### After Aggregation (1000 users)

```
command:build: 50,000   (avg 50/user)
command:test: 30,000    (avg 30/user)
platform:linux/amd64: 600  (60%)
platform:darwin/arm64: 300 (30%)
platform:windows/amd64: 100 (10%)
```

## Xem thêm

- [Cấu hình Telemetry](configuration.md)
- [Quyền riêng tư và Telemetry](privacy.md)
- [Tổng quan về Telemetry](overview.md)
