# Thiết lập Website Go Cục bộ

Tài liệu này hướng dẫn cách thiết lập và chạy website Go (go.dev) locally cho development và testing.

## Tổng quan

Website Go chính thức bao gồm:
- **go.dev**: Main website
- **pkg.go.dev**: Package documentation
- **play.golang.org**: Go Playground

## Prerequisites

```bash
# Go installed
go version  # 1.21+

# Git
git version

# Node.js (cho một số tools)
node --version  # 16+
```

## Clone Repositories

### Main Website (go.dev)

```bash
# Clone website repository
git clone https://go.googlesource.com/website
cd website

# hoặc từ GitHub mirror
git clone https://github.com/golang/website
cd website
```

### Package Documentation (pkg.go.dev)

```bash
# Clone pkgsite repository  
git clone https://go.googlesource.com/pkgsite
cd pkgsite
```

## Setup go.dev Locally

### Install Dependencies

```bash
cd website/_content

# Get Go dependencies
go mod download
```

### Run Development Server

```bash
# Từ _content directory
go run ./cmd/golangorg

# Server starts tại http://localhost:6060
```

### Access Website

```
http://localhost:6060        # Main site
http://localhost:6060/doc/   # Documentation
http://localhost:6060/pkg/   # Package index
```

## Setup pkg.go.dev Locally

### Database Setup

Pkgsite yêu cầu PostgreSQL:

```bash
# Install PostgreSQL
# Ubuntu/Debian
sudo apt-get install postgresql

# macOS
brew install postgresql

# Start PostgreSQL
sudo systemctl start postgresql  # Linux
brew services start postgresql   # macOS
```

### Create Database

```bash
# Create user và database
createuser -U postgres pkgsite
createdb -U postgres -O pkgsite discovery_postgres

# Set password
psql -U postgres -c "ALTER USER pkgsite WITH PASSWORD 'your-password';"
```

### Configure Pkgsite

```bash
cd pkgsite

# Set environment variables
export GO_DISCOVERY_DATABASE_NAME=discovery_postgres
export GO_DISCOVERY_DATABASE_USER=pkgsite
export GO_DISCOVERY_DATABASE_PASSWORD=your-password
export GO_DISCOVERY_DATABASE_HOST=localhost
```

### Run Pkgsite

```bash
# Run frontend
go run ./cmd/frontend

# Server starts tại http://localhost:8080
```

### Fetch Packages

```bash
# Fetch specific package
go run ./cmd/worker -fetch=golang.org/x/tools

# Fetch from module proxy
go run ./cmd/worker -fetch -proxy=https://proxy.golang.org
```

## Go Playground Local

### Clone Repository

```bash
git clone https://go.googlesource.com/playground
cd playground
```

### Run Playground

```bash
# Run playground server
go run ./server.go

# Or build và run
go build -o playground ./server.go
./playground

# Access tại http://localhost:8080
```

### Configuration

```bash
# Custom port
./playground -addr=:9000

# With Go version
./playground -gover=1.21.0
```

## Development Workflow

### Making Changes

```go
// Edit content
vim _content/doc/tutorial/getting-started.md

// Edit templates
vim _content/tour/basics/1.article

// Edit Go code
vim cmd/golangorg/main.go
```

### Live Reload

Website tự động reload khi files thay đổi:

```bash
# Run với auto-reload
go run ./cmd/golangorg

# Changes được reflected ngay lập tức
```

### Testing

```bash
# Run tests
go test ./...

# Specific package
go test ./cmd/golangorg

# With coverage
go test -cover ./...
```

## Content Structure

### Website Layout

```
website/
├── _content/          # Main content
│   ├── doc/          # Documentation
│   ├── blog/         # Blog posts
│   ├── tour/         # Go Tour
│   └── cmd/          # Commands
├── cmd/
│   └── golangorg/    # Server code
└── internal/         # Internal packages
```

### Adding Documentation

```markdown
---
title: My New Doc
date: 2024-01-15
---

# My New Documentation

Content here...
```

### Adding Blog Post

```markdown
---
title: Exciting New Feature
date: 2024-01-15
by:
- Author Name
tags:
- feature
---

Blog content here...
```

## Customization

### Site Configuration

```go
// cmd/golangorg/main.go

var (
    httpAddr   = flag.String("http", "localhost:6060", "HTTP serve address")
    verbose    = flag.Bool("v", false, "verbose mode")
)
```

### Templates

```html
<!-- Edit templates directory -->
_content/templates/root.tmpl

{{define "layout"}}
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>
    {{template "content" .}}
</body>
</html>
{{end}}
```

### Styling

```css
/* Edit CSS */
_content/static/css/style.css

body {
    font-family: Arial, sans-serif;
}
```

## Building for Production

### Build Static Site

```bash
# Build all
cd website
go run cmd/golangorg -build
```

### Generate HTML

```bash
# Generate static HTML
go run ./cmd/golangorg -generate

# Output trong _generated/
```

### Deploy

```bash
# Build production binary
go build -o golangorg ./cmd/golangorg

# Run production server
./golangorg -http=:80
```

## Docker Setup

### Dockerfile

```dockerfile
FROM golang:1.21 AS builder

WORKDIR /app
COPY . .

RUN go build -o golangorg ./cmd/golangorg

FROM debian:bullseye-slim

COPY --from=builder /app/golangorg /usr/local/bin/
COPY _content /app/_content

WORKDIR /app
EXPOSE 8080

CMD ["golangorg", "-http=:8080"]
```

### Build và Run

```bash
# Build image
docker build -t go-website .

# Run container
docker run -p 8080:8080 go-website
```

### Docker Compose

```yaml
version: '3'

services:
  website:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - ./_content:/app/_content
    environment:
      - GO_ENV=development
      
  pkgsite:
    build: ./pkgsite
    ports:
      - "9000:8080"
    depends_on:
      - postgres
    environment:
      - GO_DISCOVERY_DATABASE_HOST=postgres
      
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: discovery_postgres
      POSTGRES_USER: pkgsite
      POSTGRES_PASSWORD: password
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Run:

```bash
docker-compose up
```

## Troubleshooting

### Port Already in Use

```bash
# Check process
lsof -i :6060

# Kill process
kill -9 <PID>

# Or use different port
go run ./cmd/golangorg -http=:6061
```

### Database Connection Failed

```bash
# Check PostgreSQL status
systemctl status postgresql

# Test connection
psql -U pkgsite -d discovery_postgres

# Reset database
dropdb discovery_postgres
createdb -O pkgsite discovery_postgres
```

### Module Download Failed

```bash
# Clear module cache
go clean -modcache

# Re-download
go mod download
```

## Development Tips

### Hot Reload

```bash
# Install air (live reload tool)
go install github.com/cosmtrek/air@latest

# Create .air.toml
air init

# Run với auto-reload
air
```

### Debug Mode

```bash
# Run với debug logging
go run ./cmd/golangorg -v

# Or với delve
dlv debug ./cmd/golangorg
```

### Performance

```bash
# Profile server
go run ./cmd/golangorg -cpuprofile=cpu.prof

# Analyze
go tool pprof cpu.prof
```

## Contributing

### Testing Changes

```bash
# Run all tests
go test ./...

# Run specific tests
go test -run TestDocumentation

# With race detector
go test -race ./...
```

### Code Style

```bash
# Format code
gofmt -w .

# Check với golangci-lint
golangci-lint run
```

### Submit Changes

```bash
# Create branch
git checkout -b my-website-fix

# Commit
git commit -m "doc: fix typo in tutorial"

# Submit (sử dụng Gerrit)
git codereview mail
```

## Resources

### Official Repositories

- Website: https://go.googlesource.com/website
- Pkgsite: https://go.googlesource.com/pkgsite
- Playground: https://go.googlesource.com/playground

### Documentation

- Website README: https://github.com/golang/website/blob/master/README.md
- Pkgsite README: https://github.com/golang/pkgsite/blob/master/README.md

### Community

- golang-dev mailing list
- #website channel on Gophers Slack

## Xem thêm

- [Development Process](process.md)
- [Contributing to Wiki](wiki-contributing.md)
- [Đóng góp mã](contributing.md)
