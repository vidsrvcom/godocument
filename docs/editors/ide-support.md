# Hỗ trợ Plugin và IDE cho Biên tập viên

## Giới thiệu

Go có hỗ trợ tốt từ nhiều IDE và text editors phổ biến thông qua các plugins và extensions.

## Visual Studio Code

### Cài đặt

1. Cài đặt [VS Code](https://code.visualstudio.com/)
2. Cài đặt extension [Go for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=golang.Go)

### Tính năng

- IntelliSense (auto-completion)
- Go to Definition
- Find References
- Syntax highlighting
- Debugging
- Test runner
- Linting
- Formatting

### Cấu hình

```json
// settings.json
{
    "go.useLanguageServer": true,
    "go.lintTool": "golangci-lint",
    "editor.formatOnSave": true,
    "[go]": {
        "editor.defaultFormatter": "golang.go"
    }
}
```

## GoLand

[GoLand](https://www.jetbrains.com/go/) là IDE chuyên dụng cho Go từ JetBrains.

### Tính năng

- Full IDE experience
- Advanced refactoring
- Database tools
- Git integration
- Debugger
- Profiler
- Test runner

### Cấu hình GOROOT

1. File → Settings → Go → GOROOT
2. Chọn thư mục cài đặt Go

## Vim/Neovim

### vim-go

Plugin phổ biến nhất cho Vim:

```vim
" Sử dụng vim-plug
Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }
```

### Tính năng

- Syntax highlighting
- Auto-completion (với gopls)
- Go to definition
- Build và test integration
- Formatting

### Cấu hình

```vim
" .vimrc
let g:go_def_mode='gopls'
let g:go_info_mode='gopls'
let g:go_fmt_command = "goimports"
let g:go_auto_type_info = 1
```

### Neovim với LSP

```lua
-- init.lua
require('lspconfig').gopls.setup{}
```

## Emacs

### go-mode

```elisp
(use-package go-mode
  :ensure t
  :hook ((go-mode . lsp-deferred)
         (go-mode . company-mode)))

(use-package lsp-mode
  :ensure t
  :commands (lsp lsp-deferred))
```

### Tính năng

- Syntax highlighting
- Auto-completion
- Gofmt integration
- Build integration

## Sublime Text

### GoSublime (deprecated)

Sử dụng LSP thay thế:

1. Install Package Control
2. Install "LSP" package
3. Install "LSP-gopls" package

### Cấu hình

```json
// LSP.sublime-settings
{
    "clients": {
        "gopls": {
            "command": ["gopls"],
            "enabled": true,
            "selector": "source.go"
        }
    }
}
```

## Atom (Sunset)

Atom đã ngừng phát triển. Recommend chuyển sang VS Code.

## Công cụ hỗ trợ

### gopls (Go Language Server)

Tất cả editors hiện đại nên sử dụng gopls:

```bash
go install golang.org/x/tools/gopls@latest
```

### goimports

Format và quản lý imports:

```bash
go install golang.org/x/tools/cmd/goimports@latest
```

### golangci-lint

Meta-linter:

```bash
# macOS
brew install golangci-lint

# Linux/Windows
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

### delve (Debugger)

```bash
go install github.com/go-delve/delve/cmd/dlv@latest
```

## So sánh

| Editor | Miễn phí | Tính năng | Độ phức tạp |
|--------|----------|-----------|-------------|
| VS Code | ✓ | Đầy đủ | Thấp |
| GoLand | ✗ | Đầy đủ+ | Thấp |
| Vim/Neovim | ✓ | Đầy đủ | Cao |
| Emacs | ✓ | Đầy đủ | Cao |
| Sublime Text | ✗ | Cơ bản | Trung bình |

## Recommendation

- **Người mới**: VS Code với Go extension
- **Professional**: GoLand
- **Power users**: Vim/Neovim hoặc Emacs với LSP

---

*Xem thêm tại [go.dev/doc/editors](https://go.dev/doc/editors)*
