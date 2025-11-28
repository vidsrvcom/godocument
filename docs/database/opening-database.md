# Mở một Database Handle

Khi bạn có database với driver được hỗ trợ bởi Go, bạn sẽ sử dụng package `database/sql` để làm việc với nó. Package này bao gồm các kiểu và hàm giúp bạn kết nối với cơ sở dữ liệu, thực thi các giao dịch, hủy các hoạt động đang diễn ra, và nhiều hơn nữa.

Để biết chi tiết về sử dụng package `database/sql`, xem [Truy cập cơ sở dữ liệu quan hệ](../../tutorials/database-access.md).

## Cấu hình để truy cập cơ sở dữ liệu

Trong Go, bạn truy cập cơ sở dữ liệu bằng package `database/sql` và driver cơ sở dữ liệu cụ thể cho hệ thống quản lý cơ sở dữ liệu (DBMS) của bạn.

Package `database/sql` bao gồm các kiểu và hàm để kết nối với cơ sở dữ liệu, thực thi giao dịch, hủy một hoạt động đang diễn ra, và nhiều hơn nữa. Để biết thêm chi tiết, xem [Tổng quan về Truy cập Cơ sở Dữ liệu](overview.md).

Driver cơ sở dữ liệu dịch các yêu cầu và phản hồi giữa chương trình của bạn và cơ sở dữ liệu. Để biết danh sách các driver, xem [SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers).

## Các bước thực hiện

Để truy cập cơ sở dữ liệu, bạn sẽ:

1. **Locate và import một driver cơ sở dữ liệu**  
   Driver dịch các yêu cầu và phản hồi giữa chương trình và cơ sở dữ liệu.
   
   ```go
   import "github.com/go-sql-driver/mysql"
   ```

2. **Mở một database handle**  
   Database handle đại diện cho một kết nối đến cơ sở dữ liệu cụ thể.
   
   ```go
   db, err := sql.Open("mysql", "user:password@tcp(127.0.0.1:3306)/dbname")
   if err != nil {
       log.Fatal(err)
   }
   defer db.Close()
   ```

3. **Xác nhận kết nối**  
   Khi bạn mở database handle, driver có thể không kết nối ngay lập tức. Thay vào đó, nó có thể chỉ xác thực rằng các đối số bạn cung cấp là hợp lệ. Để xác nhận rằng cơ sở dữ liệu khả dụng, gọi method `Ping`:
   
   ```go
   err = db.Ping()
   if err != nil {
       log.Fatal(err)
   }
   fmt.Println("Connected!")
   ```

## Ví dụ đầy đủ

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    // Chuỗi kết nối Data Source Name (DSN)
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname"
    
    // Mở kết nối database
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Verify connection
    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("Successfully connected to database!")
}
```

## Cấu hình Connection Pool

`sql.DB` handle được thiết kế để tồn tại lâu dài. Không nên `Open()` và `Close()` cơ sở dữ liệu thường xuyên. Thay vào đó, tạo một `sql.DB` handle cho mỗi database khác biệt mà bạn cần truy cập, và giữ nó cho đến khi chương trình kết thúc.

Bạn có thể cấu hình các thiết lập của connection pool:

```go
// Đặt số lượng kết nối mở tối đa
db.SetMaxOpenConns(25)

// Đặt số lượng kết nối idle tối đa
db.SetMaxIdleConns(25)

// Đặt thời gian tối đa một kết nối có thể được tái sử dụng
db.SetConnMaxLifetime(5 * time.Minute)
```

## Xử lý lỗi

Luôn kiểm tra và xử lý lỗi khi mở kết nối cơ sở dữ liệu. Các lỗi phổ biến bao gồm:

- Thông tin xác thực không chính xác
- Cơ sở dữ liệu không khả dụng
- Chuỗi kết nối không hợp lệ
- Driver không được cài đặt hoặc import

## Thực hành tốt nhất

1. **Import driver với blank identifier**: Sử dụng `_` khi import driver để chỉ thực thi init function của nó:
   
   ```go
   import _ "github.com/go-sql-driver/mysql"
   ```

2. **Giữ database handle trong suốt vòng đời ứng dụng**: Không tạo và đóng kết nối liên tục.

3. **Sử dụng defer để đảm bảo đóng kết nối**: Điều này đảm bảo tài nguyên được giải phóng ngay cả khi có lỗi.

4. **Cấu hình connection pool phù hợp**: Điều chỉnh dựa trên nhu cầu của ứng dụng.

## Xem thêm

- [Thực thi các câu lệnh SQL](executing-statements.md)
- [Truy vấn dữ liệu](querying-data.md)
- [Quản lý kết nối](connection-management.md)
- [Tổng quan về Truy cập Cơ sở Dữ liệu](overview.md)
