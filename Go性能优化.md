# Go性能优化指南

## 1. 性能分析工具

### 1.1 使用pprof进行性能分析

**CPU分析：**
```go
import (
    "os"
    "runtime/pprof"
)

func main() {
    // CPU Profile
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // 运行需要分析的代码
}
```

**内存分析：**
```go
import "runtime/pprof"

// 记录堆内存分配
f, _ := os.Create("mem.prof")
defer f.Close()
pprof.WriteHeapProfile(f)
```

**在Web服务中使用pprof：**
```go
import (
    "net/http"
    _ "net/http/pprof"  // 自动注册pprof处理器
)

func main() {
    go func() {
        log.Println(http.ListenAndServe(":6060", nil))
    }()
    // ... 主服务代码
}
```

### 1.2 性能分析可视化

**使用go tool pprof：**
```bash
# 分析CPU profile
go tool pprof cpu.prof

# 分析内存profile
go tool pprof mem.prof

# Web界面查看
go tool pprof -http=:8080 cpu.prof
```

## 2. 算法与数据结构优化

### 2.1 常见优化策略

**使用合适的数据结构：**
```go
// 不推荐
func findItem(items []int, target int) bool {
    for _, item := range items {
        if item == target {
            return true
        }
    }
    return false
}

// 推荐：使用map提高查找效率
func findItem(items map[int]struct{}, target int) bool {
    _, exists := items[target]
    return exists
}
```

**算法复杂度优化：**
```go
// 优化前：O(n²)
func hasDuplicate(nums []int) bool {
    for i := 0; i < len(nums); i++ {
        for j := i + 1; j < len(nums); j++ {
            if nums[i] == nums[j] {
                return true
            }
        }
    }
    return false
}

// 优化后：O(n)
func hasDuplicate(nums []int) bool {
    seen := make(map[int]bool)
    for _, num := range nums {
        if seen[num] {
            return true
        }
        seen[num] = true
    }
    return false
}
```

## 3. 内存优化

### 3.1 减少内存分配

**对象池化：**
```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func processRequest(data []byte) {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    
    // 使用buf处理数据
}
```

**预分配内存：**
```go
// 避免频繁扩容
func processItems(count int) []int {
    // 预分配容量
    result := make([]int, 0, count)
    for i := 0; i < count; i++ {
        result = append(result, i)
    }
    return result
}
```

## 4. 并发优化

### 4.1 并行处理

**使用worker pool：**
```go
func worker(jobs <-chan int, results chan<- int) {
    for n := range jobs {
        results <- process(n)
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // 启动3个worker
    for w := 1; w <= 3; w++ {
        go worker(jobs, results)
    }
    
    // 发送任务
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)
    
    // 收集结果
    for a := 1; a <= 9; a++ {
        <-results
    }
}
```

### 4.2 并发控制

**使用semaphore限制并发：**
```go
type Semaphore chan struct{}

func NewSemaphore(max int) Semaphore {
    return make(chan struct{}, max)
}

func (s Semaphore) Acquire() {
    s <- struct{}{}
}

func (s Semaphore) Release() {
    <-s
}

// 使用示例
sem := NewSemaphore(10) // 最多10个并发
for _, task := range tasks {
    sem.Acquire()
    go func(t Task) {
        defer sem.Release()
        process(t)
    }(task)
}
```

## 5. 编译优化

### 5.1 编译选项

**使用编译标记：**
```bash
# 启用内联优化
go build -gcflags="-l=4"

# 禁用优化和内联（调试用）
go build -gcflags="-N -l"
```

### 5.2 代码内联

**提示编译器内联：**
```go
//go:inline
func add(x, y int) int {
    return x + y
}
```

## 6. I/O优化

### 6.1 网络I/O

**使用连接池：**
```go
type Pool struct {
    conns chan net.Conn
    addr  string
}

func NewPool(addr string, maxConns int) *Pool {
    return &Pool{
        conns: make(chan net.Conn, maxConns),
        addr:  addr,
    }
}

func (p *Pool) Get() (net.Conn, error) {
    select {
    case conn := <-p.conns:
        return conn, nil
    default:
        return net.Dial("tcp", p.addr)
    }
}

func (p *Pool) Put(conn net.Conn) {
    select {
    case p.conns <- conn:
    default:
        conn.Close()
    }
}
```

### 6.2 数据库优化

**批量操作：**
```go
// 批量插入
func batchInsert(db *sql.DB, users []User) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    
    stmt, err := tx.Prepare(`
        INSERT INTO users (name, age) 
        VALUES (?, ?)
    `)
    if err != nil {
        tx.Rollback()
        return err
    }
    defer stmt.Close()
    
    for _, user := range users {
        _, err = stmt.Exec(user.Name, user.Age)
        if err != nil {
            tx.Rollback()
            return err
        }
    }
    
    return tx.Commit()
}
```

## 7. 系统性能优化

### 7.1 CPU优化

**避免过度GC：**
```go
func processLargeData() {
    // 临时禁用GC
    debug.SetGCPercent(-1)
    defer debug.SetGCPercent(100)
    
    // 处理大量数据
}
```

### 7.2 内存优化

**减少临时对象：**
```go
// 避免字符串连接产生临时对象
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString("item")
    builder.WriteString(strconv.Itoa(i))
}
result := builder.String()
```

## 最佳实践总结

1. **性能分析**
   - 使用pprof定位瓶颈
   - 关注CPU和内存profile
   - 定期进行性能测试

2. **代码优化**
   - 选择合适的数据结构
   - 优化算法复杂度
   - 减少内存分配

3. **并发处理**
   - 合理使用goroutine
   - 控制并发数量
   - 避免竞态条件

4. **系统调优**
   - 优化GC参数
   - 合理使用缓存
   - 监控系统资源

5. **编译优化**
   - 使用适当的编译标记
   - 启用内联优化
   - 考虑交叉编译性能

6. **I/O优化**
   - 使用连接池
   - 实现批量操作
   - 优化网络通信

7. **持续优化**
   - 建立性能基准
   - 进行压力测试
   - 监控生产环境