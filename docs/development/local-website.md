# Thiết lập và sử dụng Website Go cục bộ

## Giới thiệu

Tài liệu này hướng dẫn cách thiết lập website Go cục bộ để phát triển và đóng góp.

## Tổng quan Architecture

### Các thành phần

```
go.dev website
├── _content/      # Markdown content
├── internal/      # Internal packages
├── cmd/           # Commands
└── static/        # Static assets
```

### Repositories

| Repo | Description |
|------|-------------|
| `website` | go.dev website |
| `pkgsite` | pkg.go.dev |
| `tour` | Tour of Go |

## Cài đặt

### Prerequisites

```bash
# Go 1.21+
go version

# Git
git --version

# Node.js (optional, for some tools)
node --version
```

### Clone repository

```bash
git clone https://go.googlesource.com/website
cd website
```

### Build và run

```bash
go run ./cmd/golangorg

# Server sẽ chạy tại http://localhost:6060
```

## Development Workflow

### Run local server

```bash
# Development mode với auto-reload
go run ./cmd/golangorg -dev

# Hoặc build và run
go build -o golangorg ./cmd/golangorg
./golangorg
```

### Truy cập

- Main site: http://localhost:6060
- Docs: http://localhost:6060/doc
- Blog: http://localhost:6060/blog
- Packages: http://localhost:6060/pkg

## Content Structure

### _content directory

```
_content/
├── doc/            # Documentation
│   ├── effective_go.html
│   ├── tutorial/
│   └── ...
├── blog/           # Blog posts
│   ├── go1.21.md
│   └── ...
└── ref/            # References
    ├── spec.md
    └── ...
```

### File formats

| Format | Usage |
|--------|-------|
| `.md` | Markdown content |
| `.html` | HTML content |
| `.tmpl` | Go templates |

## Editing Content

### Markdown files

```markdown
---
title: My Article
date: 2024-01-15
---

# Introduction

Content here.

## Code Example

​```go
package main

func main() {
    println("Hello")
}
​```
```

### HTML files

```html
<!--{
    "Title": "Page Title",
    "Template": true
}-->

<h2>Content</h2>
<p>Paragraph text.</p>
```

## Testing Changes

### Local testing

```bash
# Run server
go run ./cmd/golangorg -dev &

# Check pages
curl http://localhost:6060/doc/effective_go
```

### Run tests

```bash
go test ./...
```

### Link checking

```bash
# Check for broken links
go run ./cmd/linkcheck http://localhost:6060
```

## Styling và Templates

### Static assets

```
static/
├── css/
│   └── style.css
├── js/
│   └── site.js
└── images/
    └── logo.svg
```

### Templates

```
templates/
├── base.tmpl      # Base template
├── doc.tmpl       # Documentation template
└── blog.tmpl      # Blog template
```

### Modifying styles

```bash
# CSS files trong static/css/
# Rebuild không cần thiết với -dev flag
```

## Contributing Changes

### 1. Create branch

```bash
git checkout -b my-change
```

### 2. Make changes

```bash
# Edit files
vim _content/doc/my-article.md
```

### 3. Test locally

```bash
go run ./cmd/golangorg -dev
# Check http://localhost:6060/doc/my-article
```

### 4. Commit

```bash
git add .
git commit -m "doc: add my-article"
```

### 5. Submit

```bash
# Sử dụng Gerrit (không phải GitHub PRs)
git push origin HEAD:refs/for/master
```

## pkg.go.dev Development

### Clone pkgsite

```bash
git clone https://go.googlesource.com/pkgsite
cd pkgsite
```

### Run locally

```bash
go run ./cmd/frontend
# http://localhost:8080
```

### Database setup (optional)

```bash
# Sử dụng Docker
docker-compose up -d db
```

## Tour of Go Development

### Clone tour

```bash
git clone https://go.googlesource.com/tour
cd tour
```

### Run locally

```bash
go run .
# http://localhost:3999
```

### Edit lessons

```
content/
├── welcome/
│   ├── welcome.article
│   └── ...
├── basics/
└── ...
```

## Docker Setup

### Dockerfile

```dockerfile
FROM golang:1.21

WORKDIR /app
COPY . .

RUN go build -o golangorg ./cmd/golangorg

EXPOSE 6060
CMD ["./golangorg"]
```

### Run với Docker

```bash
docker build -t go-website .
docker run -p 6060:6060 go-website
```

## Troubleshooting

### Port already in use

```bash
# Tìm process
lsof -i :6060

# Kill hoặc sử dụng port khác
go run ./cmd/golangorg -http=:8080
```

### Missing dependencies

```bash
go mod download
go mod tidy
```

### Template errors

```bash
# Check template syntax
go run ./cmd/golangorg -dev 2>&1 | grep -i error
```

## Best Practices

1. **Test locally**: Trước khi submit
2. **Check formatting**: `gofmt -s -w .`
3. **Verify links**: Run link checker
4. **Small commits**: Một change per commit
5. **Clear messages**: Descriptive commit messages

---

*Xem thêm tại [go.googlesource.com/website](https://go.googlesource.com/website)*
