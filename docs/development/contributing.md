# Đóng góp mã cho Go

## Giới thiệu

Go là dự án mã nguồn mở và chào đón đóng góp từ cộng đồng.

## Quy trình

### 1. Sign the CLA

Trước khi đóng góp, bạn cần ký Contributor License Agreement.

### 2. Fork và Clone

```bash
git clone https://go.googlesource.com/go
cd go
```

### 3. Tạo branch

```bash
git checkout -b my-feature
```

### 4. Make changes

- Follow Go style
- Add tests
- Update documentation

### 5. Commit

```bash
git add .
git commit -m "module: brief description"
```

### 6. Submit

Sử dụng Gerrit (không phải GitHub PRs):

```bash
git push origin HEAD:refs/for/master
```

## Code Review

- Maintainers review changes
- Address feedback
- Iterate until approved

## Coding Guidelines

### Style

```go
// Package comment
// Package foo provides ...
package foo

// Exported function comment
// DoSomething does something important.
func DoSomething() {
    // ...
}
```

### Testing

```go
func TestDoSomething(t *testing.T) {
    result := DoSomething()
    if result != expected {
        t.Errorf("DoSomething() = %v; want %v", result, expected)
    }
}
```

## Resources

- [Contribution Guide](https://go.dev/doc/contribute)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Issue Tracker](https://github.com/golang/go/issues)

---

*Xem thêm tại [go.dev/doc/contribute](https://go.dev/doc/contribute)*
