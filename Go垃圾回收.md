# Go垃圾回收机制详解

## 1. 垃圾回收基础

### 1.1 Go GC算法原理

**三色标记-清除算法：**
```go
// 三色标记示意
type Object struct {
    marked  bool    // 对象是否被标记
    refs    []*Object  // 引用的其他对象
}

// 标记过程伪代码
func markPhase(root *Object) {
    // 白色：未被标记的对象
    // 灰色：已标记但其引用对象未标记完成
    // 黑色：已标记且其引用对象也已标记完成
    
    // 初始时所有对象都是白色
    grey := []*Object{root}
    
    for len(grey) > 0 {
        obj := grey[0]
        grey = grey[1:]
        
        // 将对象标记为黑色
        obj.marked = true
        
        // 将其引用对象加入灰色集合
        for _, ref := range obj.refs {
            if !ref.marked {
                grey = append(grey, ref)
            }
        }
    }
}
```

**分代回收机制：**
- 新生代（Young Generation）：存活时间短的对象
- 老年代（Old Generation）：经过多次GC仍存活的对象
- 对象晋升策略：存活超过一定次数的对象晋升到老年代

### 1.2 GC触发时机

1. **自动触发**：
   - 内存分配达到阈值
   - 定期触发（默认2分钟）

2. **手动触发**：
```go
runtime.GC()  // 强制触发GC
```

## 2. 性能优化

### 2.1 内存分配优化

**对象池化：**
```go
var pool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func processData() {
    // 从对象池获取
    buf := pool.Get().([]byte)
    defer pool.Put(buf)  // 使用完归还
    
    // 使用buf进行操作
}
```

**预分配内存：**
```go
// 不推荐
var slice []int
for i := 0; i < 1000; i++ {
    slice = append(slice, i)  // 可能多次扩容
}

// 推荐
slice := make([]int, 0, 1000)  // 预分配容量
for i := 0; i < 1000; i++ {
    slice = append(slice, i)
}
```

### 2.2 逃逸分析

**避免不必要的逃逸：**
```go
// 会逃逸到堆
func createOnHeap() *int {
    x := 42
    return &x
}

// 栈分配
func createOnStack() int {
    x := 42
    return x
}
```

## 3. 垃圾回收调优

### 3.1 GC参数调整

**GOGC环境变量：**
```bash
# 设置GOGC
export GOGC=50  # 更激进的GC
export GOGC=200 # 更宽松的GC
```

**运行时调整：**
```go
defer debug.SetGCPercent(debug.SetGCPercent(50))
```

### 3.2 性能监控

**使用trace工具：**
```go
f, err := os.Create("trace.out")
if err != nil {
    log.Fatal(err)
}
defer f.Close()

trace.Start(f)
defer trace.Stop()

// 运行需分析的代码
```

**运行时统计：**
```go
var stats runtime.MemStats
runtime.ReadMemStats(&stats)
fmt.Printf("Allocated: %d MB\n", stats.Alloc/1024/1024)
fmt.Printf("GC Cycles: %d\n", stats.NumGC)
```

## 4. 内存泄漏检测

### 4.1 常见泄漏场景

**Goroutine泄漏：**
```go
// 错误示例
func leakyGoroutine() {
    ch := make(chan int)
    go func() {
        // channel永远不会被读取
        ch <- 42
    }()
}

// 正确示例
func nonLeakyGoroutine() {
    ch := make(chan int)
    go func() {
        select {
        case ch <- 42:
        case <-time.After(time.Second):
            return
        }
    }()
    // 确保channel被读取或goroutine超时退出
}
```

**资源未释放：**
```go
// 使用defer确保资源释放
func processFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()  // 确保文件被关闭
    
    // 处理文件
    return nil
}
```

### 4.2 检测工具

1. **pprof工具使用：**
```go
import "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // 应用代码
}
```

2. **使用go-torch生成火焰图**

## 5. 高效内存使用

### 5.1 数据结构优化

**减少对象大小：**
```go
// 优化前
type User struct {
    ID        int64
    Name      string
    Age       int64  // 过大的类型
    IsActive  bool
    padding   byte   // 额外的填充
}

// 优化后
type User struct {
    ID        int64
    Name      string
    Age       int32  // 使用更小的类型
    IsActive  bool
    // 相似类型字段放在一起，减少填充
}
```

**使用值类型：**
```go
// 避免不必要的指针
type Point struct {
    X, Y float64
}

// 直接使用值类型
points := make([]Point, 1000)

// 而不是指针切片
// points := make([]*Point, 1000)
```

### 5.2 引用管理

**弱引用实现：**
```go
type WeakRef struct {
    value   *interface{}
    cleanup func()
}

func NewWeakRef(value interface{}, cleanup func()) *WeakRef {
    return &WeakRef{
        value:   &value,
        cleanup: cleanup,
    }
}

func (w *WeakRef) Get() interface{} {
    if w.value == nil {
        return nil
    }
    return *w.value
}
```

## 6. 优化GC暂停时间

### 6.1 减少GC压力

1. **批量处理：**
```go
// 不推荐
for _, item := range items {
    process(item)  // 每次处理都可能产生垃圾
}

// 推荐
batch := make([]Item, 0, 100)
for _, item := range items {
    batch = append(batch, item)
    if len(batch) == cap(batch) {
        processBatch(batch)  // 批量处理减少垃圾
        batch = batch[:0]
    }
}
```

2. **复用对象：**
```go
type Worker struct {
    buffer []byte
}

func (w *Worker) Process(data []byte) {
    // 复用buffer，避免每次分配
    if cap(w.buffer) < len(data) {
        w.buffer = make([]byte, len(data))
    }
    w.buffer = w.buffer[:len(data)]
    copy(w.buffer, data)
    // 处理数据
}
```

### 6.2 异步GC策略

**控制GC时机：**
```go
func processWithGCControl() {
    // 关闭GC
    debug.SetGCPercent(-1)
    
    // 处理关键业务逻辑
    
    // 手动触发GC
    go func() {
        runtime.GC()
        // 恢复GC
        debug.SetGCPercent(100)
    }()
}
```

## 最佳实践总结

1. **内存分配：**
   - 预分配合适大小的内存
   - 使用对象池复用对象
   - 避免频繁的小对象分配

2. **GC优化：**
   - 合理设置GOGC
   - 监控GC指标
   - 控制对象生命周期

3. **代码优化：**
   - 使用适当的数据类型
   - 避免不必要的指针
   - 及时释放不用的资源

4. **性能监控：**
   - 定期进行性能分析
   - 使用pprof和trace工具
   - 关注内存使用趋势

5. **系统设计：**
   - 批量处理减少GC压力
   - 合理使用异步处理
   - 控制对象生命周期