# Quá trình phát triển Go

## Giới thiệu

Tài liệu này mô tả quá trình phát triển ngôn ngữ Go và cách bạn có thể tham gia.

## Tổng quan về quy trình

### Workflow

```
Proposal → Discussion → Prototype → Review → Implementation → Release
```

### Timeline

| Phase | Duration |
|-------|----------|
| Development | 6 tháng |
| Freeze | 1 tháng |
| Beta | 1-2 tháng |
| RC | 2-4 tuần |
| Release | - |

## Release Schedule

### Major versions

```
go1.21 → go1.22 → go1.23 → ...
```

- 2 releases mỗi năm
- Khoảng tháng 2 và tháng 8
- Support 2 major versions gần nhất

### Minor versions

```
go1.21.0 → go1.21.1 → go1.21.2 → ...
```

- Theo nhu cầu
- Security fixes
- Critical bug fixes

## Proposal Process

### Tạo proposal

1. **Mở issue**: Tại [github.com/golang/go/issues](https://github.com/golang/go/issues)
2. **Thêm template**: Sử dụng proposal template
3. **Discussion**: Cộng đồng và team thảo luận
4. **Decision**: Accepted, Declined, hoặc On Hold

### Proposal template

```markdown
# Proposal: [Title]

## Summary
Brief description of the proposal.

## Background
Context and motivation.

## Proposal
Detailed description.

## Compatibility
Impact on existing code.

## Implementation
How it would be implemented.
```

### Trạng thái Proposal

| Status | Meaning |
|--------|---------|
| Proposal | Đang thảo luận |
| Accepted | Được chấp nhận |
| Declined | Bị từ chối |
| Withdrawn | Người đề xuất rút lại |
| On Hold | Tạm hoãn |

## Repositories

### Core repositories

| Repo | Description |
|------|-------------|
| `go` | Main Go implementation |
| `tools` | Go tools (gopls, godoc, etc.) |
| `website` | go.dev website |
| `exp` | Experimental packages |
| `proposal` | Design documents |

### Clone

```bash
git clone https://go.googlesource.com/go
cd go/src
./all.bash
```

## Development Tools

### Building Go từ source

```bash
# Clone
git clone https://go.googlesource.com/go goroot
cd goroot/src

# Build
./all.bash

# Sử dụng
export GOROOT=$(pwd)/..
export PATH=$GOROOT/bin:$PATH
go version
```

### Running tests

```bash
cd go/src
./all.bash  # Build và test tất cả

# Hoặc chỉ test
./run.bash --no-rebuild
```

## Issue Tracking

### Labels

| Label | Meaning |
|-------|---------|
| `Proposal` | Đề xuất tính năng mới |
| `NeedsInvestigation` | Cần điều tra thêm |
| `WaitingForInfo` | Đợi thông tin từ reporter |
| `help wanted` | Tìm contributors |
| `Documentation` | Liên quan đến docs |

### Milestones

- `Go1.22`: Sẽ được fix trong 1.22
- `Backlog`: Chưa được lên lịch
- `Unplanned`: Không có kế hoạch cụ thể

## Meetings và Communication

### Mailing lists

- [golang-dev](https://groups.google.com/g/golang-dev): Development discussions
- [golang-nuts](https://groups.google.com/g/golang-nuts): General discussions
- [golang-announce](https://groups.google.com/g/golang-announce): Announcements

### Weekly meetings

- Go team meetings (internal)
- Notes được công bố

### Gophers Slack

- [gophers.slack.com](https://gophers.slack.com)
- `#contributing` channel
- `#godev` channel

## Security Process

### Reporting vulnerabilities

1. Email: security@golang.org
2. Không post publicly
3. Team sẽ respond trong 24h

### Security releases

- Theo nhu cầu
- Thông báo qua golang-announce
- Patch cho 2 versions gần nhất

## Contribution Areas

### Cho beginners

- Documentation improvements
- Test coverage
- Typo fixes
- Issues labeled `help wanted`

### Intermediate

- Bug fixes
- Small features
- Tooling improvements

### Advanced

- Language changes
- Runtime improvements
- Compiler optimizations

## Resources

### Documentation

- [go.dev/doc](https://go.dev/doc)
- [go.dev/ref](https://go.dev/ref)
- [go.dev/blog](https://go.dev/blog)

### Code review

- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Effective Go](https://go.dev/doc/effective_go)

### History

- [Go 1 Release History](https://go.dev/doc/devel/release)
- [Go Release Notes](https://go.dev/doc/devel/release)

## Best Practices

1. **Bắt đầu nhỏ**: Với documentation hoặc tests
2. **Đọc existing code**: Hiểu style
3. **Tham gia discussions**: Trước khi implement
4. **Follow process**: Đặc biệt với changes lớn
5. **Be patient**: Review có thể mất thời gian

---

*Xem thêm tại [go.dev/doc/contribute](https://go.dev/doc/contribute)*
