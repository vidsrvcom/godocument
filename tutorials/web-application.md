# Hướng dẫn: Viết Ứng dụng Web

Hướng dẫn này sẽ hướng dẫn bạn cách viết một ứng dụng web đơn giản với Go.

Nội dung bao gồm:

- Tạo một cấu trúc dữ liệu với các phương thức load và save
- Sử dụng package `net/http` để xây dựng các ứng dụng web
- Sử dụng package `html/template` để xử lý các template HTML
- Sử dụng package `regexp` để xác thực đầu vào người dùng
- Sử dụng closures

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Kiến thức giả định

- Kinh nghiệm lập trình
- Hiểu biết về các công nghệ web cơ bản (HTTP, HTML)
- Một số kiến thức về dòng lệnh UNIX/DOS

## Bắt đầu

Hiện tại, bạn cần có một máy FreeBSD, Linux, macOS, hoặc Windows để chạy Go. Chúng tôi sẽ sử dụng `$` để đại diện cho dấu nhắc lệnh.

Cài đặt Go (xem [Hướng dẫn Cài đặt](install.md)).

Tạo một thư mục mới cho hướng dẫn này bên trong `GOPATH` của bạn và cd vào nó:

```bash
$ mkdir gowiki
$ cd gowiki
```

Tạo một file có tên `wiki.go`, mở nó trong trình soạn thảo yêu thích của bạn, và thêm các dòng sau:

```go
package main

import (
    "fmt"
    "os"
)
```

Chúng tôi import các package `fmt` và `os` từ thư viện chuẩn Go. Sau đó, khi chúng tôi triển khai thêm chức năng, chúng tôi sẽ thêm nhiều package hơn vào khai báo `import` này.

## Cấu trúc Dữ liệu

Hãy bắt đầu bằng việc định nghĩa các cấu trúc dữ liệu. Một wiki bao gồm một loạt các trang được kết nối với nhau, mỗi trang có một tiêu đề và một nội dung (nội dung trang). Ở đây, chúng tôi định nghĩa `Page` là một struct với hai trường đại diện cho tiêu đề và nội dung.

```go
type Page struct {
    Title string
    Body  []byte
}
```

Kiểu `[]byte` có nghĩa là "một byte slice". (Xem [Slices: cách sử dụng và bên trong](https://go.dev/blog/slices-intro) để biết thêm về slices.) Phần tử `Body` là một `[]byte` thay vì `string` vì đó là kiểu mà các thư viện `io` mà chúng tôi sẽ sử dụng mong đợi, như bạn sẽ thấy bên dưới.

Struct `Page` mô tả cách dữ liệu trang sẽ được lưu trữ trong bộ nhớ. Nhưng còn lưu trữ liên tục thì sao? Chúng ta có thể giải quyết điều đó bằng cách tạo một phương thức `save` trên `Page`:

```go
func (p *Page) save() error {
    filename := p.Title + ".txt"
    return os.WriteFile(filename, p.Body, 0600)
}
```

Chữ ký của phương thức này có nội dung: "Đây là một phương thức có tên `save` lấy receiver `p`, một con trỏ đến `Page`. Nó không nhận tham số, và trả về một giá trị kiểu `error`."

Phương thức này sẽ lưu `Body` của `Page` vào một file văn bản. Để đơn giản, chúng tôi sẽ sử dụng `Title` làm tên file.

Phương thức `save` trả về một giá trị `error` vì đó là kiểu trả về của `WriteFile` (một hàm thư viện chuẩn ghi một byte slice vào một file). Phương thức `save` trả về giá trị lỗi, để cho phép ứng dụng xử lý nó nếu có bất kỳ điều gì sai khi ghi file. Nếu tất cả diễn ra tốt đẹp, `Page.save()` sẽ trả về `nil` (giá trị zero cho con trỏ, interface, và một số kiểu khác).

Số nguyên bát phân `0600`, được truyền làm tham số thứ ba cho `WriteFile`, chỉ ra rằng file sẽ được tạo với quyền đọc-ghi chỉ cho người dùng hiện tại. (Xem trang man Unix `open(2)` để biết chi tiết.)

Ngoài việc lưu các trang, chúng tôi cũng sẽ muốn tải các trang:

```go
func loadPage(title string) (*Page, error) {
    filename := title + ".txt"
    body, err := os.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    return &Page{Title: title, Body: body}, nil
}
```

Hàm `loadPage` xây dựng tên file từ tham số tiêu đề, đọc nội dung file vào một biến mới `body`, và trả về một con trỏ đến một `Page` literal được xây dựng với các giá trị tiêu đề và nội dung thích hợp.

## Package net/http

Đây là một ví dụ hoạt động đầy đủ của một máy chủ web đơn giản:

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Hàm `main` bắt đầu với một lời gọi đến `http.HandleFunc`, nói với package `http` xử lý tất cả các yêu cầu đến gốc web ("/") bằng `handler`.

Sau đó nó gọi `http.ListenAndServe`, xác định rằng nó sẽ lắng nghe trên cổng 8080 trên bất kỳ interface nào (":8080"). (Đừng lo lắng về tham số thứ hai, `nil`, vào lúc này.) Hàm này sẽ block cho đến khi chương trình bị kết thúc.

`ListenAndServe` luôn trả về một lỗi, vì nó chỉ trả về khi một lỗi không mong đợi xảy ra. Để log lỗi đó, chúng tôi bọc lời gọi hàm bằng `log.Fatal`.

Hàm `handler` có kiểu `http.HandlerFunc`. Nó nhận một `http.ResponseWriter` và một `http.Request` làm đối số.

Một giá trị `http.ResponseWriter` lắp ráp phản hồi của máy chủ HTTP; bằng cách viết vào nó, chúng tôi gửi dữ liệu đến máy khách HTTP.

Một `http.Request` là một cấu trúc dữ liệu đại diện cho yêu cầu HTTP của máy khách. `r.URL.Path` là thành phần đường dẫn của URL yêu cầu. Phần `[1:]` theo sau có nghĩa là "tạo một sub-slice của `Path` từ ký tự thứ 1 đến cuối." Điều này loại bỏ "/" ở đầu khỏi tên đường dẫn.

Nếu bạn chạy chương trình này và truy cập URL:

```
http://localhost:8080/monkeys
```

chương trình sẽ trình bày một trang chứa:

```
Hi there, I love monkeys!
```

## Sử dụng net/http để phục vụ các trang wiki

Để sử dụng package `net/http`, nó phải được import:

```go
import (
    "fmt"
    "os"
    "log"
    "net/http"
)
```

Hãy tạo một handler, `viewHandler` sẽ cho phép người dùng xem một trang wiki. Nó sẽ xử lý các URL có tiền tố "/view/".

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```

Một lần nữa, lưu ý việc sử dụng `_` để bỏ qua giá trị trả về `error` từ `loadPage`. Điều này được thực hiện ở đây để đơn giản và thường được coi là thực hành xấu. Chúng tôi sẽ chú ý đến điều này sau.

Đầu tiên, hàm này trích xuất tiêu đề trang từ `r.URL.Path`, thành phần đường dẫn của URL yêu cầu. `Path` được cắt lại bằng `[len("/view/"):]` để loại bỏ thành phần "/view/" ở đầu của đường dẫn yêu cầu. Điều này là vì đường dẫn sẽ luôn bắt đầu bằng "/view/", không phải là một phần của tiêu đề trang.

Sau đó, hàm tải dữ liệu trang, định dạng trang bằng một chuỗi HTML đơn giản, và viết nó vào `w`, `http.ResponseWriter`.

## Kết luận

Hướng dẫn này đã giới thiệu các khái niệm cơ bản về xây dựng ứng dụng web với Go. Bạn đã học về:

- Cách tạo các cấu trúc dữ liệu với các phương thức
- Cách đọc và ghi file
- Cách sử dụng package `net/http` để tạo máy chủ web
- Cách xử lý các yêu cầu HTTP

Để tìm hiểu thêm về việc xây dựng ứng dụng web với Go, hãy xem [Hướng dẫn: Phát triển RESTful API với Go và Gin](web-service-gin.md).

---

*Tài liệu này được dịch từ [go.dev/doc/articles/wiki](https://go.dev/doc/articles/wiki/) để phục vụ cộng đồng lập trình viên Việt Nam.*
