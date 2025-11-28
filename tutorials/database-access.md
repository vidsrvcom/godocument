# Hướng dẫn: Truy cập cơ sở dữ liệu quan hệ

Bảng nội dung:
- [Yêu cầu tiên quyết](#yêu-cầu-tiên-quyết)
- [Tạo một thư mục cho mã của bạn](#tạo-một-thư-mục-cho-mã-của-bạn)
- [Thiết lập cơ sở dữ liệu](#thiết-lập-cơ-sở-dữ-liệu)
- [Tìm và import một driver cơ sở dữ liệu](#tìm-và-import-một-driver-cơ-sở-dữ-liệu)
- [Lấy một database handle và kết nối](#lấy-một-database-handle-và-kết-nối)
- [Truy vấn cho nhiều hàng](#truy-vấn-cho-nhiều-hàng)
- [Truy vấn cho một hàng đơn](#truy-vấn-cho-một-hàng-đơn)
- [Thêm dữ liệu](#thêm-dữ-liệu)
- [Kết luận](#kết-luận)

Hướng dẫn này giới thiệu về cơ bản của việc truy cập một cơ sở dữ liệu quan hệ với Go và package [`database/sql`](https://pkg.go.dev/database/sql) trong thư viện chuẩn của nó.

Bạn sẽ thu được nhiều nhất từ hướng dẫn này nếu bạn có một sự quen thuộc cơ bản với Go và các công cụ của nó. Nếu đây là lần đầu tiên bạn tiếp xúc với Go, vui lòng xem [Hướng dẫn: Bắt đầu với Go](getting-started.md) để biết giới thiệu nhanh.

Package `database/sql` bạn sẽ sử dụng bao gồm các kiểu và hàm để kết nối đến cơ sở dữ liệu, thực thi giao dịch, hủy một hoạt động đang tiến hành, và nhiều hơn nữa. Để biết thêm chi tiết về cách sử dụng package, xem [Truy cập cơ sở dữ liệu quan hệ](https://go.dev/doc/database/index).

Trong hướng dẫn này, bạn sẽ tạo một cơ sở dữ liệu, sau đó viết mã để truy cập cơ sở dữ liệu. Dự án ví dụ của bạn sẽ là một kho lưu trữ dữ liệu về các bản ghi nhạc jazz vintage.

Trong hướng dẫn này, bạn sẽ tiến triển qua các phần sau:

1. Tạo một thư mục cho mã của bạn.
2. Thiết lập một cơ sở dữ liệu.
3. Import driver cơ sở dữ liệu.
4. Lấy một database handle và kết nối.
5. Truy vấn cho nhiều hàng.
6. Truy vấn cho một hàng đơn.
7. Thêm dữ liệu.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

## Yêu cầu tiên quyết

- **Một bản cài đặt của [MySQL](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/)** hệ thống quản lý cơ sở dữ liệu quan hệ (DBMS).
- **Một bản cài đặt Go.** Để biết hướng dẫn cài đặt, xem [Cài đặt Go](https://go.dev/doc/install).
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.

## Tạo một thư mục cho mã của bạn

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

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn gọi là data-access.
   ```bash
   mkdir data-access
   cd data-access
   ```

3. Tạo một module trong đó bạn có thể quản lý các phụ thuộc bạn sẽ thêm trong hướng dẫn này.

   Chạy lệnh `go mod init`, đặt cho nó đường dẫn module của bạn -- ở đây, sử dụng `example/data-access`.
   ```bash
   $ go mod init example/data-access
   go: creating new go.mod: module example/data-access
   ```

   Lệnh này tạo một file go.mod trong đó các phụ thuộc bạn thêm vào sẽ được liệt kê để theo dõi. Để biết thêm, hãy xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies).

   > **Lưu ý:** Trong phát triển thực tế, bạn sẽ xác định một đường dẫn module cụ thể hơn cho nhu cầu của riêng bạn. Để biết thêm, xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ tạo một cơ sở dữ liệu.

## Thiết lập cơ sở dữ liệu

Trong bước này, bạn sẽ tạo cơ sở dữ liệu mà bạn sẽ làm việc. Bạn sẽ sử dụng CLI cho chính DBMS để tạo cơ sở dữ liệu và bảng, cũng như để thêm dữ liệu.

Bạn sẽ tạo một cơ sở dữ liệu với dữ liệu về các bản ghi vinyl trên các đĩa nhạc cổ điển jazz.

Mã ở đây sử dụng [MySQL CLI](https://dev.mysql.com/doc/refman/8.0/en/mysql.html), nhưng hầu hết các DBMS đều có CLI riêng với các tính năng tương tự.

1. Mở một dấu nhắc lệnh mới.

2. Tại dòng lệnh, đăng nhập vào DBMS của bạn, như trong ví dụ sau cho MySQL.
   ```bash
   $ mysql -u root -p
   Enter password:

   mysql>
   ```

3. Tại dấu nhắc `mysql`, tạo một cơ sở dữ liệu.
   ```sql
   mysql> create database recordings;
   ```

4. Thay đổi sang cơ sở dữ liệu bạn vừa tạo để bạn có thể thêm bảng.
   ```sql
   mysql> use recordings;
   Database changed
   ```

5. Trong trình soạn thảo văn bản của bạn, trong thư mục data-access, tạo một file gọi là create-tables.sql để giữ script SQL để thêm bảng.

6. Vào file, dán script SQL sau, sau đó lưu file.
   ```sql
   DROP TABLE IF EXISTS album;
   CREATE TABLE album (
     id         INT AUTO_INCREMENT NOT NULL,
     title      VARCHAR(128) NOT NULL,
     artist     VARCHAR(255) NOT NULL,
     price      DECIMAL(5,2) NOT NULL,
     PRIMARY KEY (`id`)
   );

   INSERT INTO album
     (title, artist, price)
   VALUES
     ('Blue Train', 'John Coltrane', 56.99),
     ('Giant Steps', 'John Coltrane', 63.99),
     ('Jeru', 'Gerry Mulligan', 17.99),
     ('Sarah Vaughan', 'Sarah Vaughan', 34.98);
   ```

   Trong script SQL này, bạn:

   - Xóa (drop) một bảng gọi là `album`. Thực thi lệnh này trước cho phép bạn chạy lại script dễ dàng hơn sau này nếu bạn muốn bắt đầu lại với bảng.
   - Tạo một bảng `album` với bốn cột: `title`, `artist`, và `price`. Mỗi hàng có một `id` là một số tự động tạo để tăng dần.
   - Thêm bốn hàng với các giá trị.

7. Từ dấu nhắc lệnh `mysql`, chạy script bạn vừa tạo.

   Bạn sẽ sử dụng lệnh `source` theo dạng sau:
   ```bash
   mysql> source /path/to/create-tables.sql
   ```

8. Tại dấu nhắc lệnh DBMS của bạn, sử dụng một câu lệnh `SELECT` để xác minh bạn đã tạo thành công bảng với dữ liệu.
   ```sql
   mysql> select * from album;
   +----+---------------+----------------+-------+
   | id | title         | artist         | price |
   +----+---------------+----------------+-------+
   |  1 | Blue Train    | John Coltrane  | 56.99 |
   |  2 | Giant Steps   | John Coltrane  | 63.99 |
   |  3 | Jeru          | Gerry Mulligan | 17.99 |
   |  4 | Sarah Vaughan | Sarah Vaughan  | 34.98 |
   +----+---------------+----------------+-------+
   4 rows in set (0.00 sec)
   ```

Tiếp theo, bạn sẽ viết một số mã Go để kết nối.

## Tìm và import một driver cơ sở dữ liệu

Bây giờ bạn đã có một cơ sở dữ liệu với một số dữ liệu, hãy bắt đầu mã Go của bạn.

Tìm và import một driver cơ sở dữ liệu sẽ dịch các yêu cầu bạn thực hiện thông qua các hàm trong package `database/sql` thành các yêu cầu mà cơ sở dữ liệu hiểu.

1. Trong trình duyệt của bạn, truy cập [trang SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers) để xác định một driver bạn có thể sử dụng.

   Sử dụng danh sách trên trang để xác định driver bạn sẽ sử dụng. Để truy cập MySQL trong hướng dẫn này, bạn sẽ sử dụng [Go-MySQL-Driver](https://github.com/go-sql-driver/mysql/).

2. Lưu ý tên package cho driver -- ở đây, `github.com/go-sql-driver/mysql`.

3. Sử dụng trình soạn thảo văn bản của bạn, tạo một file trong đó để viết mã Go của bạn và lưu file là main.go trong thư mục data-access bạn đã tạo trước đó.

4. Vào main.go, dán mã sau để import package driver.
   ```go
   package main

   import "github.com/go-sql-driver/mysql"
   ```

   Trong mã này, bạn:

   - Thêm mã của bạn vào một package `main` để bạn có thể thực thi nó độc lập.
   - Import driver MySQL `github.com/go-sql-driver/mysql`.

Với driver đã được import, bạn sẽ bắt đầu viết mã để truy cập cơ sở dữ liệu.

## Lấy một database handle và kết nối

Bây giờ viết một số mã Go cho phép bạn truy cập cơ sở dữ liệu với một database handle.

Bạn sẽ sử dụng một con trỏ đến một [`sql.DB`](https://pkg.go.dev/database/sql#DB) struct, đại diện cho quyền truy cập vào một cơ sở dữ liệu cụ thể.

### Viết mã

1. Vào main.go, bên dưới mã `import` bạn vừa thêm, dán mã Go sau để tạo một database handle.
   ```go
   var db *sql.DB

   func main() {
       // Nắm bắt các thuộc tính kết nối.
       cfg := mysql.Config{
           User:   os.Getenv("DBUSER"),
           Passwd: os.Getenv("DBPASS"),
           Net:    "tcp",
           Addr:   "127.0.0.1:3306",
           DBName: "recordings",
       }
       // Lấy một database handle.
       var err error
       db, err = sql.Open("mysql", cfg.FormatDSN())
       if err != nil {
           log.Fatal(err)
       }

       pingErr := db.Ping()
       if pingErr != nil {
           log.Fatal(pingErr)
       }
       fmt.Println("Connected!")
   }
   ```

   Trong mã này, bạn:

   - Khai báo một biến `db` kiểu `*sql.DB`. Đây là database handle của bạn.

     Làm cho `db` một biến toàn cục đơn giản hóa ví dụ này. Trong sản xuất, bạn sẽ tránh biến toàn cục, chẳng hạn như bằng cách truyền biến cho các hàm cần nó hoặc bao bọc nó trong một struct.

   - Sử dụng [`Config`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config) của driver MySQL -- và phương thức [`FormatDSN`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config.FormatDSN) của kiểu -- để thu thập các thuộc tính kết nối và định dạng chúng thành một DSN cho một chuỗi kết nối.

     Kiểu `Config` giúp mã dễ đọc hơn so với một chuỗi kết nối.

   - Gọi [`sql.Open`](https://pkg.go.dev/database/sql#Open) để khởi tạo biến `db`, truyền giá trị trả về của `FormatDSN`.

   - Kiểm tra một lỗi từ `sql.Open`. Nó có thể thất bại nếu, ví dụ, các chi tiết kết nối cơ sở dữ liệu của bạn không được định dạng đúng.

     Để đơn giản hóa mã, bạn đang gọi `log.Fatal` để kết thúc thực thi và in lỗi ra console. Trong mã sản xuất, bạn sẽ muốn xử lý lỗi một cách tinh tế hơn.

   - Gọi [`DB.Ping`](https://pkg.go.dev/database/sql#DB.Ping) để xác nhận rằng kết nối đến cơ sở dữ liệu hoạt động. Tại thời gian chạy, `sql.Open` có thể không kết nối ngay lập tức, phụ thuộc vào driver. Bạn đang sử dụng `Ping` ở đây để xác nhận rằng package `database/sql` có thể kết nối khi cần.

   - Kiểm tra một lỗi từ `Ping`, trong trường hợp kết nối thất bại.

   - In một thông báo nếu `Ping` kết nối thành công.

2. Gần đầu main.go, ngay dưới khai báo package, import các package bạn sẽ cần để hỗ trợ mã bạn vừa viết.

   Đầu file bây giờ sẽ trông như thế này:
   ```go
   package main

   import (
       "database/sql"
       "fmt"
       "log"
       "os"

       "github.com/go-sql-driver/mysql"
   )
   ```

3. Lưu main.go.

### Chạy mã

1. Bắt đầu theo dõi module MySQL driver như một phụ thuộc.

   Sử dụng [`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) để thêm module github.com/go-sql-driver/mysql như một phụ thuộc cho module của bạn. Sử dụng đối số dấu chấm để có nghĩa là "lấy các phụ thuộc cho mã trong thư mục hiện tại."
   ```bash
   $ go get .
   go get: added github.com/go-sql-driver/mysql v1.6.0
   ```

   Go đã tải xuống phụ thuộc này vì bạn đã thêm nó vào khai báo `import` trong bước trước. Để biết thêm về theo dõi phụ thuộc, xem [Thêm một phụ thuộc](https://go.dev/doc/modules/managing-dependencies#adding_dependency).

2. Từ dấu nhắc lệnh, đặt các biến môi trường `DBUSER` và `DBPASS` để chương trình Go sử dụng.

   Trên Linux hoặc Mac:
   ```bash
   $ export DBUSER=username
   $ export DBPASS=password
   ```

   Trên Windows:
   ```bash
   C:\Users\you\data-access> set DBUSER=username
   C:\Users\you\data-access> set DBPASS=password
   ```

3. Từ dòng lệnh trong thư mục chứa main.go, chạy mã bằng cách gõ `go run` với đối số dấu chấm để có nghĩa là "chạy package trong thư mục hiện tại."
   ```bash
   $ go run .
   Connected!
   ```

Bạn có thể kết nối! Tiếp theo, bạn sẽ truy vấn cho một số dữ liệu.

## Truy vấn cho nhiều hàng

Trong phần này, bạn sẽ sử dụng Go để thực thi một truy vấn SQL được thiết kế để trả về nhiều hàng.

Đối với các truy vấn SQL có thể trả về nhiều hàng, bạn sử dụng phương thức `Query` từ package `database/sql`, sau đó lặp qua các hàng mà nó trả về. (Bạn sẽ tìm hiểu cách truy vấn cho một hàng đơn sau, trong phần [Truy vấn cho một hàng đơn](#truy-vấn-cho-một-hàng-đơn).)

### Viết mã

1. Vào main.go, ngay trên `func main`, dán định nghĩa sau của một struct `Album`. Bạn sẽ sử dụng điều này để giữ dữ liệu hàng được trả về từ truy vấn.
   ```go
   type Album struct {
       ID     int64
       Title  string
       Artist string
       Price  float32
   }
   ```

2. Bên dưới `func main`, dán hàm `albumsByArtist` sau để truy vấn cơ sở dữ liệu.
   ```go
   // albumsByArtist truy vấn cho albums có tên nghệ sĩ được chỉ định.
   func albumsByArtist(name string) ([]Album, error) {
       // Một albums slice để giữ dữ liệu từ các hàng được trả về.
       var albums []Album

       rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
       if err != nil {
           return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
       }
       defer rows.Close()
       // Lặp qua các hàng, sử dụng Scan để gán dữ liệu cột cho
       // các trường struct.
       for rows.Next() {
           var alb Album
           if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
               return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
           }
           albums = append(albums, alb)
       }
       if err := rows.Err(); err != nil {
           return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
       }
       return albums, nil
   }
   ```

   Trong mã này, bạn:

   - Khai báo một `albums` slice kiểu `Album` mà bạn đã định nghĩa. Điều này sẽ giữ dữ liệu từ các hàng được trả về. Tên và kiểu struct tương ứng với tên và kiểu cột cơ sở dữ liệu.

   - Sử dụng [`DB.Query`](https://pkg.go.dev/database/sql#DB.Query) để thực thi một câu lệnh `SELECT` để truy vấn cho albums với tên nghệ sĩ được chỉ định.

     Tham số đầu tiên của `Query` là câu lệnh SQL. Sau tham số, bạn có thể truyền không hoặc nhiều tham số của bất kỳ kiểu nào. Những tham số này cung cấp một cách để bạn xác định các giá trị cho các tham số trong truy vấn SQL của bạn. Bằng cách tách câu lệnh SQL khỏi các giá trị tham số -- thay vì nối chúng với, ví dụ, `fmt.Sprintf` -- bạn cho phép package `database/sql` gửi các giá trị tách biệt với văn bản SQL, loại bỏ bất kỳ rủi ro SQL injection nào.

   - Defer đóng `rows` để bất kỳ tài nguyên nào nó giữ sẽ được giải phóng khi hàm thoát.

   - Lặp qua các hàng được trả về, sử dụng [`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan) để gán các giá trị cột của mỗi hàng cho các trường struct `Album`.

     `Scan` nhận một danh sách các con trỏ đến các giá trị Go, nơi các giá trị cột sẽ được viết. Ở đây, bạn truyền các con trỏ đến các trường trong biến `alb`, được tạo sử dụng toán tử `&`. `Scan` viết thông qua các con trỏ để cập nhật các trường struct.

   - Bên trong vòng lặp, kiểm tra một lỗi từ việc scan các giá trị cột vào các trường struct.

   - Bên trong vòng lặp, thêm `alb` mới vào slice `albums`.

   - Sau vòng lặp, kiểm tra một lỗi từ hoạt động truy vấn tổng thể sử dụng `rows.Err`. Lưu ý rằng nếu bản thân truy vấn thất bại, việc kiểm tra một lỗi ở đây là cách duy nhất để tìm hiểu rằng kết quả là không đầy đủ.

3. Cập nhật hàm `main` của bạn để gọi `albumsByArtist`.

   Vào cuối `func main`, thêm mã sau.
   ```go
   albums, err := albumsByArtist("John Coltrane")
   if err != nil {
       log.Fatal(err)
   }
   fmt.Printf("Albums found: %v\n", albums)
   ```

   Trong mã mới, bạn bây giờ:

   - Gọi hàm `albumsByArtist` bạn đã thêm, gán giá trị trả về của nó cho một biến `albums` mới.
   - In kết quả.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
```

Tiếp theo, bạn sẽ truy vấn cho một hàng đơn.

## Truy vấn cho một hàng đơn

Trong phần này, bạn sẽ sử dụng Go để truy vấn cho một hàng đơn trong cơ sở dữ liệu.

Đối với các truy vấn SQL bạn biết sẽ trả về tối đa một hàng, bạn có thể sử dụng `QueryRow`, đơn giản hơn so với sử dụng vòng lặp `Query`.

### Viết mã

1. Bên dưới `albumsByArtist`, dán hàm `albumByID` sau.
   ```go
   // albumByID truy vấn cho album với ID được chỉ định.
   func albumByID(id int64) (Album, error) {
       // Một album để giữ dữ liệu từ hàng được trả về.
       var alb Album

       row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
       if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
           if err == sql.ErrNoRows {
               return alb, fmt.Errorf("albumsById %d: no such album", id)
           }
           return alb, fmt.Errorf("albumsById %d: %v", id, err)
       }
       return alb, nil
   }
   ```

   Trong mã này, bạn:

   - Sử dụng [`DB.QueryRow`](https://pkg.go.dev/database/sql#DB.QueryRow) để thực thi một câu lệnh `SELECT` để truy vấn cho một album với ID được chỉ định.

     Nó trả về một [`sql.Row`](https://pkg.go.dev/database/sql#Row). Để đơn giản hóa mã gọi (mã của bạn!), `QueryRow` không trả về một lỗi. Thay vào đó, nó sắp xếp để trả về bất kỳ lỗi truy vấn nào (như `sql.ErrNoRows`) từ `Rows.Scan` sau này.

   - Sử dụng [`Row.Scan`](https://pkg.go.dev/database/sql#Row.Scan) để sao chép các giá trị cột vào các trường struct.

   - Kiểm tra một lỗi từ `Scan`.

     Lỗi đặc biệt `sql.ErrNoRows` chỉ ra rằng truy vấn không trả về hàng nào. Thường lỗi đó đáng được thay thế bằng văn bản cụ thể hơn, như "no such album" ở đây.

2. Cập nhật `main` để gọi `albumByID`.

   Vào cuối `func main`, thêm mã sau.
   ```go
   // ID cứng 2 ở đây để test.
   alb, err := albumByID(2)
   if err != nil {
       log.Fatal(err)
   }
   fmt.Printf("Album found: %v\n", alb)
   ```

   Trong mã mới, bạn bây giờ:

   - Gọi hàm `albumByID` bạn đã thêm.
   - In album được trả về.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
```

Tiếp theo, bạn sẽ thêm một album vào cơ sở dữ liệu.

## Thêm dữ liệu

Trong phần này, bạn sẽ sử dụng Go để thực thi một câu lệnh SQL `INSERT` để thêm một hàng mới vào cơ sở dữ liệu.

Bạn đã thấy cách sử dụng `Query` và `QueryRow` với các câu lệnh SQL trả về dữ liệu. Để thực thi các câu lệnh SQL *không* trả về dữ liệu, bạn sử dụng `Exec`.

### Viết mã

1. Bên dưới `albumByID`, dán hàm `addAlbum` sau để chèn một album mới vào cơ sở dữ liệu, sau đó lưu main.go.
   ```go
   // addAlbum thêm album được chỉ định vào cơ sở dữ liệu,
   // trả về ID album của mục mới
   func addAlbum(alb Album) (int64, error) {
       result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
       if err != nil {
           return 0, fmt.Errorf("addAlbum: %v", err)
       }
       id, err := result.LastInsertId()
       if err != nil {
           return 0, fmt.Errorf("addAlbum: %v", err)
       }
       return id, nil
   }
   ```

   Trong mã này, bạn:

   - Sử dụng [`DB.Exec`](https://pkg.go.dev/database/sql#DB.Exec) để thực thi một câu lệnh `INSERT`.

     Giống như `Query`, `Exec` nhận một câu lệnh SQL theo sau bởi các giá trị tham số cho câu lệnh SQL.

   - Kiểm tra một lỗi từ nỗ lực `INSERT`.

   - Truy xuất ID của hàng cơ sở dữ liệu được chèn sử dụng [`Result.LastInsertId`](https://pkg.go.dev/database/sql#Result.LastInsertId).

   - Kiểm tra một lỗi từ nỗ lực truy xuất ID.

2. Cập nhật `main` để gọi hàm `addAlbum` mới.

   Vào cuối `func main`, thêm mã sau.
   ```go
   albID, err := addAlbum(Album{
       Title:  "The Modern Sound of Betty Carter",
       Artist: "Betty Carter",
       Price:  49.99,
   })
   if err != nil {
       log.Fatal(err)
   }
   fmt.Printf("ID of added album: %v\n", albID)
   ```

   Trong mã mới, bạn bây giờ:

   - Gọi `addAlbum` với một album mới, gán ID của album bạn đang thêm cho một biến `albID`.

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
```bash
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
ID of added album: 5
```

## Kết luận

Xin chúc mừng! Bạn vừa sử dụng Go để thực hiện các hành động đơn giản với một cơ sở dữ liệu quan hệ.

Các chủ đề được đề xuất tiếp theo:

- Hãy xem Hướng dẫn Truy cập Dữ liệu, cung cấp một cái nhìn rộng hơn về làm việc với cơ sở dữ liệu sử dụng Go, bao gồm nhiều thao tác nâng cao hơn.
- Nếu bạn mới bắt đầu với Go, bạn sẽ tìm thấy các thực hành tốt nhất hữu ích được mô tả trong [Effective Go](https://go.dev/doc/effective_go) và [Cách viết mã Go](https://go.dev/doc/code).
- [Go Tour](https://go.dev/tour/) là một giới thiệu tuyệt vời từng bước về các cơ bản của Go.

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/database-access](https://go.dev/doc/tutorial/database-access) để phục vụ cộng đồng lập trình viên Việt Nam.*
