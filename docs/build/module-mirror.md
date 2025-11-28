# Module Mirror và Checksum Database

## Giới thiệu

Go sử dụng module proxy và checksum database để đảm bảo tính toàn vẹn và khả dụng của dependencies.

## Module Proxy

### GOPROXY

```bash
# Mặc định
GOPROXY=https://proxy.golang.org,direct
```

### Cách hoạt động

1. Go tool request module từ proxy
2. Proxy cache và serve modules
3. Fallback đến direct nếu cần

### Tùy chọn

```bash
# Chỉ sử dụng proxy
GOPROXY=https://proxy.golang.org

# Chỉ direct
GOPROXY=direct

# Private proxy
GOPROXY=https://internal.proxy.com,https://proxy.golang.org,direct
```

## Checksum Database

### GOSUMDB

```bash
# Mặc định
GOSUMDB=sum.golang.org
```

### Cách hoạt động

- Verify checksums của modules
- Phát hiện tampering
- Đảm bảo reproducible builds

## Private Modules

### GOPRIVATE

```bash
go env -w GOPRIVATE=*.corp.example.com,github.com/myorg/*
```

### GONOSUMDB

```bash
go env -w GONOSUMDB=*.corp.example.com
```

### GONOPROXY

```bash
go env -w GONOPROXY=*.corp.example.com
```

## Best Practices

1. **Sử dụng proxy mặc định cho public modules**
2. **Cấu hình GOPRIVATE cho internal modules**
3. **Giữ go.sum trong version control**

---

*Xem thêm tại [go.dev/ref/mod#module-proxy](https://go.dev/ref/mod#module-proxy)*
