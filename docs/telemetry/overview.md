# Tổng quan về Đo lường từ xa (Telemetry)

## Giới thiệu

Go Telemetry là thu thập dữ liệu sử dụng và chẩn đoán ẩn danh từ chuỗi công cụ Go để cải thiện trải nghiệm nhà phát triển.

## Tại sao Telemetry?

- Hiểu cách Go được sử dụng
- Phát hiện bugs và issues
- Ưu tiên improvements
- Cải thiện tooling

## Cấu hình

### Kiểm tra trạng thái

```bash
go telemetry
```

### Bật telemetry

```bash
go telemetry on
```

### Tắt telemetry

```bash
go telemetry off
```

### Local only

```bash
go telemetry local
```

## Dữ liệu được thu thập

- Lệnh Go được sử dụng
- Version của Go
- OS và architecture
- Các options và flags
- Thông tin lỗi (ẩn danh)

## Quyền riêng tư

- Không thu thập source code
- Không thu thập paths đầy đủ
- Không thu thập thông tin cá nhân
- Dữ liệu được tổng hợp ẩn danh

## Xem dữ liệu local

```bash
go telemetry view
```

---

*Xem thêm tại [go.dev/doc/telemetry](https://go.dev/doc/telemetry)*
