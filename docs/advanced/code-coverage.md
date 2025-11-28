# Bảo hiểm Code (Code Coverage)

## Giới thiệu

Go cung cấp công cụ code coverage tích hợp để đo lường phần mã nào đã được thực thi bởi tests.

## Cơ bản

### Chạy tests với coverage

```bash
go test -cover ./...
```

Output:

```
PASS
coverage: 80.0% of statements
```

### Tạo coverage profile

```bash
go test -coverprofile=coverage.out ./...
```

### Xem coverage báo cáo

```bash
# Text report
go tool cover -func=coverage.out

# HTML report
go tool cover -html=coverage.out -o coverage.html
```

## Coverage Modes

### set (mặc định)

Đếm xem mỗi statement có được thực thi không.

```bash
go test -covermode=set ./...
```

### count

Đếm số lần mỗi statement được thực thi.

```bash
go test -covermode=count ./...
```

### atomic

Như count nhưng thread-safe (cho tests với goroutines).

```bash
go test -covermode=atomic ./...
```

## Coverage cho Applications (Go 1.20+)

### Build với coverage

```bash
go build -cover -o myapp
```

### Run và collect

```bash
GOCOVERDIR=./coverdata ./myapp
```

### Merge và analyze

```bash
go tool covdata textfmt -i=./coverdata -o coverage.txt
go tool cover -html=coverage.txt
```

## Tích hợp CI/CD

### GitHub Actions

```yaml
name: Coverage
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
      - name: Test with coverage
        run: go test -coverprofile=coverage.out ./...
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.out
```

## Best Practices

1. **Focus on important code**: Không cần 100% coverage
2. **Cover edge cases**: Tests chất lượng quan trọng hơn số lượng
3. **Monitor trends**: Theo dõi coverage theo thời gian

---

*Xem thêm tại [go.dev/testing/coverage](https://go.dev/testing/coverage)*
