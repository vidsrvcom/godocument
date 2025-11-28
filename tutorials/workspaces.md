# Hướng dẫn: Bắt đầu với multi-module workspaces

Bảng nội dung:
- [Yêu cầu tiên quyết](#yêu-cầu-tiên-quyết)
- [Tạo một module cho mã của bạn](#tạo-một-module-cho-mã-của-bạn)
- [Tạo workspace](#tạo-workspace)
- [Tải xuống và sửa đổi module golang.org/x/example](#tải-xuống-và-sửa-đổi-module-golangorgxexample)
- [Tìm hiểu thêm về workspaces](#tìm-hiểu-thêm-về-workspaces)

Hướng dẫn này giới thiệu về cơ bản của multi-module workspaces trong Go. Với multi-module workspaces, bạn có thể nói với lệnh Go rằng bạn đang viết mã trong nhiều module cùng một lúc và dễ dàng xây dựng và chạy mã trong các module đó.

Trong hướng dẫn này, bạn sẽ tạo hai module trong một workspace được chia sẻ, thực hiện các thay đổi trên các module đó, và xem kết quả của những thay đổi đó trong một bản build.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Yêu cầu tiên quyết

- **Một bản cài đặt Go 1.18 trở lên.**
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.

## Tạo một module cho mã của bạn

Để bắt đầu, tạo một module cho mã bạn sẽ viết.

1. Mở một dấu nhắc lệnh và thay đổi đến thư mục home của bạn.

   Trên Linux hoặc Mac:
   ```bash
   cd
   ```

   Trên Windows:
   ```bash
   cd %HOMEPATH%
   ```

   Phần còn lại của hướng dẫn sẽ hiển thị dấu $ như một dấu nhắc. Các lệnh bạn sử dụng cũng sẽ hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn gọi là workspace.
   ```bash
   $ mkdir workspace
   $ cd workspace
   ```

3. Khởi tạo module

   Ví dụ của chúng tôi sẽ tạo một module mới `hello` phụ thuộc vào module golang.org/x/example/hello/reverse.

   Tạo module hello:
   ```bash
   $ mkdir hello
   $ cd hello
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

   Thêm một phụ thuộc vào module golang.org/x/example/hello/reverse bằng cách sử dụng `go get`.
   ```bash
   $ go get golang.org/x/example/hello/reverse
   ```

4. Tạo hello.go trong thư mục hello với nội dung sau:
   ```go
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"))
   }
   ```

   Bây giờ, chạy chương trình hello:
   ```bash
   $ go run .
   olleH
   ```

## Tạo workspace

Trong bước này, chúng ta sẽ tạo một file `go.work` để xác định một workspace với module.

### Khởi tạo workspace

Trong thư mục `workspace`, chạy:
```bash
$ go work init ./hello
```

Lệnh `go work init` nói với `go` để tạo một file `go.work` cho một workspace chứa các module trong thư mục `./hello`.

Lệnh `go` tạo ra một file `go.work` trông như thế này:
```
go 1.18

use ./hello
```

File `go.work` có cú pháp tương tự như `go.mod`.

Chỉ thị `go` nói với Go phiên bản Go nào file nên được thông dịch với. Nó tương tự như chỉ thị `go` trong file `go.mod`.

Chỉ thị `use` nói với Go rằng module trong thư mục `hello` nên là các module chính khi thực hiện một bản build.

Vì vậy trong bất kỳ thư mục con nào của `workspace` module sẽ hoạt động.

### Chạy chương trình trong thư mục workspace

Trong thư mục `workspace`, chạy:
```bash
$ go run ./hello
olleH
```

Lệnh Go bao gồm tất cả các module trong workspace như các module chính. Điều này cho phép chúng tôi tham chiếu đến một package trong module, ngay cả bên ngoài module. Chạy lệnh `go run` bên ngoài module hoặc workspace sẽ dẫn đến một lỗi vì lệnh `go` sẽ không biết module nào để sử dụng.

Tiếp theo, chúng tôi sẽ thêm một bản sao cục bộ của module `golang.org/x/example/hello/reverse` vào workspace. Sau đó chúng tôi sẽ thêm một hàm mới vào package `reverse` mà chúng tôi có thể sử dụng thay vì `String`.

## Tải xuống và sửa đổi module golang.org/x/example

Trong bước này, chúng tôi sẽ tải xuống một bản sao của repository Git chứa module `golang.org/x/example`, thêm nó vào workspace, và sau đó thêm một hàm mới vào nó mà chúng tôi sẽ sử dụng từ chương trình hello.

1. Clone repository

   Từ thư mục workspace, chạy lệnh `git` để clone repository:
   ```bash
   $ git clone https://go.googlesource.com/example
   Cloning into 'example'...
   remote: Total 165 (delta 27), reused 165 (delta 27)
   Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
   Resolving deltas: 100% (27/27), done.
   ```

2. Thêm module vào workspace

   Module golang.org/x/example được clone vào thư mục ./example. Mã nguồn của module golang.org/x/example/hello/reverse nằm trong ./example/hello/reverse.

   Thêm module vào workspace:
   ```bash
   $ go work use ./example/hello
   ```

   Lệnh `go work use` thêm một mục dòng mới vào file `go.work` tham chiếu đến module trong `./example/hello`:
   ```
   go 1.18

   use (
       ./hello
       ./example/hello
   )
   ```

   Workspace bây giờ bao gồm cả module `example.com/hello` và module `golang.org/x/example/hello`.

   Điều này sẽ cho phép chúng tôi sử dụng mã mới mà chúng tôi sẽ viết trong bản sao của module `golang.org/x/example/hello/reverse` thay vì phiên bản của module mà chúng tôi đã tải xuống với lệnh `go get`.

3. Thêm hàm mới.

   Chúng tôi sẽ thêm một hàm mới để đảo ngược một số vào package `golang.org/x/example/hello/reverse`.

   Tạo một file mới có tên `int.go` trong thư mục `workspace/example/hello/reverse` chứa nội dung sau:
   ```go
   package reverse

   import "strconv"

   // Int trả về đảo ngược số nguyên của i dưới dạng một int.
   func Int(i int) int {
       i, _ = strconv.Atoi(String(strconv.Itoa(i)))
       return i
   }
   ```

4. Sửa đổi chương trình hello để sử dụng hàm.

   Sửa đổi nội dung của `workspace/hello/hello.go` để chứa nội dung sau:
   ```go
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"), reverse.Int(24601))
   }
   ```

### Chạy mã trong workspace

Từ thư mục workspace, chạy
```bash
$ go run ./hello
olleH 10642
```

Lệnh Go tìm module `example.com/hello` được xác định trong dòng lệnh trong thư mục `hello` được xác định bởi file `go.work`, và tương tự phân giải import `golang.org/x/example/hello/reverse` bằng cách sử dụng file `go.work`.

`go.work` có thể được sử dụng thay cho việc thêm các chỉ thị [`replace`](https://go.dev/ref/mod#go-mod-file-replace) để làm việc trên nhiều module.

Vì hai module đều trong cùng một workspace nên dễ dàng thực hiện thay đổi trong một module và sử dụng nó trong một module khác.

## Tìm hiểu thêm về workspaces

Lệnh `go` có một vài lệnh con để làm việc với workspaces ngoài `go work init` mà chúng tôi đã thấy trước đó trong hướng dẫn:

- `go work use [-r] [dir]` thêm một chỉ thị `use` vào file `go.work` cho `dir`, nếu nó tồn tại, và xóa chỉ thị `use` nếu đối số thư mục không tồn tại. Cờ `-r` kiểm tra các thư mục con của `dir` một cách đệ quy.
- `go work edit` chỉnh sửa file `go.work` tương tự như `go mod edit`
- `go work sync` đồng bộ các phụ thuộc từ danh sách build workspace vào từng file `go.mod` của các module workspace.

Xem [Workspaces](https://go.dev/ref/mod#workspaces) trong Go Modules Reference để biết thêm chi tiết về workspaces và file `go.work`.

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/workspaces](https://go.dev/doc/tutorial/workspaces) để phục vụ cộng đồng lập trình viên Việt Nam.*
