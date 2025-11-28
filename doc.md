# Tài liệu Go

Ngôn ngữ lập trình Go là một dự án mã nguồn mở nhằm giúp các lập trình viên làm việc hiệu quả hơn.

Go mang tính biểu đạt cao, súc tích, sạch sẽ và hiệu quả. Các cơ chế đồng thời của nó giúp dễ dàng viết các chương trình tận dụng tối đa các máy đa lõi và mạng, trong khi hệ thống kiểu mới lạ cho phép xây dựng chương trình linh hoạt và theo module. Go biên dịch nhanh thành mã máy nhưng có sự tiện lợi của garbage collection và sức mạnh của runtime reflection. Đây là một ngôn ngữ biên dịch tĩnh, có kiểu nhanh mà cảm giác như ngôn ngữ động, thông dịch.

---

## Bắt đầu

### [Cài đặt Go](tutorials/install.md)

Hướng dẫn tải xuống và cài đặt Go.

### [Hướng dẫn: Bắt đầu với Go](tutorials/getting-started.md)

Một hướng dẫn giới thiệu ngắn gọn "Hello, World" để thiết lập và sử dụng Go. Tìm hiểu một chút về mã Go, công cụ, package và module.

### [Hướng dẫn: Tạo một module Go](tutorials/create-module.md)

Một hướng dẫn giới thiệu ngắn về các hàm, xử lý lỗi, mảng, map, unit testing và biên dịch.

### [Hướng dẫn: Bắt đầu với multi-module workspaces](tutorials/workspaces.md)

Giới thiệu về cơ bản của việc tạo và sử dụng multi-module workspaces trong Go. Multi-module workspaces hữu ích cho việc thực hiện các thay đổi trên nhiều module.

### [Hướng dẫn: Phát triển RESTful API với Go và Gin](tutorials/web-service-gin.md)

Giới thiệu về viết RESTful web service API với Go và Gin Web Framework.

### [Hướng dẫn: Bắt đầu với generics](tutorials/generics.md)

Với generics, bạn có thể khai báo và sử dụng các hàm hoặc kiểu được viết để làm việc với bất kỳ tập hợp kiểu nào do mã gọi cung cấp.

### [Hướng dẫn: Bắt đầu với fuzzing](tutorials/fuzzing.md)

Fuzzing có thể tạo ra các đầu vào cho các bài test của bạn có thể bắt được các trường hợp biên mà bạn có thể đã bỏ lỡ.

### [Viết Ứng dụng Web](tutorials/web-application.md)

Xây dựng một ứng dụng web đơn giản.

### [Cách viết mã Go](tutorials/how-to-write-go-code.md)

Tài liệu này giải thích cách phát triển một tập hợp đơn giản các Go package bên trong một module, và cho thấy cách sử dụng lệnh go để build và test các package.

### [Hướng dẫn: Truy cập cơ sở dữ liệu quan hệ](tutorials/database-access.md)

Giới thiệu về cơ bản của việc truy cập cơ sở dữ liệu quan hệ với Go và package database/sql trong thư viện chuẩn.

### [Hướng dẫn: Tìm và sửa các phụ thuộc có lỗ hổng bảo mật](tutorials/vulnerability.md)

Hướng dẫn sử dụng govulncheck để tìm và sửa các lỗ hổng bảo mật trong các phụ thuộc của bạn.

---

## Sử dụng và hiểu Go

### [Effective Go](docs/using-go/effective-go.md)

Một tài liệu cung cấp các mẹo để viết mã Go rõ ràng, theo phong cách chuẩn. Đây là tài liệu phải đọc cho bất kỳ lập trình viên Go mới nào. Nó bổ sung cho tour và đặc tả ngôn ngữ, cả hai đều nên đọc trước.

### [Đặc tả ngôn ngữ Go](docs/using-go/spec.md)

Đặc tả chính thức của ngôn ngữ Go.

### [Tài liệu thư viện chuẩn](docs/using-go/stdlib.md)

Tài liệu cho thư viện chuẩn Go.

### [Go Modules Reference](docs/using-go/modules-reference.md)

Tham chiếu chi tiết cho hệ thống quản lý phụ thuộc của Go.

### [go.mod file reference](docs/using-go/gomod-reference.md)

Tham chiếu cho các chỉ thị có trong file go.mod.

### [Tham chiếu lệnh Go](docs/using-go/go-command.md)

Tài liệu cho công cụ Go.

### [Tham chiếu Comment tài liệu](docs/using-go/doc-comment.md)

Viết các comment tài liệu cho godoc.

### [Một Tour về Go](docs/using-go/tour.md)

Một tour tương tác về Go trong 4 phần. Phần đầu tiên bao gồm cú pháp cơ bản và cấu trúc dữ liệu; phần thứ hai thảo luận về các phương thức và interface; phần thứ ba giới thiệu về generics của Go; và phần thứ tư là về các nguyên thủy đồng thời của Go. Mỗi phần kết thúc với một vài bài tập để bạn có thể thực hành những gì bạn đã học. Bạn có thể tham gia tour trực tuyến hoặc cài đặt nó cục bộ với:

```bash
$ go install golang.org/x/website/tour@latest
```

Điều này sẽ đặt binary tour trong thư mục bin của GOPATH của bạn.

### [Mô hình Bộ nhớ Go](docs/using-go/memory-model.md)

Mô hình bộ nhớ Go xác định các điều kiện theo đó các hoạt động đọc một biến trong một goroutine có thể được đảm bảo quan sát các giá trị được tạo ra bởi các hoạt động ghi vào cùng một biến trong một goroutine khác.

### [Câu hỏi thường gặp (FAQ)](docs/using-go/faq.md)

Câu trả lời cho các câu hỏi phổ biến về Go.

---

## Tham chiếu

### [Bản phát hành](docs/references/release-history.md)

Lịch sử phát hành Go.

### [Cam kết Tương thích Go 1](docs/references/go1-compat.md)

Cam kết Go 1 về tính tương thích. Các chương trình Go viết theo đặc tả Go 1 sẽ tiếp tục biên dịch và chạy đúng, không thay đổi, trong suốt vòng đời của phiên bản 1 của Go.

### [Go Wiki](docs/references/wiki.md)

Một wiki được duy trì bởi cộng đồng Go.

### [Tài liệu Tham khảo về Môi trường](docs/references/environment.md)

Các biến môi trường ảnh hưởng đến hành vi của các công cụ Go.

---

## Hỗ trợ biên tập viên và IDE

### [Hỗ trợ plugin và IDE cho biên tập viên](docs/editors/ide-support.md)

Một tài liệu tóm tắt các plugin và IDE phổ biến được sử dụng với Go.

### [Gopls](docs/editors/gopls.md)

Gopls là Go language server cung cấp các tính năng như tự động hoàn thành, định dạng và chẩn đoán cho các IDE và biên tập văn bản.

---

## Bảo mật

### [Tổng quan về Bảo mật](docs/security/overview.md)

Tổng quan về các tài nguyên bảo mật cho các nhà phát triển Go.

### [Chính sách Bảo mật Go](docs/security/security-policy.md)

Mô tả cách nhóm Go xử lý các lỗi bảo mật trong ngôn ngữ Go, thư viện, và các công cụ. Bao gồm hướng dẫn báo cáo các vấn đề bảo mật.

### [Quản lý Lỗ hổng Bảo mật](docs/security/vulnerability-management.md)

Tổng quan về cách Go hỗ trợ phát hiện và giải quyết các lỗ hổng bảo mật trong các phụ thuộc của dự án Go.

### [Cơ sở dữ liệu Lỗ hổng Go](docs/security/vulnerability-database.md)

Cơ sở dữ liệu lỗ hổng Go là nguồn tổng hợp thông tin về các lỗ hổng bảo mật đã biết trong các module Go công khai.

### [govulncheck](docs/security/govulncheck.md)

Govulncheck là công cụ dòng lệnh báo cáo các lỗ hổng bảo mật đã biết ảnh hưởng đến các package Go trong dự án của bạn. Nó phân tích cơ sở mã để chỉ để lộ các lỗ hổng thực sự ảnh hưởng đến mã của bạn.

### [Các Thực hành Tốt nhất về Bảo mật](docs/security/best-practices.md)

Hướng dẫn về các thực hành tốt nhất để viết mã Go an toàn, bao gồm xác thực đầu vào, xử lý lỗi, và sử dụng các package mật mã.

---

## Công cụ chẩn đoán

### [Tổng quan về công cụ chẩn đoán](docs/diagnostics/overview.md)

Một tổng quan về các công cụ và phương pháp chẩn đoán các vấn đề trong chương trình Go.

### [Hướng dẫn Garbage Collector](docs/diagnostics/gc-guide.md)

Một hướng dẫn về cách Go quản lý bộ nhớ và cách tận dụng tối đa nó.

### [Hướng dẫn thu thập Profile](docs/diagnostics/pgo-guide.md)

Một hướng dẫn về cách sử dụng profile hướng dẫn tối ưu hóa (PGO) trong Go.

---

## Các bài viết

### [Wiki Go](docs/articles/wiki.md)

Wiki Go, được duy trì bởi cộng đồng Go, bao gồm các bài viết về ngôn ngữ Go, công cụ và các chủ đề khác.

### [Tài liệu không phải tiếng Anh](docs/articles/non-english.md)

Xem trang Tài liệu không phải tiếng Anh tại Wiki Go để biết các bản địa hóa của tài liệu Go.

---

## Truy cập Cơ sở Dữ liệu

### [Tổng quan về Truy cập Cơ sở Dữ liệu](docs/database/overview.md)

Tổng quan về cách truy cập cơ sở dữ liệu từ Go.

### Mở một xử lý cơ sở dữ liệu

Cách kết nối với cơ sở dữ liệu sử dụng package database/sql trong Go.

### Thực thi các câu lệnh SQL không trả về dữ liệu

Cách thực thi các câu lệnh INSERT, UPDATE, DELETE, và các câu lệnh SQL khác không trả về dữ liệu.

### Truy vấn dữ liệu

Cách truy vấn dữ liệu từ cơ sở dữ liệu trong Go.

### Sử dụng prepared statements

Cách sử dụng prepared statements để thực thi các câu lệnh SQL hiệu quả và an toàn hơn.

### Thực thi giao dịch

Cách thực thi các giao dịch cơ sở dữ liệu để đảm bảo tính toàn vẹn dữ liệu.

### Xử lý lỗi

Các thực hành tốt nhất cho việc xử lý lỗi khi làm việc với cơ sở dữ liệu trong Go.

### Tránh các vấn đề SQL injection

Cách viết các truy vấn SQL an toàn để tránh các lỗ hổng SQL injection.

### Quản lý kết nối

Cách quản lý và tối ưu hóa connection pool trong ứng dụng Go.

---

## Cgo

### [Cgo](docs/cgo/cgo.md)

Liên kết mã Go với các package C bên ngoài.

### [Gọi Go từ C](docs/cgo/calling-go-from-c.md)

Cách gọi hàm Go từ mã C.

---

## Các chủ đề nâng cao

### Bảo hiểm Code

Mô tả cách sử dụng công cụ bảo hiểm code được tích hợp với go test.

### Tham chiếu Format Testdata

Tham chiếu cho syntax của file "txtar" testdata.

### Cấu hình Fuzz

Tham chiếu cho các cấu hình có thể đọc và giải quyết được bằng công cụ go.

### Quản lý phụ thuộc

Khi mã của bạn sử dụng các package bên ngoài, các package đó (được phân phối dưới dạng module) trở thành các phụ thuộc.

### Đánh giá hiệu suất

Mô tả cách viết và đọc benchmark tests trong Go.

### Fuzz Testing

Mô tả cách sử dụng fuzzer bản địa của Go.

### Assembly

Mô tả cách sử dụng hợp ngữ Go để thực hiện các tác vụ cấp thấp.

### Race Detector

Mô tả cách sử dụng công cụ phát hiện data race.

### Làm việc với các đường dẫn file

Mô tả các vấn đề mà các đường dẫn file có thể gây ra và cách làm việc với chúng một cách an toàn trong Go.

### Chế độ FIPS 140

Mô tả cách kích hoạt chế độ tuân thủ FIPS 140 trong Go.

---

## Đo lường từ xa Go (Telemetry)

### Tổng quan về Đo lường từ xa

Đo lường từ xa là thu thập dữ liệu sử dụng và chẩn đoán ẩn danh từ chuỗi công cụ Go để cải thiện trải nghiệm nhà phát triển.

### Cấu hình Đo lường từ xa

Cách bật, tắt hoặc cấu hình đo lường từ xa trong cài đặt Go của bạn sử dụng lệnh `go telemetry`.

### Dữ liệu được thu thập

Các loại dữ liệu được thu thập bởi hệ thống đo lường từ xa Go và cách chúng được sử dụng để cải thiện Go.

### Quyền riêng tư và Đo lường từ xa

Thông tin về quyền riêng tư liên quan đến việc thu thập và xử lý dữ liệu đo lường từ xa.

---

## Xây dựng và phát hành

### Module Mirror và Checksum Database

Cách sử dụng module mirror và checksum database.

### Private Module

Cách cấu hình các module riêng tư với go.

### Xây dựng lại có thể tái tạo

Cách xác minh rằng một bản build Go được tái tạo chính xác.

---

## Phát triển Go

### Quá trình phát triển

Tham gia vào phát triển Go.

### Đóng góp mã

Quy trình để gửi các thay đổi cho dự án Go.

### Đóng góp vào Wiki

Quy trình để đóng góp vào Wiki Go.

### Thiết lập và sử dụng Website Go cục bộ

Cách thiết lập website Go cục bộ để phát triển.

### Source code

Xem mã nguồn Go.

---

## Cộng đồng

Cộng đồng Go rất sôi nổi và thân thiện. Để tham gia:

- **Go Forum**: Diễn đàn thảo luận Go.
- **Gophers Slack**: Kênh chat cho các lập trình viên Go.
- **Go Meetups**: Các nhóm gặp mặt địa phương.
- **GopherCon**: Hội nghị chính thức về Go.

---

## Liên kết hữu ích

- [Go.dev](https://go.dev) - Trang chủ chính thức
- [Go Playground](https://go.dev/play/) - Chạy mã Go trực tuyến
- [Go Blog](https://go.dev/blog/) - Blog chính thức của Go
- [Go Packages](https://pkg.go.dev) - Tìm kiếm các package Go

---

*Tài liệu này được dịch từ [go.dev/doc](https://go.dev/doc/) để phục vụ cộng đồng lập trình viên Việt Nam.*
