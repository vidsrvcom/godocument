# Cách viết mã Go

Tài liệu này giải thích cách phát triển một tập hợp đơn giản các Go package bên trong một module, và cho thấy cách sử dụng lệnh `go` để build và test các package.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Tổ chức mã

Các chương trình Go được tổ chức thành các package. Một package là một tập hợp các file nguồn trong cùng một thư mục được biên dịch cùng nhau. Các hàm, kiểu, biến, và hằng số được định nghĩa trong một file nguồn được hiển thị cho tất cả các file khác trong cùng package.

Một repository chứa một hoặc nhiều module. Một module là một tập hợp các Go package liên quan được phát hành cùng nhau. Một Go repository thường chỉ chứa một module, nằm ở gốc của repository. Một file có tên `go.mod` ở đó khai báo đường dẫn module: tiền tố đường dẫn import cho tất cả các package trong module. Module chứa các package trong thư mục chứa file `go.mod` của nó cũng như các thư mục con của thư mục đó.

Lưu ý rằng bạn không cần xuất bản mã của mình lên một repository từ xa trước khi bạn có thể build nó. Một module có thể được định nghĩa cục bộ mà không thuộc về một repository. Tuy nhiên, đó là thói quen tốt để tổ chức mã của bạn như thể bạn sẽ xuất bản nó một ngày nào đó.

Đường dẫn của mỗi module không chỉ đóng vai trò là tiền tố đường dẫn import cho các package của nó, mà còn cho biết lệnh `go` nên tìm ở đâu để tải nó xuống. Ví dụ, để tải module `golang.org/x/tools`, lệnh `go` sẽ tham khảo repository được chỉ định bởi `https://golang.org/x/tools` (được mô tả thêm [ở đây](https://go.dev/cmd/go/#hdr-Relative_import_paths)).

Một đường dẫn import là một chuỗi được sử dụng để import một package. Đường dẫn import của một package là đường dẫn module của nó được nối với thư mục con của nó trong module. Ví dụ, module `github.com/google/go-cmp` chứa một package trong thư mục `cmp/`. Đường dẫn import của package đó là `github.com/google/go-cmp/cmp`. Các package trong thư viện chuẩn không có tiền tố đường dẫn module.

## Chương trình đầu tiên của bạn

Để biên dịch và chạy một chương trình đơn giản, trước tiên chọn một đường dẫn module (chúng tôi sẽ sử dụng `example/user/hello`) và tạo một file `go.mod` khai báo nó:

```bash
$ mkdir hello # Hoặc clone nó nếu nó đã tồn tại trong kiểm soát phiên bản.
$ cd hello
$ go mod init example/user/hello
go: creating new go.mod: module example/user/hello
$ cat go.mod
module example/user/hello

go 1.16
$
```

Câu lệnh đầu tiên trong một file nguồn Go phải là `package name`. Các lệnh thực thi phải luôn sử dụng `package main`.

Tiếp theo, tạo một file có tên `hello.go` bên trong thư mục đó chứa mã Go sau:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, world.")
}
```

Bây giờ bạn có thể build và cài đặt chương trình đó bằng công cụ `go`:

```bash
$ go install example/user/hello
$
```

Lệnh này build lệnh `hello`, tạo ra một binary thực thi. Sau đó nó cài đặt binary đó như `$HOME/go/bin/hello` (hoặc, trên Windows, `%USERPROFILE%\go\bin\hello.exe`).

Thư mục cài đặt được kiểm soát bởi các biến môi trường `GOPATH` và `GOBIN`. Nếu `GOBIN` được đặt, các binary được cài đặt vào thư mục đó. Nếu `GOPATH` được đặt, các binary được cài đặt vào thư mục con `bin` của thư mục đầu tiên trong danh sách `GOPATH`. Nếu không, các binary được cài đặt vào thư mục con `bin` của `GOPATH` mặc định (`$HOME/go` hoặc `%USERPROFILE%\go`).

Bạn có thể sử dụng lệnh `go env` để đặt giá trị mặc định cho các biến môi trường cho các lệnh `go` tương lai một cách di động:

```bash
$ go env -w GOBIN=/somewhere/else/bin
$
```

Để hủy đặt một biến đã được đặt trước đó bởi `go env -w`, sử dụng `go env -u`:

```bash
$ go env -u GOBIN
$
```

Các lệnh như `go install` áp dụng trong ngữ cảnh của module chứa thư mục làm việc hiện tại. Nếu thư mục làm việc không nằm trong module `example/user/hello`, `go install` có thể thất bại.

Để thuận tiện, các lệnh `go` chấp nhận các đường dẫn tương đối với thư mục làm việc, và mặc định là package trong thư mục làm việc hiện tại nếu không có đường dẫn nào khác được cung cấp. Vì vậy trong thư mục làm việc của chúng tôi, các lệnh sau đều tương đương:

```bash
$ go install example/user/hello
$ go install .
$ go install
```

Tiếp theo, hãy chạy chương trình để đảm bảo nó hoạt động. Để thuận tiện hơn, chúng tôi sẽ thêm thư mục cài đặt vào `PATH` của chúng tôi để dễ dàng chạy các binary:

```bash
# Windows users should consult https://github.com/golang/go/wiki/SettingGOPATH
# for setting %PATH%.
$ export PATH=$PATH:$(dirname $(go list -f '{{.Target}}' .))
$ hello
Hello, world.
$
```

Nếu bạn đang sử dụng một hệ thống kiểm soát nguồn, bây giờ sẽ là thời điểm tốt để khởi tạo một repository, thêm các file, và commit thay đổi đầu tiên của bạn. Một lần nữa, bước này là tùy chọn: bạn không cần sử dụng kiểm soát nguồn để viết mã Go.

```bash
$ git init
Initialized empty Git repository in /home/user/hello/.git/
$ git add go.mod hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 7 insertion(+)
 create mode 100644 go.mod hello.go
$
```

## Importing các package từ module của bạn

Hãy viết một package `morestrings` và sử dụng nó từ chương trình `hello`. Đầu tiên, tạo một thư mục cho package có tên `$HOME/hello/morestrings`, và sau đó một file có tên `reverse.go` trong thư mục đó với nội dung sau:

```go
// Package morestrings triển khai các hàm bổ sung để thao tác UTF-8
// encoded strings, ngoài những gì được cung cấp trong package "strings"
// của thư viện chuẩn.
package morestrings

// ReverseRunes trả về chuỗi đối số của nó đã đảo ngược theo rune từ trái sang phải.
func ReverseRunes(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

Bởi vì hàm `ReverseRunes` của chúng tôi bắt đầu bằng một chữ hoa, nó được [export](https://go.dev/ref/spec#Exported_identifiers), và có thể được sử dụng trong các package khác import package `morestrings` của chúng tôi.

Hãy test rằng package biên dịch bằng `go build`:

```bash
$ cd $HOME/hello/morestrings
$ go build
$
```

Điều này sẽ không tạo ra một file đầu ra. Thay vào đó nó lưu package đã biên dịch trong cache build cục bộ.

Sau khi xác nhận rằng package `morestrings` build, hãy sử dụng nó từ chương trình `hello`. Để làm như vậy, sửa đổi `$HOME/hello/hello.go` ban đầu của bạn để sử dụng package morestrings:

```go
package main

import (
    "fmt"

    "example/user/hello/morestrings"
)

func main() {
    fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```

Cài đặt chương trình `hello`:

```bash
$ go install example/user/hello
```

Chạy phiên bản mới của chương trình, bạn sẽ thấy một thông báo mới, đã đảo ngược:

```bash
$ hello
Hello, Go!
```

## Importing các package từ các module từ xa

Một đường dẫn import có thể mô tả cách lấy mã nguồn package sử dụng một hệ thống kiểm soát sửa đổi như Git hoặc Mercurial. Công cụ `go` sử dụng thuộc tính này để tự động tải các package từ các repository từ xa. Ví dụ, để sử dụng `github.com/google/go-cmp/cmp` trong chương trình của bạn:

```go
package main

import (
    "fmt"

    "example/user/hello/morestrings"
    "github.com/google/go-cmp/cmp"
)

func main() {
    fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
    fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```

Bây giờ bạn có một phụ thuộc vào một module bên ngoài, bạn cần tải module đó và ghi lại phiên bản của nó trong file `go.mod` của bạn. Lệnh `go mod tidy` thêm các yêu cầu module còn thiếu cho các package đã import và loại bỏ các yêu cầu về các module không còn được sử dụng nữa.

```bash
$ go mod tidy
go: finding module for package github.com/google/go-cmp/cmp
go: found github.com/google/go-cmp/cmp in github.com/google/go-cmp v0.5.4
$ go install example/user/hello
$ hello
Hello, Go!
  string(
-     "Hello World",
+     "Hello Go",
  )
$ cat go.mod
module example/user/hello

go 1.16

require github.com/google/go-cmp v0.5.4
$
```

## Testing

Go có một framework test nhẹ bao gồm lệnh `go test` và package `testing`.

Bạn viết một test bằng cách tạo một file có tên kết thúc bằng `_test.go` chứa các hàm có tên `TestXXX` với chữ ký `func (t *testing.T)`. Framework test chạy mỗi hàm như vậy; nếu hàm gọi một hàm lỗi như `t.Error` hoặc `t.Fail`, test được coi là đã thất bại.

Thêm một test vào package `morestrings` bằng cách tạo file `$HOME/hello/morestrings/reverse_test.go` chứa mã Go sau.

```go
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
    cases := []struct {
        in, want string
    }{
        {"Hello, world", "dlrow ,olleH"},
        {"Hello, 世界", "界世 ,olleH"},
        {"", ""},
    }
    for _, c := range cases {
        got := ReverseRunes(c.in)
        if got != c.want {
            t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
        }
    }
}
```

Sau đó chạy test bằng `go test`:

```bash
$ cd $HOME/hello/morestrings
$ go test
PASS
ok      example/user/hello/morestrings 0.165s
$
```

Chạy `go help test` và xem [tài liệu package testing](https://go.dev/pkg/testing/) để biết thêm chi tiết.

## Tiếp theo là gì

Đăng ký vào danh sách mail [golang-announce](https://groups.google.com/group/golang-announce) để được thông báo khi một phiên bản ổn định mới của Go được phát hành.

Xem [Effective Go](https://go.dev/doc/effective_go.html) để biết các mẹo viết mã Go rõ ràng, theo phong cách chuẩn.

Tham gia [Một Tour về Go](https://go.dev/tour/) để học ngôn ngữ một cách đúng đắn.

Truy cập [trang tài liệu](../doc.md) để xem một tập hợp các bài viết chuyên sâu về ngôn ngữ Go và các thư viện và công cụ của nó.

## Nhận trợ giúp

Để được trợ giúp thời gian thực, hãy hỏi các gopher hữu ích trong cộng đồng chạy [máy chủ Discord Gophers](https://discord.gg/golang) hoặc [kênh #go-nuts IRC](https://go.dev/wiki/Irc).

Kênh thảo luận chính thức cho ngôn ngữ Go là [Go forum](https://forum.golangbridge.org/).

Báo cáo lỗi bằng cách sử dụng [Go issue tracker](https://go.dev/issue).

---

*Tài liệu này được dịch từ [go.dev/doc/code](https://go.dev/doc/code) để phục vụ cộng đồng lập trình viên Việt Nam.*
