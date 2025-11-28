# Quyền riêng tư và Telemetry

Quyền riêng tư là ưu tiên hàng đầu trong thiết kế Go telemetry. Tài liệu này giải thích chi tiết về các biện pháp bảo vệ quyền riêng tư và cam kết của Go team.

## Nguyên tắc Quyền riêng tư

### 1. Opt-in mặc định

```bash
# Telemetry TẮT mặc định
# User phải explicitly enable

go telemetry on  # Requires user action
```

### 2. Transparency

```bash
# User có thể xem chính xác dữ liệu được thu thập
go telemetry local
go telemetry view

# Có thể review trước khi enable uploading
```

### 3. Minimal Data Collection

Chỉ thu thập dữ liệu cần thiết để cải thiện Go:

- ✅ Command counters
- ✅ Platform stats (aggregate)
- ❌ **KHÔNG** source code
- ❌ **KHÔNG** personal information

### 4. Anonymization

Tất cả dữ liệu được anonymize:

```
KHÔNG có:
- User IDs
- Machine IDs
- IP addresses
- Hostnames
- Email addresses
```

### 5. User Control

```bash
# Bật uploading
go telemetry on

# Chỉ local collection
go telemetry local

# Tắt hoàn toàn
go telemetry off

# Xóa dữ liệu
rm -rf ~/.config/go/telemetry/
```

## Dữ liệu được Bảo vệ

### Personal Information

**KHÔNG bao giờ** thu thập:

- ❌ Names
- ❌ Email addresses
- ❌ Company/organization names
- ❌ Geographic location
- ❌ IP addresses
- ❌ User accounts

### Source Code

**KHÔNG bao giờ** thu thập:

- ❌ File contents
- ❌ Function implementations
- ❌ Variable names
- ❌ Comments
- ❌ Package names (specific)
- ❌ Import paths (specific)

### File Paths

Paths được sanitize trong crash reports:

```
❌ Original:
/home/john/secret-project/internal/auth/password.go:123

✅ Sanitized:
<redacted>/password.go:123
```

### Environment

**KHÔNG** thu thập:

- ❌ Environment variables
- ❌ System configuration
- ❌ Network settings
- ❌ Installed software

## Data Anonymization

### Aggregation

Dữ liệu được aggregate từ nhiều users:

```
Instead of:
  User A: go build (10 times)
  User B: go build (15 times)
  User C: go build (20 times)

Reported as:
  Total: go build (45 times)
  Median: 15
```

### No Individual Tracking

```
❌ KHÔNG có:
  "User X used go build 10 times"

✅ Chỉ có:
  "go build was used 10,000 times by all users"
```

### Random Sampling

Nếu cần sampling, sử dụng randomization:

```go
// Conceptual example
if rand.Float64() < 0.1 {  // 10% sample
    recordMetric()
}
```

## Data Transmission Security

### HTTPS Encryption

```
All uploads:
- Protocol: HTTPS (TLS 1.3)
- Endpoint: telemetry.go.dev
- Certificate: Pinned
```

### No Tracking

```
Server không track:
- IP addresses (logged nhưng không stored)
- Session IDs
- Request patterns
```

## Data Storage

### Client-side

```bash
# Local storage location
~/.config/go/telemetry/

# Permissions
drwx------ (0700) - Only owner can access

# Retention
- Automatic rotation
- Max 7 days local storage
```

### Server-side

Google's telemetry servers:

- No user identifiers stored
- Aggregated data only  
- Automatic retention limits
- GDPR compliant
- SOC 2 certified

## Legal Compliance

### GDPR (Europe)

- ✅ Explicit opt-in consent
- ✅ Right to access data
- ✅ Right to deletion
- ✅ Data minimization
- ✅ Anonymous data

### CCPA (California)

- ✅ Disclosure of data collected
- ✅ Opt-out mechanism
- ✅ No sale of personal information
- ✅ Transparent privacy practices

### Other Regulations

Compatible với:
- PIPEDA (Canada)
- LGPD (Brazil)
- PDPA (Singapore)

## User Rights

### Right to Know

```bash
# Xem dữ liệu được thu thập
go telemetry view

# Export data
go telemetry view > my-data.txt
```

### Right to Control

```bash
# Disable telemetry
go telemetry off

# Enable với control
go telemetry local  # No upload
go telemetry on     # With upload
```

### Right to Delete

```bash
# Delete local data
rm -rf ~/.config/go/telemetry/

# Server-side: aggregated data không có individual records để delete
```

### Right to Opt-out

```bash
# Permanently opt-out
go telemetry off

# Prevent accidental enable
echo "off" > ~/.config/go/telemetry/mode
chmod 444 ~/.config/go/telemetry/mode  # Read-only
```

## Privacy by Design

### Default Privacy

```
Telemetry OFF by default
Không tự động enable
Không có "dark patterns"
Clear prompts và documentation
```

### Minimal Collection

```
Chỉ counters và aggregates
Không có detailed logs
Không có realtime tracking
Batch uploads (not streaming)
```

### Data Lifecycle

```
1. Collection: Local counters only
2. Aggregation: Daily/weekly batches
3. Upload: Encrypted HTTPS
4. Storage: Aggregated only
5. Retention: Auto-pruning
6. Deletion: User controlled
```

## Transparency Measures

### Open Documentation

- Public privacy policy
- Detailed data collection docs
- Regular transparency reports
- Open source telemetry client

### User Visibility

```bash
# Users có thể:
# 1. See exact data collected
go telemetry view

# 2. Choose upload or not
go telemetry local

# 3. Disable anytime
go telemetry off
```

### Public Reporting

Go team publishes:
- Aggregated statistics
- High-level insights
- Feature priorities based on data

## Enterprise Privacy

### Corporate Policies

```bash
# Organization có thể:

# 1. Disable telemetry globally
export GOTELEMETRY=off

# 2. Set policy
go telemetry off  # trong corporate image

# 3. Network restrictions
# Block telemetry.go.dev nếu cần
```

### Data Residency

```
Telemetry servers:
- Located: Google Cloud (multiple regions)
- Data processing: Compliant với local laws
- Cross-border: Minimal, aggregated only
```

## Incident Response

### Data Breach Protocol

Nếu có incident:

1. **Immediate notification** của affected users
2. **Investigation và remediation**
3. **Public disclosure**
4. **Enhanced security measures**

Note: **Aggregated, anonymous data** giảm thiểu impact.

### Security Measures

```
- Regular security audits
- Penetration testing
- Secure development practices
- Incident response plan
```

## Trust và Verification

### Verify Implementation

```bash
# Source code:
# https://go.googlesource.com/go

# Telemetry implementation:
# src/cmd/go/internal/telemetry/

# Users có thể audit code
```

### Third-party Audits

- Independent security assessments
- Privacy compliance audits
- Regular reviews

## Questions và Concerns

### "How do I know data is really anonymized?"

```
1. Review collection code (open source)
2. View local data (go telemetry view)
3. Check uploaded format (no IDs)
4. Read privacy policy
5. Trust but verify
```

### "What if I change my mind?"

```bash
# Có thể disable bất cứ lúc nào
go telemetry off

# Delete local data
rm -rf ~/.config/go/telemetry/
```

### "Can Google track me?"

```
NO. Telemetry data không liên kết:
- Với Google accounts
- Với IP addresses
- Với search history
- Với bất kỳ identifier nào
```

### "What about corporate secrets?"

```
SAFE. Telemetry KHÔNG thu thập:
- Source code
- Package names
- File names
- Environment variables
- Network information
```

## Best Practices

### For Individual Developers

1. **Review before enabling**:
   ```bash
   go telemetry local
   # Use Go normally
   go telemetry view
   # Then decide
   ```

2. **Stay informed**: Read privacy docs

3. **Exercise rights**: Disable nếu không comfortable

### For Organizations

1. **Set clear policy**: Document telemetry stance
2. **Respect employee choice**: Trong personal projects
3. **Disable in production**: Nếu có concerns
4. **Review periodically**: Re-evaluate policy

## Contact

### Privacy Questions

- Email: golang-dev@googlegroups.com
- Issue tracker: https://github.com/golang/go/issues
- Privacy policy: https://go.dev/doc/telemetry/privacy

### Report Concerns

```
Nếu có privacy concerns:
1. File issue: github.com/golang/go/issues
2. Email: security@golang.org (for security)
3. Public discussion: golang-dev mailing list
```

## Summary

Go telemetry được thiết kế với privacy-first approach:

✅ Opt-in mặc định  
✅ Anonymous và aggregated  
✅ Transparent và verifiable  
✅ User control và rights  
✅ Minimal data collection  
✅ Secure transmission και storage  
✅ Legal compliance  
✅ Regular audits  

## Xem thêm

- [Cấu hình Telemetry](configuration.md)
- [Dữ liệu được thu thập](data-collected.md)
- [Tổng quan về Telemetry](overview.md)
- Privacy Policy: https://go.dev/doc/telemetry/privacy
