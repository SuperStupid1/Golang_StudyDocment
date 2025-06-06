# Golang后端开发面试题集合

## 目录

1. [Go语言基础](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#go语言基础)
2. [数据结构与类型](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#数据结构与类型)
3. [并发编程](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#并发编程)
4. [内存管理与GC](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#内存管理与gc)
5. [网络编程](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#网络编程)
6. [数据库操作](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#数据库操作)
7. [微服务与分布式](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#微服务与分布式)
8. [性能优化](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#性能优化)
9. [项目实战](https://claude.ai/chat/67e8931c-22c1-47ac-97db-22ef780c4a02#项目实战)

------

## Go语言基础

### 1. Go语言的特点和优势是什么？

**答案：**

- **编译型语言**：编译速度快，生成单一可执行文件
- **静态类型**：类型安全，编译时检查
- **垃圾回收**：自动内存管理
- **并发支持**：goroutine和channel提供原生并发支持
- **简洁语法**：语法简单，学习成本低
- **高性能**：接近C语言的执行效率
- **跨平台**：支持多种操作系统和架构
- **丰富的标准库**：网络、加密、压缩等功能完善

### 2. 解释Go语言中的包（package）机制

**答案：**

```go
// 包声明
package main

// 导入包
import (
    "fmt"
    "strings"
    _ "github.com/go-sql-driver/mysql" // 空导入
    . "math"  // 点导入
    m "math"  // 别名导入
)

// 包的初始化顺序：
// 1. 包级别变量初始化
// 2. init()函数执行
// 3. main()函数执行（如果是main包）
```

**要点：**

- 包名应该简洁、小写
- 包内大写字母开头的标识符是导出的（public）
- 小写字母开头的标识符是包内私有的
- 每个Go文件必须属于一个包

### 3. Go语言中的变量声明方式有哪些？

**答案：**

```go
// 1. var关键字声明
var name string
var age int = 25
var height, weight float64

// 2. 短变量声明（只能在函数内使用）
name := "张三"
age, height := 25, 175.5

// 3. 批量声明
var (
    name   string
    age    int
    height float64
)

// 4. 类型推断
var name = "张三"  // 推断为string类型
```

### 4. 什么是零值？Go语言中各种类型的零值是什么？

**答案：**

```go
var b bool        // false
var i int         // 0
var f float64     // 0.0
var s string      // ""
var p *int        // nil
var slice []int   // nil
var m map[string]int // nil
var ch chan int   // nil
var fn func()     // nil
var inter interface{} // nil
```

**重要概念：**

- Go语言中所有变量都有零值，不存在未初始化的变量
- 零值使得程序更安全，避免了未定义行为

### 5. 解释Go语言中的常量（const）

**答案：**

```go
// 普通常量
const PI = 3.14159
const name = "Go语言"

// 批量声明
const (
    StatusOK = 200
    StatusNotFound = 404
    StatusError = 500
)

// iota枚举器
const (
    Sunday = iota    // 0
    Monday          // 1
    Tuesday         // 2
    Wednesday       // 3
    Thursday        // 4
    Friday          // 5
    Saturday        // 6
)

// 复杂的iota使用
const (
    _  = iota             // 0，忽略
    KB = 1 << (10 * iota) // 1024
    MB                    // 1048576
    GB                    // 1073741824
)
```

------

## 数据结构与类型

### 6. 说说Go语言中的数组和切片的区别

**答案：**

**数组（Array）：**

```go
// 声明和初始化
var arr1 [5]int                    // 零值数组
arr2 := [5]int{1, 2, 3, 4, 5}     // 初始化数组
arr3 := [...]int{1, 2, 3}         // 让编译器推断长度

// 特点：
// 1. 长度固定，是类型的一部分
// 2. 值类型，传递时会复制整个数组
// 3. 长度不同的数组是不同类型
```

**切片（Slice）：**

```go
// 声明和初始化
var slice1 []int                   // nil切片
slice2 := []int{1, 2, 3}          // 字面量创建
slice3 := make([]int, 5)          // make创建，长度为5
slice4 := make([]int, 5, 10)      // 长度5，容量10

// 从数组或切片创建
arr := [5]int{1, 2, 3, 4, 5}
slice5 := arr[1:4]                // [2, 3, 4]

// 特点：
// 1. 长度可变
// 2. 引用类型，底层指向数组
// 3. 有长度（len）和容量（cap）概念
```

**关键区别：**

- 数组长度固定，切片长度可变
- 数组是值类型，切片是引用类型
- 切片更灵活，是实际开发中的首选

### 7. 解释切片的扩容机制

**答案：**

```go
func main() {
    slice := make([]int, 0, 1)
    
    for i := 0; i < 10; i++ {
        slice = append(slice, i)
        fmt.Printf("len: %d, cap: %d\n", len(slice), cap(slice))
    }
}

// 输出：
// len: 1, cap: 1
// len: 2, cap: 2
// len: 3, cap: 4
// len: 4, cap: 4
// len: 5, cap: 8
// len: 6, cap: 8
// len: 7, cap: 8
// len: 8, cap: 8
// len: 9, cap: 16
// len: 10, cap: 16
```

**扩容规则：**

1. 如果新容量大于原容量的2倍，则新容量等于所需容量
2. 如果原容量小于1024，则新容量为原容量的2倍
3. 如果原容量大于等于1024，则新容量为原容量的1.25倍
4. 最终容量还会根据元素类型进行内存对齐调整

### 8. Go语言中的map如何使用？有什么特点？

**答案：**

```go
// 声明和初始化
var m1 map[string]int              // nil map，不能直接赋值
m2 := make(map[string]int)         // 空map
m3 := map[string]int{              // 字面量初始化
    "apple":  5,
    "banana": 3,
}

// 基本操作
m2["key"] = 100                    // 赋值
value := m2["key"]                 // 取值
value, ok := m2["key"]             // 安全取值
delete(m2, "key")                  // 删除

// 遍历
for key, value := range m3 {
    fmt.Printf("%s: %d\n", key, value)
}

// 只遍历key
for key := range m3 {
    fmt.Println(key)
}
```

**特点：**

- 引用类型，零值为nil
- 键必须是可比较的类型
- 遍历顺序是随机的
- 并发读写不安全，需要使用sync.Map或加锁
- 扩容时可能发生rehash

### 9. 说说Go语言中的指针

**答案：**

```go
// 指针声明和使用
var p *int                         // 指针变量，零值为nil
var x int = 42
p = &x                            // 取地址

fmt.Println(*p)                   // 解引用，输出42
*p = 100                          // 通过指针修改值
fmt.Println(x)                    // 输出100

// 指针函数参数
func modify(p *int) {
    *p = 200
}

// 返回指针
func createInt() *int {
    x := 42
    return &x                     // 可以返回局部变量地址
}
```

**特点：**

- Go语言有指针但没有指针算术
- 不能对指针进行加减运算
- 自动垃圾回收，不需要手动释放内存
- 函数参数传递指针可以避免大对象复制

### 10. 解释Go语言中的接口（interface）

**答案：**

```go
// 接口定义
type Writer interface {
    Write([]byte) (int, error)
}

type Reader interface {
    Read([]byte) (int, error)
}

// 接口组合
type ReadWriter interface {
    Reader
    Writer
}

// 实现接口（隐式实现）
type File struct {
    name string
}

func (f *File) Write(data []byte) (int, error) {
    fmt.Printf("Writing %s to file %s\n", string(data), f.name)
    return len(data), nil
}

func (f *File) Read(data []byte) (int, error) {
    fmt.Printf("Reading from file %s\n", f.name)
    return 0, nil
}

// 使用接口
func process(rw ReadWriter) {
    data := []byte("hello")
    rw.Write(data)
    rw.Read(data)
}

// 空接口
var v interface{} = 42
var v2 interface{} = "hello"

// 类型断言
if str, ok := v2.(string); ok {
    fmt.Println("v2 is string:", str)
}

// 类型选择
switch v := v.(type) {
case int:
    fmt.Println("v is int:", v)
case string:
    fmt.Println("v is string:", v)
default:
    fmt.Println("unknown type")
}
```

**重要概念：**

- 接口是一组方法签名的集合
- 隐式实现，无需显式声明
- 空接口interface{}可以接受任何类型
- 接口值包含动态类型和动态值

------

## 并发编程

### 11. 什么是goroutine？如何创建和使用？

**答案：**

```go
import (
    "fmt"
    "time"
)

// 普通函数
func sayHello(name string) {
    for i := 0; i < 5; i++ {
        fmt.Printf("Hello %s %d\n", name, i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // 创建goroutine
    go sayHello("Goroutine")
    
    // 匿名函数goroutine
    go func() {
        fmt.Println("Anonymous goroutine")
    }()
    
    // 主goroutine
    sayHello("Main")
    
    // 等待goroutine完成
    time.Sleep(1 * time.Second)
}
```

**特点：**

- 轻量级线程，创建成本低
- 由Go运行时调度，不是OS线程
- 栈大小初始为2KB，可动态增长
- M:N调度模型（M个goroutine对应N个OS线程）

### 12. 什么是channel？如何使用？

**答案：**

```go
// 创建channel
ch1 := make(chan int)           // 无缓冲channel
ch2 := make(chan int, 10)       // 有缓冲channel，容量为10
ch3 := make(chan<- int)         // 只写channel
ch4 := make(<-chan int)         // 只读channel

// 基本操作
go func() {
    ch1 <- 42                   // 发送数据
}()

value := <-ch1                  // 接收数据
value, ok := <-ch1              // 接收数据，检查channel是否关闭

close(ch1)                      // 关闭channel

// 遍历channel
ch := make(chan int, 5)
go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}()

for value := range ch {
    fmt.Println(value)
}

// select语句
select {
case value := <-ch1:
    fmt.Println("Received from ch1:", value)
case ch2 <- 42:
    fmt.Println("Sent to ch2")
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
default:
    fmt.Println("No channel operation ready")
}
```

**Channel类型：**

- **无缓冲channel**：同步通信，发送和接收必须同时准备好
- **有缓冲channel**：异步通信，缓冲区未满时发送不阻塞

### 13. 解释Go语言的并发模型和调度器

**答案：**

**GMP模型：**

- **G (Goroutine)**：用户级线程，包含栈、程序计数器等信息
- **M (Machine)**：系统线程，真正执行计算的单元
- **P (Processor)**：逻辑处理器，维护goroutine队列

```go
// GOMAXPROCS设置逻辑处理器数量
import "runtime"

func main() {
    // 获取逻辑处理器数量
    fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))
    
    // 设置逻辑处理器数量
    runtime.GOMAXPROCS(4)
    
    // 获取当前goroutine数量
    fmt.Println("NumGoroutine:", runtime.NumGoroutine())
}
```

**调度策略：**

1. **协作式调度**：goroutine主动让出CPU
2. **抢占式调度**：运行时间过长会被强制调度
3. **工作窃取**：空闲的P会从其他P的队列中窃取G

### 14. 如何实现goroutine之间的同步？

**答案：**

**1. 使用WaitGroup：**

```go
import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }
    
    wg.Wait()
    fmt.Println("All workers completed")
}
```

**2. 使用Mutex：**

```go
import (
    "fmt"
    "sync"
)

type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}
```

**3. 使用Channel：**

```go
func main() {
    done := make(chan bool)
    
    go func() {
        fmt.Println("Goroutine working...")
        time.Sleep(time.Second)
        done <- true
    }()
    
    <-done
    fmt.Println("Goroutine completed")
}
```

### 15. 什么是竞态条件？如何避免？

**答案：**

**竞态条件示例：**

```go
// 错误示例：存在竞态条件
var counter int

func increment() {
    for i := 0; i < 1000; i++ {
        counter++  // 非原子操作，存在竞态条件
    }
}

func main() {
    go increment()
    go increment()
    
    time.Sleep(time.Second)
    fmt.Println("Counter:", counter)  // 结果不确定
}
```

**解决方案：**

**1. 使用Mutex：**

```go
var (
    counter int
    mu      sync.Mutex
)

func increment() {
    for i := 0; i < 1000; i++ {
        mu.Lock()
        counter++
        mu.Unlock()
    }
}
```

**2. 使用原子操作：**

```go
import "sync/atomic"

var counter int64

func increment() {
    for i := 0; i < 1000; i++ {
        atomic.AddInt64(&counter, 1)
    }
}
```

**3. 使用Channel：**

```go
func main() {
    ch := make(chan int)
    counter := 0
    
    // 串行处理
    go func() {
        for delta := range ch {
            counter += delta
        }
    }()
    
    // 并发发送
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 100; j++ {
                ch <- 1
            }
        }()
    }
    
    wg.Wait()
    close(ch)
}
```

------

## 内存管理与GC

### 16. Go语言的垃圾回收机制是怎样的？

**答案：**

**三色标记算法：**

- **白色**：未被访问的对象，回收候选
- **灰色**：已被访问但子对象未被扫描的对象
- **黑色**：已被访问且子对象已被扫描的对象

```go
import (
    "runtime"
    "runtime/debug"
    "time"
)

func main() {
    // 查看GC统计信息
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("Alloc = %d KB", bToKb(m.Alloc))
    fmt.Printf("TotalAlloc = %d KB", bToKb(m.TotalAlloc))
    fmt.Printf("Sys = %d KB", bToKb(m.Sys))
    fmt.Printf("NumGC = %v\n", m.NumGC)
    
    // 手动触发GC
    runtime.GC()
    
    // 设置GC目标百分比
    debug.SetGCPercent(50)  // 默认100%
    
    // 禁用GC
    debug.SetGCPercent(-1)
}

func bToKb(b uint64) uint64 {
    return b / 1024
}
```

**GC特点：**

- 并发垃圾回收器，STW时间很短
- 写屏障技术保证并发安全
- 分代假设优化（新对象更容易成为垃圾）

### 17. 什么情况下会发生内存泄漏？如何避免？

**答案：**

**常见内存泄漏场景：**

**1. Goroutine泄漏：**

```go
// 问题代码：goroutine无法正常退出
func problematicFunction() {
    ch := make(chan int)
    
    go func() {
        // 这个goroutine会一直阻塞，永远不会退出
        val := <-ch  // 没有发送者，永远阻塞
        fmt.Println(val)
    }()
    
    // 函数结束，但goroutine依然存在
}

// 正确做法：使用context或超时
func correctFunction() {
    ch := make(chan int)
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    go func() {
        select {
        case val := <-ch:
            fmt.Println(val)
        case <-ctx.Done():
            fmt.Println("Goroutine timeout")
            return
        }
    }()
}
```

**2. 切片内存泄漏：**

```go
// 问题代码：切片引用大数组
func problematicSlice() []int {
    bigArray := make([]int, 1000000)
    // 填充数据...
    
    // 返回小切片，但底层数组仍被引用
    return bigArray[:5]  // 内存泄漏
}

// 正确做法：复制数据
func correctSlice() []int {
    bigArray := make([]int, 1000000)
    // 填充数据...
    
    result := make([]int, 5)
    copy(result, bigArray[:5])
    return result  // bigArray可以被GC回收
}
```

**3. 定时器泄漏：**

```go
// 问题代码：定时器未停止
func problematicTimer() {
    ticker := time.NewTicker(1 * time.Second)
    // 忘记停止定时器
    // ticker.Stop()  // 需要调用这个
}

// 正确做法
func correctTimer() {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()  // 确保定时器被停止
    
    // 使用定时器...
}
```

### 18. 解释Go语言中的逃逸分析

**答案：**

```go
// 查看逃逸分析
// go build -gcflags="-m" main.go

// 情况1：变量逃逸到堆
func createPointer() *int {
    x := 42  // x逃逸到堆，因为返回了它的地址
    return &x
}

// 情况2：变量在栈上
func useLocal() {
    x := 42  // x在栈上，函数结束后自动释放
    fmt.Println(x)
}

// 情况3：接口导致逃逸
func printValue(v interface{}) {
    fmt.Println(v)  // v逃逸到堆
}

// 情况4：切片扩容导致逃逸
func appendSlice() {
    s := make([]int, 0, 10)  // 初始在栈上
    for i := 0; i < 100; i++ {
        s = append(s, i)  // 扩容后逃逸到堆
    }
}

// 情况5：闭包捕获变量
func createClosure() func() int {
    x := 42
    return func() int {
        return x  // x逃逸到堆，被闭包捕获
    }
}
```

**逃逸分析规则：**

- 返回局部变量的指针
- 变量被闭包捕获
- 变量赋值给接口
- 切片/map动态扩容
- 变量过大（>64KB）

------

## 网络编程

### 19. 如何使用Go语言创建HTTP服务器？

**答案：**

**基础HTTP服务器：**

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

// 处理器函数
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

// JSON响应处理器
func jsonHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message": "Hello JSON",
        "time":    time.Now(),
        "method":  r.Method,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// 中间件示例
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    }
}

func main() {
    // 路由设置
    http.HandleFunc("/", loggingMiddleware(helloHandler))
    http.HandleFunc("/json", loggingMiddleware(jsonHandler))
    
    // 静态文件服务
    fs := http.FileServer(http.Dir("./static/"))
    http.Handle("/static/", http.StripPrefix("/static/", fs))
    
    // 启动服务器
    server := &http.Server{
        Addr:         ":8080",
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  15 * time.Second,
    }
    
    log.Println("Server starting on :8080")
    log.Fatal(server.ListenAndServe())
}
```

### 20. 如何处理HTTP请求的不同方法（GET、POST等）？

**答案：**

```go
func apiHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        handleGet(w, r)
    case http.MethodPost:
        handlePost(w, r)
    case http.MethodPut:
        handlePut(w, r)
    case http.MethodDelete:
        handleDelete(w, r)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func handleGet(w http.ResponseWriter, r *http.Request) {
    // 获取查询参数
    id := r.URL.Query().Get("id")
    name := r.URL.Query().Get("name")
    
    // 获取路径参数
    path := r.URL.Path
    
    response := map[string]string{
        "method": "GET",
        "id":     id,
        "name":   name,
        "path":   path,
    }
    
    json.NewEncoder(w).Encode(response)
}

func handlePost(w http.ResponseWriter, r *http.Request) {
    // 解析JSON请求体
    var requestData map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&requestData); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    defer r.Body.Close()
    
    // 解析表单数据
    if err := r.ParseForm(); err != nil {
        http.Error(w, "Failed to parse form", http.StatusBadRequest)
        return
    }
    
    formData := r.Form.Get("data")
    
    response := map[string]interface{}{
        "method":   "POST",
        "json":     requestData,
        "form":     formData,
        "headers":  r.Header,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// 文件上传处理
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Only POST allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // 限制上传大小（10MB）
    r.ParseMultipartForm(10 << 20)
    
    file, handler, err := r.FormFile("file")
    if err != nil {
        http.Error(w, "Error retrieving file", http.StatusBadRequest)
        return
    }
    defer file.Close()
    
    // 创建目标文件
    dst, err := os.Create("./uploads/" + handler.Filename)
    if err != nil {
        http.Error(w, "Error creating file", http.StatusInternalServerError)
        return
    }
    defer dst.Close()
    
    // 复制文件内容
    if _, err := io.Copy(dst, file); err != nil {
        http.Error(w, "Error saving file", http.StatusInternalServerError)
        return
    }
    
    fmt.Fprintf(w, "File uploaded successfully: %s", handler.Filename)
}
```

### 21. 如何创建HTTP客户端并发送请求？

**答案：**

```go
package main

import (
    "bytes"
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

// HTTP客户端封装
type HTTPClient struct {
    client *http.Client
}

func NewHTTPClient(timeout time.Duration) *HTTPClient {
    return &HTTPClient{
        client: &http.Client{
            Timeout: timeout,
            Transport: &http.Transport{
                MaxIdleConns:        100,
                MaxIdleConnsPerHost: 10,
                IdleConnTimeout:     90 * time.Second,
            },
        },
    }
}

// GET请求
func (c *HTTPClient) Get(url string, headers map[string]string) ([]byte, error) {
    req, err := http.NewRequest(http.MethodGet, url, nil)
    if err != nil {
        return nil, err
    }
    
    // 添加请求头
    for key, value := range headers {
        req.Header.Set(key, value)
    }
    
    resp, err := c.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("HTTP error: %d", resp.StatusCode)
    }
    
    return io.ReadAll(resp.Body)
}

// POST请求
func (c *HTTPClient) Post(url string, data interface{}, headers map[string]string) ([]byte, error) {
    jsonData, err := json.Marshal(data)
    if err != nil {
        return nil, err
    }
    
    req, err := http.NewRequest(http.MethodPost, url, bytes.NewBuffer(jsonData))
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Content-Type", "application/json")
    for key, value := range headers {
        req.Header.Set(key, value)
    }
    
    resp, err := c.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}

// 带超时的请求
func (c *HTTPClient) GetWithTimeout(url string, timeout time.Duration) ([]byte, error) {
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := c.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}

// 使用示例
func main() {
    client := NewHTTPClient(30 * time.Second)
    
    // GET请求
    data, err := client.Get("https://api.github.com/users/octocat", map[string]string{
        "User-Agent": "Go-HTTP-Client",
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(data))
    
    // POST请求
    postData := map[string]interface{}{
        "name":  "test",
        "value": 123,
    }
    
    respData, err := client.Post("https://httpbin.org/post", postData, nil)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(respData))
}
```

### 22. 如何使用Go语言创建WebSocket服务？

**答案：**

```go
// 需要使用第三方库：go get github.com/gorilla/websocket
package main

import (
    "log"
    "net/http"
    "github.com/gorilla/websocket"
)

// WebSocket升级器
var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true  // 允许所有来源，生产环境需要验证
    },
}

// 客户端连接结构
type Client struct {
    conn   *websocket.Conn
    send   chan []byte
    hub    *Hub
    id     string
}

// 消息中心
type Hub struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
}

func newHub() *Hub {
    return &Hub{
        clients:    make(map[*Client]bool),
        broadcast:  make(chan []byte),
        register:   make(chan *Client),
        unregister: make(chan *Client),
    }
}

func (h *Hub) run() {
    for {
        select {
        case client := <-h.register:
            h.clients[client] = true
            log.Printf("Client %s connected", client.id)
            
        case client := <-h.unregister:
            if _, ok := h.clients[client]; ok {
                delete(h.clients, client)
                close(client.send)
                log.Printf("Client %s disconnected", client.id)
            }
            
        case message := <-h.broadcast:
            for client := range h.clients {
                select {
                case client.send <- message:
                default:
                    close(client.send)
                    delete(h.clients, client)
                }
            }
        }
    }
}

// WebSocket处理器
func wsHandler(hub *Hub, w http.ResponseWriter, r *http.Request) {
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println("WebSocket upgrade error:", err)
        return
    }
    
    client := &Client{
        conn: conn,
        send: make(chan []byte, 256),
        hub:  hub,
        id:   r.Header.Get("X-Client-ID"),
    }
    
    client.hub.register <- client
    
    // 启动读写goroutine
    go client.writePump()
    go client.readPump()
}

func (c *Client) readPump() {
    defer func() {
        c.hub.unregister <- c
        c.conn.Close()
    }()
    
    for {
        _, message, err := c.conn.ReadMessage()
        if err != nil {
            break
        }
        
        // 广播消息给所有客户端
        c.hub.broadcast <- message
    }
}

func (c *Client) writePump() {
    defer c.conn.Close()
    
    for {
        select {
        case message, ok := <-c.send:
            if !ok {
                c.conn.WriteMessage(websocket.CloseMessage, []byte{})
                return
            }
            
            c.conn.WriteMessage(websocket.TextMessage, message)
        }
    }
}

func main() {
    hub := newHub()
    go hub.run()
    
    http.HandleFunc("/ws", func(w http.ResponseWriter, r *http.Request) {
        wsHandler(hub, w, r)
    })
    
    log.Println("WebSocket server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

------

## 数据库操作

### 23. 如何在Go语言中连接和操作MySQL数据库？

**答案：**

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "time"
    
    _ "github.com/go-sql-driver/mysql"
)

// 用户结构体
type User struct {
    ID       int       `json:"id"`
    Name     string    `json:"name"`
    Email    string    `json:"email"`
    CreateAt time.Time `json:"created_at"`
}

// 数据库配置
type DBConfig struct {
    Host     string
    Port     int
    Username string
    Password string
    Database string
}

// 数据库连接
func NewDB(config DBConfig) (*sql.DB, error) {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local",
        config.Username, config.Password, config.Host, config.Port, config.Database)
    
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }
    
    // 设置连接池参数
    db.SetMaxOpenConns(100)
    db.SetMaxIdleConns(10)
    db.SetConnMaxLifetime(time.Hour)
    
    // 测试连接
    if err := db.Ping(); err != nil {
        return nil, err
    }
    
    return db, nil
}

// 用户数据访问对象
type UserDAO struct {
    db *sql.DB
}

func NewUserDAO(db *sql.DB) *UserDAO {
    return &UserDAO{db: db}
}

// 创建用户
func (dao *UserDAO) CreateUser(user *User) error {
    query := `INSERT INTO users (name, email) VALUES (?, ?)`
    result, err := dao.db.Exec(query, user.Name, user.Email)
    if err != nil {
        return err
    }
    
    id, err := result.LastInsertId()
    if err != nil {
        return err
    }
    
    user.ID = int(id)
    return nil
}

// 根据ID获取用户
func (dao *UserDAO) GetUserByID(id int) (*User, error) {
    query := `SELECT id, name, email, created_at FROM users WHERE id = ?`
    row := dao.db.QueryRow(query, id)
    
    user := &User{}
    err := row.Scan(&user.ID, &user.Name, &user.Email, &user.CreateAt)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, nil  // 用户不存在
        }
        return nil, err
    }
    
    return user, nil
}

// 获取所有用户
func (dao *UserDAO) GetAllUsers() ([]*User, error) {
    query := `SELECT id, name, email, created_at FROM users ORDER BY id`
    rows, err := dao.db.Query(query)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []*User
    for rows.Next() {
        user := &User{}
        err := rows.Scan(&user.ID, &user.Name, &user.Email, &user.CreateAt)
        if err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    
    return users, nil
}

// 更新用户
func (dao *UserDAO) UpdateUser(user *User) error {
    query := `UPDATE users SET name = ?, email = ? WHERE id = ?`
    result, err := dao.db.Exec(query, user.Name, user.Email, user.ID)
    if err != nil {
        return err
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return err
    }
    
    if rowsAffected == 0 {
        return fmt.Errorf("user with id %d not found", user.ID)
    }
    
    return nil
}

// 删除用户
func (dao *UserDAO) DeleteUser(id int) error {
    query := `DELETE FROM users WHERE id = ?`
    result, err := dao.db.Exec(query, id)
    if err != nil {
        return err
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return err
    }
    
    if rowsAffected == 0 {
        return fmt.Errorf("user with id %d not found", id)
    }
    
    return nil
}

// 事务操作示例
func (dao *UserDAO) TransferUserData(fromID, toID int) error {
    tx, err := dao.db.Begin()
    if err != nil {
        return err
    }
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
        }
    }()
    
    // 执行第一个操作
    _, err = tx.Exec("UPDATE users SET balance = balance - 100 WHERE id = ?", fromID)
    if err != nil {
        tx.Rollback()
        return err
    }
    
    // 执行第二个操作
    _, err = tx.Exec("UPDATE users SET balance = balance + 100 WHERE id = ?", toID)
    if err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}

// 使用示例
func main() {
    config := DBConfig{
        Host:     "localhost",
        Port:     3306,
        Username: "root",
        Password: "password",
        Database: "testdb",
    }
    
    db, err := NewDB(config)
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }
    defer db.Close()
    
    userDAO := NewUserDAO(db)
    
    // 创建用户
    user := &User{
        Name:  "张三",
        Email: "zhangsan@example.com",
    }
    
    if err := userDAO.CreateUser(user); err != nil {
        log.Fatal("Failed to create user:", err)
    }
    
    fmt.Printf("Created user with ID: %d\n", user.ID)
    
    // 查询用户
    foundUser, err := userDAO.GetUserByID(user.ID)
    if err != nil {
        log.Fatal("Failed to get user:", err)
    }
    
    fmt.Printf("Found user: %+v\n", foundUser)
}
```

### 24. 如何使用GORM进行数据库操作？

**答案：**

```go
package main

import (
    "log"
    "time"
    
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

// 用户模型
type User struct {
    ID        uint      `gorm:"primaryKey" json:"id"`
    Name      string    `gorm:"size:100;not null" json:"name"`
    Email     string    `gorm:"size:100;unique;not null" json:"email"`
    Age       int       `json:"age"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
    Profile   Profile   `gorm:"foreignKey:UserID" json:"profile"`
    Orders    []Order   `gorm:"foreignKey:UserID" json:"orders"`
}

// 用户资料模型
type Profile struct {
    ID     uint   `gorm:"primaryKey" json:"id"`
    UserID uint   `json:"user_id"`
    Avatar string `json:"avatar"`
    Bio    string `json:"bio"`
}

// 订单模型
type Order struct {
    ID       uint      `gorm:"primaryKey" json:"id"`
    UserID   uint      `json:"user_id"`
    Amount   float64   `json:"amount"`
    Status   string    `gorm:"default:'pending'" json:"status"`
    CreateAt time.Time `json:"created_at"`
}

// 数据库初始化
func InitDB() *gorm.DB {
    dsn := "root:password@tcp(127.0.0.1:3306)/testdb?charset=utf8mb4&parseTime=True&loc=Local"
    
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }
    
    // 自动迁移
    err = db.AutoMigrate(&User{}, &Profile{}, &Order{})
    if err != nil {
        log.Fatal("Failed to migrate database:", err)
    }
    
    return db
}

// 用户服务
type UserService struct {
    db *gorm.DB
}

func NewUserService(db *gorm.DB) *UserService {
    return &UserService{db: db}
}

// 创建用户
func (s *UserService) CreateUser(user *User) error {
    return s.db.Create(user).Error
}

// 根据ID获取用户（包含关联数据）
func (s *UserService) GetUserByID(id uint) (*User, error) {
    var user User
    err := s.db.Preload("Profile").Preload("Orders").First(&user, id).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

// 分页获取用户列表
func (s *UserService) GetUsers(page, pageSize int) ([]User, int64, error) {
    var users []User
    var total int64
    
    // 计算总数
    if err := s.db.Model(&User{}).Count(&total).Error; err != nil {
        return nil, 0, err
    }
    
    // 分页查询
    offset := (page - 1) * pageSize
    err := s.db.Offset(offset).Limit(pageSize).Find(&users).Error
    if err != nil {
        return nil, 0, err
    }
    
    return users, total, nil
}

// 更新用户
func (s *UserService) UpdateUser(id uint, updates map[string]interface{}) error {
    return s.db.Model(&User{}).Where("id = ?", id).Updates(updates).Error
}

// 软删除用户
func (s *UserService) DeleteUser(id uint) error {
    return s.db.Delete(&User{}, id).Error
}

// 条件查询
func (s *UserService) FindUsers(conditions map[string]interface{}) ([]User, error) {
    var users []User
    query := s.db
    
    for key, value := range conditions {
        query = query.Where(key+" = ?", value)
    }
    
    err := query.Find(&users).Error
    return users, err
}

// 复杂查询示例
func (s *UserService) GetActiveUsersWithOrders() ([]User, error) {
    var users []User
    err := s.db.
        Preload("Orders", "status = ?", "completed").
        Where("age >= ?", 18).
        Find(&users).Error
    
    return users, err
}

// 原生SQL查询
func (s *UserService) GetUserStatistics() (map[string]interface{}, error) {
    var result struct {
        TotalUsers int     `json:"total_users"`
        AvgAge     float64 `json:"avg_age"`
        MaxAge     int     `json:"max_age"`
        MinAge     int     `json:"min_age"`
    }
    
    err := s.db.Raw(`
        SELECT 
            COUNT(*) as total_users,
            AVG(age) as avg_age,
            MAX(age) as max_age,
            MIN(age) as min_age
        FROM users
    `).Scan(&result).Error
    
    if err != nil {
        return nil, err
    }
    
    return map[string]interface{}{
        "total_users": result.TotalUsers,
        "avg_age":     result.AvgAge,
        "max_age":     result.MaxAge,
        "min_age":     result.MinAge,
    }, nil
}

// 事务操作
func (s *UserService) CreateUserWithProfile(user *User, profile *Profile) error {
    return s.db.Transaction(func(tx *gorm.DB) error {
        // 创建用户
        if err := tx.Create(user).Error; err != nil {
            return err
        }
        
        // 设置profile的用户ID
        profile.UserID = user.ID
        
        // 创建profile
        if err := tx.Create(profile).Error; err != nil {
            return err
        }
        
        return nil
    })
}

// 使用示例
func main() {
    db := InitDB()
    userService := NewUserService(db)
    
    // 创建用户
    user := &User{
        Name:  "李四",
        Email: "lisi@example.com",
        Age:   25,
    }
    
    profile := &Profile{
        Avatar: "avatar.jpg",
        Bio:    "这是一个测试用户",
    }
    
    if err := userService.CreateUserWithProfile(user, profile); err != nil {
        log.Fatal("Failed to create user with profile:", err)
    }
    
    log.Printf("Created user: %+v", user)
}


```

### 25. 如何处理数据库连接池？

**答案：**

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "log"
    "sync"
    "time"
    
    _ "github.com/go-sql-driver/mysql"
)

// 数据库连接池管理器
type DBPool struct {
    db     *sql.DB
    config PoolConfig
    mutex  sync.RWMutex
    stats  PoolStats
}

type PoolConfig struct {
    MaxOpenConns    int           // 最大打开连接数
    MaxIdleConns    int           // 最大空闲连接数
    ConnMaxLifetime time.Duration // 连接最大生命周期
    ConnMaxIdleTime time.Duration // 连接最大空闲时间
}

type PoolStats struct {
    OpenConnections int
    IdleConnections int
    WaitCount       int64
    WaitDuration    time.Duration
}

func NewDBPool(dsn string, config PoolConfig) (*DBPool, error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }
    
    // 设置连接池参数
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    db.SetConnMaxLifetime(config.ConnMaxLifetime)
    db.SetConnMaxIdleTime(config.ConnMaxIdleTime)
    
    // 测试连接
    if err := db.Ping(); err != nil {
        return nil, err
    }
    
    pool := &DBPool{
        db:     db,
        config: config,
    }
    
    // 启动监控goroutine
    go pool.monitor()
    
    return pool, nil
}

// 监控连接池状态
func (p *DBPool) monitor() {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        stats := p.db.Stats()
        
        p.mutex.Lock()
        p.stats = PoolStats{
            OpenConnections: stats.OpenConnections,
            IdleConnections: stats.Idle,
            WaitCount:       stats.WaitCount,
            WaitDuration:    stats.WaitDuration,
        }
        p.mutex.Unlock()
        
        log.Printf("DB Pool Stats - Open: %d, Idle: %d, Wait: %d, WaitDuration: %v",
            stats.OpenConnections, stats.Idle, stats.WaitCount, stats.WaitDuration)
        
        // 检查连接池健康状态
        if stats.WaitCount > 100 {
            log.Println("Warning: High wait count detected, consider increasing MaxOpenConns")
        }
        
        if stats.WaitDuration > 5*time.Second {
            log.Println("Warning: High wait duration detected, database may be slow")
        }
    }
}

// 获取连接池统计信息
func (p *DBPool) GetStats() PoolStats {
    p.mutex.RLock()
    defer p.mutex.RUnlock()
    return p.stats
}

// 执行查询（带超时）
func (p *DBPool) QueryWithTimeout(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    return p.db.QueryContext(ctx, query, args...)
}

// 执行事务（带超时）
func (p *DBPool) TransactionWithTimeout(ctx context.Context, fn func(*sql.Tx) error) error {
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()
    
    tx, err := p.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
            panic(r)
        }
    }()
    
    if err := fn(tx); err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}

// 健康检查
func (p *DBPool) HealthCheck() error {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    return p.db.PingContext(ctx)
}

// 关闭连接池
func (p *DBPool) Close() error {
    return p.db.Close()
}

// 使用示例
func main() {
    config := PoolConfig{
        MaxOpenConns:    50,
        MaxIdleConns:    10,
        ConnMaxLifetime: time.Hour,
        ConnMaxIdleTime: 10 * time.Minute,
    }
    
    dsn := "root:password@tcp(localhost:3306)/testdb?charset=utf8mb4&parseTime=True&loc=Local"
    pool, err := NewDBPool(dsn, config)
    if err != nil {
        log.Fatal("Failed to create DB pool:", err)
    }
    defer pool.Close()
    
    // 并发测试连接池
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            ctx := context.Background()
            rows, err := pool.QueryWithTimeout(ctx, "SELECT * FROM users LIMIT 10")
            if err != nil {
                log.Printf("Query failed for goroutine %d: %v", id, err)
                return
            }
            defer rows.Close()
            
            // 处理结果...
            log.Printf("Goroutine %d completed query", id)
        }(i)
    }
    
    wg.Wait()
    
    // 打印最终统计信息
    stats := pool.GetStats()
    fmt.Printf("Final stats: %+v\n", stats)
}


```

---

## 微服务与分布式

### 26. 如何设计和实现RESTful API？

**答案：**

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
    
    "github.com/gorilla/mux"
)

// API响应结构
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Message string      `json:"message,omitempty"`
    Error   string      `json:"error,omitempty"`
}

// 分页响应结构
type PaginatedResponse struct {
    Data       interface{} `json:"data"`
    Total      int64       `json:"total"`
    Page       int         `json:"page"`
    PageSize   int         `json:"page_size"`
    TotalPages int         `json:"total_pages"`
}

// 用户API控制器
type UserController struct {
    userService *UserService
}

func NewUserController(userService *UserService) *UserController {
    return &UserController{userService: userService}
}

// 创建用户 POST /api/v1/users
func (c *UserController) CreateUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        c.respondError(w, http.StatusBadRequest, "Invalid JSON format")
        return
    }
    
    // 验证输入
    if err := c.validateUser(&user); err != nil {
        c.respondError(w, http.StatusBadRequest, err.Error())
        return
    }
    
    if err := c.userService.CreateUser(&user); err != nil {
        c.respondError(w, http.StatusInternalServerError, "Failed to create user")
        return
    }
    
    c.respondSuccess(w, http.StatusCreated, user)
}

// 获取用户 GET /api/v1/users/{id}
func (c *UserController) GetUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id, err := strconv.ParseUint(vars["id"], 10, 32)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    user, err := c.userService.GetUserByID(uint(id))
    if err != nil {
        c.respondError(w, http.StatusNotFound, "User not found")
        return
    }
    
    c.respondSuccess(w, http.StatusOK, user)
}

// 获取用户列表 GET /api/v1/users
func (c *UserController) GetUsers(w http.ResponseWriter, r *http.Request) {
    // 解析查询参数
    page, _ := strconv.Atoi(r.URL.Query().Get("page"))
    if page < 1 {
        page = 1
    }
    
    pageSize, _ := strconv.Atoi(r.URL.Query().Get("page_size"))
    if pageSize < 1 || pageSize > 100 {
        pageSize = 10
    }
    
    users, total, err := c.userService.GetUsers(page, pageSize)
    if err != nil {
        c.respondError(w, http.StatusInternalServerError, "Failed to get users")
        return
    }
    
    totalPages := int((total + int64(pageSize) - 1) / int64(pageSize))
    
    response := PaginatedResponse{
        Data:       users,
        Total:      total,
        Page:       page,
        PageSize:   pageSize,
        TotalPages: totalPages,
    }
    
    c.respondSuccess(w, http.StatusOK, response)
}

// 更新用户 PUT /api/v1/users/{id}
func (c *UserController) UpdateUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id, err := strconv.ParseUint(vars["id"], 10, 32)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    var updates map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&updates); err != nil {
        c.respondError(w, http.StatusBadRequest, "Invalid JSON format")
        return
    }
    
    if err := c.userService.UpdateUser(uint(id), updates); err != nil {
        c.respondError(w, http.StatusInternalServerError, "Failed to update user")
        return
    }
    
    c.respondSuccess(w, http.StatusOK, map[string]string{"message": "User updated successfully"})
}

// 删除用户 DELETE /api/v1/users/{id}
func (c *UserController) DeleteUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    id, err := strconv.ParseUint(vars["id"], 10, 32)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    if err := c.userService.DeleteUser(uint(id)); err != nil {
        c.respondError(w, http.StatusInternalServerError, "Failed to delete user")
        return
    }
    
    c.respondSuccess(w, http.StatusOK, map[string]string{"message": "User deleted successfully"})
}

// 用户搜索 GET /api/v1/users/search
func (c *UserController) SearchUsers(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query()
    conditions := make(map[string]interface{})
    
    if name := query.Get("name"); name != "" {
        conditions["name LIKE"] = "%" + name + "%"
    }
    
    if email := query.Get("email"); email != "" {
        conditions["email"] = email
    }
    
    if ageStr := query.Get("age"); ageStr != "" {
        if age, err := strconv.Atoi(ageStr); err == nil {
            conditions["age"] = age
        }
    }
    
    users, err := c.userService.FindUsers(conditions)
    if err != nil {
        c.respondError(w, http.StatusInternalServerError, "Failed to search users")
        return
    }
    
    c.respondSuccess(w, http.StatusOK, users)
}

// 中间件：请求日志
func (c *UserController) LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // 包装ResponseWriter以捕获状态码
        wrapped := &responseWriter{ResponseWriter: w, statusCode: http.StatusOK}
        
        next.ServeHTTP(wrapped, r)
        
        log.Printf("%s %s %d %v", r.Method, r.URL.Path, wrapped.statusCode, time.Since(start))
    })
}

// 中间件：CORS
func (c *UserController) CORSMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
        
        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// 中间件：认证
func (c *UserController) AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            c.respondError(w, http.StatusUnauthorized, "Missing authorization header")
            return
        }
        
        // 验证token（这里简化处理）
        if !strings.HasPrefix(authHeader, "Bearer ") {
            c.respondError(w, http.StatusUnauthorized, "Invalid authorization format")
            return
        }
        
        token := strings.TrimPrefix(authHeader, "Bearer ")
        if !c.validateToken(token) {
            c.respondError(w, http.StatusUnauthorized, "Invalid token")
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// 辅助方法
func (c *UserController) respondSuccess(w http.ResponseWriter, statusCode int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)
    
    response := APIResponse{
        Success: true,
        Data:    data,
    }
    
    json.NewEncoder(w).Encode(response)
}

func (c *UserController) respondError(w http.ResponseWriter, statusCode int, message string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)
    
    response := APIResponse{
        Success: false,
        Error:   message,
    }
    
    json.NewEncoder(w).Encode(response)
}

func (c *UserController) validateUser(user *User) error {
    if user.Name == "" {
        return fmt.Errorf("name is required")
    }
    if user.Email == "" {
        return fmt.Errorf("email is required")
    }
    // 更多验证逻辑...
    return nil
}

func (c *UserController) validateToken(token string) bool {
    // 实际项目中应该验证JWT token
    return token == "valid-token"
}

// ResponseWriter包装器
type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}

// 路由设置
func SetupRoutes(userController *UserController) *mux.Router {
    r := mux.NewRouter()
    
    // API版本前缀
    api := r.PathPrefix("/api/v1").Subrouter()
    
    // 应用全局中间件
    api.Use(userController.LoggingMiddleware)
    api.Use(userController.CORSMiddleware)
    
    // 公开路由
    api.HandleFunc("/users", userController.GetUsers).Methods("GET")
    api.HandleFunc("/users/{id:[0-9]+}", userController.GetUser).Methods("GET")
    api.HandleFunc("/users/search", userController.SearchUsers).Methods("GET")
    
    // 需要认证的路由
    protected := api.PathPrefix("").Subrouter()
    protected.Use(userController.AuthMiddleware)
    protected.HandleFunc("/users", userController.CreateUser).Methods("POST")
    protected.HandleFunc("/users/{id:[0-9]+}", userController.UpdateUser).Methods("PUT")
    protected.HandleFunc("/users/{id:[0-9]+}", userController.DeleteUser).Methods("DELETE")
    
    return r
}

func main() {
    // 初始化依赖
    db := InitDB()
    userService := NewUserService(db)
    userController := NewUserController(userService)
    
    // 设置路由
    router := SetupRoutes(userController)
    
    // 启动服务器
    server := &http.Server{
        Addr:         ":8080",
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }
    
    log.Println("Server starting on :8080")
    log.Fatal(server.ListenAndServe())
}


```

### 27. 如何实现服务发现和负载均衡？

**答案：**

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "math/rand"
    "net/http"
    "sync"
    "time"
)

// 服务实例
type ServiceInstance struct {
    ID       string            `json:"id"`
    Name     string            `json:"name"`
    Address  string            `json:"address"`
    Port     int               `json:"port"`
    Metadata map[string]string `json:"metadata"`
    Health   bool              `json:"health"`
    LastSeen time.Time         `json:"last_seen"`
}

func (s *ServiceInstance) URL() string {
    return fmt.Sprintf("http://%s:%d", s.Address, s.Port)
}

// 服务注册表
type ServiceRegistry struct {
    services map[string][]*ServiceInstance
    mutex    sync.RWMutex
}

func NewServiceRegistry() *ServiceRegistry {
    registry := &ServiceRegistry{
        services: make(map[string][]*ServiceInstance),
    }
    
    // 启动健康检查
    go registry.healthCheck()
    
    return registry
}

// 注册服务
func (r *ServiceRegistry) Register(instance *ServiceInstance) {
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    instance.Health = true
    instance.LastSeen = time.Now()
    
    if r.services[instance.Name] == nil {
        r.services[instance.Name] = make([]*ServiceInstance, 0)
    }
    
    // 检查是否已存在
    for i, existing := range r.services[instance.Name] {
        if existing.ID == instance.ID {
            r.services[instance.Name][i] = instance
            return
        }
    }
    
    // 添加新实例
    r.services[instance.Name] = append(r.services[instance.Name], instance)
    log.Printf("Service registered: %s (%s)", instance.Name, instance.ID)
}

// 注销服务
func (r *ServiceRegistry) Deregister(serviceName, instanceID string) {
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    instances := r.services[serviceName]
    for i, instance := range instances {
        if instance.ID == instanceID {
            r.services[serviceName] = append(instances[:i], instances[i+1:]...)
            log.Printf("Service deregistered: %s (%s)", serviceName, instanceID)
            break
        }
    }
}

// 发现服务
func (r *ServiceRegistry) Discover(serviceName string) []*ServiceInstance {
    r.mutex.RLock()
    defer r.mutex.RUnlock()
    
    instances := r.services[serviceName]
    healthyInstances := make([]*ServiceInstance, 0)
    
    for _, instance := range instances {
        if instance.Health {
            healthyInstances = append(healthyInstances, instance)
        }
    }
    
    return healthyInstances
}

// 健康检查
func (r *ServiceRegistry) healthCheck() {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        r.mutex.Lock()
        for serviceName, instances := range r.services {
            for _, instance := range instances {
                go r.checkInstanceHealth(serviceName, instance)
            }
        }
        r.mutex.Unlock()
    }
}

func (r *ServiceRegistry) checkInstanceHealth(serviceName string, instance *ServiceInstance) {
    client := &http.Client{Timeout: 5 * time.Second}
    
    resp, err := client.Get(instance.URL() + "/health")
    if err != nil || resp.StatusCode != http.StatusOK {
        r.mutex.Lock()
        instance.Health = false
        r.mutex.Unlock()
        log.Printf("Health check failed for %s (%s)", serviceName, instance.ID)
        return
    }
    
    r.mutex.Lock()
    instance.Health = true
    instance.LastSeen = time.Now()
    r.mutex.Unlock()
}

// 负载均衡器接口
type LoadBalancer interface {
    Select(instances []*ServiceInstance) *ServiceInstance
}

// 轮询负载均衡
type RoundRobinBalancer struct {
    current int
    mutex   sync.Mutex
}

func (r *RoundRobinBalancer) Select(instances []*ServiceInstance) *ServiceInstance {
    if len(instances) == 0 {
        return nil
    }
    
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    instance := instances[r.current%len(instances)]
    r.current++
    
    return instance
}

// 随机负载均衡
type RandomBalancer struct{}

func (r *RandomBalancer) Select(instances []*ServiceInstance) *ServiceInstance {
    if len(instances) == 0 {
        return nil
    }
    
    return instances[rand.Intn(len(instances))]
}

// 加权随机负载均衡
type WeightedRandomBalancer struct{}

func (w *WeightedRandomBalancer) Select(instances []*ServiceInstance) *ServiceInstance {
    if len(instances) == 0 {
        return nil
    }
    
    totalWeight := 0
    for _, instance := range instances {
        if weight := instance.Metadata["weight"]; weight != "" {
            if w, err := strconv.Atoi(weight); err == nil {
                totalWeight += w
            } else {
                totalWeight += 1
            }
        } else {
            totalWeight += 1
        }
    }
    
    if totalWeight == 0 {
        return instances[rand.Intn(len(instances))]
    }
    
    randomWeight := rand.Intn(totalWeight)
    currentWeight := 0
    
    for _, instance := range instances {
        weight := 1
        if w := instance.Metadata["weight"]; w != "" {
            if parsed, err := strconv.Atoi(w); err == nil {
                weight = parsed
            }
        }
        
        currentWeight += weight
        if currentWeight > randomWeight {
            return instance
        }
    }
    
    return instances[len(instances)-1]
}

// 服务发现客户端
type ServiceDiscoveryClient struct {
    registry     *ServiceRegistry
    loadBalancer LoadBalancer
    httpClient   *http.Client
}

func NewServiceDiscoveryClient(registry *ServiceRegistry, balancer LoadBalancer) *ServiceDiscoveryClient {
    return &ServiceDiscoveryClient{
        registry:     registry,
        loadBalancer: balancer,
        httpClient:   &http.Client{Timeout: 10 * time.Second},
    }
}

// 调用服务
func (c *ServiceDiscoveryClient) CallService(serviceName, path string, method string, body interface{}) (*http.Response, error) {
    instances := c.registry.Discover(serviceName)
    if len(instances) == 0 {
        return nil, fmt.Errorf("no healthy instances found for service: %s", serviceName)
    }
    
    instance := c.loadBalancer.Select(instances)
    if instance == nil {
        return nil, fmt.Errorf("load balancer returned no instance")
    }
    
    url := instance.URL() + path
    
    var req *http.Request
    var err error
    
    if body != nil {
        jsonBody, err := json.Marshal(body)
        if err != nil {
            return nil, err
        }
        req, err = http.NewRequest(method, url, bytes.NewBuffer(jsonBody))
        if err != nil {
            return nil, err
        }
        req.Header.Set("Content-Type", "application/json")
    } else {
        req, err = http.NewRequest(method, url, nil)
        if err != nil {
            return nil, err
        }
    }
    
    return c.httpClient.Do(req)
}

// 服务提供者
type ServiceProvider struct {
    instance *ServiceInstance
    registry *ServiceRegistry
    server   *http.Server
}

func NewServiceProvider(instance *ServiceInstance, registry *ServiceRegistry) *ServiceProvider {
    return &ServiceProvider{
        instance: instance,
        registry: registry,
    }
}

func (p *ServiceProvider) Start() error {
    mux := http.NewServeMux()
    
    // 健康检查端点
    mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
    })
    
    // 业务端点
    mux.HandleFunc("/api/hello", func(w http.ResponseWriter, r *http.Request) {
        response := map[string]string{
            "message":     "Hello from " + p.instance.ID,
            "instance_id": p.instance.ID,
            "timestamp":   time.Now().Format(time.RFC3339),
        }
        json.NewEncoder(w).Encode(response)
    })
    
    p.server = &http.Server{
        Addr:    fmt.Sprintf(":%d", p.instance.Port),
        Handler: mux,
    }
    
    // 注册服务
    p.registry.Register(p.instance)
    
    // 启动服务器
    log.Printf("Service %s starting on port %d", p.instance.ID, p.instance.Port)
    return p.server.ListenAndServe()
}

func (p *ServiceProvider) Stop() error {
    // 注销服务
    p.registry.Deregister(p.instance.Name, p.instance.ID)
    
    // 停止服务器
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    return p.server.Shutdown(ctx)
}