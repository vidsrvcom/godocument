# Hướng dẫn: Bắt đầu với fuzzing

Bảng nội dung:
- [Yêu cầu tiên quyết](#yêu-cầu-tiên-quyết)
- [Tạo thư mục cho mã của bạn](#tạo-thư-mục-cho-mã-của-bạn)
- [Thêm mã để test](#thêm-mã-để-test)
- [Thêm một unit test](#thêm-một-unit-test)
- [Thêm một fuzz test](#thêm-một-fuzz-test)
- [Sửa lỗi không hợp lệ](#sửa-lỗi-không-hợp-lệ)
- [Sửa lỗi double-reverse](#sửa-lỗi-double-reverse)
- [Kết luận](#kết-luận)

Hướng dẫn này giới thiệu về cơ bản của fuzzing trong Go. Với fuzzing, các đầu vào ngẫu nhiên được chạy qua các test của bạn để cố gắng tìm các lỗ hổng hoặc đầu vào gây crash. Một số ví dụ về các lỗ hổng có thể được tìm thấy thông qua fuzzing là SQL injection, buffer overflow, denial of service và các cuộc tấn công cross-site scripting.

Trong hướng dẫn này, bạn sẽ viết một fuzz test cho một hàm đơn giản, chạy lệnh go, và debug và sửa các vấn đề trong mã.

Để được giúp đỡ với thuật ngữ trong suốt hướng dẫn này, xem [Tổng quan về Fuzzing trong Go](https://go.dev/security/fuzz/).

Bạn sẽ tiến triển qua các phần sau:

1. Tạo một thư mục cho mã của bạn.
2. Thêm mã để test.
3. Thêm một unit test.
4. Thêm một fuzz test.
5. Sửa hai lỗi.
6. Khám phá các tài nguyên bổ sung.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

> **Lưu ý:** Fuzzing được hỗ trợ bắt đầu từ Go 1.18, và phải được xây dựng với cờ coverage instrumentation. Để tải phiên bản Go mới nhất, xem [Tải xuống và cài đặt](https://go.dev/doc/install).

## Yêu cầu tiên quyết

- **Một bản cài đặt Go 1.18 trở lên.** Để biết hướng dẫn cài đặt, xem [Cài đặt Go](https://go.dev/doc/install).
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.
- **Một môi trường hỗ trợ fuzzing.** Fuzzing với Go coverage instrumentation hiện chỉ có sẵn trên AMD64 và ARM64 architectures.

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

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn gọi là fuzz.
   ```bash
   $ mkdir fuzz
   $ cd fuzz
   ```

3. Tạo một module để chứa mã của bạn.

   Chạy lệnh `go mod init`, đặt cho nó đường dẫn module cho module mới của bạn.
   ```bash
   $ go mod init example/fuzz
   go: creating new go.mod: module example/fuzz
   ```

   > **Lưu ý:** Đối với mã sản xuất, bạn sẽ xác định một đường dẫn module cụ thể hơn cho nhu cầu của riêng bạn. Để biết thêm, hãy chắc chắn xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một số mã đơn giản để đảo ngược một chuỗi, mà chúng tôi sẽ fuzz sau này.

## Thêm mã để test

Trong bước này, bạn sẽ thêm một hàm để đảo ngược một chuỗi.

### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, trong thư mục fuzz tạo một file gọi là main.go.

2. Vào file main.go, ở đầu file, dán khai báo package sau.
   ```go
   package main
   ```

3. Bên dưới khai báo package, dán khai báo hàm sau.
   ```go
   func Reverse(s string) string {
       b := []byte(s)
       for i, j := 0, len(b)-1; i < len(b)/2; i, j = i+1, j-1 {
           b[i], b[j] = b[j], b[i]
       }
       return string(b)
   }
   ```

   Hàm này sẽ nhận một `string`, lặp qua nó một byte tại một thời điểm, và trả về chuỗi đã đảo ngược ở cuối.

   > **Lưu ý:** Mã này dựa trên hàm `stringutil.Reverse` từ `golang.org/x/example`.

4. Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo một chuỗi, đảo ngược nó, in đầu ra, và lặp lại.
   ```go
   func main() {
       input := "The quick brown fox jumped over the lazy dog"
       rev := Reverse(input)
       doubleRev := Reverse(rev)
       fmt.Printf("original: %q\n", input)
       fmt.Printf("reversed: %q\n", rev)
       fmt.Printf("reversed again: %q\n", doubleRev)
   }
   ```

   Hàm này sẽ chạy một vài vòng Reverse, sau đó in đầu ra ra dòng lệnh.

5. Hàm `main` sử dụng package fmt, vì vậy bạn sẽ cần import nó.

   Các dòng mã đầu tiên sẽ trông như thế này:
   ```go
   package main

   import "fmt"
   ```

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
original: "The quick brown fox jumped over the lazy dog"
reversed: "god yzal eht revo depmuj xof nworb kciuq ehT"
reversed again: "The quick brown fox jumped over the lazy dog"
```

Bạn có thể thấy rằng đảo ngược chuỗi ban đầu, sau đó đảo ngược nó lần nữa, tạo ra đầu ra ban đầu.

Bây giờ khi mã đang chạy, đã đến lúc test nó.

## Thêm một unit test

Trong bước này, bạn sẽ viết một unit test cơ bản cho hàm `Reverse`.

### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, trong thư mục fuzz tạo một file gọi là reverse_test.go.

2. Dán mã sau vào reverse_test.go.
   ```go
   package main

   import (
       "testing"
   )

   func TestReverse(t *testing.T) {
       testcases := []struct {
           in, want string
       }{
           {"Hello, world", "dlrow ,olleH"},
           {" ", " "},
           {"!12345", "54321!"},
       }
       for _, tc := range testcases {
           rev := Reverse(tc.in)
           if rev != tc.want {
               t.Errorf("Reverse: %q, want %q", rev, tc.want)
           }
       }
   }
   ```

   Unit test đơn giản này sẽ test một tập hợp các đầu vào testcase được mã hóa cứng.

### Chạy mã

Chạy unit test bằng `go test`
```bash
$ go test
PASS
ok      example/fuzz  0.013s
```

Tiếp theo, bạn sẽ thay đổi unit test thành một fuzz test.

## Thêm một fuzz test

Unit test có những hạn chế, cụ thể là mỗi đầu vào phải được thêm vào test bởi nhà phát triển. Một lợi ích của fuzzing là nó đưa ra các đầu vào cho mã của bạn, và có thể xác định các trường hợp biên mà các trường hợp test bạn đưa ra không đạt được.

Trong phần này bạn sẽ chuyển đổi unit test thành một fuzz test để bạn có thể tạo ra nhiều đầu vào hơn với ít công việc hơn!

Lưu ý rằng bạn có thể giữ unit tests, benchmarks, và fuzz tests trong cùng file *_test.go, nhưng đối với ví dụ này, bạn sẽ chuyển đổi unit test thành một fuzz test.

### Viết mã

Trong trình soạn thảo văn bản của bạn, thay thế unit test trong reverse_test.go bằng fuzz test sau.
```go
package main

import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Sử dụng f.Add để cung cấp một seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

Fuzzing cũng có một số hạn chế. Trong unit test của bạn, bạn có thể dự đoán đầu ra mong đợi của hàm `Reverse`, và xác minh rằng đầu ra thực tế đáp ứng các kỳ vọng đó.

Ví dụ, trong trường hợp test `Reverse("Hello, world")` unit test xác định return là `"dlrow ,olleH"`.

Khi fuzzing, bạn không thể dự đoán đầu ra mong đợi, vì bạn không có quyền kiểm soát các đầu vào.

Tuy nhiên, có một vài thuộc tính của hàm `Reverse` mà bạn có thể xác minh trong một fuzz test. Hai thuộc tính được kiểm tra trong fuzz test này là:

1. Đảo ngược một chuỗi hai lần giữ nguyên giá trị ban đầu
2. Chuỗi đã đảo ngược giữ nguyên trạng thái là UTF-8 hợp lệ.

Lưu ý sự khác biệt về cú pháp giữa unit test và fuzz test:

- Hàm bắt đầu với `FuzzXxx` thay vì `TestXxx`, và nhận `*testing.F` thay vì `*testing.T`
- Nơi bạn mong đợi thấy một xác nhận `t.Run`, bạn sẽ thấy `f.Fuzz` thay vì, nó nhận một fuzz target function có các tham số là `*testing.T` và các kiểu được fuzz. Các đầu vào từ unit test của bạn được cung cấp như seed corpus inputs bằng cách sử dụng `f.Add`.

Đảm bảo package mới `unicode/utf8` đã được import.

Với unit test đã được chuyển đổi thành một fuzz test, đã đến lúc chạy test lần nữa.

### Chạy mã

1. Chạy fuzz test mà không fuzzing nó để đảm bảo rằng các seed inputs pass.
   ```bash
   $ go test
   PASS
   ok      example/fuzz  0.013s
   ```

   Nếu bạn có các files khác trong thư mục đó, bạn cũng có thể chỉ định để chạy fuzz test, bằng `go test -run=FuzzReverse`.

2. Chạy `FuzzReverse` với fuzzing, để xem liệu bất kỳ đầu vào nào được tạo ngẫu nhiên có gây ra lỗi hay không. Điều này được thực thi sử dụng `go test` với một cờ mới, `-fuzz`, được đặt thành tham số `Fuzz`. Sao chép lệnh dưới đây.
   ```bash
   $ go test -fuzz=Fuzz
   ```

   Một lỗi xảy ra!
   ```
   fuzz: elapsed: 0s, gathering baseline coverage: 0/3 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 3/3 completed, now fuzzing with 8 workers
   fuzz: minimizing 38-byte failing input file...
   --- FAIL: FuzzReverse (0.01s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:20: Reverse produced invalid UTF-8 string "\x9c\xc5"

       Failing input written to testdata/fuzz/FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
       To re-run:
       go test -run=FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
   FAIL
   exit status 1
   FAIL    example/fuzz  0.030s
   ```

   Một lỗi xảy ra trong khi fuzzing, và đầu vào gây ra vấn đề được viết vào một file seed corpus sẽ được chạy lần sau `go test` được gọi, ngay cả khi không có cờ `-fuzz`. Để xem đầu vào gây ra lỗi, mở file corpus được viết vào thư mục testdata/fuzz/FuzzReverse trong một trình soạn thảo văn bản. File seed corpus của bạn có thể chứa một chuỗi khác, nhưng định dạng sẽ giống nhau.
   ```
   go test fuzz v1
   string("泃")
   ```

   Dòng đầu tiên của file corpus cho biết phiên bản encoding. Mỗi dòng sau cung cấp các giá trị của từng loại tạo nên corpus entry. Vì fuzz target chỉ nhận 1 đầu vào, chỉ có 1 giá trị sau phiên bản.

3. Chạy `go test` lần nữa mà không có cờ `-fuzz`; file seed corpus mới sẽ được sử dụng:
   ```bash
   $ go test
   --- FAIL: FuzzReverse (0.00s)
       --- FAIL: FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a (0.00s)
           reverse_test.go:20: Reverse produced invalid UTF-8 string "\x9c\xc5"
   FAIL
   exit status 1
   FAIL    example/fuzz  0.016s
   ```

   Vì test của chúng tôi đã thất bại, đã đến lúc debug.

## Sửa lỗi không hợp lệ

Trong phần này, bạn sẽ debug lỗi và sửa nó.

Hãy thoải mái dành một chút thời gian để suy nghĩ về điều này và thử tự sửa vấn đề trước khi tiếp tục.

### Chẩn đoán lỗi

Có một vài cách khác nhau để debug lỗi này. Nếu bạn đang sử dụng VS Code làm trình soạn thảo văn bản của mình, bạn có thể [thiết lập debugger của bạn](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) để điều tra.

Trong hướng dẫn này, bạn sẽ log thông tin hữu ích ra terminal.

Trước tiên, hãy xem xét các tài liệu cho [`utf8.ValidString`](https://pkg.go.dev/unicode/utf8#ValidString).

```
ValidString reports whether s consists entirely of valid UTF-8-encoded runes.
```

Hàm `Reverse` hiện tại đảo ngược chuỗi byte-by-byte, và đó là vấn đề của chúng tôi. Để bảo tồn các runes được mã hóa UTF-8 của chuỗi ban đầu, chúng ta phải thay vào đó đảo ngược chuỗi rune-by-rune.

Để kiểm tra tại sao đầu vào (trong trường hợp này, ký tự Trung Quốc `泃`) gây ra `Reverse` tạo ra một chuỗi UTF-8 không hợp lệ khi đảo ngược, bạn có thể kiểm tra số lượng runes trong chuỗi đã đảo ngược.

### Viết mã

1. Trong trình soạn thảo văn bản của bạn, thay thế fuzz target trong `FuzzReverse` bằng những gì sau đây.
   ```go
   f.Fuzz(func(t *testing.T, orig string) {
       rev := Reverse(orig)
       doubleRev := Reverse(rev)
       t.Logf("Number of runes: orig=%d, rev=%d, doubleRev=%d", utf8.RuneCountInString(orig), utf8.RuneCountInString(rev), utf8.RuneCountInString(doubleRev))
       if orig != doubleRev {
           t.Errorf("Before: %q, after: %q", orig, doubleRev)
       }
       if utf8.ValidString(orig) && !utf8.ValidString(rev) {
           t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
       }
   })
   ```

   Câu lệnh `t.Logf` này sẽ in ra console nếu lỗi xảy ra, hoặc nếu thực thi test với `-v`, có thể giúp bạn debug vấn đề cụ thể này.

### Chạy mã

Chạy test bằng `go test`
```bash
$ go test
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6b1e6c8a0bdc0e37ecee (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=3, doubleRev=1
        reverse_test.go:21: Reverse produced invalid UTF-8 string "\x9c\xc5"
FAIL
exit status 1
FAIL    example/fuzz  0.025s
```

Toàn bộ seed corpus được sử dụng làm đầu vào mỗi khi `go test` được chạy, bao gồm corpus entry ban đầu, và các entries thất bại đã được lưu trữ trong testdata/fuzz.

Để chỉ chạy failing input cho debugging, cung cấp corpus entry cho go test với cờ `-run`.
```bash
$ go test -run=FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6b1e6c8a0bdc0e37ecee
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6b1e6c8a0bdc0e37ecee (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=3, doubleRev=1
        reverse_test.go:21: Reverse produced invalid UTF-8 string "\x9c\xc5"
FAIL
exit status 1
FAIL    example/fuzz  0.016s
```

Biết rằng đầu vào là một ký tự duy nhất, nhưng việc đảo ngược đã tạo ra một chuỗi với 3 byte. Điều này xác nhận rằng chúng ta đang đảo ngược chuỗi byte-by-byte thay vì rune-by-rune.

### Sửa lỗi

Để sửa lỗi này, hãy đi qua chuỗi theo runes, thay vì theo byte.

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse()` hiện có bằng những gì sau đây.
   ```go
   func Reverse(s string) string {
       r := []rune(s)
       for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r)
   }
   ```

   Thay đổi chính là `Reverse` bây giờ đang lặp qua từng rune trong chuỗi, thay vì từng byte.

### Chạy mã

1. Chạy test bằng `go test`
   ```bash
   $ go test
   PASS
   ok      example/fuzz  0.016s
   ```

   Test bây giờ pass!

2. Fuzz nó lại với `go test -fuzz=Fuzz` để xem liệu có bất kỳ lỗi mới nào không.
   ```bash
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/37 completed
   fuzz: minimizing 506-byte failing input file...
   fuzz: elapsed: 0s, gathering baseline coverage: 5/37 completed
   --- FAIL: FuzzReverse (0.02s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:33: Before: "\x91", after: "�"

       Failing input written to testdata/fuzz/FuzzReverse/1ffc28f7538e29571f7c5c61f5c8851d5
       To re-run:
       go test -run=FuzzReverse/1ffc28f7538e29571f7c5c61f5c8851d5
   FAIL
   exit status 1
   FAIL    example/fuzz  0.032s
   ```

   Chúng ta có thể thấy rằng chuỗi khác với chuỗi ban đầu sau khi đảo ngược nó hai lần. Lần này đầu vào tự nó là Unicode không hợp lệ. Làm thế nào điều này có thể xảy ra nếu chúng ta đang fuzzing với strings?

   Hãy debug lần nữa.

## Sửa lỗi double-reverse

Trong phần này, bạn sẽ debug lỗi double-reverse, và sửa nó.

Hãy thoải mái dành một chút thời gian để suy nghĩ về điều này và thử tự sửa vấn đề trước khi tiếp tục.

### Chẩn đoán lỗi

Như trước, có một vài cách bạn có thể debug lỗi này. Trong trường hợp này, sử dụng một debugger sẽ là một cách tiếp cận tốt.

Trong hướng dẫn này, bạn sẽ log thông tin hữu ích trong hàm `Reverse`.

Nhìn cẩn thận vào chuỗi đã đảo ngược để phát hiện lỗi. Trong Go, [một chuỗi là một slice bytes chỉ đọc](https://go.dev/blog/strings), và có thể chứa các bytes không phải UTF-8 hợp lệ. Chuỗi ban đầu là một byte slice với một byte duy nhất, `'\x91'`. Khi chuỗi đầu vào được đặt thành `[]rune`, Go mã hóa byte slice thành UTF-8, và thay thế byte bằng ký tự rune UTF-8, `�`. Khi chúng ta so sánh chuỗi UTF-8 thay thế với byte slice đầu vào, chúng rõ ràng không bằng nhau.

### Viết mã

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse` bằng những gì sau đây.
   ```go
   func Reverse(s string) string {
       fmt.Printf("input: %q\n", s)
       r := []rune(s)
       fmt.Printf("runes: %q\n", r)
       for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r)
   }
   ```

   Điều này sẽ giúp chúng ta hiểu những gì đang xảy ra bên trong khi chuyển đổi chuỗi thành một slice của runes.

### Chạy mã

Lần này, chúng ta chỉ muốn chạy failing test để kiểm tra các logs. Để làm điều này, chúng ta sẽ sử dụng `go test -run`.
```bash
$ go test -run=FuzzReverse/1ffc28f7538e29571f7c5c61f5c8851d5
input: "\x91"
runes: ['�']
input: "�"
runes: ['�']
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/1ffc28f7538e29571f7c5c61f5c8851d5 (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=1, doubleRev=1
        reverse_test.go:18: Before: "\x91", after: "�"
FAIL
exit status 1
FAIL    example/fuzz  0.016s
```

Để chạy failing corpus entry cụ thể, bạn có thể cung cấp `{FuzzTestName}/{filename}` cho `-run`. Điều này có thể hữu ích khi debugging.

Biết rằng đầu vào là Unicode không hợp lệ, hãy sửa lỗi trong hàm `Reverse` của chúng ta.

### Sửa lỗi

Để sửa vấn đề này, hãy trả về một lỗi nếu đầu vào cho `Reverse` không phải là UTF-8 hợp lệ.

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse` hiện có bằng những gì sau đây.
   ```go
   func Reverse(s string) (string, error) {
       if !utf8.ValidString(s) {
           return s, errors.New("input is not valid UTF-8")
       }
       r := []rune(s)
       for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r), nil
   }
   ```

   Thay đổi này sẽ trả về một lỗi nếu chuỗi đầu vào chứa các ký tự không phải UTF-8 hợp lệ.

2. Vì hàm `Reverse` bây giờ trả về một lỗi, sửa đổi hàm `main` để loại bỏ thông báo lỗi bổ sung.

   Hàm `main` hiện có nên trông như sau:
   ```go
   func main() {
       input := "The quick brown fox jumped over the lazy dog"
       rev, revErr := Reverse(input)
       doubleRev, doubleRevErr := Reverse(rev)
       fmt.Printf("original: %q\n", input)
       fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
       fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
   }
   ```

3. Bạn sẽ cần import các packages errors và unicode/utf8. Câu lệnh import trong main.go nên trông như sau.
   ```go
   import (
       "errors"
       "fmt"
       "unicode/utf8"
   )
   ```

4. Sửa đổi file reverse_test.go để kiểm tra lỗi và bỏ qua test nếu lỗi được tạo ra khi đảo ngược.
   ```go
   func FuzzReverse(f *testing.F) {
       testcases := []string{"Hello, world", " ", "!12345"}
       for _, tc := range testcases {
           f.Add(tc)  // Sử dụng f.Add để cung cấp một seed corpus
       }
       f.Fuzz(func(t *testing.T, orig string) {
           rev, err1 := Reverse(orig)
           if err1 != nil {
               return
           }
           doubleRev, err2 := Reverse(rev)
           if err2 != nil {
               return
           }
           if orig != doubleRev {
               t.Errorf("Before: %q, after: %q", orig, doubleRev)
           }
           if utf8.ValidString(orig) && !utf8.ValidString(rev) {
               t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
           }
       })
   }
   ```

   Bạn cũng có thể gọi `t.Skip()` thay vì return ở đây.

### Chạy mã

1. Chạy test sử dụng `go test`
   ```bash
   $ go test
   PASS
   ok      example/fuzz  0.019s
   ```

2. Fuzz với `go test -fuzz=Fuzz`, sau đó sau một vài giây pass, dừng fuzzing với `ctrl-C`. Fuzzing sẽ pass trừ khi bạn gặp một lỗi do timeout hoặc giới hạn bộ nhớ.
   ```bash
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/38 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 38/38 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 86342 (28778/sec), new interesting: 2 (total: 35)
   fuzz: elapsed: 6s, execs: 193490 (35714/sec), new interesting: 4 (total: 37)
   fuzz: elapsed: 9s, execs: 304390 (36961/sec), new interesting: 4 (total: 37)
   ...
   fuzz: elapsed: 3m45s, execs: 7246222 (32357/sec), new interesting: 8 (total: 41)
   ^Cfuzz: elapsed: 3m48s, execs: 7335316 (31648/sec), new interesting: 8 (total: 41)
   PASS
   ok      example/fuzz  228.000s
   ```

   Fuzz test sẽ chạy mãi mãi trừ khi nó thất bại hoặc bạn dừng test. Cờ `-fuzztime` hữu ích nếu bạn muốn đặt một khoảng thời gian cho fuzzing.

3. Fuzz với `go test -fuzz=Fuzz -fuzztime 30s` sẽ fuzz trong 30 giây trước khi dừng nếu không có thất bại nào được tìm thấy trước đó.
   ```bash
   $ go test -fuzz=Fuzz -fuzztime 30s
   fuzz: elapsed: 0s, gathering baseline coverage: 0/5 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 5/5 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 80290 (26763/sec), new interesting: 12 (total: 12)
   fuzz: elapsed: 6s, execs: 210803 (43501/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 9s, execs: 292882 (27360/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 12s, execs: 371872 (26329/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 15s, execs: 517169 (48433/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 18s, execs: 663276 (48699/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 21s, execs: 771698 (36143/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 24s, execs: 924768 (50990/sec), new interesting: 16 (total: 16)
   fuzz: elapsed: 27s, execs: 1082025 (52427/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 30s, execs: 1172817 (30281/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 31s, execs: 1172817 (0/sec), new interesting: 17 (total: 17)
   PASS
   ok      example/fuzz  31.025s
   ```

   Fuzzing đã pass!

Ngoài cờ `-fuzz`, một số cờ mới đã được thêm vào `go test` để được sử dụng khi fuzzing:

- `-fuzztime`: tổng thời gian hoặc số lần lặp lại để fuzz target sẽ được thực thi trước khi thoát, mặc định là vô hạn.
- `-fuzzminimizetime`: thời gian hoặc số lần lặp lại để fuzz target sẽ được thực thi trong mỗi nỗ lực minimize, mặc định là 60 giây. Bạn có thể hoàn toàn tắt việc minimize bằng cách đặt `-fuzzminimizetime 0` khi fuzzing.
- `-parallel`: số lượng fuzzing processes chạy cùng một lúc, mặc định là `$GOMAXPROCS`. Hiện tại, việc đặt `-cpu` trong khi fuzzing không có tác dụng.

## Kết luận

Tốt lắm! Bạn vừa giới thiệu bản thân với fuzzing trong Go.

Bước tiếp theo là chọn một hàm trong mã của bạn mà bạn muốn fuzz, và thử nó! Nếu fuzzing tìm thấy một lỗi trong mã của bạn, hãy xem xét thêm nó vào [trophy case](https://github.com/golang/go/wiki/Fuzzing-trophy-case).

Nếu bạn gặp bất kỳ vấn đề nào hoặc có một ý tưởng cho một tính năng, [file một issue](https://github.com/golang/go/issues/new/?&labels=fuzz).

Để thảo luận và nhận phản hồi chung về tính năng, bạn cũng có thể tham gia [#fuzzing channel trong Gophers Slack](https://invite.slack.golangbridge.org/).

Kiểm tra tài liệu tại [go.dev/security/fuzz](https://go.dev/security/fuzz) để đọc thêm.

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/fuzz](https://go.dev/doc/tutorial/fuzz) để phục vụ cộng đồng lập trình viên Việt Nam.*
