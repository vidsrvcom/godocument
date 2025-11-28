# Gopls - Go Language Server

## Giới thiệu

Gopls (phát âm: "go please") là Go language server chính thức. Nó cung cấp các tính năng IDE như auto-completion, go-to-definition, find references cho các editors hỗ trợ Language Server Protocol (LSP).

## Cài đặt

```bash
go install golang.org/x/tools/gopls@latest
```

Đảm bảo `$GOPATH/bin` (hoặc `$HOME/go/bin`) trong PATH.

## Tính năng

### Auto-completion

Gợi ý tự động khi gõ code:

```go
fmt.Pr // Hiển thị: Println, Printf, Print, ...
```

### Go to Definition

Nhảy đến định nghĩa của symbol (F12 trong VS Code).

### Find References

Tìm tất cả các nơi sử dụng symbol.

### Rename

Đổi tên symbol trong toàn bộ workspace.

### Hover

Hiển thị documentation khi hover qua symbol.

### Signature Help

Hiển thị tham số function khi gọi.

### Diagnostics

Phát hiện lỗi và warnings trong thời gian thực.

### Code Actions

- Organize imports
- Extract variable/function
- Inline variable
- Fill struct
- Generate interface stubs

### Formatting

Tự động format code sử dụng gofmt/goimports.

## Cấu hình

### VS Code

```json
// settings.json
{
    "go.useLanguageServer": true,
    "gopls": {
        "ui.completion.usePlaceholders": true,
        "ui.semanticTokens": true
    }
}
```

### Vim/Neovim

```vim
" coc.nvim
:CocInstall coc-go
```

Hoặc với native LSP:

```lua
-- init.lua
require('lspconfig').gopls.setup{
    settings = {
        gopls = {
            analyses = {
                unusedparams = true,
            },
            staticcheck = true,
        },
    },
}
```

### Emacs

```elisp
(use-package lsp-mode
  :commands lsp
  :hook (go-mode . lsp-deferred))
```

## Gopls Settings

### Analyses

Bật các static analysis checks:

```json
{
    "gopls": {
        "analyses": {
            "unusedparams": true,
            "shadow": true,
            "nilness": true
        }
    }
}
```

Các analyses phổ biến:
| Analysis | Mô tả |
|----------|-------|
| unusedparams | Tham số không sử dụng |
| shadow | Variable shadowing |
| nilness | Nil checks |
| unusedwrite | Writes không sử dụng |
| fieldalignment | Struct field alignment |

### Staticcheck

Tích hợp staticcheck:

```json
{
    "gopls": {
        "staticcheck": true
    }
}
```

### Build Flags

```json
{
    "gopls": {
        "buildFlags": ["-tags=integration"]
    }
}
```

### Env

```json
{
    "gopls": {
        "env": {
            "GOFLAGS": "-mod=vendor"
        }
    }
}
```

### Formatting

```json
{
    "gopls": {
        "formatting.gofumpt": true
    }
}
```

### UI Settings

```json
{
    "gopls": {
        "ui.completion.usePlaceholders": true,
        "ui.completion.matcher": "Fuzzy",
        "ui.semanticTokens": true,
        "ui.codelenses": {
            "gc_details": true,
            "generate": true,
            "test": true
        }
    }
}
```

## Code Lenses

Gopls cung cấp code lenses:

- `gc_details`: Hiển thị thông tin từ compiler
- `generate`: Chạy go generate
- `test`: Chạy tests

## Troubleshooting

### Kiểm tra phiên bản

```bash
gopls version
```

### Debug logs

```bash
gopls -rpc.trace serve
```

### Trong VS Code

1. View → Output
2. Chọn "gopls" từ dropdown

### Các vấn đề phổ biến

#### Gopls không tìm thấy packages

```bash
# Đảm bảo go.mod tồn tại
go mod init example.com/myproject

# Tải dependencies
go mod tidy
```

#### Slow performance

```json
{
    "gopls": {
        "memoryMode": "DegradeClosed"
    }
}
```

#### Quá nhiều CPU usage

Thử cấu hình:

```json
{
    "gopls": {
        "diagnosticsDelay": "500ms"
    }
}
```

## Cập nhật

### Cập nhật gopls

```bash
go install golang.org/x/tools/gopls@latest
```

### Kiểm tra updates

```bash
go list -m -u golang.org/x/tools/gopls
```

## So sánh với tools khác

| Feature | gopls | gocode (deprecated) | godef (deprecated) |
|---------|-------|---------------------|-------------------|
| Completion | ✓ | ✓ | ✗ |
| Definition | ✓ | ✗ | ✓ |
| References | ✓ | ✗ | ✗ |
| Rename | ✓ | ✗ | ✗ |
| Format | ✓ | ✗ | ✗ |
| Diagnostics | ✓ | ✗ | ✗ |

## Resources

- [gopls documentation](https://pkg.go.dev/golang.org/x/tools/gopls)
- [gopls settings](https://github.com/golang/tools/blob/master/gopls/doc/settings.md)
- [Issue tracker](https://github.com/golang/go/issues?q=is%3Aissue+is%3Aopen+label%3Agopls)

---

*Xem tài liệu đầy đủ tại [go.dev/s/gopls](https://go.dev/s/gopls)*
