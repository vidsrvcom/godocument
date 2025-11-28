# Hướng dẫn: Tạo một module Go

Đây là phần đầu tiên của một hướng dẫn giới thiệu một vài tính năng cơ bản của ngôn ngữ Go. Nếu bạn mới bắt đầu với Go, hãy chắc chắn xem [Hướng dẫn: Bắt đầu với Go](getting-started.md), nơi giới thiệu lệnh `go`, module Go và mã Go rất đơn giản.

Trong hướng dẫn này bạn sẽ tạo hai module. Đầu tiên là một thư viện dự định được import bởi các thư viện hoặc ứng dụng khác. Thứ hai là một ứng dụng caller sẽ sử dụng thư viện đầu tiên.

Chuỗi hướng dẫn này bao gồm bảy chủ đề ngắn, mỗi chủ đề minh họa một phần khác nhau của ngôn ngữ.

1. Tạo một module -- Viết một module nhỏ với các hàm bạn có thể gọi từ một module khác.
2. [Gọi mã của bạn từ một module khác](#gọi-mã-của-bạn-từ-một-module-khác) -- Import và sử dụng module mới của bạn.
3. [Trả về và xử lý lỗi](#trả-về-và-xử-lý-lỗi) -- Thêm xử lý lỗi đơn giản.
4. [Trả về một lời chào ngẫu nhiên](#trả-về-một-lời-chào-ngẫu-nhiên) -- Xử lý dữ liệu trong slice (mảng có kích thước động của Go).
5. [Trả về lời chào cho nhiều người](#trả-về-lời-chào-cho-nhiều-người) -- Lưu trữ các cặp key/value trong map.
6. [Thêm bài test](#thêm-bài-test) -- Sử dụng các quy ước testing tích hợp của Go để test mã của bạn.
7. [Biên dịch và cài đặt ứng dụng](#biên-dịch-và-cài-đặt-ứng-dụng) -- Biên dịch và cài đặt mã của bạn cục bộ.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Yêu cầu tiên quyết

- **Một chút kinh nghiệm lập trình.** Mã ở đây khá đơn giản, nhưng sẽ hữu ích nếu biết một số điều về hàm, vòng lặp và mảng.
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt. Hầu hết các trình soạn thảo văn bản đều có hỗ trợ tốt cho Go. Phổ biến nhất là VSCode (miễn phí), GoLand (trả phí), và Vim (miễn phí).
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.

## Bắt đầu một module mà người khác có thể sử dụng

Bắt đầu bằng việc tạo một module Go. Trong một module, bạn thu thập một hoặc nhiều package liên quan cho một tập hợp các hàm rời rạc và hữu ích. Ví dụ, bạn có thể tạo một module với các package có các hàm để phân tích tài chính, để những người khác viết ứng dụng tài chính có thể sử dụng công việc của bạn. Để biết thêm về phát triển module, xem [Phát triển và xuất bản module](https://go.dev/doc/modules/developing).

Mã Go được nhóm vào các package, và các package được nhóm vào các module. Module của bạn xác định các phụ thuộc cần thiết để chạy mã của bạn, bao gồm phiên bản Go và tập hợp các module khác mà nó yêu cầu.

Khi bạn thêm hoặc cải thiện chức năng trong module của mình, bạn xuất bản các phiên bản mới của module. Các nhà phát triển viết mã gọi các hàm trong module của bạn có thể import các package đã cập nhật của module và test với phiên bản mới trước khi đưa vào sử dụng sản xuất.

1. Mở một dấu nhắc lệnh và `cd` đến thư mục home của bạn.

   Trên Linux hoặc Mac:
   ```bash
   cd
   ```

   Trên Windows:
   ```bash
   cd %HOMEPATH%
   ```

2. Tạo một thư mục `greetings` cho mã nguồn module Go của bạn.

   Ví dụ, từ thư mục home của bạn sử dụng các lệnh sau:
   ```bash
   mkdir greetings
   cd greetings
   ```

3. Bắt đầu module của bạn bằng lệnh `go mod init`.

   Chạy lệnh `go mod init`, đặt cho nó đường dẫn module của bạn -- ở đây, sử dụng `example.com/greetings`. Nếu bạn xuất bản một module, đây *phải* là một đường dẫn mà từ đó module của bạn có thể được tải xuống bởi các công cụ Go. Đó sẽ là repository của mã của bạn.

   Để biết thêm về đặt tên module của bạn với đường dẫn module, xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies#naming_module).
   ```bash
   $ go mod init example.com/greetings
   go: creating new go.mod: module example.com/greetings
   ```

   Lệnh `go mod init` tạo một file go.mod để theo dõi các phụ thuộc của mã của bạn. Cho đến nay, file chỉ bao gồm tên của module của bạn và phiên bản Go mà mã của bạn hỗ trợ. Nhưng khi bạn thêm các phụ thuộc, file go.mod sẽ liệt kê các phiên bản mà mã của bạn phụ thuộc vào. Điều này giữ cho các bản build có thể tái tạo và cho phép bạn kiểm soát trực tiếp các phiên bản module nào để sử dụng.

4. Trong trình soạn thảo văn bản của bạn, tạo một file trong đó để viết mã của bạn và gọi nó là greetings.go.

5. Dán mã sau vào file greetings.go của bạn và lưu file.
   ```go
   package greetings

   import "fmt"

   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) string {
       // Trả về một lời chào nhúng tên vào một thông báo.
       message := fmt.Sprintf("Hi, %v. Welcome!", name)
       return message
   }
   ```

   Đây là mã đầu tiên cho module của bạn. Nó trả về một lời chào cho bất kỳ caller nào yêu cầu. Bạn sẽ viết mã gọi hàm này trong bước tiếp theo.

   Trong mã này, bạn:

   - Khai báo một package `greetings` để thu thập các hàm liên quan.
   - Triển khai một hàm `Hello` để trả về lời chào.

     Hàm này nhận một tham số `name` có kiểu là `string`. Hàm cũng trả về một `string`. Trong Go, một hàm có tên bắt đầu bằng chữ hoa có thể được gọi bởi một hàm không nằm trong cùng package. Điều này được gọi là tên được *export* trong Go. Để biết thêm về tên export, xem [Tên export](https://go.dev/tour/basics/3) trong Go tour.

     ![Hình minh họa hàm](https://go.dev/doc/tutorial/images/function-syntax.png)

   - Khai báo một biến `message` để giữ lời chào của bạn.

     Trong Go, toán tử `:=` là một shortcut để khai báo và khởi tạo một biến trong một dòng (Go sử dụng giá trị ở bên phải để xác định kiểu của biến). Lấy cách dài, bạn có thể đã viết điều này như:
     ```go
     var message string
     message = fmt.Sprintf("Hi, %v. Welcome!", name)
     ```

   - Sử dụng hàm `Sprintf` của package `fmt` để tạo một thông báo lời chào. Đối số đầu tiên là một chuỗi định dạng, và `Sprintf` thay thế định dạng `%v` của tham số `name` bằng giá trị của tham số `name`. Việc chèn giá trị của tham số `name` hoàn thành văn bản lời chào.
   - Trả về văn bản lời chào đã được định dạng cho caller.

## Gọi mã của bạn từ một module khác

Trong phần trước, bạn đã tạo một module `greetings`. Trong phần này, bạn sẽ viết mã để gọi hàm `Hello` trong module bạn vừa viết. Bạn sẽ viết mã bạn có thể thực thi như một ứng dụng, và mã này gọi mã trong module `greetings`.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

1. Tạo một thư mục `hello` cho mã nguồn module Go caller của bạn. Đây là nơi bạn sẽ viết caller của mình.

   Sau khi bạn tạo thư mục này, bạn sẽ có cả thư mục hello và greetings ở cùng cấp trong hệ thống phân cấp, như thế này:
   ```
   <home>/
    |-- greetings/
    |-- hello/
   ```

   Ví dụ, nếu dấu nhắc lệnh của bạn đang ở thư mục greetings, bạn có thể sử dụng các lệnh sau:
   ```bash
   cd ..
   mkdir hello
   cd hello
   ```

2. Kích hoạt theo dõi phụ thuộc cho mã bạn sắp viết.

   Để kích hoạt theo dõi phụ thuộc cho mã của bạn, chạy lệnh `go mod init`, đặt cho nó tên của module mà mã của bạn sẽ nằm trong.

   Cho mục đích của hướng dẫn này, sử dụng `example.com/hello` cho đường dẫn module.
   ```bash
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

3. Trong trình soạn thảo văn bản của bạn, trong thư mục hello, tạo một file trong đó để viết mã của bạn và gọi nó là hello.go.

4. Viết mã để gọi hàm `Hello`, sau đó in giá trị trả về của hàm.

   Dán mã sau vào hello.go:
   ```go
   package main

   import (
       "fmt"

       "example.com/greetings"
   )

   func main() {
       // Lấy một thông báo lời chào và in nó.
       message := greetings.Hello("Gladys")
       fmt.Println(message)
   }
   ```

   Trong mã này, bạn:

   - Khai báo một package `main`. Trong Go, mã được thực thi như một ứng dụng phải nằm trong một package `main`.
   - Import hai package: `example.com/greetings` và package `fmt`. Điều này cho mã của bạn quyền truy cập vào các hàm trong các package đó. Việc import `example.com/greetings` (package chứa trong module bạn đã tạo trước đó) cho bạn quyền truy cập vào hàm `Hello`. Bạn cũng import `fmt`, với các hàm để xử lý văn bản đầu vào và đầu ra (như in văn bản ra console).
   - Lấy một lời chào bằng cách gọi hàm `Hello` của package `greetings`.

5. Chỉnh sửa module `example.com/hello` để sử dụng module `example.com/greetings` cục bộ của bạn.

   Đối với việc sử dụng sản xuất, bạn sẽ xuất bản module `example.com/greetings` từ repository của nó (với đường dẫn module phản ánh vị trí đã xuất bản của nó), nơi các công cụ Go có thể tìm thấy nó để tải xuống. Vào lúc này, vì bạn chưa xuất bản module, bạn cần điều chỉnh module `example.com/hello` để nó có thể tìm thấy mã `example.com/greetings` trên hệ thống file cục bộ của bạn.

   Để làm điều đó, sử dụng lệnh `go mod edit` để chỉnh sửa module `example.com/hello` nhằm chuyển hướng các công cụ Go từ đường dẫn module của nó (nơi module không có) đến thư mục cục bộ (nơi nó có).

   1. Từ dấu nhắc lệnh trong thư mục hello, chạy lệnh sau:
      ```bash
      $ go mod edit -replace example.com/greetings=../greetings
      ```

      Lệnh xác định rằng `example.com/greetings` nên được thay thế bằng `../greetings` cho mục đích định vị phụ thuộc. Sau khi bạn chạy lệnh, file go.mod trong thư mục hello sẽ bao gồm một chỉ thị [`replace`](https://go.dev/doc/modules/gomod-ref#replace):
      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings
      ```

   2. Từ dấu nhắc lệnh trong thư mục hello, chạy lệnh `go mod tidy` để đồng bộ các phụ thuộc của module `example.com/hello`, thêm những phụ thuộc được yêu cầu bởi mã, nhưng chưa được theo dõi trong module.
      ```bash
      $ go mod tidy
      go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
      ```

      Sau khi lệnh hoàn thành, file go.mod của module `example.com/hello` sẽ trông như thế này:
      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings

      require example.com/greetings v0.0.0-00010101000000-000000000000
      ```

      Lệnh đã tìm thấy mã cục bộ trong thư mục greetings, sau đó thêm một chỉ thị [`require`](https://go.dev/doc/modules/gomod-ref#require) để xác định rằng `example.com/hello` yêu cầu `example.com/greetings`. Bạn đã tạo phụ thuộc này khi bạn import package `greetings` trong hello.go.

      Số theo sau đường dẫn module là một *số phiên bản giả* -- một số được tạo ra thay cho một số phiên bản ngữ nghĩa (mà module không có).

      Để tham chiếu một module *đã xuất bản*, file go.mod thường sẽ bỏ qua chỉ thị `replace` và sử dụng một chỉ thị `require` với một số phiên bản được gắn thẻ ở cuối.
      ```
      require example.com/greetings v1.1.0
      ```
      Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](https://go.dev/doc/modules/version-numbers).

6. Tại dấu nhắc lệnh trong thư mục `hello`, chạy mã của bạn để xác nhận rằng nó hoạt động.
   ```bash
   $ go run .
   Hi, Gladys. Welcome!
   ```

Xin chúc mừng! Bạn đã viết hai module hoạt động.

## Trả về và xử lý lỗi

Xử lý lỗi là một tính năng thiết yếu của mã vững chắc. Trong phần này, bạn sẽ thêm một chút mã để trả về một lỗi từ module greetings, sau đó xử lý nó trong caller.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

1. Trong greetings/greetings.go, thêm mã được đánh dấu dưới đây.

   Không có lý do để gửi một lời chào nếu bạn không biết chào ai. Trả về một lỗi cho caller nếu tên là trống. Sao chép mã sau vào greetings.go và lưu file.
   ```go
   package greetings

   import (
       "errors"
       "fmt"
   )

   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) (string, error) {
       // Nếu không có tên nào được cung cấp, trả về một lỗi với một thông báo.
       if name == "" {
           return "", errors.New("empty name")
       }

       // Nếu một tên được nhận, trả về một giá trị nhúng tên
       // vào một thông báo lời chào.
       message := fmt.Sprintf("Hi, %v. Welcome!", name)
       return message, nil
   }
   ```

   Trong mã này, bạn:

   - Thay đổi hàm để nó trả về hai giá trị: một `string` và một `error`. Caller của bạn sẽ kiểm tra giá trị thứ hai để xem có xảy ra lỗi không. (Bất kỳ hàm Go nào cũng có thể trả về nhiều giá trị. Để biết thêm, xem [Effective Go](https://go.dev/doc/effective_go.html#multiple-returns).)
   - Import package Go `errors` để bạn có thể sử dụng hàm [`errors.New`](https://pkg.go.dev/errors/#New) của nó.
   - Thêm một câu lệnh `if` để kiểm tra một yêu cầu không hợp lệ (một tên trống) và trả về một lỗi nếu yêu cầu không hợp lệ. Hàm `errors.New` trả về một `error` với thông báo của bạn bên trong nó.
   - Thêm `nil` (có nghĩa là không có lỗi) làm giá trị thứ hai trong trả về thành công. Bằng cách đó, caller có thể thấy rằng hàm đã thành công.

2. Trong file hello/hello.go của bạn, xử lý lỗi bây giờ được trả về bởi hàm `Hello`, cùng với giá trị không lỗi.

   Dán mã sau vào hello.go.
   ```go
   package main

   import (
       "fmt"
       "log"

       "example.com/greetings"
   )

   func main() {
       // Đặt thuộc tính của Logger được xác định trước, bao gồm
       // cờ tiền tố log và một cờ để vô hiệu hóa việc in
       // thời gian, file nguồn, và số dòng.
       log.SetPrefix("greetings: ")
       log.SetFlags(0)

       // Yêu cầu một thông báo lời chào.
       message, err := greetings.Hello("")
       // Nếu một lỗi được trả về, in nó ra console và
       // thoát chương trình.
       if err != nil {
           log.Fatal(err)
       }

       // Nếu không có lỗi được trả về, in thông báo
       // trả về ra console.
       fmt.Println(message)
   }
   ```

   Trong mã này, bạn:

   - Cấu hình package [`log`](https://pkg.go.dev/log/) để in tên lệnh ("greetings: ") ở đầu các thông báo log của nó, không có dấu thời gian hoặc thông tin file nguồn.
   - Gán cả hai giá trị trả về của `Hello`, bao gồm `error`, cho các biến.
   - Thay đổi đối số `Hello` từ tên của Gladys thành một chuỗi trống để bạn có thể thử mã xử lý lỗi của mình.
   - Tìm kiếm một giá trị lỗi không phải nil. Không có lý do để tiếp tục trong trường hợp này.
   - Sử dụng các hàm trong package `log` của thư viện chuẩn để xuất thông tin lỗi. Nếu bạn nhận được một lỗi, bạn sử dụng hàm [`Fatal`](https://pkg.go.dev/log?tab=doc#Fatal) của package `log` để in lỗi và dừng chương trình.

3. Tại dấu nhắc lệnh trong thư mục `hello`, chạy hello.go để xác nhận mã hoạt động.

   Bây giờ khi bạn truyền vào một tên trống, bạn sẽ nhận được một lỗi.
   ```bash
   $ go run .
   greetings: empty name
   exit status 1
   ```

Đó là xử lý lỗi phổ biến trong Go: Trả về một giá trị lỗi làm giá trị bổ sung để caller có thể kiểm tra nó.

## Trả về một lời chào ngẫu nhiên

Trong phần này, bạn sẽ thay đổi mã của mình để thay vì trả về một lời chào duy nhất mỗi lần, nó trả về một trong một số thông báo lời chào được xác định trước.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

Để làm điều này, bạn sẽ sử dụng một Go slice. Một slice giống như một mảng, ngoại trừ việc kích thước của nó thay đổi động khi bạn thêm và xóa các mục. Slice là một trong những kiểu hữu ích nhất trong Go. Bạn sẽ thêm một slice nhỏ để chứa ba thông báo lời chào, sau đó để mã của bạn trả về một trong các thông báo ngẫu nhiên. Để biết thêm về slice, xem [Slice trong Go blog](https://go.dev/blog/slices-intro).

1. Trong greetings/greetings.go, thay đổi mã của bạn để nó trông như sau.
   ```go
   package greetings

   import (
       "errors"
       "fmt"
       "math/rand"
   )

   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) (string, error) {
       // Nếu không có tên nào được cung cấp, trả về một lỗi với một thông báo.
       if name == "" {
           return "", errors.New("empty name")
       }
       // Tạo một thông báo sử dụng một định dạng ngẫu nhiên.
       message := fmt.Sprintf(randomFormat(), name)
       return message, nil
   }

   // randomFormat trả về một trong một tập hợp các định dạng thông báo lời chào. Thông báo
   // trả về được chọn ngẫu nhiên.
   func randomFormat() string {
       // Một slice các định dạng thông báo.
       formats := []string{
           "Hi, %v. Welcome!",
           "Great to see you, %v!",
           "Hail, %v! Well met!",
       }

       // Trả về một định dạng thông báo được chọn ngẫu nhiên bằng cách
       // xác định một chỉ số ngẫu nhiên cho slice.
       return formats[rand.Intn(len(formats))]
   }
   ```

   Trong mã này, bạn:

   - Thêm một hàm `randomFormat` trả về một định dạng được chọn ngẫu nhiên cho một thông báo lời chào. Lưu ý rằng `randomFormat` bắt đầu bằng một chữ thường, làm cho nó chỉ có thể truy cập được từ mã trong package của chính nó (nói cách khác, nó không được export).
   - Trong `randomFormat`, khai báo một slice `formats` với ba định dạng thông báo. Khi khai báo một slice, bạn bỏ qua kích thước của nó trong các dấu ngoặc, như thế này: `[]string`. Điều này cho Go biết rằng kích thước của mảng bên dưới một slice có thể được thay đổi động.
   - Sử dụng package [`math/rand`](https://pkg.go.dev/math/rand/) để tạo một số ngẫu nhiên để chọn một mục từ slice.
   - Trong `Hello`, gọi hàm `randomFormat` để lấy một định dạng cho thông báo bạn sẽ trả về, sau đó sử dụng định dạng và giá trị `name` cùng nhau để tạo thông báo.
   - Trả về thông báo (hoặc một lỗi) như bạn đã làm trước đó.

2. Trong hello/hello.go, thay đổi mã của bạn để nó trông như sau.

   Bạn chỉ thêm tên của Gladys (hoặc một tên khác, nếu bạn thích) làm một đối số cho lời gọi hàm `Hello` trong hello.go.
   ```go
   package main

   import (
       "fmt"
       "log"

       "example.com/greetings"
   )

   func main() {
       // Đặt thuộc tính của Logger được xác định trước, bao gồm
       // cờ tiền tố log và một cờ để vô hiệu hóa việc in
       // thời gian, file nguồn, và số dòng.
       log.SetPrefix("greetings: ")
       log.SetFlags(0)

       // Yêu cầu một thông báo lời chào.
       message, err := greetings.Hello("Gladys")
       // Nếu một lỗi được trả về, in nó ra console và
       // thoát chương trình.
       if err != nil {
           log.Fatal(err)
       }

       // Nếu không có lỗi được trả về, in thông báo
       // trả về ra console.
       fmt.Println(message)
   }
   ```

3. Tại dấu nhắc lệnh trong thư mục hello, chạy hello.go để xác nhận mã hoạt động. Chạy nó nhiều lần, chú ý rằng lời chào thay đổi.
   ```bash
   $ go run .
   Great to see you, Gladys!

   $ go run .
   Hi, Gladys. Welcome!

   $ go run .
   Hail, Gladys! Well met!
   ```

## Trả về lời chào cho nhiều người

Trong phần cuối cùng, bạn sẽ xử lý một yêu cầu nhiều người. Nói cách khác, bạn sẽ xử lý một đầu vào nhiều giá trị, sau đó ghép các giá trị trong đầu vào đó với một đầu ra nhiều giá trị. Để làm điều này, bạn sẽ cần truyền một tập hợp tên cho một hàm có thể trả về một lời chào cho từng tên đó.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

Nhưng có một phức tạp. Thay đổi tham số của hàm `Hello` từ một tên đơn thành một tập hợp tên sẽ thay đổi chữ ký của hàm. Nếu bạn đã xuất bản module `example.com/greetings` và người dùng đã viết mã gọi `Hello`, thay đổi đó sẽ *làm hỏng* chương trình của họ.

Trong tình huống này, một lựa chọn tốt hơn là viết một hàm mới với một tên khác. Hàm mới sẽ nhận nhiều tham số. Điều đó bảo tồn hàm cũ cho tính tương thích ngược.

1. Trong greetings/greetings.go, thay đổi mã của bạn để nó trông như sau.
   ```go
   package greetings

   import (
       "errors"
       "fmt"
       "math/rand"
   )

   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) (string, error) {
       // Nếu không có tên nào được cung cấp, trả về một lỗi với một thông báo.
       if name == "" {
           return "", errors.New("empty name")
       }
       // Tạo một thông báo sử dụng một định dạng ngẫu nhiên.
       message := fmt.Sprintf(randomFormat(), name)
       return message, nil
   }

   // Hellos trả về một map liên kết từng tên được đặt tên với một
   // thông báo lời chào.
   func Hellos(names []string) (map[string]string, error) {
       // Một map để liên kết tên với thông báo.
       messages := make(map[string]string)
       // Lặp qua slice tên nhận được, gọi
       // hàm Hello để lấy một thông báo cho từng tên.
       for _, name := range names {
           message, err := Hello(name)
           if err != nil {
               return nil, err
           }
           // Trong map, liên kết thông báo đã truy xuất với
           // tên.
           messages[name] = message
       }
       return messages, nil
   }

   // randomFormat trả về một trong một tập hợp các định dạng thông báo lời chào. Thông báo
   // trả về được chọn ngẫu nhiên.
   func randomFormat() string {
       // Một slice các định dạng thông báo.
       formats := []string{
           "Hi, %v. Welcome!",
           "Great to see you, %v!",
           "Hail, %v! Well met!",
       }

       // Trả về một định dạng thông báo được chọn ngẫu nhiên bằng cách
       // xác định một chỉ số ngẫu nhiên cho slice.
       return formats[rand.Intn(len(formats))]
   }
   ```

   Trong mã này, bạn:

   - Thêm một hàm `Hellos` có tham số là một slice tên thay vì một tên đơn. Ngoài ra, bạn thay đổi một trong các kiểu trả về của nó từ một `string` thành một `map` để bạn có thể trả về các tên được ánh xạ tới các thông báo lời chào.
   - Để hàm `Hellos` mới gọi hàm `Hello` hiện có. Điều này giúp giảm trùng lặp trong khi giữ lại cả hai hàm.
   - Tạo một `messages` map để liên kết từng tên nhận được (làm key) với một thông báo được tạo (làm value). Trong Go, bạn khởi tạo một map với cú pháp sau: `make(map[key-type]value-type)`. Bạn để hàm `Hellos` trả về map này cho caller. Để biết thêm về map, xem [Map trong Go blog](https://go.dev/blog/maps).
   - Lặp qua các tên mà hàm của bạn nhận được, kiểm tra rằng mỗi tên có một giá trị không rỗng, sau đó liên kết một thông báo với từng tên. Trong vòng lặp `for` này, `range` trả về hai giá trị: chỉ số của mục hiện tại trong vòng lặp và một bản sao của giá trị của mục. Bạn không cần chỉ số, vì vậy bạn sử dụng định danh trống Go (một dấu gạch dưới) để bỏ qua nó. Để biết thêm, xem [Định danh trống](https://go.dev/doc/effective_go.html#blank) trong Effective Go.

2. Trong mã gọi hello/hello.go của bạn, truyền một slice tên, sau đó in nội dung của bản đồ tên/thông báo bạn nhận được.

   Trong hello.go, thay đổi mã của bạn để nó trông như sau.
   ```go
   package main

   import (
       "fmt"
       "log"

       "example.com/greetings"
   )

   func main() {
       // Đặt thuộc tính của Logger được xác định trước, bao gồm
       // cờ tiền tố log và một cờ để vô hiệu hóa việc in
       // thời gian, file nguồn, và số dòng.
       log.SetPrefix("greetings: ")
       log.SetFlags(0)

       // Một slice tên.
       names := []string{"Gladys", "Samantha", "Darrin"}

       // Yêu cầu thông báo lời chào cho các tên.
       messages, err := greetings.Hellos(names)
       if err != nil {
           log.Fatal(err)
       }
       // Nếu không có lỗi được trả về, in bản đồ tên
       // và thông báo trả về ra console.
       fmt.Println(messages)
   }
   ```

   Với những thay đổi này, bạn:

   - Tạo một biến `names` làm kiểu slice giữ ba tên.
   - Truyền biến `names` làm đối số cho hàm `Hellos`.

3. Tại dấu nhắc lệnh, thay đổi đến thư mục chứa hello/hello.go, sau đó sử dụng `go run` để xác nhận mã hoạt động.

   Đầu ra sẽ là một chuỗi đại diện của map liên kết các tên với thông báo của bạn, như sau:
   ```bash
   $ go run .
   map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
   ```

## Thêm bài test

Bây giờ bạn đã có mã của mình ở một nơi ổn định (làm tốt lắm, nhân tiện), hãy thêm một bài test. Testing mã của bạn trong quá trình phát triển có thể bộc lộ các lỗi xảy ra trong khi bạn thực hiện các thay đổi. Trong chủ đề này, bạn thêm một bài test cho hàm `Hello`.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

Hỗ trợ testing tích hợp của Go giúp dễ dàng test khi bạn đi. Cụ thể, sử dụng các quy ước đặt tên, package `testing` của Go, và lệnh `go test`, bạn có thể nhanh chóng viết và thực thi các bài test.

1. Trong thư mục greetings, tạo một file gọi là greetings_test.go.

   Kết thúc tên file bằng _test.go nói với lệnh `go test` rằng file này chứa các hàm test.

2. Trong greetings_test.go, dán mã sau và lưu file.
   ```go
   package greetings

   import (
       "regexp"
       "testing"
   )

   // TestHelloName gọi greetings.Hello với một tên, kiểm tra
   // một giá trị trả về hợp lệ.
   func TestHelloName(t *testing.T) {
       name := "Gladys"
       want := regexp.MustCompile(`\b` + name + `\b`)
       msg, err := Hello("Gladys")
       if !want.MatchString(msg) || err != nil {
           t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
       }
   }

   // TestHelloEmpty gọi greetings.Hello với một chuỗi trống,
   // kiểm tra một lỗi.
   func TestHelloEmpty(t *testing.T) {
       msg, err := Hello("")
       if msg != "" || err == nil {
           t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
       }
   }
   ```

   Trong mã này, bạn:

   - Triển khai các hàm test trong cùng package với mã bạn đang test.
   - Tạo hai hàm test để test hàm `greetings.Hello`. Tên hàm test có dạng `TestName`, trong đó *Name* nói điều gì đó về bài test cụ thể. Ngoài ra, các hàm test nhận một con trỏ đến kiểu [`testing.T`](https://pkg.go.dev/testing/#T) của package `testing` làm tham số. Bạn sử dụng các phương thức của tham số này để báo cáo và ghi log từ bài test của bạn.
   - Triển khai hai bài test:

     - `TestHelloName` gọi hàm `Hello`, truyền một giá trị `name` với mà hàm sẽ có thể trả về một thông báo phản hồi hợp lệ. Nếu lời gọi trả về một lỗi hoặc một phản hồi không mong đợi (một phản hồi không bao gồm tên bạn đã truyền vào), bạn sử dụng phương thức `Fatalf` của tham số `t` để in một thông báo ra console và kết thúc thực thi.
     - `TestHelloEmpty` gọi hàm `Hello` với một chuỗi trống. Bài test này được thiết kế để xác nhận rằng xử lý lỗi của bạn hoạt động đúng. Nếu lời gọi trả về một chuỗi không rỗng hoặc không có lỗi, bạn sử dụng phương thức `Fatalf` của tham số `t` để in một thông báo ra console và kết thúc thực thi.

3. Tại dấu nhắc lệnh trong thư mục greetings, chạy lệnh [`go test`](https://go.dev/cmd/go/#hdr-Test_packages) để thực thi bài test.

   Lệnh `go test` thực thi các hàm test (có tên bắt đầu bằng `Test`) trong các file test (có tên kết thúc bằng _test.go) trong thư mục hiện tại. Bạn có thể thêm cờ `-v` để lấy đầu ra chi tiết liệt kê tất cả các bài test và kết quả của chúng.

   Các bài test sẽ pass.
   ```bash
   $ go test
   PASS
   ok      example.com/greetings   0.364s

   $ go test -v
   === RUN   TestHelloName
   --- PASS: TestHelloName (0.00s)
   === RUN   TestHelloEmpty
   --- PASS: TestHelloEmpty (0.00s)
   PASS
   ok      example.com/greetings   0.372s
   ```

4. Phá vỡ hàm `greetings.Hello` để xem một bài test thất bại.

   Hàm `TestHelloName` test kiểm tra giá trị trả về cho tên bạn đã chỉ định làm tham số cho hàm `Hello`. Để xem kết quả bài test thất bại, thay đổi hàm `greetings.Hello` để nó không còn bao gồm tên nữa.

   Trong greetings/greetings.go, dán mã sau thay cho hàm `Hello`. Lưu ý rằng các dòng được đánh dấu thay đổi giá trị mà hàm trả về, như thể đối số `name` đã bị loại bỏ một cách vô tình.
   ```go
   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) (string, error) {
       // Nếu không có tên nào được cung cấp, trả về một lỗi với một thông báo.
       if name == "" {
           return "", errors.New("empty name")
       }
       // Tạo một thông báo sử dụng một định dạng ngẫu nhiên.
       // message := fmt.Sprintf(randomFormat(), name)
       message := fmt.Sprintf(randomFormat())
       return message, nil
   }
   ```

5. Tại dấu nhắc lệnh trong thư mục greetings, chạy `go test` để thực thi bài test.

   Lần này, chạy `go test` mà không có cờ `-v`. Đầu ra sẽ chỉ bao gồm kết quả cho các bài test thất bại, điều này có thể hữu ích khi bạn có nhiều bài test. Bài test `TestHelloName` sẽ thất bại -- `TestHelloEmpty` vẫn pass.
   ```bash
   $ go test
   --- FAIL: TestHelloName (0.00s)
       greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
   FAIL
   exit status 1
   FAIL    example.com/greetings   0.182s
   ```

6. Trong greetings/greetings.go, dán mã sau thay cho hàm `Hello`. Điều này phục hồi nó về trạng thái hoạt động ban đầu.
   ```go
   // Hello trả về một lời chào cho người được đặt tên.
   func Hello(name string) (string, error) {
       // Nếu không có tên nào được cung cấp, trả về một lỗi với một thông báo.
       if name == "" {
           return "", errors.New("empty name")
       }
       // Tạo một thông báo sử dụng một định dạng ngẫu nhiên.
       message := fmt.Sprintf(randomFormat(), name)
       return message, nil
   }
   ```

7. Chạy `go test` lại để xác nhận các bài test đều pass.
   ```bash
   $ go test
   PASS
   ok      example.com/greetings   0.263s
   ```

## Biên dịch và cài đặt ứng dụng

Trong chủ đề cuối cùng này, bạn sẽ học một vài lệnh `go` mới. Trong khi lệnh `go run` là một shortcut hữu ích để biên dịch và chạy một chương trình khi bạn đang thực hiện các thay đổi thường xuyên, nó không tạo ra một binary thực thi.

Chủ đề này giới thiệu hai lệnh bổ sung để build mã:

- Lệnh [`go build`](https://go.dev/cmd/go/#hdr-Compile_packages_and_dependencies) biên dịch các package, cùng với các phụ thuộc của chúng, nhưng nó không cài đặt kết quả.
- Lệnh [`go install`](https://go.dev/ref/mod#go-install) biên dịch và cài đặt các package.

> **Lưu ý:** Chủ đề này là một phần của một hướng dẫn nhiều phần bắt đầu với [Tạo một module Go](#bắt-đầu-một-module-mà-người-khác-có-thể-sử-dụng).

1. Từ dấu nhắc lệnh trong thư mục hello, chạy lệnh `go build` để biên dịch mã thành một thực thi.
   ```bash
   $ go build
   ```

2. Từ dấu nhắc lệnh trong thư mục hello, chạy thực thi `hello` mới (hoặc `hello.exe` trên Windows) để xác nhận mã hoạt động.
   ```bash
   $ ./hello
   map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hi, Samantha. Welcome!]
   ```

   Bạn đã biên dịch ứng dụng thành một thực thi để bạn có thể chạy nó. Nhưng để chạy nó hiện tại, dấu nhắc của bạn cần phải ở trong thư mục của thực thi, hoặc phải xác định đường dẫn đến thực thi.

   Tiếp theo, bạn sẽ cài đặt thực thi để bạn có thể chạy nó mà không cần xác định đường dẫn của nó.

3. Khám phá đường dẫn cài đặt Go, nơi lệnh `go` sẽ cài đặt package hiện tại.

   Bạn có thể khám phá đường dẫn cài đặt bằng cách chạy lệnh [`go list`](https://go.dev/cmd/go/#hdr-List_packages_or_modules), như trong ví dụ sau:
   ```bash
   $ go list -f '{{.Target}}'
   ```

   Ví dụ, đầu ra của lệnh có thể nói `/home/gopher/bin/hello`, có nghĩa là các binary được cài đặt vào /home/gopher/bin. Bạn sẽ cần thư mục cài đặt này trong bước tiếp theo.

4. Thêm thư mục cài đặt Go vào đường dẫn shell của hệ thống của bạn.

   Bằng cách đó, bạn sẽ có thể chạy thực thi của chương trình mà không cần xác định nơi thực thi nằm.

   - Trên Linux hoặc Mac, chạy lệnh sau:
     ```bash
     $ export PATH=$PATH:/path/to/your/install/directory
     ```

   - Trên Windows, chạy lệnh sau:
     ```bash
     $ set PATH=%PATH%;C:\path\to\your\install\directory
     ```

   Như một thay thế, nếu bạn đã có một thư mục như `$HOME/bin` trong đường dẫn shell của bạn và bạn muốn cài đặt các chương trình Go của bạn ở đó, bạn có thể thay đổi đích cài đặt bằng cách đặt biến `GOBIN` bằng lệnh [`go env`](https://go.dev/cmd/go/#hdr-Print_Go_environment_information):
   ```bash
   $ go env -w GOBIN=/path/to/your/bin
   ```

   hoặc

   ```bash
   $ go env -w GOBIN=C:\path\to\your\bin
   ```

5. Sau khi bạn đã cập nhật đường dẫn shell, chạy lệnh `go install` để biên dịch và cài đặt package.
   ```bash
   $ go install
   ```

6. Chạy ứng dụng của bạn đơn giản bằng cách gõ tên của nó. Để làm cho điều này thú vị hơn, mở một dấu nhắc lệnh mới và chạy tên thực thi `hello` trong một thư mục nào đó khác.
   ```bash
   $ hello
   map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
   ```

Đó là tất cả! Hướng dẫn này kết thúc tại đây.

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/create-module](https://go.dev/doc/tutorial/create-module) để phục vụ cộng đồng lập trình viên Việt Nam.*
