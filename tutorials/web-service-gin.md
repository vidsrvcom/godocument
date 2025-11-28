# Hướng dẫn: Phát triển RESTful API với Go và Gin

Bảng nội dung:
- [Yêu cầu tiên quyết](#yêu-cầu-tiên-quyết)
- [Thiết kế API endpoints](#thiết-kế-api-endpoints)
- [Tạo thư mục cho mã của bạn](#tạo-thư-mục-cho-mã-của-bạn)
- [Tạo dữ liệu](#tạo-dữ-liệu)
- [Viết handler để trả về tất cả các mục](#viết-handler-để-trả-về-tất-cả-các-mục)
- [Viết handler để thêm một mục mới](#viết-handler-để-thêm-một-mục-mới)
- [Viết handler để trả về một mục cụ thể](#viết-handler-để-trả-về-một-mục-cụ-thể)
- [Kết luận](#kết-luận)

Hướng dẫn này giới thiệu về cơ bản của việc viết một RESTful web service API với Go và [Gin Web Framework](https://gin-gonic.com/docs/) (Gin).

Bạn sẽ thu được nhiều nhất từ hướng dẫn này nếu bạn có một sự quen thuộc cơ bản với Go và các công cụ của nó. Nếu đây là lần đầu tiên bạn tiếp xúc với Go, vui lòng xem [Hướng dẫn: Bắt đầu với Go](getting-started.md) để biết giới thiệu nhanh.

Gin đơn giản hóa nhiều tác vụ mã hóa liên quan đến việc xây dựng các ứng dụng web, bao gồm các web services. Trong hướng dẫn này, bạn sẽ sử dụng Gin để định tuyến các yêu cầu, truy xuất chi tiết yêu cầu và marshal JSON cho các phản hồi.

Trong hướng dẫn này, bạn sẽ xây dựng một RESTful API server với hai endpoints. Dự án ví dụ của bạn sẽ là một kho lưu trữ dữ liệu về các bản ghi nhạc jazz vintage.

Hướng dẫn bao gồm các phần sau:

1. Thiết kế API endpoints.
2. Tạo một thư mục cho mã của bạn.
3. Tạo dữ liệu.
4. Viết một handler để trả về tất cả các mục.
5. Viết một handler để thêm một mục mới.
6. Viết một handler để trả về một mục cụ thể.

> **Lưu ý:** Để xem các hướng dẫn khác, xem [Hướng dẫn](../doc.md).

> **Mẹo:** Bạn cũng có thể thấy [Module truy cập cơ sở dữ liệu quan hệ](https://go.dev/doc/tutorial/database-access) và [Phát triển module](https://go.dev/doc/modules/developing) hữu ích.

## Yêu cầu tiên quyết

- **Một bản cài đặt Go 1.16 trở lên.** Để biết hướng dẫn cài đặt, xem [Cài đặt Go](https://go.dev/doc/install).
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, và trên PowerShell hoặc cmd trong Windows.
- **Công cụ curl.** Trên Linux và Mac, công cụ này đã được cài đặt sẵn. Trên Windows, nó được bao gồm trên Windows 10 Insider build 17063 trở lên. Đối với các phiên bản Windows trước đó, bạn có thể cần phải cài đặt nó. Để biết thêm, xem [Tar và Curl đến Windows](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20171219-tar-and-curl-come-to-windows).

## Thiết kế API endpoints

Bạn sẽ xây dựng một API cung cấp quyền truy cập vào một cửa hàng bán các bản ghi trên vinyl. Vì vậy, bạn sẽ cần cung cấp các endpoints thông qua đó một client có thể lấy và thêm albums cho người dùng.

Khi phát triển một API, bạn thường bắt đầu bằng việc thiết kế các endpoints. Người dùng API của bạn sẽ thành công hơn nếu các endpoints dễ hiểu.

Đây là các endpoints bạn sẽ tạo trong hướng dẫn này.

| Đường dẫn          | Phương thức | Mô tả                            |
|--------------------|-------------|----------------------------------|
| /albums            | GET         | Lấy danh sách tất cả các albums  |
| /albums            | POST        | Thêm một album mới từ JSON yêu cầu |
| /albums/:id        | GET         | Lấy một album theo ID của nó     |

Tiếp theo, bạn sẽ tạo một thư mục cho mã của bạn.

## Tạo thư mục cho mã của bạn

Để bắt đầu, tạo một dự án cho mã bạn sẽ viết.

1. Mở một dấu nhắc lệnh và thay đổi đến thư mục home của bạn.

   Trên Linux hoặc Mac:
   ```bash
   cd
   ```

   Trên Windows:
   ```bash
   cd %HOMEPATH%
   ```

2. Sử dụng dấu nhắc lệnh, tạo một thư mục cho mã của bạn gọi là web-service-gin.
   ```bash
   mkdir web-service-gin
   cd web-service-gin
   ```

3. Tạo một module trong đó bạn có thể quản lý các phụ thuộc.

   Chạy lệnh `go mod init`, đặt cho nó đường dẫn của module mà mã của bạn sẽ nằm trong.
   ```bash
   $ go mod init example/web-service-gin
   go: creating new go.mod: module example/web-service-gin
   ```

   Lệnh này tạo một file go.mod trong đó các phụ thuộc bạn thêm vào sẽ được liệt kê để theo dõi. Để biết thêm về đặt tên một module với đường dẫn module, xem [Quản lý phụ thuộc](https://go.dev/doc/modules/managing-dependencies#naming_module).

Tiếp theo, bạn sẽ thiết kế các cấu trúc dữ liệu để xử lý dữ liệu.

## Tạo dữ liệu

Để giữ cho mọi thứ đơn giản cho hướng dẫn, bạn sẽ lưu trữ dữ liệu trong bộ nhớ. Một API điển hình hơn sẽ tương tác với cơ sở dữ liệu.

Lưu ý rằng việc lưu trữ dữ liệu trong bộ nhớ có nghĩa là tập hợp các albums sẽ bị mất mỗi khi bạn dừng server, sau đó được tạo lại khi bạn khởi động lại nó.

### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, trong thư mục web-service-gin, tạo một file gọi là main.go. Bạn sẽ viết mã Go của bạn trong file này.

2. Vào đầu main.go, dán khai báo package sau.
   ```go
   package main
   ```

   Một package độc lập (không phải là một thư viện) luôn nằm trong package `main`.

3. Bên dưới khai báo package, dán khai báo sau của một struct `album`. Bạn sẽ sử dụng điều này để lưu trữ dữ liệu album trong bộ nhớ.

   Các struct tags như `json:"artist"` xác định tên của một trường nên là gì khi nội dung của struct được serialize thành JSON. Nếu không có chúng, JSON sẽ sử dụng tên trường viết hoa của struct -- một phong cách không phổ biến trong JSON.
   ```go
   // album đại diện cho dữ liệu về một bản ghi album.
   type album struct {
       ID     string  `json:"id"`
       Title  string  `json:"title"`
       Artist string  `json:"artist"`
       Price  float64 `json:"price"`
   }
   ```

4. Bên dưới khai báo struct bạn vừa thêm, dán slice sau của các struct `album` chứa dữ liệu bạn sẽ sử dụng để bắt đầu.
   ```go
   // albums slice để seed dữ liệu album bản ghi.
   var albums = []album{
       {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
       {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
       {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
   }
   ```

Tiếp theo, bạn sẽ viết mã để triển khai endpoint đầu tiên của bạn.

## Viết handler để trả về tất cả các mục

Khi client thực hiện một yêu cầu tại `GET /albums`, bạn muốn trả về tất cả các albums dưới dạng JSON.

Để làm điều này, bạn sẽ viết những thứ sau:

- Logic để chuẩn bị một phản hồi
- Mã để ánh xạ đường dẫn yêu cầu đến logic của bạn

Lưu ý rằng đây là ngược lại so với cách chúng sẽ được thực thi tại thời gian chạy, nhưng bạn đang thêm các phụ thuộc trước, sau đó là mã phụ thuộc vào chúng.

### Viết mã

1. Bên dưới mã struct bạn đã thêm trong phần trước, dán mã sau để lấy danh sách album.

   Hàm `getAlbums` này tạo JSON từ slice của các struct `album`, viết JSON vào phản hồi.
   ```go
   // getAlbums phản hồi với danh sách tất cả các albums dưới dạng JSON.
   func getAlbums(c *gin.Context) {
       c.IndentedJSON(http.StatusOK, albums)
   }
   ```

   Trong mã này, bạn:

   - Viết một hàm `getAlbums` nhận một tham số [`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context). Lưu ý rằng bạn có thể đặt cho hàm này bất kỳ tên nào bạn thích -- cả Gin và Go đều không yêu cầu một định dạng tên hàm cụ thể.

     `gin.Context` là phần quan trọng nhất của Gin. Nó mang thông tin yêu cầu, xác thực và serialize JSON, và nhiều hơn nữa. (Mặc dù có tên tương tự, điều này khác với package [`context`](https://go.dev/pkg/context/) tích hợp của Go.)

   - Gọi [`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON) để serialize struct thành JSON và thêm nó vào phản hồi.

     Đối số đầu tiên của hàm là mã trạng thái HTTP bạn muốn gửi cho client. Ở đây, bạn đang truyền hằng số [`StatusOK`](https://pkg.go.dev/net/http#StatusOK) từ package `net/http` để chỉ ra `200 OK`.

     Lưu ý rằng bạn có thể thay thế `Context.IndentedJSON` bằng một lời gọi đến [`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON) để gửi JSON gọn gàng hơn. Trong thực tế, dạng thụt lề dễ làm việc hơn khi debugging và sự khác biệt về kích thước thường là nhỏ.

2. Gần đầu main.go, ngay dưới khai báo slice `albums`, dán mã dưới đây để gán hàm handler cho một đường dẫn endpoint.

   Điều này thiết lập một liên kết trong đó `getAlbums` xử lý các yêu cầu đến đường dẫn endpoint `/albums`.
   ```go
   func main() {
       router := gin.Default()
       router.GET("/albums", getAlbums)

       router.Run("localhost:8080")
   }
   ```

   Trong mã này, bạn:

   - Khởi tạo một Gin router bằng cách sử dụng [`Default`](https://pkg.go.dev/github.com/gin-gonic/gin#Default).

   - Sử dụng hàm [`GET`](https://pkg.go.dev/github.com/gin-gonic/gin#RouterGroup.GET) để liên kết phương thức HTTP `GET` và đường dẫn `/albums` với một hàm handler.

     Lưu ý rằng bạn đang truyền *tên* của hàm `getAlbums`. Điều này khác với việc truyền *kết quả* của hàm, mà bạn sẽ làm bằng cách truyền `getAlbums()` (lưu ý dấu ngoặc đơn).

   - Sử dụng hàm [`Run`](https://pkg.go.dev/github.com/gin-gonic/gin#Engine.Run) để gắn router vào một `http.Server` và khởi động server.

3. Gần đầu main.go, ngay dưới khai báo package, import các package bạn sẽ cần để hỗ trợ mã bạn vừa viết.

   Các dòng import đầu tiên sẽ trông như thế này:
   ```go
   package main

   import (
       "net/http"

       "github.com/gin-gonic/gin"
   )
   ```

4. Lưu main.go.

### Chạy mã

1. Bắt đầu theo dõi module Gin như một phụ thuộc.

   Tại dòng lệnh, sử dụng [`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) để thêm module github.com/gin-gonic/gin như một phụ thuộc cho module của bạn. Sử dụng một đối số dấu chấm để có nghĩa là "lấy các phụ thuộc cho mã trong thư mục hiện tại."
   ```bash
   $ go get .
   go: downloading github.com/gin-gonic/gin v1.7.2
   go: downloading github.com/gin-contrib/sse v0.1.0
   go: downloading github.com/mattn/go-isatty v0.0.12
   go: downloading github.com/go-playground/validator/v10 v10.4.1
   go: downloading golang.org/x/sys v0.0.0-20200116001909-b77594299b42
   go: downloading github.com/go-playground/universal-translator v0.17.0
   go: downloading github.com/leodido/go-urn v1.2.0
   go: downloading github.com/ugorji/go v1.1.7
   go: downloading github.com/golang/protobuf v1.3.3
   go: downloading github.com/json-iterator/go v1.1.9
   go: downloading github.com/ugorji/go/codec v1.1.7
   go: downloading gopkg.in/yaml.v2 v2.2.8
   go: downloading github.com/go-playground/locales v0.13.0
   go: downloading github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421
   go: downloading github.com/modern-go/reflect2 v0.0.0-20180701023420-4b7aa43c6742
   go: downloading github.com/go-playground/validator v9.31.0+incompatible
   go get: added github.com/gin-gonic/gin v1.7.2
   ```

   Go đã giải quyết và tải xuống phụ thuộc này để đáp ứng khai báo `import` bạn đã thêm trong bước trước.

2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã. Sử dụng một đối số dấu chấm để có nghĩa là "chạy mã trong thư mục hiện tại."
   ```bash
   $ go run .
   ```

   Khi mã đang chạy, bạn có một server HTTP đang chạy mà bạn có thể gửi các yêu cầu đến.

3. Từ một cửa sổ dòng lệnh mới, sử dụng `curl` để thực hiện một yêu cầu đến web service đang chạy của bạn.
   ```bash
   $ curl http://localhost:8080/albums
   ```

   Lệnh sẽ hiển thị dữ liệu bạn đã seed cho service.
   ```json
   [
           {
                   "id": "1",
                   "title": "Blue Train",
                   "artist": "John Coltrane",
                   "price": 56.99
           },
           {
                   "id": "2",
                   "title": "Jeru",
                   "artist": "Gerry Mulligan",
                   "price": 17.99
           },
           {
                   "id": "3",
                   "title": "Sarah Vaughan and Clifford Brown",
                   "artist": "Sarah Vaughan",
                   "price": 39.99
           }
   ]
   ```

Bạn đã khởi động một API! Trong phần tiếp theo, bạn sẽ tạo một endpoint khác với mã để xử lý một yêu cầu `POST` để thêm một album.

## Viết handler để thêm một mục mới

Khi client thực hiện một yêu cầu `POST` tại `/albums`, bạn muốn thêm album được mô tả trong body yêu cầu vào dữ liệu albums hiện có.

Để làm điều này, bạn sẽ viết những thứ sau:

- Logic để thêm album mới vào danh sách hiện có.
- Một chút mã để định tuyến yêu cầu POST đến logic của bạn.

### Viết mã

1. Thêm mã để thêm dữ liệu albums vào danh sách các albums.

   Đâu đó sau các câu lệnh `import`, dán mã sau. (Cuối file là một nơi tốt cho mã này, nhưng Go không thực thi thứ tự mà bạn khai báo các hàm.)
   ```go
   // postAlbums thêm một album từ JSON nhận được trong body yêu cầu.
   func postAlbums(c *gin.Context) {
       var newAlbum album

       // Gọi BindJSON để bind JSON nhận được vào
       // newAlbum.
       if err := c.BindJSON(&newAlbum); err != nil {
           return
       }

       // Thêm album mới vào slice.
       albums = append(albums, newAlbum)
       c.IndentedJSON(http.StatusCreated, newAlbum)
   }
   ```

   Trong mã này, bạn:

   - Sử dụng [`Context.BindJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.BindJSON) để bind body yêu cầu vào `newAlbum`.
   - Thêm struct `album` được khởi tạo từ JSON vào slice `albums`.
   - Thêm một mã trạng thái `201` vào phản hồi, cùng với JSON đại diện cho album bạn đã thêm.

2. Thay đổi hàm `main` của bạn để nó bao gồm hàm `router.POST`, như sau.
   ```go
   func main() {
       router := gin.Default()
       router.GET("/albums", getAlbums)
       router.POST("/albums", postAlbums)

       router.Run("localhost:8080")
   }
   ```

   Trong mã này, bạn:

   - Liên kết phương thức `POST` tại đường dẫn `/albums` với hàm `postAlbums`.

     Với Gin, bạn có thể liên kết một handler với một tổ hợp phương thức HTTP và đường dẫn. Bằng cách này, bạn có thể định tuyến riêng biệt các yêu cầu được gửi đến một đường dẫn duy nhất dựa trên phương thức mà client đang sử dụng.

### Chạy mã

1. Nếu server vẫn đang chạy từ phần cuối, hãy dừng nó.

2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
   ```bash
   $ go run .
   ```

3. Từ một cửa sổ dòng lệnh khác, sử dụng `curl` để thực hiện một yêu cầu đến web service đang chạy của bạn.
   ```bash
   $ curl http://localhost:8080/albums \
       --include \
       --header "Content-Type: application/json" \
       --request "POST" \
       --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
   ```

   Lệnh sẽ hiển thị các header và JSON cho album đã thêm.
   ```
   HTTP/1.1 201 Created
   Content-Type: application/json; charset=utf-8
   Date: Wed, 02 Jun 2021 00:34:12 GMT
   Content-Length: 116

   {
       "id": "4",
       "title": "The Modern Sound of Betty Carter",
       "artist": "Betty Carter",
       "price": 49.99
   }
   ```

4. Như trong phần trước, sử dụng `curl` để truy xuất danh sách đầy đủ các albums, mà bạn có thể sử dụng để xác nhận rằng album mới đã được thêm.
   ```bash
   $ curl http://localhost:8080/albums \
       --header "Content-Type: application/json" \
       --request "GET"
   ```

   Lệnh sẽ hiển thị danh sách albums.
   ```json
   [
           {
                   "id": "1",
                   "title": "Blue Train",
                   "artist": "John Coltrane",
                   "price": 56.99
           },
           {
                   "id": "2",
                   "title": "Jeru",
                   "artist": "Gerry Mulligan",
                   "price": 17.99
           },
           {
                   "id": "3",
                   "title": "Sarah Vaughan and Clifford Brown",
                   "artist": "Sarah Vaughan",
                   "price": 39.99
           },
           {
                   "id": "4",
                   "title": "The Modern Sound of Betty Carter",
                   "artist": "Betty Carter",
                   "price": 49.99
           }
   ]
   ```

Trong phần tiếp theo, bạn sẽ thêm mã để xử lý một `GET` cho một mục cụ thể.

## Viết handler để trả về một mục cụ thể

Khi client thực hiện một yêu cầu đến `GET /albums/[id]`, bạn muốn trả về album có ID khớp với tham số đường dẫn `id`.

Để làm điều này, bạn sẽ:

- Thêm logic để truy xuất album được yêu cầu.
- Ánh xạ đường dẫn đến logic.

### Viết mã

1. Bên dưới hàm `postAlbums` bạn đã thêm trong phần trước, dán mã sau để truy xuất một album cụ thể.

   Hàm `getAlbumByID` này sẽ trích xuất ID trong đường dẫn yêu cầu, sau đó định vị một album khớp.
   ```go
   // getAlbumByID định vị album có giá trị ID khớp với tham số id
   // được gửi bởi client, sau đó trả về album đó như một phản hồi.
   func getAlbumByID(c *gin.Context) {
       id := c.Param("id")

       // Lặp qua danh sách các albums, tìm kiếm
       // một album có giá trị ID khớp với tham số.
       for _, a := range albums {
           if a.ID == id {
               c.IndentedJSON(http.StatusOK, a)
               return
           }
       }
       c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
   }
   ```

   Trong mã này, bạn:

   - Sử dụng [`Context.Param`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.Param) để truy xuất tham số đường dẫn `id` từ URL. Khi bạn ánh xạ handler này đến một đường dẫn, bạn sẽ bao gồm một placeholder cho tham số trong đường dẫn.

   - Lặp qua các struct `album` trong slice, tìm kiếm một struct có trường `ID` khớp với giá trị tham số `id`. Nếu tìm thấy, bạn serialize struct `album` đó thành JSON và trả về nó như một phản hồi với mã HTTP `200 OK`.

     Như đã đề cập ở trên, một service trong thế giới thực có thể sẽ sử dụng một truy vấn cơ sở dữ liệu để thực hiện tra cứu này.

   - Trả về một lỗi HTTP `404` với [`http.StatusNotFound`](https://pkg.go.dev/net/http#StatusNotFound) nếu album không được tìm thấy.

2. Cuối cùng, thay đổi `main` của bạn để nó bao gồm một lời gọi mới đến `router.GET`, trong đó đường dẫn bây giờ là `/albums/:id`, như được hiển thị trong ví dụ sau.
   ```go
   func main() {
       router := gin.Default()
       router.GET("/albums", getAlbums)
       router.GET("/albums/:id", getAlbumByID)
       router.POST("/albums", postAlbums)

       router.Run("localhost:8080")
   }
   ```

   Trong mã này, bạn:

   - Liên kết đường dẫn `/albums/:id` với hàm `getAlbumByID`. Trong Gin, dấu hai chấm trước một mục trong đường dẫn biểu thị rằng mục đó là một tham số đường dẫn.

### Chạy mã

1. Nếu server vẫn đang chạy từ phần cuối, hãy dừng nó.

2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã để khởi động server.
   ```bash
   $ go run .
   ```

3. Từ một cửa sổ dòng lệnh khác, sử dụng `curl` để thực hiện một yêu cầu đến web service đang chạy của bạn.
   ```bash
   $ curl http://localhost:8080/albums/2
   ```

   Lệnh sẽ hiển thị JSON cho album có ID bạn đã sử dụng. Nếu album không được tìm thấy, bạn sẽ nhận được JSON với một thông báo lỗi.
   ```json
   {
           "id": "2",
           "title": "Jeru",
           "artist": "Gerry Mulligan",
           "price": 17.99
   }
   ```

## Kết luận

Xin chúc mừng! Bạn vừa sử dụng Go và Gin để viết một RESTful web service đơn giản.

Các chủ đề được đề xuất tiếp theo:

- Nếu bạn mới bắt đầu với Go, bạn sẽ tìm thấy các thực hành tốt nhất hữu ích được mô tả trong [Effective Go](https://go.dev/doc/effective_go) và [Cách viết mã Go](https://go.dev/doc/code).
- [Go Tour](https://go.dev/tour/) là một giới thiệu tuyệt vời từng bước về các cơ bản của Go.
- Để biết thêm về Gin, xem [Tài liệu Gin Web Framework](https://gin-gonic.com/docs/) hoặc [Godoc của Gin](https://pkg.go.dev/github.com/gin-gonic/gin).

## Mã hoàn chỉnh

Phần này chứa mã cho ứng dụng bạn xây dựng với hướng dẫn này.

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

// album đại diện cho dữ liệu về một bản ghi album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice để seed dữ liệu album bản ghi.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}

// getAlbums phản hồi với danh sách tất cả các albums dưới dạng JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums thêm một album từ JSON nhận được trong body yêu cầu.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Gọi BindJSON để bind JSON nhận được vào
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Thêm album mới vào slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID định vị album có giá trị ID khớp với tham số id
// được gửi bởi client, sau đó trả về album đó như một phản hồi.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Lặp qua danh sách các albums, tìm kiếm
    // một album có giá trị ID khớp với tham số.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```

---

*Tài liệu này được dịch từ [go.dev/doc/tutorial/web-service-gin](https://go.dev/doc/tutorial/web-service-gin) để phục vụ cộng đồng lập trình viên Việt Nam.*
