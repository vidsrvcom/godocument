# Mô hình Bộ nhớ Go

## Giới thiệu

Mô hình bộ nhớ Go xác định các điều kiện theo đó các hoạt động đọc một biến trong một goroutine có thể được đảm bảo quan sát các giá trị được tạo ra bởi các hoạt động ghi vào cùng một biến trong một goroutine khác.

## Lời khuyên

Các chương trình có nhiều goroutine truy cập dữ liệu chia sẻ phải tuân theo các quy tắc đồng bộ hóa.

**Nếu bạn phải đọc phần còn lại của tài liệu này để hiểu hành vi của chương trình, bạn đang quá thông minh. Đừng thông minh như vậy.**

## Happens Before

### Định nghĩa

Trong một goroutine đơn, các đọc và ghi phải hoạt động như thể chúng thực thi theo thứ tự được chỉ định trong chương trình.

Để xác định yêu cầu đọc và ghi, chúng ta định nghĩa *happens before*, một thứ tự một phần trên việc thực thi các hoạt động bộ nhớ trong chương trình Go.

- Nếu sự kiện e1 *happens before* sự kiện e2, thì chúng ta nói rằng e2 *happens after* e1.
- Nếu e1 không xảy ra trước e2 và e2 không xảy ra trước e1, thì e1 và e2 xảy ra *đồng thời*.

### Quy tắc

Một đọc r của biến v được *phép* quan sát một ghi w của v nếu:
1. r không xảy ra trước w.
2. Không có ghi khác w' của v xảy ra sau w nhưng trước r.

Để đảm bảo rằng một đọc r của biến v quan sát một giá trị cụ thể được ghi bởi w:
1. w xảy ra trước r.
2. Mọi ghi khác của biến chia sẻ v xảy ra trước w hoặc sau r.

## Đồng bộ hóa

### Khởi tạo

Khởi tạo chương trình chạy trong một goroutine đơn, nhưng goroutine đó có thể tạo các goroutine khác.

Nếu package p import package q, việc hoàn thành các hàm init của q *happens before* bắt đầu bất kỳ hàm init nào của p.

Việc khởi động hàm main.main *happens after* tất cả các hàm init đã hoàn thành.

### Tạo Goroutine

Câu lệnh `go` khởi động một goroutine mới *happens before* goroutine mới bắt đầu thực thi.

```go
var a string

func f() {
    print(a)
}

func hello() {
    a = "hello, world"
    go f()
}
```

Gọi `hello` sẽ in "hello, world" tại một thời điểm nào đó trong tương lai (có thể sau khi hello đã trả về).

### Goroutine destruction

Sự kết thúc của một goroutine không được đảm bảo *happens before* bất kỳ sự kiện nào trong chương trình.

```go
var a string

func hello() {
    go func() { a = "hello" }()
    print(a)
}
```

Đoạn mã này không có đồng bộ hóa nên không đảm bảo goroutine ghi sẽ được quan sát bởi goroutine khác.

### Channel communication

Giao tiếp qua channel là phương pháp đồng bộ hóa chính giữa các goroutine.

**Quy tắc:**

1. Một gửi trên channel *happens before* nhận tương ứng từ channel đó hoàn thành.

```go
var c = make(chan int, 10)
var a string

func f() {
    a = "hello, world"
    c <- 0
}

func main() {
    go f()
    <-c
    print(a)
}
```

2. Việc đóng channel *happens before* nhận trả về giá trị zero.

3. Nhận từ unbuffered channel *happens before* gửi trên channel đó hoàn thành.

```go
var c = make(chan int)
var a string

func f() {
    a = "hello, world"
    <-c
}

func main() {
    go f()
    c <- 0
    print(a)
}
```

4. Nhận thứ k từ channel với capacity C *happens before* gửi thứ k+C hoàn thành.

### Locks

Package `sync` cung cấp hai loại lock: `sync.Mutex` và `sync.RWMutex`.

**Quy tắc:**

Với bất kỳ `sync.Mutex` hoặc `sync.RWMutex` nào, gọi n của `l.Unlock()` *happens before* gọi m của `l.Lock()` trả về, với n < m.

```go
var l sync.Mutex
var a string

func f() {
    a = "hello, world"
    l.Unlock()
}

func main() {
    l.Lock()
    go f()
    l.Lock()
    print(a)
}
```

### Once

Package `sync` cung cấp `sync.Once` cho khởi tạo an toàn.

```go
var a string
var once sync.Once

func setup() {
    a = "hello, world"
}

func doprint() {
    once.Do(setup)
    print(a)
}

func twoprint() {
    go doprint()
    go doprint()
}
```

`setup` sẽ chỉ được gọi một lần và `print` sẽ in "hello, world" hai lần.

### Atomic Values

Package `sync/atomic` cung cấp các thao tác atomic.

```go
var x atomic.Int64

func f() {
    x.Store(1)
}

func main() {
    go f()
    if x.Load() == 1 {
        // x đã được thiết lập
    }
}
```

## Đồng bộ hóa không chính xác

### Race condition

```go
var a, b int

func f() {
    a = 1
    b = 2
}

func g() {
    print(b)
    print(a)
}

func main() {
    go f()
    g()
}
```

Có thể in "2, 0" vì không có đồng bộ hóa.

### Double-checked locking

Đây là một nỗ lực để tránh overhead của đồng bộ hóa:

```go
var a string
var done bool

func setup() {
    a = "hello, world"
    done = true
}

func doprint() {
    if !done {
        once.Do(setup)
    }
    print(a)
}
```

**Không có đảm bảo** rằng việc quan sát `done` được ghi sẽ quan sát `a` được ghi.

### Busy waiting

```go
var a string
var done bool

func setup() {
    a = "hello, world"
    done = true
}

func main() {
    go setup()
    for !done {
    }
    print(a)
}
```

**Không có đảm bảo** chương trình này sẽ kết thúc hoặc in "hello, world".

## Kết luận

Để viết chương trình Go đồng thời chính xác:
1. Sử dụng channels để giao tiếp
2. Sử dụng mutex hoặc atomic operations khi cần
3. Đừng giả định về thứ tự thực thi giữa các goroutine
4. Sử dụng race detector: `go test -race`

---

*Xem tài liệu đầy đủ tại [go.dev/ref/mem](https://go.dev/ref/mem)*
