# Quy trình Phát triển Go

Tài liệu này mô tả quy trình phát triển và đóng góp cho dự án Go, bao gồm cả Go toolchain, standard library, và related projects.

## Tổng quan

Go development process được thiết kế để:
- Open và transparent
- Inclusive và welcoming
- Focused và efficient  
- Community-driven

## Getting Started

### Prerequisites

```bash
# Git
git --version

# Go (để build Go from source)
go version

# C compiler (cho runtime)
gcc --version
# hoặc
clang --version
```

### Clone Repository

```bash
# Clone main Go repository
git clone https://go.googlesource.com/go
cd go

# Checkout branch
git checkout master  # hoặc release branch
```

### Build từ Source

```bash
cd src
./all.bash  # Linux/macOS
# hoặc
all.bat     # Windows

# Sau khi build thành công
../bin/go version
```

## Contribution Workflow

### 1. Đăng ký CLA

Trước khi contribute, phải sign Contributor License Agreement:

- Cá nhân: https://cla.developers.google.com/
- Corporate: Contact golang-dev

### 2. Tìm Issue hoặc Propose Change

#### Existing Issues

```
Browse issues: https://github.com/golang/go/issues
Filter: label:help-wanted
Good first issues: label:good-first-issue
```

#### Propose New Feature

```markdown
Subject: proposal: [package] brief description

## Background

Why is this needed?

## Proposal

What do you propose?

## Implementation

How would you implement it?

## Compatibility

Any breaking changes?
```

### 3. Discuss với Community

- File issue trước khi code
- Get feedback early
- Discuss design
- Avoid duplicate work

### 4. Make Changes

#### Create Branch

```bash
git checkout -b my-feature
```

#### Write Code

Follow [Code Style](#code-style):

```go
// Package comment
package mypackage

import (
    "fmt"
)

// Exported function với doc comment
func MyFunction() {
    // Implementation
}
```

#### Write Tests

```go
// mypackage_test.go
package mypackage_test

import "testing"

func TestMyFunction(t *testing.T) {
    // Test implementation
}
```

### 5. Test Changes

#### Run Tests

```bash
# All tests
cd src
./all.bash

# Specific package
go test ./path/to/package

# With race detector
go test -race ./...

# All packages
go test std cmd
```

#### Benchmarks

```bash
# Run benchmarks
go test -bench=. ./path/to/package

# With mem profiling
go test -bench=. -benchmem
```

### 6. Commit Changes

#### Commit Message Format

```
package: brief description

Detailed explanation of changes.
Explain why, not just what.

Fixes #12345
```

Example:

```
encoding/json: improve Unmarshal performance

Optimize allocation in Unmarshal by reusing buffers
where possible. This reduces memory pressure and
improves performance by approximately 20% for typical
JSON documents.

Benchmark results:
BenchmarkUnmarshal-8  old: 1000 ns/op  new: 800 ns/op

Fixes #45678
```

### 7. Code Review

#### Use Gerrit

Go sử dụng Gerrit cho code review:

```bash
# Install git-codereview tool
go install golang.org/x/review/git-codereview@latest

# Configure
git codereview change

# Upload for review
git codereview mail

# Address feedback
git add .
git commit --amend
git codereview mail
```

#### Review Process

1. **Upload CL (Change List)**
2. **Automatic checks** run (tests, builds)
3. **Reviewers** assigned automatically
4. **Feedback** và discussion
5. **Revisions** nếu cần
6. **Approval** từ maintainers
7. **Submit** khi approved

### 8. Address Review Feedback

```bash
# Make changes
vim myfile.go

# Amend commit
git add .
git commit --amend

# Re-upload
git codereview mail
```

### 9. Submit

Sau khi approved:

```bash
# Submit (merges vào main branch)
git codereview submit
```

## Code Style

### Formatting

```bash
# Format code
gofmt -w .

# Or with imports
goimports -w .
```

### Documentation

```go
// Package comment mô tả package
package mypackage

// FunctionName brief description.
//
// Detailed explanation if needed.
// Multiple paragraphs OK.
func FunctionName() {
}
```

### Testing

```go
// Table-driven tests
func TestFunction(t *testing.T) {
    tests := []struct {
        name string
        input int
        want int
    }{
        {"zero", 0, 0},
        {"positive", 5, 5},
        {"negative", -5, 5},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Function(tt.input); got != tt.want {
                t.Errorf("Function(%d) = %d, want %d", tt.input, got, tt.want)
            }
        })
    }
}
```

## Development Tools

### git-codereview

```bash
# Install
go install golang.org/x/review/git-codereview@latest

# Commands
git codereview change <branch>  # Create/switch branch
git codereview mail             # Upload for review
git codereview sync             # Sync với upstream
git codereview submit           # Submit approved change
```

### gopls

```bash
# Install language server
go install golang.org/x/tools/gopls@latest

# Use với editor (VS Code, vim, etc.)
```

### go test

```bash
# Run tests
go test ./...

# Verbose
go test -v

# Specific test
go test -run TestName

# Coverage
go test -cover

# Race detection
go test -race
```

## Release Process

### Release Cycle

- **Major releases**: Mỗi 6 tháng (Feb, Aug)
- **Minor releases**: Bug fixes, security patches
- **Beta/RC**: Trước major release

### Release Branch

```bash
# Release branches
git checkout release-branch.go1.21
```

## Communication Channels

### Mailing Lists

- **golang-nuts**: General discussion
- **golang-dev**: Development discussion
- **golang-announce**: Announcements

### Issue Tracker

- GitHub: https://github.com/golang/go/issues
- Discussion labels
- Proposals

### Slack

- Gophers Slack: https://gophers.slack.com
- Many topic channels
- Realtime discussion

### Forums

- Go Forum: https://forum.golangbridge.org
- Reddit: r/golang
- Stack Overflow: [go] tag

## Proposal Process

### When to Propose

- New language features
- Standard library additions
- Major changes
- Breaking changes

### Proposal Template

```markdown
# Proposal: [Title]

Author(s): @username
Last updated: YYYY-MM-DD
Discussion at: https://golang.org/issue/NNNNN

## Abstract

One paragraph summary.

## Background

Why is this needed?

## Proposal

Detailed proposal.

## Rationale

Why this approach?

## Compatibility

Any breaking changes?

## Implementation

How would this be implemented?

## Open issues

What's still unclear?
```

### Review Process

1. **File proposal issue**
2. **Community feedback** (21 days minimum)
3. **Proposal review meeting**
4. **Accept, decline, or hold**
5. **If accepted**: Implementation

## Best Practices

### For Contributors

1. **Start small**: Bug fixes, documentation
2. **Read existing code**: Learn style
3. **Ask questions**: Before coding
4. **Test thoroughly**: Multiple platforms
5. **Follow guidelines**: Code style, commits
6. **Be patient**: Review takes time
7. **Stay engaged**: Respond to feedback

### For Reviewers

1. **Be constructive**: Helpful feedback
2. **Be specific**: Point to examples
3. **Be timely**: Don't block contributors
4. **Be respectful**: Everyone is learning

## Resources

### Documentation

- Go.dev: https://go.dev/doc/
- Effective Go: https://go.dev/doc/effective_go
- Code Review Comments: https://go.dev/wiki/CodeReviewComments

### Guides

- Contributing: https://go.dev/doc/contribute
- git-codereview: https://pkg.go.dev/golang.org/x/review/git-codereview
- Go Developer Guide: https://go.dev/doc/devel

### Tools

- Gerrit Code Review: https://go-review.googlesource.com
- Build Dashboard: https://build.golang.org
- Performance Dashboard: https://perf.golang.org

## Example: Complete Contribution

```bash
#!/bin/bash
# Complete contribution workflow

# 1. Setup
git clone https://go.googlesource.com/go
cd go
git codereview change my-feature

# 2. Make changes
vim src/mypackage/myfile.go
vim src/mypackage/myfile_test.go

# 3. Test
cd src
./all.bash

# 4. Commit
git add .
git commit -m "mypackage: add new feature

Detailed explanation.

Fixes #12345"

# 5. Upload for review
git codereview mail

# 6. Address feedback
vim src/mypackage/myfile.go
git add .
git commit --amend
git codereview mail

# 7. Submit (after approval)
git codereview submit
```

## Xem thêm

- [Đóng góp mã](contributing.md)
- [Contributing to Wiki](wiki-contributing.md)
- [Source Code](source-code.md)
