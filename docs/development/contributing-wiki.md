# Đóng góp vào Wiki Go

## Giới thiệu

Go Wiki là nguồn tài liệu cộng đồng chứa nhiều bài viết, hướng dẫn và tài liệu tham khảo về Go.

## Truy cập Wiki

### URL

[github.com/golang/go/wiki](https://github.com/golang/go/wiki)

### Các trang phổ biến

| Page | Description |
|------|-------------|
| [Home](https://github.com/golang/go/wiki) | Trang chủ |
| [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments) | Code review guidelines |
| [CommonMistakes](https://github.com/golang/go/wiki/CommonMistakes) | Các lỗi thường gặp |
| [LearnConcurrency](https://github.com/golang/go/wiki/LearnConcurrency) | Học concurrency |
| [TableDrivenTests](https://github.com/golang/go/wiki/TableDrivenTests) | Table-driven tests |

## Quy trình đóng góp

### Yêu cầu

1. **GitHub account**: Cần để edit
2. **CLA**: Contributor License Agreement (cho edits lớn)
3. **Hiểu content**: Đọc trước khi edit

### Edit trực tiếp

1. Navigate đến page cần edit
2. Click "Edit" button
3. Thực hiện changes
4. Add commit message
5. Click "Save Page"

### Cho changes lớn

1. **Discuss first**: Mở issue để thảo luận
2. **Draft locally**: Viết content
3. **Request review**: Trong pull request hoặc issue
4. **Incorporate feedback**: Cập nhật theo feedback
5. **Publish**: Sau khi được approve

## Content Guidelines

### Format

```markdown
# Page Title

## Introduction
Brief overview of the topic.

## Main Content
Detailed explanation.

### Subsection
More details.

## Examples
Code examples with explanations.

## Related Pages
Links to related wiki pages.
```

### Code blocks

```markdown
​```go
package main

func main() {
    fmt.Println("Hello, World!")
}
​```
```

### Links

```markdown
[Internal link](PageName)
[External link](https://example.com)
```

## Các loại đóng góp

### 1. Fix typos và errors

- Sửa lỗi chính tả
- Fix broken links
- Update outdated info

### 2. Improve existing content

- Add examples
- Clarify explanations
- Add missing details

### 3. New pages

- Tutorials
- Best practices
- Tool guides
- Language features

### 4. Translations

- Contribute to non-English docs
- Link từ main wiki

## Best Practices

### Writing style

1. **Be concise**: Ngắn gọn, súc tích
2. **Use examples**: Minh họa với code
3. **Stay neutral**: Không promotional
4. **Be accurate**: Verify trước khi publish
5. **Keep updated**: Cập nhật cho Go versions mới

### Organization

```markdown
# Topic

## Overview
What is this about?

## When to Use
When should you use this?

## How to Use
Step-by-step guide.

## Examples
Practical examples.

## Common Pitfalls
What to avoid.

## See Also
Related topics.
```

## Các trang cần contribution

### Documentation gaps

- New language features
- Tooling updates
- Best practices

### Translation

- Non-English documentation
- Localization efforts

### Community resources

- Tutorials
- Case studies
- Tool comparisons

## Moderation

### Guidelines

1. **Relevant**: Liên quan đến Go
2. **Accurate**: Thông tin chính xác
3. **Respectful**: Không xúc phạm
4. **Non-promotional**: Không quảng cáo

### Xử lý violations

- Maintainers có thể revert changes
- Repeat offenders có thể bị block

## Tools

### Preview locally

```bash
# Clone wiki
git clone https://github.com/golang/go.wiki.git

# Preview với markdown viewer
# hoặc GitHub preview
```

### Markdown editors

- VSCode với Markdown preview
- Typora
- Online markdown editors

## Examples của trang tốt

### Code Review Comments

```markdown
# Go Code Review Comments

This page collects common comments made during reviews of Go code.

## Gofmt

Run gofmt on your code to automatically fix the majority of mechanical style issues.

## Comment Sentences

Comments should be complete sentences.

​```go
// Request represents a request to run a command.
type Request struct { ...
​```
```

### Table-Driven Tests

```markdown
# TableDrivenTests

Go's testing package encourages table-driven tests.

## Example

​```go
func TestAdd(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {1, 2, 3},
        {0, 0, 0},
    }
    for _, tt := range tests {
        if got := Add(tt.a, tt.b); got != tt.want {
            t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
        }
    }
}
​```
```

## Liên hệ

### Questions

- Mở issue tại [github.com/golang/go/issues](https://github.com/golang/go/issues)
- Tag với `Documentation`

### Discussions

- golang-dev mailing list
- Gophers Slack #documentation

---

*Xem thêm tại [github.com/golang/go/wiki](https://github.com/golang/go/wiki)*
