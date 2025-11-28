# Hướng dẫn: Bắt đầu với Go

Trong hướng dẫn này, bạn sẽ có một giới thiệu ngắn gọn về lập trình Go. Trong quá trình này, bạn sẽ:

- Cài đặt Go (nếu bạn chưa cài đặt).
- Viết một số mã "Hello, World!" đơn giản.
- Sử dụng lệnh `go` để chạy mã của bạn.
- Sử dụng công cụ khám phá package Go để tìm các package bạn có thể sử dụng trong mã của riêng bạn.
- Gọi các hàm của một module bên ngoài.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Yêu cầu tiên quyết

- **Một chút kinh nghiệm lập trình.** Mã ở đây khá đơn giản, nhưng sẽ hữu ích nếu biết một số điều về hàm.
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt. Hầu hết các trình soạn thảo văn bản đều có hỗ trợ tốt cho Go. Phổ biến nhất là VSCode (miễn phí), GoLand (trả phí), và Vim (miễn phí).
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.

## Cài đặt Go

Chỉ cần sử dụng [Hướng dẫn Tải xuống và cài đặt](https://go.dev/doc/install).

## Viết một số mã

Bắt đầu với Hello, World.

1. Mở một dấu nhắc lệnh và cd đến thư mục home của bạn.

   Trên Linux hoặc Mac:
   ```bash
   cd
   ```

   Trên Windows:
   ```bash
   cd %HOMEPATH%
   ```

2. Tạo một thư mục hello cho mã nguồn Go đầu tiên của bạn.

   Ví dụ, sử dụng các lệnh sau:
   ```bash
   mkdir hello
   cd hello
   ```

3. Kích hoạt theo dõi phụ thuộc cho mã của bạn.

   Khi mã của bạn import các package chứa trong các module khác, bạn quản lý các phụ thuộc đó thông qua module của riêng mã của bạn. Module đó được định nghĩa bởi một file go.mod theo dõi các module cung cấp các package đó. File go.mod đó nằm cùng với mã của bạn, bao gồm cả trong repository mã nguồn của bạn.

   Để kích hoạt theo dõi phụ thuộc cho mã của bạn bằng cách tạo file go.mod, chạy lệnh [`go mod init`](https://go.dev/ref/mod#go-mod-init), đặt cho nó tên của module mà mã của bạn sẽ nằm trong. Tên là đường dẫn module của module.

   Trong phát triển thực tế, đường dẫn module thường sẽ là vị trí repository nơi mã nguồn của bạn sẽ được lưu giữ. Ví dụ, đường dẫn module có thể là `github.com/mymodule`. Nếu bạn định xuất bản module của mình cho người khác sử dụng, đường dẫn module *phải* là một vị trí mà từ đó các công cụ Go có thể tải xuống module của bạn. Để biết thêm về đặt tên một module với đường dẫn module, xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies#naming_module).

   Cho mục đích của hướng dẫn này, chỉ cần sử dụng `example/hello`.
   ```bash
   $ go mod init example/hello
   go: creating new go.mod: module example/hello
   ```

4. Trong trình soạn thảo văn bản của bạn, tạo một file hello.go trong đó để viết mã của bạn.

5. Dán mã sau vào file hello.go của bạn và lưu file.
   ```go
   package main

   import "fmt"

   func main() {
       fmt.Println("Hello, World!")
   }
   ```

   Đây là mã Go của bạn. Trong mã này, bạn:

   - Khai báo một `main` package (một package là cách để nhóm các hàm, và nó bao gồm tất cả các file trong cùng một thư mục).
   - Import package [`fmt`](https://pkg.go.dev/fmt/) phổ biến, chứa các hàm để định dạng văn bản, bao gồm in ra console. Package này là một trong các package [thư viện chuẩn](https://pkg.go.dev/std) bạn nhận được khi cài đặt Go.
   - Triển khai một hàm `main` để in một thông báo ra console. Một hàm `main` thực thi theo mặc định khi bạn chạy `main` package.

6. Chạy mã của bạn để xem lời chào.
   ```bash
   $ go run .
   Hello, World!
   ```

   Lệnh [`go run`](https://go.dev/cmd/go/#hdr-Compile_and_run_Go_program) là một trong nhiều lệnh `go` bạn sẽ sử dụng để hoàn thành công việc với Go. Sử dụng lệnh sau để lấy danh sách các lệnh khác:
   ```bash
   $ go help
   ```

## Gọi mã trong một package bên ngoài

Khi bạn cần mã của mình làm điều gì đó có thể đã được triển khai bởi người khác, bạn có thể tìm kiếm các package có các hàm bạn có thể sử dụng trong mã của mình.

1. Làm cho thông báo in ra của bạn thú vị hơn một chút với một hàm từ một module bên ngoài.

   1. Truy cập pkg.go.dev và [tìm kiếm package "quote"](https://pkg.go.dev/search?q=quote).
   2. Tìm và click vào package `rsc.io/quote` trong kết quả tìm kiếm (nếu bạn thấy `rsc.io/quote/v4`, bỏ qua nó vào lúc này).
   3. Trong phần **Documentation**, dưới **Index**, lưu ý danh sách các hàm bạn có thể gọi từ mã của mình. Bạn sẽ sử dụng hàm `Go`.
   4. Ở đầu trang này, lưu ý rằng package `quote` được bao gồm trong module `rsc.io/quote`.

   Bạn có thể sử dụng trang pkg.go.dev để tìm các module đã xuất bản có các package bạn có thể sử dụng trong mã của riêng bạn. Các package được xuất bản trong các module -- như `rsc.io/quote` -- nơi người khác có thể sử dụng chúng. Các module được cải thiện với các phiên bản mới theo thời gian, và bạn có thể nâng cấp mã của mình để sử dụng các phiên bản cải tiến.

2. Trong mã Go của bạn, import package `rsc.io/quote` và thêm một lời gọi đến hàm `Go` của nó.

   Sau khi thêm các dòng được đánh dấu, mã của bạn sẽ bao gồm những điều sau:
   ```go
   package main

   import "fmt"

   import "rsc.io/quote"

   func main() {
       fmt.Println(quote.Go())
   }
   ```

3. Thêm các yêu cầu module mới và sums.

   Go sẽ thêm module `quote` như một yêu cầu, cũng như một file go.sum để sử dụng trong xác thực module. Để biết thêm, xem [Xác thực các module trong Go Modules Reference](https://go.dev/ref/mod#authenticating).
   ```bash
   $ go mod tidy
   go: finding module for package rsc.io/quote
   go: found rsc.io/quote in rsc.io/quote v1.5.2
   ```

4. Chạy mã của bạn để xem thông báo được tạo bởi hàm bạn đang gọi.
   ```bash
   $ go run .
   Don't communicate by sharing memory, share memory by communicating.
   ```

   Lưu ý rằng mã của bạn gọi hàm `Go`, in ra một thông báo thông minh về giao tiếp.

   Khi bạn chạy `go mod tidy`, nó đã định vị và tải xuống module `rsc.io/quote` chứa package bạn đã import. Theo mặc định, nó đã tải xuống phiên bản mới nhất -- v1.5.2.

## Viết thêm mã

Với phần giới thiệu nhanh này, bạn đã cài đặt Go và học được một số điều cơ bản. Để viết thêm mã với một hướng dẫn khác, hãy xem [Tạo một module Go](create-module.md).

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/getting-started](https://go.dev/doc/tutorial/getting-started) để phục vụ cộng đồng lập trình viên Việt Nam.*
