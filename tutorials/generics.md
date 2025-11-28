# Hướng dẫn: Bắt đầu với generics

Bảng nội dung:
- [Yêu cầu tiên quyết](#yêu-cầu-tiên-quyết)
- [Tạo thư mục cho mã của bạn](#tạo-thư-mục-cho-mã-của-bạn)
- [Thêm các hàm không phải generic](#thêm-các-hàm-không-phải-generic)
- [Thêm một hàm generic để xử lý nhiều kiểu](#thêm-một-hàm-generic-để-xử-lý-nhiều-kiểu)
- [Xóa các ràng buộc kiểu khi gọi hàm generic](#xóa-các-ràng-buộc-kiểu-khi-gọi-hàm-generic)
- [Khai báo một ràng buộc kiểu](#khai-báo-một-ràng-buộc-kiểu)
- [Kết luận](#kết-luận)

Hướng dẫn này giới thiệu về cơ bản của generics trong Go. Với generics, bạn có thể khai báo và sử dụng các hàm hoặc kiểu được viết để làm việc với bất kỳ tập hợp kiểu nào do mã gọi cung cấp.

Trong hướng dẫn này, bạn sẽ khai báo hai hàm đơn giản không phải generic, sau đó bắt được cùng logic đó trong một hàm generic duy nhất.

Bạn sẽ tiến triển qua các phần sau:

1. Tạo một thư mục cho mã của bạn.
2. Thêm các hàm không phải generic.
3. Thêm một hàm generic để xử lý nhiều kiểu.
4. Xóa các đối số kiểu khi gọi hàm generic.
5. Khai báo một ràng buộc kiểu.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

> **Lưu ý:** Nếu bạn thích, bạn có thể sử dụng [Go playground ở chế độ "Go dev branch"](https://go.dev/play/?v=gotip) để chỉnh sửa và chạy chương trình của bạn thay vì.

## Yêu cầu tiên quyết

- **Một bản cài đặt Go 1.18 trở lên.** Để biết hướng dẫn cài đặt, xem [Cài đặt Go](https://go.dev/doc/install).
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.

## Tạo thư mục cho mã của bạn

Để bắt đầu, tạo một thư mục cho mã bạn sẽ viết.

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

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn gọi là generics.
   ```bash
   $ mkdir generics
   $ cd generics
   ```

3. Tạo một module để chứa mã của bạn.

   Chạy lệnh `go mod init`, đặt cho nó đường dẫn module cho module mới của bạn.
   ```bash
   $ go mod init example/generics
   go: creating new go.mod: module example/generics
   ```

> **Lưu ý:** Đối với mã sản xuất, bạn sẽ xác định một đường dẫn module cụ thể hơn cho nhu cầu của riêng bạn. Để biết thêm, hãy xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một số mã đơn giản để làm việc với maps.

## Thêm các hàm không phải generic

Trong bước này, bạn sẽ thêm hai hàm mà mỗi hàm cộng các giá trị của một map với nhau và trả về tổng.

Bạn đang khai báo hai hàm thay vì một bởi vì bạn đang làm việc với hai loại map khác nhau: một lưu trữ các giá trị `int64` và một lưu trữ các giá trị `float64`.

### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, trong thư mục generics tạo một file gọi là main.go để viết mã Go của bạn. Bạn sẽ viết mã vào file này.

2. Vào main.go, ở đầu file, dán khai báo package sau.
   ```go
   package main
   ```

   Một chương trình độc lập (trái ngược với một thư viện) luôn nằm trong package `main`.

3. Bên dưới khai báo package, dán hai khai báo hàm sau.
   ```go
   // SumInts cộng các giá trị của m với nhau.
   func SumInts(m map[string]int64) int64 {
       var s int64
       for _, v := range m {
           s += v
       }
       return s
   }

   // SumFloats cộng các giá trị của m với nhau.
   func SumFloats(m map[string]float64) float64 {
       var s float64
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   Trong mã này, bạn:

   - Khai báo hai hàm để cộng các giá trị của một map và trả về tổng.
     - `SumFloats` nhận một map của `string` đến các giá trị `float64`.
     - `SumInts` nhận một map của `string` đến các giá trị `int64`.

4. Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo hai map và sử dụng chúng làm đối số khi gọi các hàm bạn đã khai báo trong bước trước.
   ```go
   func main() {
       // Khởi tạo một map cho các giá trị integer.
       ints := map[string]int64{
           "first":  34,
           "second": 12,
       }

       // Khởi tạo một map cho các giá trị float.
       floats := map[string]float64{
           "first":  35.98,
           "second": 26.99,
       }

       fmt.Printf("Non-Generic Sums: %v and %v\n",
           SumInts(ints),
           SumFloats(floats))
   }
   ```

   Trong mã này, bạn:

   - Khởi tạo một map của các giá trị `float64` và một map của các giá trị `int64`, mỗi map có hai mục.
   - Gọi hai hàm bạn đã khai báo trước đó để tìm tổng của các giá trị của mỗi map.
   - In kết quả.

5. Gần đầu main.go, ngay dưới khai báo package, import package bạn sẽ cần để hỗ trợ mã bạn vừa viết.

   Các dòng mã đầu tiên sẽ trông như thế này:
   ```go
   package main

   import "fmt"
   ```

6. Lưu main.go.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Non-Generic Sums: 46 and 62.97
```

Với generics, bạn có thể viết một hàm ở đây thay vì hai. Tiếp theo, bạn sẽ thêm một hàm generic duy nhất cho các map chứa các giá trị integer hoặc float.

## Thêm một hàm generic để xử lý nhiều kiểu

Trong phần này, bạn sẽ thêm một hàm generic duy nhất có thể nhận một map chứa các giá trị integer hoặc float, thay thế hiệu quả hai hàm bạn vừa viết bằng một hàm duy nhất.

Để hỗ trợ các giá trị của cả hai kiểu, hàm đơn đó sẽ cần một cách để khai báo kiểu nào nó hỗ trợ. Mặt khác, mã gọi sẽ cần một cách để xác định liệu nó đang gọi với một map giá trị integer hay float.

Để hỗ trợ điều này, bạn sẽ viết một hàm khai báo *các tham số kiểu* ngoài các tham số hàm thông thường của nó. Các tham số kiểu này làm cho hàm trở nên generic, cho phép nó hoạt động với các đối số của các kiểu khác nhau. Bạn sẽ gọi hàm với *các đối số kiểu* và các đối số hàm thông thường.

Mỗi tham số kiểu có một *ràng buộc kiểu* hoạt động như một loại meta-type cho tham số kiểu. Mỗi ràng buộc kiểu xác định các đối số kiểu được phép mà mã gọi có thể sử dụng cho tham số kiểu tương ứng.

Trong khi một ràng buộc của tham số kiểu thường đại diện cho một tập hợp các kiểu, tại thời gian biên dịch tham số kiểu đại diện cho một kiểu duy nhất -- kiểu được cung cấp như một đối số kiểu bởi mã gọi. Nếu kiểu của đối số kiểu không được phép bởi ràng buộc của tham số kiểu, mã sẽ không biên dịch.

Hãy nhớ rằng một tham số kiểu phải hỗ trợ tất cả các thao tác mà mã generic đang thực hiện trên nó. Ví dụ, nếu mã hàm của bạn đang cố gắng thực hiện các thao tác `string` (như lập chỉ mục) trên một tham số có ràng buộc bao gồm các kiểu số, mã sẽ không biên dịch.

Trong mã bạn sắp viết, bạn sẽ sử dụng một ràng buộc cho phép các kiểu integer hoặc float.

### Viết mã

1. Bên dưới hai hàm bạn đã thêm trước đó, dán hàm generic sau.
   ```go
   // SumIntsOrFloats tính tổng các giá trị của map m. Nó hỗ trợ cả int64 và float64
   // làm kiểu cho các giá trị của map.
   func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   Trong mã này, bạn:

   - Khai báo một hàm `SumIntsOrFloats` với hai tham số kiểu (bên trong dấu ngoặc vuông), `K` và `V`, và một đối số sử dụng các tham số kiểu, `m` kiểu `map[K]V`. Hàm trả về một giá trị kiểu `V`.

   - Xác định cho tham số kiểu `K` ràng buộc kiểu `comparable`. Dự định đặc biệt cho các trường hợp như thế này, ràng buộc `comparable` được khai báo trước trong Go. Nó cho phép bất kỳ kiểu nào có thể được sử dụng làm toán hạng của các toán tử so sánh `==` và `!=`. Go yêu cầu các key của map phải là comparable. Vì vậy, khai báo `K` là `comparable` là cần thiết để bạn có thể sử dụng `K` như key trong biến map. Nó cũng đảm bảo rằng mã gọi sử dụng một kiểu được phép cho các key của map.

   - Xác định cho tham số kiểu `V` một ràng buộc là một union của hai kiểu: `int64` và `float64`. Sử dụng `|` xác định một union của hai kiểu, có nghĩa là ràng buộc này cho phép cả hai kiểu. Cả hai kiểu sẽ được phép bởi trình biên dịch như các đối số trong mã gọi.

   - Xác định rằng đối số `m` là kiểu `map[K]V`, trong đó `K` và `V` là các kiểu đã được xác định cho các tham số kiểu. Lưu ý rằng chúng ta biết `map[K]V` là một kiểu map hợp lệ vì `K` là một kiểu comparable. Nếu chúng ta không khai báo `K` là comparable, trình biên dịch sẽ từ chối tham chiếu đến `map[K]V`.

2. Trong main.go, bên dưới mã bạn đã có, dán mã sau.
   ```go
   fmt.Printf("Generic Sums: %v and %v\n",
       SumIntsOrFloats[string, int64](ints),
       SumIntsOrFloats[string, float64](floats))
   ```

   Trong mã này, bạn:

   - Gọi hàm generic bạn vừa khai báo, truyền từng map bạn đã tạo.

   - Xác định các đối số kiểu -- tên kiểu trong dấu ngoặc vuông -- để rõ ràng về các kiểu nên thay thế cho các tham số kiểu trong hàm bạn đang gọi.

     Như bạn sẽ thấy trong phần tiếp theo, bạn thường có thể bỏ qua các đối số kiểu trong lời gọi hàm. Go thường có thể suy ra chúng từ mã của bạn.

   - In các tổng được trả về bởi hàm.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

Để chạy mã của bạn, trong mỗi lời gọi trình biên dịch đã thay thế các tham số kiểu bằng các kiểu cụ thể được xác định trong lời gọi đó.

Trong việc gọi hàm generic bạn đã viết, bạn đã xác định các đối số kiểu nói cho trình biên dịch kiểu nào để sử dụng thay cho các tham số kiểu của hàm. Như bạn sẽ thấy trong phần tiếp theo, trong nhiều trường hợp bạn có thể bỏ qua các đối số kiểu này vì trình biên dịch có thể suy ra chúng.

## Xóa các ràng buộc kiểu khi gọi hàm generic

Trong phần này, bạn sẽ thêm một phiên bản sửa đổi của lời gọi hàm generic, thực hiện một thay đổi nhỏ để đơn giản hóa mã gọi. Bạn sẽ xóa các đối số kiểu, vốn không cần thiết trong trường hợp này.

Bạn có thể bỏ qua các đối số kiểu trong mã gọi khi trình biên dịch Go có thể suy ra các kiểu bạn muốn sử dụng. Trình biên dịch suy ra các đối số kiểu từ các kiểu của các đối số hàm.

Lưu ý rằng điều này không phải lúc nào cũng có thể. Ví dụ, nếu bạn cần gọi một hàm generic không có đối số, bạn sẽ cần phải bao gồm các đối số kiểu trong lời gọi hàm.

### Viết mã

- Trong main.go, bên dưới mã bạn đã có, dán mã sau.
   ```go
   fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
       SumIntsOrFloats(ints),
       SumIntsOrFloats(floats))
   ```

   Trong mã này, bạn:

   - Gọi hàm generic, bỏ qua các đối số kiểu.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

Tiếp theo, bạn sẽ đơn giản hóa hàm hơn nữa bằng cách chuyển union của các integers và floats thành một ràng buộc kiểu bạn có thể sử dụng lại, chẳng hạn như từ mã khác.

## Khai báo một ràng buộc kiểu

Trong phần cuối cùng này, bạn sẽ di chuyển ràng buộc bạn đã định nghĩa trước đó vào interface của riêng nó để bạn có thể sử dụng lại nó ở nhiều nơi. Khai báo các ràng buộc theo cách này giúp đơn giản hóa mã, chẳng hạn như khi một ràng buộc phức tạp hơn.

Bạn khai báo một ràng buộc kiểu như một interface. Ràng buộc cho phép bất kỳ kiểu nào triển khai interface. Ví dụ, nếu bạn khai báo một interface ràng buộc kiểu với ba phương thức, sau đó sử dụng nó với một tham số kiểu trong một hàm generic, các đối số kiểu được sử dụng để gọi hàm phải có tất cả các phương thức đó.

Các interface ràng buộc cũng có thể tham chiếu đến các kiểu cụ thể, như bạn sẽ thấy trong phần này.

### Viết mã

1. Ngay trên `main`, ngay sau các câu lệnh import, dán mã sau để khai báo một ràng buộc kiểu.
   ```go
   type Number interface {
       int64 | float64
   }
   ```

   Trong mã này, bạn:

   - Khai báo kiểu interface `Number` để sử dụng như một ràng buộc kiểu.
   - Khai báo một union của `int64` và `float64` bên trong interface.

     Về cơ bản, bạn đang di chuyển union từ khai báo hàm vào một ràng buộc kiểu mới. Bằng cách đó, khi bạn muốn ràng buộc một tham số kiểu thành `int64` hoặc `float64`, bạn có thể sử dụng ràng buộc kiểu `Number` này thay vì viết ra `int64 | float64`.

2. Bên dưới các hàm bạn đã có, dán hàm generic sau.
   ```go
   // SumNumbers tính tổng các giá trị của map m. Nó hỗ trợ cả integers
   // và floats làm các giá trị của map.
   func SumNumbers[K comparable, V Number](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   Trong mã này, bạn:

   - Khai báo một hàm generic với cùng logic như hàm generic bạn đã khai báo trước đó, nhưng với interface kiểu mới thay vì union làm ràng buộc kiểu. Như trước, bạn sử dụng các tham số kiểu cho các kiểu đối số và trả về.

3. Trong main.go, bên dưới mã bạn đã có, dán mã sau.
   ```go
   fmt.Printf("Generic Sums with Constraint: %v and %v\n",
       SumNumbers(ints),
       SumNumbers(floats))
   ```

   Trong mã này, bạn:

   - Gọi `SumNumbers` với từng map, in tổng từ các giá trị của mỗi map.

     Như trong phần trước, bạn bỏ qua các đối số kiểu (tên kiểu trong dấu ngoặc vuông) trong lời gọi hàm generic. Trình biên dịch Go có thể suy ra đối số kiểu từ các đối số khác.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

## Kết luận

Làm tốt lắm! Bạn vừa giới thiệu bản thân với generics trong Go.

Các chủ đề được đề xuất tiếp theo:

- [Go Tour](https://go.dev/tour/) là một giới thiệu tuyệt vời từng bước về các cơ bản của Go.
- Bạn sẽ tìm thấy các thực hành tốt nhất hữu ích được mô tả trong [Effective Go](https://go.dev/doc/effective_go) và [Cách viết mã Go](https://go.dev/doc/code).

## Mã hoàn chỉnh

Bạn có thể chạy chương trình này trong [Go Playground](https://go.dev/play/p/apNmfVwogK0?v=gotip). Trên Playground chỉ cần nhấn nút **Run**.

```go
package main

import "fmt"

type Number interface {
    int64 | float64
}

func main() {
    // Khởi tạo một map cho các giá trị integer.
    ints := map[string]int64{
        "first":  34,
        "second": 12,
    }

    // Khởi tạo một map cho các giá trị float.
    floats := map[string]float64{
        "first":  35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))

    fmt.Printf("Generic Sums: %v and %v\n",
        SumIntsOrFloats[string, int64](ints),
        SumIntsOrFloats[string, float64](floats))

    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
        SumIntsOrFloats(ints),
        SumIntsOrFloats(floats))

    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
        SumNumbers(ints),
        SumNumbers(floats))
}

// SumInts cộng các giá trị của m với nhau.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats cộng các giá trị của m với nhau.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}

// SumIntsOrFloats tính tổng các giá trị của map m. Nó hỗ trợ cả int64 và float64
// làm kiểu cho các giá trị của map.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}

// SumNumbers tính tổng các giá trị của map m. Nó hỗ trợ cả integers
// và floats làm các giá trị của map.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/generics](https://go.dev/doc/tutorial/generics) để phục vụ cộng đồng lập trình viên Việt Nam.*
