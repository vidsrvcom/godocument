# Hướng dẫn: Tải xuống và cài đặt Go

Hướng dẫn này mô tả cách tải xuống và cài đặt Go.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Bước 1: Tải xuống Go

Truy cập [trang tải xuống Go](https://go.dev/dl/) và tải xuống phiên bản phù hợp với hệ điều hành của bạn.

- **Windows**: Tải file `.msi`
- **macOS**: Tải file `.pkg`
- **Linux**: Tải file `.tar.gz`

## Bước 2: Cài đặt Go

### Linux

1. Xóa bất kỳ cài đặt Go nào trước đó bằng cách xóa thư mục `/usr/local/go` (nếu nó tồn tại), sau đó giải nén archive bạn vừa tải xuống vào `/usr/local`, tạo một cây Go mới trong `/usr/local/go`:

   ```bash
   $ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.x.x.linux-amd64.tar.gz
   ```

   > **Lưu ý:** Bạn có thể cần chạy lệnh này với quyền root hoặc thông qua `sudo`.

   > **Quan trọng:** Không giải nén archive vào một cây `/usr/local/go` hiện có. Điều này được biết là tạo ra các cài đặt Go bị hỏng.

2. Thêm `/usr/local/go/bin` vào biến môi trường `PATH`.

   Bạn có thể làm điều này bằng cách thêm dòng sau vào `$HOME/.profile` hoặc `/etc/profile` (cho cài đặt trên toàn hệ thống):

   ```bash
   export PATH=$PATH:/usr/local/go/bin
   ```

   > **Lưu ý:** Các thay đổi được thực hiện cho một file profile có thể không áp dụng cho đến lần đăng nhập tiếp theo. Để áp dụng các thay đổi ngay lập tức, chỉ cần chạy các lệnh shell trực tiếp hoặc thực thi chúng từ profile sử dụng lệnh như `source $HOME/.profile`.

3. Xác minh rằng bạn đã cài đặt Go bằng cách mở một terminal và gõ lệnh sau:

   ```bash
   $ go version
   ```

4. Xác nhận rằng lệnh in phiên bản Go đã cài đặt.

### Mac

1. Mở file package bạn đã tải xuống và làm theo các hướng dẫn để cài đặt Go.

   Package cài đặt Go distribution vào `/usr/local/go`. Package sẽ đặt thư mục `/usr/local/go/bin` vào biến môi trường `PATH` của bạn. Bạn có thể cần khởi động lại bất kỳ phiên Terminal nào đang mở để thay đổi có hiệu lực.

2. Xác minh rằng bạn đã cài đặt Go bằng cách mở một terminal và gõ lệnh sau:

   ```bash
   $ go version
   ```

3. Xác nhận rằng lệnh in phiên bản Go đã cài đặt.

### Windows

1. Mở file MSI bạn đã tải xuống và làm theo các hướng dẫn để cài đặt Go.

   Theo mặc định, installer sẽ cài đặt Go vào `Program Files` hoặc `Program Files (x86)`. Bạn có thể thay đổi vị trí nếu cần. Sau khi cài đặt, bạn sẽ cần đóng và mở lại bất kỳ dấu nhắc lệnh nào đang mở để các thay đổi đối với môi trường được phản ánh tại dấu nhắc lệnh.

2. Xác minh rằng bạn đã cài đặt Go.

   1. Trong Windows, click vào menu Start.
   2. Trong hộp tìm kiếm của menu, gõ `cmd`, sau đó nhấn phím Enter.
   3. Trong cửa sổ Command Prompt xuất hiện, gõ lệnh sau:

      ```bash
      $ go version
      ```

3. Xác nhận rằng lệnh in phiên bản Go đã cài đặt.

## Bước 3: Cấu hình môi trường (Tùy chọn)

Sau khi cài đặt Go, bạn có thể muốn cấu hình một số biến môi trường:

- **GOPATH**: Thư mục workspace của Go (mặc định là `$HOME/go` hoặc `%USERPROFILE%\go`)
- **GOBIN**: Thư mục nơi các binary được cài đặt (mặc định là `$GOPATH/bin`)

## Bước tiếp theo

Với Go đã được cài đặt, bạn có thể bắt đầu viết mã Go! Xem [Hướng dẫn: Bắt đầu với Go](getting-started.md) để tạo chương trình Go đầu tiên của bạn.

---

*Tài liệu này được dịch từ [go.dev/doc/install](https://go.dev/doc/install) để phục vụ cộng đồng lập trình viên Việt Nam.*
