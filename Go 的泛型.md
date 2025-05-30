# Go 的泛型

Go 在 **1.18 版本正式引入了泛型（Generics）**，这是 Go 语言历史上一次重要的更新。它解决了之前只能通过接口（`interface{}`）或代码生成来实现“通用逻辑”的问题，使得代码更加类型安全、简洁和高效。

---

## 🧠 一、什么是泛型？

> 泛型是一种编程技术，允许你编写适用于多种类型的函数或数据结构，而无需重复编写代码。

### 示例对比：

#### 使用 `interface{}` 的方式（旧版）：
```go
func Print(v interface{}) {
    fmt.Println(v)
}
```

- 类型不安全：运行时才能确定类型。
- 性能差：需要进行类型装箱拆箱（interface boxing/unboxing）。

#### 使用泛型的方式（新版）：
```go
func Print[T any](v T) {
    fmt.Println(v)
}
```

- 类型安全：编译期就能检查类型。
- 高效：避免了不必要的接口转换。
- 更清晰的语义。

---

## 📌 二、Go 泛型的核心概念

### 1. 类型参数（Type Parameters）

在函数或结构体定义中使用 `[T any]` 这样的语法表示一个类型参数。

```go
func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}
```

你可以这样调用：

```go
nums := []int{1, 2, 3}
strs := Map(nums, func(n int) string {
    return strconv.Itoa(n)
})
```

---

### 2. 类型约束（Type Constraints）

Go 泛型支持对类型参数进行限制，使用 `~` 和 `|` 来表达更复杂的约束。

#### 示例：

```go
type Number interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64
}

func Sum[T Number](a, b T) T {
    return a + b
}
```

- `T Number` 表示 T 必须是 Number 接口中列出的任意一种类型。
- `~` 表示底层类型匹配（例如 `type MyInt int` 可以匹配 `~int`）
- `|` 表示“或”关系

---

### 3. 实例化（Instantiation）

Go 支持**类型推导（type inference）**，所以大多数情况下你不需要显式指定类型参数：

```go
result := Add(3, 5) // 自动推断为 Add[int]
```

但也可以显式指定：

```go
result := Add[int](3, 5)
```

---

## 🧪 三、泛型结构体和方法

### 定义泛型结构体：

```go
type Stack[T any] struct {
    data []T
}

func (s *Stack[T]) Push(v T) {
    s.data = append(s.data, v)
}

func (s *Stack[T]) Pop() T {
    n := len(s.data)
    v := s.data[n-1]
    s.data = s.data[:n-1]
    return v
}
```

使用：

```go
s := &Stack[int]{}
s.Push(10)
fmt.Println(s.Pop()) // 输出 10
```

---

## 🔍 四、泛型接口

Go 允许定义带有类型参数的接口，用于抽象行为：

```go
type Stringer[T any] interface {
    ToString() string
}
```

不过目前 Go 的泛型接口能力还在逐步完善中（Go 1.18 ~ Go 1.21 中不断完善），部分高级特性如递归约束等正在逐步支持。

---

## ⚠️ 五、泛型 vs 空接口（interface{}）

| 对比项     | `interface{}`                    | 泛型                         |
| ---------- | -------------------------------- | ---------------------------- |
| 类型安全性 | 否（需运行时检查）               | 是（编译期检查）             |
| 性能       | 相对较慢（涉及装箱拆箱）         | 更快（直接实例化）           |
| 内存占用   | 多余的元信息                     | 更紧凑                       |
| 代码可读性 | 不够明确                         | 明确类型                     |
| 适用场景   | 真正需要任意类型（如 JSON 解析） | 类型安全、性能敏感的通用逻辑 |

---

## ✅ 六、泛型的应用场景

| 场景                | 说明                                         |
| ------------------- | -------------------------------------------- |
| 数据结构            | 列表、栈、队列、映射、集合等                 |
| 工具函数            | `Map`, `Filter`, `Reduce` 等                 |
| 序列化/反序列化辅助 | 构建通用解析器                               |
| 标准库增强          | `slices.Map`, `maps.Keys` 等（Go 1.21 引入） |
| ORM / 数据访问层    | 提高类型安全性和复用性                       |
| 错误处理与封装      | 通用错误包装、提取逻辑                       |

---

## 🧰 七、Go 泛型的标准库支持（Go 1.21 新增）

Go 1.21 开始，标准库开始提供了一些泛型工具函数：

### 示例：`slices` 包

```go
import "slices"

nums := []int{3, 1, 4, 1, 5}
sorted := slices.Clone(nums)
slices.Sort(sorted)
```

### 常见函数包括：

- `slices.Clone`
- `slices.Compact`
- `slices.Contains`
- `slices.Equal`
- `slices.Index`
- `slices.Sort`

---

## 🧱 八、泛型的底层实现机制（简要）

Go 编译器会根据不同的类型参数生成对应的函数副本（类似 C++ 模板的实例化）。

- **静态单赋值（SSA）优化**
- **类型专用函数生成**
- **共享相同的函数签名代码（如果类型相同）**

优点：

- 高性能
- 编译期类型检查
- 避免反射开销

缺点：

- 会导致二进制体积增大（多个类型生成多个函数副本）

---

## 📚 九、总结

| 特性         | 描述                             |
| ------------ | -------------------------------- |
| 引入时间     | Go 1.18（2022 年 3 月）          |
| 主要作用     | 编写类型安全、可复用的通用代码   |
| 优势         | 类型安全、性能更高、代码更清晰   |
| 适用对象     | 数据结构、工具函数、标准库       |
| 与空接口对比 | 更安全、更快、更适合现代开发     |
| 未来趋势     | 泛型将成为 Go 的核心编程范式之一 |

