# Go面向对象编程

## 1. 理解Go的接口与多态实现

### 1.1 接口基础

**核心概念：**
- 接口定义行为而非实现
- 隐式接口实现机制（无需显式声明implements）
- 接口的零值为nil
- 空接口(interface{})用于处理任意类型

**接口定义示例：**
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

### 1.2 多态实现

**实现多态的完整示例：**
```go
type Animal interface {
    Speak() string
    Move() string
}

type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return fmt.Sprintf("%s says: Woof!", d.Name)
}

func (d Dog) Move() string {
    return fmt.Sprintf("%s runs on four legs", d.Name)
}

type Bird struct {
    Species string
}

func (b Bird) Speak() string {
    return fmt.Sprintf("%s says: Chirp!", b.Species)
}

func (b Bird) Move() string {
    return fmt.Sprintf("%s flies in the sky", b.Species)
}

// 多态函数
func DescribeAnimal(a Animal) {
    fmt.Println(a.Speak())
    fmt.Println(a.Move())
}

// 使用示例
func main() {
    dog := Dog{Name: "Max"}
    bird := Bird{Species: "Sparrow"}
    
    DescribeAnimal(dog)  // 多态调用
    DescribeAnimal(bird) // 多态调用
}
```

## 2. 结构体与方法的定义与使用

### 2.1 结构体基础

**结构体定义与初始化：**
```go
type Person struct {
    Name     string
    Age      int
    Address  string
    private  string // 小写开头表示私有字段
}

// 初始化方式
person1 := Person{Name: "Alice", Age: 25}

// 推荐：使用构造函数
func NewPerson(name string, age int) *Person {
    return &Person{
        Name: name,
        Age:  age,
    }
}
```

### 2.2 方法定义与接收者

**值接收者vs指针接收者：**
```go
// 值接收者：适用于不需要修改接收者的场景
func (p Person) Greet() string {
    return fmt.Sprintf("Hello, my name is %s", p.Name)
}

// 指针接收者：适用于需要修改接收者或避免值拷贝的场景
func (p *Person) SetAge(age int) {
    p.Age = age
}

// 方法继承与重写
type Employee struct {
    Person  // 嵌入Person结构体
    Company string
}

// Employee特有的Greet方法（重写）
func (e Employee) Greet() string {
    return fmt.Sprintf("%s, I work at %s", e.Person.Greet(), e.Company)
}
```

## 3. 组合(Composition)替代继承

### 3.1 组合基础

**通过组合实现代码复用：**
```go
// 基础组件
type Engine struct {
    Power     int
    FuelType  string
}

func (e Engine) Start() string {
    return fmt.Sprintf("Engine starting with %d horsepower", e.Power)
}

// 通过组合构建更复杂的类型
type Car struct {
    Engine      // 嵌入Engine
    Brand       string
    Model       string
    manufactured time.Time
}

// 扩展行为
func (c Car) Description() string {
    return fmt.Sprintf("%s %s with %s", c.Brand, c.Model, c.Start())
}
```

### 3.2 多重组合

**处理多重组合的最佳实践：**
```go
type Logger struct {
    prefix string
}

func (l Logger) Log(message string) {
    fmt.Printf("[%s] %s\n", l.prefix, message)
}

type Database struct {
    Logger      // 嵌入Logger
    connection string
}

type WebService struct {
    Logger      // 嵌入Logger
    Database    // 嵌入Database
    port     int
}

// 初始化多重组合
service := WebService{
    Logger:   Logger{prefix: "WEB"},
    Database: Database{Logger: Logger{prefix: "DB"}, connection: "postgres://"},
    port:     8080,
}
```

## 4. 接口的嵌套与组合

### 4.1 接口组合

**构建复杂接口：**
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// 组合多个接口
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// 实现组合接口
type File struct {
    *os.File
}

func NewFile(filename string) (ReadWriteCloser, error) {
    file, err := os.OpenFile(filename, os.O_RDWR, 0666)
    if err != nil {
        return nil, err
    }
    return &File{File: file}, nil
}
```

### 4.2 接口隔离原则

**设计小而专注的接口：**
```go
// 好的设计：小而专注的接口
type Authenticator interface {
    Authenticate(username, password string) (bool, error)
}

type UserFinder interface {
    FindUser(id string) (*User, error)
}

// 组合特定场景需要的接口
type UserService interface {
    Authenticator
    UserFinder
}
```

## 5. 设计可扩展系统

### 5.1 面向接口编程

**设计原则：**
- 依赖接口而非具体实现
- 保持接口小而专注
- 遵循单一职责原则

**实践示例：**
```go
type MessageSender interface {
    Send(message string) error
}

type EmailSender struct {
    smtpServer string
}

func (e EmailSender) Send(message string) error {
    // 实现邮件发送逻辑
    return nil
}

type SMSSender struct {
    provider string
}

func (s SMSSender) Send(message string) error {
    // 实现短信发送逻辑
    return nil
}

// 使用接口作为依赖
type NotificationService struct {
    sender MessageSender
}

func (n NotificationService) Notify(message string) error {
    return n.sender.Send(message)
}
```

## 6. 依赖注入实践

### 6.1 构造函数注入

**基本依赖注入模式：**
```go
type Database interface {
    Query(query string) ([]byte, error)
    Execute(command string) error
}

type UserRepository struct {
    db Database
}

// 通过构造函数注入依赖
func NewUserRepository(db Database) *UserRepository {
    return &UserRepository{db: db}
}

type UserService struct {
    repo *UserRepository
    logger Logger
}

// 多依赖注入
func NewUserService(repo *UserRepository, logger Logger) *UserService {
    return &UserService{
        repo: repo,
        logger: logger,
    }
}
```

### 6.2 依赖注入最佳实践

**使用依赖注入容器：**
```go
type Container struct {
    services map[reflect.Type]interface{}
}

func NewContainer() *Container {
    return &Container{
        services: make(map[reflect.Type]interface{}),
    }
}

func (c *Container) Register(service interface{}) {
    t := reflect.TypeOf(service)
    c.services[t] = service
}

func (c *Container) Resolve(t reflect.Type) interface{} {
    return c.services[t]
}

// 使用示例
container := NewContainer()
container.Register(NewDatabase())
container.Register(NewLogger())

db := container.Resolve(reflect.TypeOf((*Database)(nil)).Elem()).(Database)
logger := container.Resolve(reflect.TypeOf((*Logger)(nil)).Elem()).(Logger)
```

## 最佳实践总结

1. **接口设计原则**
   - 保持接口小而专注
   - 遵循接口隔离原则
   - 优先使用组合而非继承

2. **结构体和方法**
   - 适当选择值接收者或指针接收者
   - 使用构造函数初始化复杂结构体
   - 合理使用嵌入实现代码复用

3. **依赖注入**
   - 通过接口解耦组件
   - 在构造函数中注入依赖
   - 考虑使用依赖注入容器

4. **代码组织**
   - 按职责划分包和模块
   - 避免循环依赖
   - 保持适当的抽象层级

5. **测试建议**
   - 依赖接口便于mock测试
   - 使用表驱动测试
   - 关注边界条件测试