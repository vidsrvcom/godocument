# Fuzz Testing

## Giới thiệu

Go 1.18 giới thiệu fuzzing tích hợp để phát hiện bugs bằng cách tự động tạo inputs ngẫu nhiên.

## Viết Fuzz Tests

### Cơ bản

```go
func FuzzReverse(f *testing.F) {
    // Seed corpus
    f.Add("hello")
    f.Add("world")
    
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        if s != doubleRev {
            t.Errorf("Reverse twice: got %q, want %q", doubleRev, s)
        }
    })
}
```

## Chạy Fuzz Tests

```bash
# Chạy một lần
go test -fuzz=FuzzReverse -fuzztime=30s

# Chạy với CPU
go test -fuzz=FuzzReverse -fuzztime=30s -parallel=4
```

## Corpus

Fuzz inputs được lưu trong `testdata/fuzz/`:

```
testdata/fuzz/FuzzReverse/
├── seed_corpus/
│   └── input1
└── crashers/
    └── crash_abc123
```

## Best Practices

1. **Bắt đầu với seeds tốt**
2. **Test invariants, không phải specific outputs**
3. **Chạy đủ lâu**: 30s - vài phút

---

*Xem thêm tại [go.dev/doc/fuzz](https://go.dev/doc/fuzz)*
