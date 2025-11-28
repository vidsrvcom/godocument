# Đóng góp vào Go Wiki

Go Wiki là tài nguyên community-maintained chứa guides, tutorials, và best practices. Tài liệu này hướng dẫn cách đóng góp vào Wiki.

## Giới thiệu

Go Wiki location:
- Main Wiki: https://go.dev/wiki
- GitHub: https://github.com/golang/go/wiki

Wiki chứa:
- How-to guides
- Tutorials
- Resources
- Tips và tricks
- Community content

## Getting Started

### Prerequisites

- GitHub account
- Basic markdown knowledge
- Familiarity với Go

### First Time Setup

1. **Visit Wiki**: https://github.com/golang/go/wiki
2. **Read existing pages**: Get familiar với style
3. **Check Page List**: Tránh duplicate content

## Contributing Guidelines

### When to Add Content

✅ **Good additions**:
- Practical how-to guides
- Common gotchas và solutions
- Tool usage examples
- Platform-specific guides
- Best practices

❌ **Avoid**:
- Basic info (covered in official docs)
- Opinionated rants
- Promotional content
- Outdated information

### Page Naming

```
Good:
- HowToDoX
- CommonMistakes
- SettingUpGoOnWindows

Bad:
- page1
- misc
- tips
```

## Writing Content

### Markdown Format

```markdown
# Page Title

Brief introduction explaining the purpose.

## Section 1

Content here.

### Subsection

More specific content.

## Code Examples

Code goes here:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Wiki!")
}
```

## Links

Link to [other pages](OtherPage) or [external sites](https://example.com).
```

### Code Examples

```markdown
## Example: JSON Parsing

Here's how to parse JSON in Go:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
)

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    data := `{"name":"Alice","age":30}`
    
    var person Person
    if err := json.Unmarshal([]byte(data), &person); err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("%+v\n", person)
}
```

This will output: `{Name:Alice Age:30}`
```

### Linking Pages

```markdown
## See Also

- [Related Topic](RelatedTopic) - Internal wiki link
- [Official Docs](https://go.dev/doc/) - External link
- [Package Documentation](https://pkg.go.dev/encoding/json) - Package docs
```

## Editing Process

### Create New Page

1. **Navigate to Wiki**: https://github.com/golang/go/wiki
2. **Click "New Page"** button
3. **Enter page name**: Follow naming convention
4. **Write content**: Use markdown
5. **Preview**: Check formatting
6. **Save**: Add commit message

### Edit Existing Page

1. **Navigate to page**
2. **Click "Edit"** button
3. **Make changes**
4. **Preview**
5. **Save với meaningful commit message**

### Commit Messages

```
Good:
- "Add section on error handling"
- "Update Go version to 1.21"
- "Fix typo in code example"
- "Add Windows-specific instructions"

Bad:
- "update"
- "fix"
- "changes"
```

## Page Structure

### Standard Template

```markdown
# Page Title

> **Note**: Brief context or important info

## Overview

What this page covers.

## Prerequisites

What reader needs to know:
- Go installation
- Basic Go knowledge
- etc.

## Step-by-Step Guide

### Step 1: First Thing

Detailed instructions.

```bash
command here
```

### Step 2: Next Thing

More instructions.

```go
code here
```

## Common Issues

### Issue 1

Problem description và solution.

### Issue 2

Another issue and fix.

## Best Practices

1. Do this
2. Don't do that
3. Consider this

## See Also

- [Related Page](RelatedPage)
- [Official Docs](https://go.dev/doc/)

## References

- Source 1
- Source 2
```

## Style Guidelines

### Formatting

```markdown
# Headers

Use ATX-style headers (# ## ###)

# H1 - Page title only
## H2 - Major sections
### H3 - Subsections

# Emphasis

*italic* or _italic_
**bold** or __bold__
`code`

# Lists

- Unordered
- List
- Items

1. Ordered
2. List
3. Items

# Code

Inline `code` với backticks.

Block code:
```go
code here
```
```

### Voice và Tone

- Clear và concise
- Helpful và friendly
- Neutral (avoid opinion)
- Educational

```markdown
Good:
"To install Go on Windows, download the MSI installer..."

Bad:
"Obviously, you should use Windows for Go development..."
```

### Code Quality

```go
// Good: Complete, runnable example
package main

import "fmt"

func main() {
    // Clear comment
    result := add(2, 3)
    fmt.Println(result) // Output: 5
}

func add(a, b int) int {
    return a + b
}

// Bad: Incomplete snippet
result := add(2, 3)  // Where's the package? Imports?
```

## Maintenance

### Keeping Content Updated

```markdown
## Update Log

Last updated: 2024-01-15 (Go 1.21)

Previous updates:
- 2023-08-01: Updated for Go 1.20
- 2023-02-15: Initial version
```

### Deprecation Notices

```markdown
> **⚠️ Deprecated**: This method is deprecated as of Go 1.20.
> Use [NewMethod](NewMethodPage) instead.
```

### Version Information

```markdown
## Go Version

This guide applies to:
- Go 1.21+
- Older versions: see [Legacy Guide](LegacyGuide)
```

## Review Process

### Self-Review Checklist

- [ ] Correct markdown syntax
- [ ] Code examples run correctly
- [ ] Links work
- [ ] Spelling và grammar
- [ ] Appropriate detail level
- [ ] Up-to-date information

### Peer Review

1. Share draft với community
2. Ask for feedback on golang-nuts
3. Incorporate suggestions
4. Publish when ready

## Common Pages to Contribute To

### Popular Pages

- **How to Write Go Code**: Best practices
- **CommonMistakes**: Pitfalls và fixes
- **GoGetTools**: Tool installation guides
- **Mobile**: Mobile development
- **Modules**: Module management

### Page Categories

- **Installation**: Platform-specific setup
- **Development**: Tools và workflows
- **Deployment**: Production tips
- **Performance**: Optimization guides
- **Testing**: Testing strategies

## Tools và Resources

### Markdown Editors

- GitHub web editor (built-in)
- VS Code với Markdown Preview
- Typora
- Any text editor

### Markdown References

- [GitHub Flavored Markdown](https://github.github.com/gfm/)
- [Markdown Guide](https://www.markdownguide.org/)

### Go Playground

Link to runnable examples:

```markdown
See [this example](https://go.dev/play/p/ABC123) on Go Playground.
```

## Best Practices

### Do's

✅ Keep examples simple và focused  
✅ Provide complete, runnable code  
✅ Include expected output  
✅ Link to official docs  
✅ Update outdated content  
✅ Fix typos và errors  
✅ Add helpful context  

### Don'ts

❌ Don't duplicate official docs  
❌ Don't include incomplete code  
❌ Don't use deprecated features  
❌ Don't add opinionated content  
❌ Don't make breaking edits without discussion  

## Community

### Get Help

- **golang-nuts**: General questions
- **golang-dev**: Development discussion
- **Golang Slack**: Realtime help

### Discuss Changes

Before major changes:

1. Post on golang-nuts
2. File GitHub issue
3. Get community input
4. Proceed với consensus

## Example Contributions

### Simple Addition

```markdown
## Adding a Tip

Added to CommonMistakes page:

### Forgetting to Check Errors

A common mistake is ignoring errors:

```go
// Bad
file, _ := os.Open("file.txt")

// Good
file, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
```

Always check errors explicitly.
```

### New Page

```markdown
# How to Deploy Go Apps with Docker

## Overview

Guide to containerizing Go applications.

## Prerequisites

- Go 1.21+
- Docker installed
- Basic Docker knowledge

## Creating a Dockerfile

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM debian:bullseye-slim
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

## Building

```bash
docker build -t myapp .
docker run myapp
```

## See Also

- [Official Docker Docs](https://docs.docker.com/)
- [Go Modules](Modules)
```

## Xem thêm

- [Development Process](process.md)
- [Đóng góp mã](contributing.md)
- [Go Wiki](https://go.dev/wiki)
