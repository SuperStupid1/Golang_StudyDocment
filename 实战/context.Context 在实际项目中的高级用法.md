# 🚀 `context.Context` 在实际项目中的高级用法

---

## 一、基本结构回顾

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- `Done()`：返回一个 channel，当上下文被取消或超时时关闭。
- `Err()`：获取取消的原因（如 context.Canceled、context.DeadlineExceeded）。
- `Value()`：传递请求作用域的数据。
- `Deadline()`：获取截止时间。

---

## 二、实际项目中的高级用法

### ✅ 1. 控制并发任务的退出（Goroutine Pool）

#### 场景：

你有一个后台服务处理多个任务，比如从队列中消费消息。你希望在服务关闭时能优雅地停止所有 goroutine。

#### 示例代码：

```go
func worker(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("Worker %d stopped: %v\n", id, ctx.Err())
			return
		default:
			fmt.Printf("Worker %d is working...\n", id)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	for i := 1; i <= 3; i++ {
		go worker(ctx, i)
	}

	time.Sleep(2 * time.Second)
	cancel() // 停止所有 worker
	time.Sleep(1 * time.Second)
}
```

> ⚠️ 注意：使用 `select` + `ctx.Done()` 是防止 goroutine 泄漏的关键。

---

### ✅ 2. 设置请求超时（HTTP / RPC / DB 查询）

#### 场景：

你发起一个外部 HTTP 请求或数据库查询，但你不希望它永远等待，而是设置最大等待时间。

#### 示例代码：

```go
func fetchWithTimeout() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	req, _ := http.NewRequest("GET", "https://slow-api.com/data", nil)
	req = req.WithContext(ctx)

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		fmt.Println("Request failed:", err)
		return
	}
	fmt.Println("Response status:", resp.Status)
}
```

> 如果超过 2 秒还没返回结果，请求将自动取消。

---

### ✅ 3. 链路追踪（Trace ID / Request ID）

#### 场景：

你在构建一个微服务系统，每个请求都要带上一个唯一标识（trace_id），方便日志追踪和调试。

#### 实现方式：

使用 `context.WithValue()` 来携带 trace_id。

```go
func handleRequest(traceID string) {
	ctx := context.WithValue(context.Background(), "trace_id", traceID)

	go processTask(ctx)
	time.Sleep(1 * time.Second)
}

func processTask(ctx context.Context) {
	traceID, _ := ctx.Value("trace_id").(string)
	fmt.Printf("[%s] Processing task...\n", traceID)
}
```

> ⚠️ 注意：
> - `WithValue` 只适合传**不可变的请求元信息**，不能传复杂对象。
> - 推荐使用类型安全的 key（避免冲突）：

```go
type contextKey string
const TraceIDKey contextKey = "trace_id"
```

---

### ✅ 4. 多级 Cancel 传播（嵌套 Context）

#### 场景：

你有多个 goroutine 并行执行任务，它们共享一个父 context。一旦父 context 被取消，所有子任务也应该终止。

#### 示例：

```go
func parentChildContext() {
	parentCtx, parentCancel := context.WithCancel(context.Background())

	childCtx, childCancel := context.WithCancel(parentCtx)
	defer childCancel()

	go func() {
		for {
			select {
			case <-childCtx.Done():
				fmt.Println("Child stopped:", childCtx.Err())
				return
			default:
				fmt.Println("Child is running...")
				time.Sleep(500 * time.Millisecond)
			}
		}
	}()

	time.Sleep(2 * time.Second)
	parentCancel() // 取消父 context，子 context 也会自动取消
	time.Sleep(1 * time.Second)
}
```

输出：
```
Child is running...
Child is running...
Child stopped: context canceled
```

---

### ✅ 5. 结合中间件统一管理请求上下文（Go Web 框架）

#### 场景：

在 Gin、Echo 等 Web 框架中，为每个请求创建一个带超时的 context，并注入 trace_id、user_id 等信息。

#### Gin 示例：

```go
func TimeoutMiddleware(timeout time.Duration) gin.HandlerFunc {
	return func(c *gin.Context) {
		ctx, cancel := context.WithTimeout(c.Request.Context(), timeout)
		defer cancel()

		c.Request = c.Request.WithContext(ctx)

		c.Next()
	}
}
```

注册中间件：

```go
r := gin.Default()
r.Use(TimeoutMiddleware(5 * time.Second))
```

> 所有后续 handler 中使用的 `c.Request.Context()` 都会受到超时控制。

---

### ✅ 6. 结合数据库操作（ORM / SQL）

很多 ORM 库（如 GORM）支持 context，你可以为数据库操作设置超时：

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

var user User
db.WithContext(ctx).Where("id = ?", 1).First(&user)
```

如果查询超过 3 秒，会自动中断并返回错误。

---

### ✅ 7. 资源清理与释放（如锁、连接池）

#### 场景：

你在获取分布式锁、连接池资源后，需要确保即使发生 panic 或超时也能正确释放资源。

```go
func withLockAndCleanup(ctx context.Context) {
	lock := NewRedisLock(rdb, "my_lock", "worker_1", 10*time.Second)

	if !lock.Acquire(ctx) {
		fmt.Println("Failed to acquire lock")
		return
	}
	defer lock.Release(ctx)

	// do something
	fmt.Println("Processing with lock")
}
```

> 即使 panic，`defer` 也能保证锁被释放。

---

## 🧩 总结：Context 的常见用途分类

| 使用场景               | 描述                                     |
| ---------------------- | ---------------------------------------- |
| Goroutine 生命周期控制 | 控制协程何时退出                         |
| 请求超时控制           | 限制单个请求的最大执行时间               |
| 上下文数据传递         | 传递 trace_id、user_id 等信息            |
| 取消通知广播           | 父 context 取消，所有子 context 自动取消 |
| 链式调用控制           | 微服务间调用链的上下文透传               |
| 资源清理               | 锁、连接、文件句柄的释放                 |

---

## ✅ 最佳实践建议

| 建议                                                  | 说明                        |
| ----------------------------------------------------- | --------------------------- |
| ❌ 不要滥用 `context.TODO()` 和 `context.Background()` | 应该优先使用传入的 context  |
| ✅ 使用 `select` 监听 `ctx.Done()`                     | 防止 goroutine 泄漏         |
| ✅ 使用 `defer cancel()` 释放资源                      | 防止 context 泄漏           |
| ✅ 使用类型安全的 key 传递数据                         | 避免 key 冲突               |
| ✅ 在中间件中封装 context 控制逻辑                     | 统一处理超时、cancel、trace |
| ✅ 结合日志记录 trace_id                               | 方便排查问题                |

---

