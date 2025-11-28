# Quyền riêng tư và Đo lường từ xa

## Giới thiệu

Tài liệu này cung cấp thông tin về quyền riêng tư liên quan đến việc thu thập và xử lý dữ liệu đo lường từ xa trong Go.

## Cam kết quyền riêng tư

### Nguyên tắc cốt lõi

1. **Opt-in**: Telemetry phải được bật một cách chủ động
2. **Transparency**: Rõ ràng về những gì được thu thập
3. **Minimal data**: Chỉ thu thập những gì cần thiết
4. **No PII**: Không thu thập thông tin cá nhân
5. **User control**: Người dùng có toàn quyền kiểm soát

## Những gì KHÔNG được thu thập

### Hoàn toàn không thu thập

| Category | Ví dụ |
|----------|-------|
| Source code | Bất kỳ code nào |
| File paths | `/home/user/project/main.go` |
| File names | `secret_api.go` |
| Package names | `github.com/company/internal` |
| Error messages | Chi tiết lỗi compile |
| Environment variables | Giá trị của env vars |
| Command arguments | `go build -o myapp` |
| IP addresses | Address của máy |
| Usernames | Tên người dùng hệ thống |
| Hostnames | Tên máy tính |

### Được sanitize trước khi thu thập

```
Original: /home/john/company/secret-project/main.go
Sanitized: <elided>
```

## Những gì ĐƯỢC thu thập

### Với sự đồng ý

| Category | Ví dụ | Mục đích |
|----------|-------|----------|
| Go version | `go1.21.5` | Compatibility |
| OS/Arch | `linux/amd64` | Platform support |
| Command counts | `build: 42` | Usage patterns |
| Feature flags | `cgo: used` | Feature adoption |
| Error counts | `compile: 10` | Quality metrics |

## Bảo vệ dữ liệu

### Tại client

```go
// Dữ liệu được xử lý locally trước
// 1. Loại bỏ thông tin nhạy cảm
// 2. Aggregate theo tuần
// 3. Thêm random noise
```

### Khi truyền

- HTTPS encryption
- Certificate pinning
- No persistent identifiers

### Tại server

- Aggregation thêm
- Differential privacy
- Limited retention
- Access controls

## Differential Privacy

### Khái niệm

Differential privacy đảm bảo rằng không thể xác định dữ liệu của một cá nhân từ dữ liệu tổng hợp.

### Kỹ thuật sử dụng

1. **Random sampling**: Không phải tất cả data points được upload
2. **Bucketing**: Giá trị được nhóm vào ranges
3. **Noise addition**: Random noise được thêm
4. **K-anonymity**: Dữ liệu chỉ publish nếu có ≥k users

### Ví dụ

```
Your data: 47 builds
After noise: 45-50 builds
Published: "40-60 builds category"
```

## Quyền của người dùng

### Kiểm soát hoàn toàn

```bash
# Tắt telemetry
go telemetry off

# Xem dữ liệu local
go telemetry view

# Xóa dữ liệu local
rm -rf ~/.config/go/telemetry/local/*
```

### Không có hậu quả

- Tắt telemetry không ảnh hưởng đến Go functionality
- Không có "premium" features cho users bật telemetry
- Không có rate limiting hay restrictions

## Data Retention

### Local data

```
~/.config/go/telemetry/
├── local/      # Dữ liệu chưa upload, tự động rotate
└── upload/     # Dữ liệu đã upload, có thể xóa
```

### Server-side

- Raw data: Giữ tối đa 90 ngày
- Aggregated data: Giữ indefinitely (ẩn danh hoàn toàn)

## Audit và Transparency

### Open source

- Toàn bộ telemetry code là open source
- Review tại: [go.googlesource.com/telemetry](https://go.googlesource.com/telemetry)

### Published data

- Dữ liệu tổng hợp được công bố
- Charts tại: [telemetry.go.dev](https://telemetry.go.dev)

### Counter registry

- Danh sách tất cả counters được document
- Không có "hidden" counters

## Corporate/Enterprise

### Cho Organizations

```bash
# Disable cho toàn organization
# Trong shared profile hoặc dotfiles
export GOTELEMETRY=off
```

### Compliance considerations

| Standard | Telemetry Impact |
|----------|-----------------|
| GDPR | No PII collected, safe |
| HIPAA | No PHI collected, safe |
| SOC 2 | Document in policies |
| ISO 27001 | Include in risk assessment |

## FAQ

### "Telemetry có track tôi không?"

**Không.** Không có persistent identifier. Mỗi upload là độc lập.

### "Telemetry có gửi code không?"

**Không.** Chỉ counters và metrics, không bao giờ gửi code.

### "Tôi có thể trust được không?"

- Code là open source
- Third-party audits
- Transparent về data collected

### "Tắt telemetry có ảnh hưởng gì?"

**Không.** Go hoạt động hoàn toàn bình thường.

## Best Practices

1. **Review trước khi enable**: Xem `go telemetry view`
2. **Understand data**: Đọc documentation
3. **Organization policies**: Align với company guidelines
4. **Educate team**: Share privacy information
5. **Contribute**: Báo cáo nếu phát hiện privacy issues

## Liên hệ

### Privacy concerns

- Email: security@golang.org
- Issue tracker: [github.com/golang/go/issues](https://github.com/golang/go/issues)

### Telemetry feedback

- Proposal discussions trên GitHub
- Go blog announcements

---

*Xem thêm tại [go.dev/doc/telemetry](https://go.dev/doc/telemetry)*
