# Go标准库面试题

## 输入输出(IO)篇

### 问题1：fmt包的主要功能和使用场景是什么？

**答案：**
fmt包主要用于格式化输入输出：

```go
// 格式化输出
fmt.Printf("数值: %d, 字符串: %s\n", 100, "hello")

// 格式化字符串
s := fmt.Sprintf("%.2f", 3.1415926)

// 扫描输入
var name string
fmt.Scanln(&name)
```

常用占位符：
- %v：默认格式
- %+v：添加字段名
- %#v：Go语法格式

### 问题2：如何使用bufio实现高效的文件读写？

**答案：**
使用缓冲区提升IO性能：

```go
func readFileByLine() error {
    file, err := os.Open("test.txt")
    if err != nil {
        return err
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
    return scanner.Err()
}

func writeWithBuffer() error {
    file, err := os.Create("output.txt")
    if err != nil {
        return err
    }
    defer file.Close()

    writer := bufio.NewWriter(file)
    writer.WriteString("Hello World\n")
    return writer.Flush()
}
```

## 字符串处理篇

### 问题3：strings包提供了哪些常用的字符串操作函数？

**答案：**
常用函数示例：

```go
func stringOps() {
    s := "Hello, World!"
    
    // 查找
    idx := strings.Index(s, "World")    // 7
    contains := strings.Contains(s, "Hello") // true
    
    // 转换
    upper := strings.ToUpper(s)
    lower := strings.ToLower(s)
    
    // 分割与连接
    parts := strings.Split(s, ", ")  // ["Hello" "World!"])
    joined := strings.Join(parts, "--") // "Hello--World!"
    
    // 修剪
    trimmed := strings.TrimSpace(" hello ") // "hello"
}
```

### 问题4：如何使用regexp包处理正则表达式？

**答案：**
正则表达式常见用法：

```go
func regexpExample() {
    // 编译正则表达式
    emailPattern := `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
    re := regexp.MustCompile(emailPattern)
    
    // 匹配
    email := "test@example.com"
    isValid := re.MatchString(email)
    
    // 查找
    text := "联系方式：test@example.com 和 admin@example.com"
    emails := re.FindAllString(text, -1)
    
    // 替换
    masked := re.ReplaceAllString(text, "[EMAIL]")
}
```

## 时间处理篇

### 问题5：time包如何处理时间和日期？

**答案：**
时间处理示例：

```go
func timeExample() {
    // 获取当前时间
    now := time.Now()
    
    // 时间格式化
    formatted := now.Format("2006-01-02 15:04:05")
    
    // 时间解析
    t, _ := time.Parse("2006-01-02", "2023-12-31")
    
    // 时间计算
    future := now.Add(24 * time.Hour)
    past := now.AddDate(-1, 0, 0)  // 一年前
    
    // 时间比较
    duration := future.Sub(now)
    isAfter := future.After(now)
}
```

## 网络编程篇

### 问题6：如何使用net/http包构建Web服务器？

**答案：**
基本的HTTP服务器示例：

```go
func httpServerExample() {
    // 处理函数
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, %s!", r.URL.Path[1:])
    })
    
    // 启动服务器
    http.ListenAndServe(":8080", nil)
}

// HTTP客户端示例
func httpClientExample() error {
    resp, err := http.Get("http://example.com")
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    return err
}
```

## 并发控制篇

### 问题7：sync包中的互斥锁和等待组如何使用？

**答案：**
并发控制示例：

```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func waitGroupExample() {
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            // 执行任务
        }(i)
    }
    
    wg.Wait()
}
```

### 问题8：context包如何用于控制goroutine？

**答案：**
context使用示例：

```go
func contextExample() {
    // 创建带取消的context
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // 使用context控制goroutine
    go func() {
        select {
        case <-ctx.Done():
            fmt.Println("任务被取消:", ctx.Err())
            return
        default:
            // 执行任务
        }
    }()
}
```

## 数据序列化篇

### 问题9：如何使用encoding/json处理JSON数据？

**答案：**
JSON处理示例：

```go
type Person struct {
    Name    string   `json:"name"`
    Age     int      `json:"age"`
    Hobbies []string `json:"hobbies,omitempty"`
}

func jsonExample() error {
    // 编码
    p := Person{Name: "张三", Age: 25}
    data, err := json.Marshal(p)
    if err != nil {
        return err
    }
    
    // 解码
    var p2 Person
    err = json.Unmarshal(data, &p2)
    return err
}
```

### 问题10：encoding/xml包如何处理XML数据？

**答案：**
XML处理示例：

```go
type Config struct {
    XMLName xml.Name `xml:"config"`
    Server  string   `xml:"server,attr"`
    Port    int      `xml:"port"`
    Timeout int      `xml:"timeout"`
}

func xmlExample() error {
    cfg := Config{
        Server:  "localhost",
        Port:    8080,
        Timeout: 30,
    }
    
    // 编码
    data, err := xml.MarshalIndent(cfg, "", "  ")
    if err != nil {
        return err
    }
    
    // 解码
    var cfg2 Config
    err = xml.Unmarshal(data, &cfg2)
    return err
}
```

## 错误处理与日志篇

### 问题11：如何使用log包记录日志？

**答案：**
日志记录示例：

```go
func logExample() {
    // 设置日志格式
    log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
    
    // 写入文件
    f, err := os.OpenFile("app.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    log.SetOutput(f)
    log.Printf("应用启动: %s", time.Now())
}
```

### 问题12：如何实现自定义错误类型？

**答案：**
自定义错误示例：

```go
type ValidationError struct {
    Field string
    Error string
}

func (v *ValidationError) Error() string {
    return fmt.Sprintf("字段 %s 验证失败: %s", v.Field, v.Error)
}

func validate(age int) error {
    if age < 0 {
        return &ValidationError{
            Field: "age",
            Error: "年龄不能为负数",
        }
    }
    return nil
}

// 错误包装
func wrapError() error {
    err := validate(-1)
    if err != nil {
        return fmt.Errorf("验证失败: %w", err)
    }
    return nil
}
```

