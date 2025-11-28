# Dữ liệu được thu thập

## Giới thiệu

Tài liệu này mô tả các loại dữ liệu được thu thập bởi hệ thống đo lường từ xa Go và cách chúng được sử dụng để cải thiện Go.

## Nguyên tắc thu thập

### Chỉ thu thập

- Dữ liệu sử dụng tổng hợp
- Thông tin chẩn đoán ẩn danh
- Counters và metrics

### Không thu thập

- Source code
- File paths đầy đủ
- Thông tin cá nhân
- Secrets hoặc credentials
- IP addresses

## Các loại dữ liệu

### 1. Thông tin môi trường

```json
{
  "GOOS": "linux",
  "GOARCH": "amd64",
  "GoVersion": "go1.21.5"
}
```

| Field | Mô tả |
|-------|-------|
| GOOS | Operating system |
| GOARCH | Architecture |
| GoVersion | Phiên bản Go |

### 2. Command Counters

```json
{
  "go/invocations:build": 42,
  "go/invocations:test": 28,
  "go/invocations:run": 15,
  "go/invocations:mod": 10
}
```

Đếm số lần sử dụng các lệnh Go.

### 3. Feature Usage

```json
{
  "go/build:cgo": 5,
  "go/build:race": 3,
  "go/build:cover": 2,
  "go/test:fuzz": 1
}
```

Tracking các features được sử dụng.

### 4. Error Counts

```json
{
  "go/errors:compile": 10,
  "go/errors:link": 2,
  "gopls/errors:analysis": 5
}
```

Không bao gồm nội dung lỗi, chỉ đếm số lượng.

### 5. Gopls Metrics

```json
{
  "gopls/client:vscode": 100,
  "gopls/client:vim": 20,
  "gopls/completion:offered": 500,
  "gopls/completion:accepted": 200
}
```

Metrics từ Go language server.

## Cấu trúc dữ liệu

### Weekly Report

```json
{
  "Week": "2024-01-15",
  "LastWeek": "2024-01-08",
  "X": "random-id",
  "Programs": [
    {
      "Program": "cmd/go",
      "Version": "go1.21.5",
      "GOOS": "linux",
      "GOARCH": "amd64",
      "GoVersion": "go1.21.5",
      "Counters": {
        "go/invocations:build": 42,
        "go/invocations:test": 28
      }
    }
  ]
}
```

### Thành phần

| Field | Mô tả |
|-------|-------|
| Week | Tuần bắt đầu |
| LastWeek | Tuần trước |
| X | Random ID cho privacy |
| Programs | Danh sách chương trình |
| Counters | Các counter values |

## Counters Registry

### Go tool counters

| Counter | Mô tả |
|---------|-------|
| `go/invocations:*` | Lệnh go được gọi |
| `go/build:*` | Build flags |
| `go/test:*` | Test options |
| `go/mod:*` | Module operations |

### Gopls counters

| Counter | Mô tả |
|---------|-------|
| `gopls/client:*` | Editor clients |
| `gopls/completion:*` | Completion metrics |
| `gopls/diagnostics:*` | Diagnostics |

## Aggregation

### Trước khi upload

Dữ liệu được tổng hợp:

1. **Bucketing**: Giá trị được nhóm vào ranges
2. **Sampling**: Một số data points được sample
3. **Noise**: Random noise được thêm cho privacy

### Ví dụ bucketing

```
Actual: 47 builds
Bucket: 40-60 builds
Uploaded: "40-60"
```

## Cách dữ liệu được sử dụng

### Cải thiện Go

- **Performance optimization**: Biết features nào cần tối ưu
- **Bug prioritization**: Focus vào issues ảnh hưởng nhiều người
- **Feature development**: Hiểu usage patterns
- **Documentation**: Cải thiện docs cho features phổ biến

### Ví dụ thực tế

1. **Generics adoption**: Tracking sử dụng generics
2. **Fuzzing usage**: Đánh giá adoption của fuzzing
3. **Module proxy**: Hiểu patterns truy cập modules
4. **Editor support**: Cải thiện gopls cho editors phổ biến

## Xem dữ liệu local

### Command

```bash
go telemetry view
```

### Filtering

```bash
# Xem counters cụ thể
go telemetry view | jq '.Programs[].Counters | to_entries[] | select(.key | startswith("go/"))'
```

## Transparency

### Public charts

Dữ liệu tổng hợp được công bố tại:
- [telemetry.go.dev](https://telemetry.go.dev)

### Counters documentation

Danh sách đầy đủ các counters:
- [go.dev/doc/telemetry#counters](https://go.dev/doc/telemetry#counters)

## Best Practices

1. **Review local data**: Trước khi enable upload
2. **Understand what's collected**: Đọc documentation
3. **Contribute feedback**: Nếu có concerns
4. **Enable nếu comfortable**: Giúp cải thiện Go

---

*Xem thêm tại [go.dev/doc/telemetry](https://go.dev/doc/telemetry)*
