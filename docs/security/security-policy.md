# Chính sách Bảo mật Go

## Giới thiệu

Tài liệu này mô tả cách nhóm Go xử lý các lỗi bảo mật trong ngôn ngữ Go, thư viện và các công cụ.

## Phạm vi

Chính sách này áp dụng cho:
- Go programming language
- Standard library
- Các công cụ trong main Go distribution
- golang.org/x/* repositories

## Báo cáo Vấn đề Bảo mật

### Cách Báo cáo

Gửi email đến: **security@golang.org**

Email nên bao gồm:
1. Mô tả rõ ràng về vấn đề
2. Steps to reproduce
3. Phiên bản Go affected
4. Impact tiềm năng
5. Proposed fix (nếu có)

### KHÔNG NÊN

- Không công bố công khai trước khi có fix
- Không báo cáo qua GitHub Issues
- Không báo cáo qua Slack/Forum

### Template báo cáo

```
Subject: [Security] Brief description

## Summary
Clear description of the vulnerability.

## Affected Versions
- Go 1.20.x
- Go 1.21.x

## Steps to Reproduce
1. Step 1
2. Step 2
3. ...

## Impact
What can an attacker do?

## Suggested Fix (optional)
How might this be fixed?
```

## Quy trình Xử lý

### 1. Xác nhận (24-48 giờ)

Nhóm bảo mật sẽ:
- Xác nhận đã nhận report
- Đánh giá ban đầu
- Assign người phụ trách

### 2. Đánh giá (1-7 ngày)

- Xác minh vulnerability
- Đánh giá severity
- Xác định phạm vi ảnh hưởng

### 3. Phát triển Fix

- Develop patch
- Internal review
- Testing

### 4. Disclosure

- Coordinate với reporter
- Chuẩn bị security advisory
- Release fix

### 5. Public Announcement

- Publish advisory
- Notify community
- Update documentation

## Severity Levels

### Critical

- Remote code execution
- Privilege escalation
- Data breach
- No user interaction required

### High

- Denial of service (significant)
- Authentication bypass
- Information disclosure (significant)

### Medium

- Denial of service (limited)
- Information disclosure (limited)
- CSRF, XSS trong standard library

### Low

- Minor information leak
- Theoretical attacks
- Defense in depth improvements

## Responsible Disclosure

### Timeline

| Severity | Target Fix Time |
|----------|-----------------|
| Critical | 7 ngày |
| High | 30 ngày |
| Medium | 90 ngày |
| Low | Best effort |

### Embargo Period

- Thông tin giữ bí mật cho đến khi fix được release
- Researcher được credit trong advisory
- Coordinated disclosure với các vendors affected

## Security Advisories

### Nơi Publish

- [go.dev/security](https://go.dev/security)
- [CVE database](https://cve.mitre.org/)
- [GitHub Security Advisories](https://github.com/golang/go/security/advisories)

### Định dạng Advisory

```markdown
## CVE-YYYY-XXXXX: Brief Title

### Severity
High

### Affected Versions
- Go 1.20 before 1.20.4
- Go 1.19 before 1.19.9

### Description
Detailed description of the vulnerability.

### Workaround
Temporary mitigation if available.

### Credits
Thanks to [Reporter Name] for reporting this issue.
```

## Supported Versions

Go hỗ trợ hai phiên bản minor gần nhất:

| Version | Support Status |
|---------|---------------|
| Go 1.22 | ✅ Supported |
| Go 1.21 | ✅ Supported |
| Go 1.20 | ❌ End of life |

## Liên hệ

- **Security reports**: security@golang.org
- **General questions**: golang-nuts mailing list
- **Development**: golang-dev mailing list

## FAQ

### Q: Có bug bounty không?

A: Không. Go không có chương trình bug bounty chính thức.

### Q: Khi nào issue được công bố?

A: Sau khi fix được release và có đủ thời gian cho users update.

### Q: Tôi có được credit không?

A: Có, reporters được credit trong security advisory.

### Q: Report của tôi có giữ bí mật không?

A: Có, trong suốt quá trình xử lý. Chỉ công bố sau khi fix.

---

*Xem thêm tại [go.dev/security/policy](https://go.dev/security/policy)*
