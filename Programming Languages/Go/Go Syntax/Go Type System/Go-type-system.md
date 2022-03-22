# Go 类型系统

目录：

- [Go 类型系统](#go-类型系统)
  - [结构体](#结构体)
    - [声明及初始化](#声明及初始化)
    - [结构体比较](#结构体比较)
    - [结构体嵌入和匿名成员](#结构体嵌入和匿名成员)
    - [结构体访问权限控制](#结构体访问权限控制)
  - [方法](#方法)
  - [接口](#接口)
  - [参考资料](#参考资料)

## 结构体

结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。

### 声明及初始化

结构体类型通过组合一系列固定且唯一的字段来声明。每个字段都会用一个已知类型声明，该已知类型可以是内置类型，也可以是其他用户定义的类型。

一个命名为 `S` 的结构体类型将不能再包含 `S` 类型的成员：因为一个聚合的值不能包含它自身。（该限制同样适用于数组。）但是 `S` 类型的结构体可以包含 `*S` 指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。

结构体声明，以关键字 `type` 开始，之后是 `新类型的名字`，最后是关键字 `struct`。

```go
// 定义一个 user 类型
type user struct {
	name       string
	email      string
	ext        int
	privileged bool
}
```

**eg 1. 使用结构类型声明变量，并初始化为零值**

```go
var bill user
```

关键字 `var` 创建一个变量并初始化为其零值（结构体类型的零值是每个成员都是零值）。

**eg 2. 使用结构体字面量声明变量**

```go
// eg 2.1 声明字段名字 - 对声明顺序无要求
lisa := user{
	name : "Lisa",
	email: "Lisa@email.com",
	ext: 123,
	privileged: true,
}

// eg 2.2 省略字段名字 - 声明顺序需与结构声明中的字段顺序一致
lisa := user{"Lisa", "Lisa@email.com", 123, true}

// eg 2.3 嵌套声明
type admin struct {
	person user
	level  string
}

fred := admin{
	person: user{
		name:       "Lisa",
		email:      "Lisa@email.com",
		ext:        123,
		privileged: true,
	},
	level:  "super",
}
```

**eg 3. 基于已有类型声明新类型**

```go
type Duration int64
```

通过这种方法，能够从内置类型中创建出很多更加明确的类型，并赋予更高级的功能。**Go 并不认为 `Duration` 和 `int64` 是同一种类型，两种不同类型即使互相兼容，也不能互相赋值。** 编译器不会对不同类型的值做隐式转换。

### 结构体比较

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，两个结构体将可以使用 `==` 或 `!=` 运算符进行比较。相等比较运算符 `==` 将比较两个结构体的每个成员。

同时，可比较的结构体类型和其他可比较的类型一样，可以用于 map 的 key 类型。

### 结构体嵌入和匿名成员

Go 可以让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。得益于匿名嵌入的特性，我们可以直接访问叶子属性而不需要给出完整的路径：

```go
type Point struct {
    X, Y int
}

type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}

var w Wheel
w.X = 8            // 等价于 to w.Circle.Point.X = 8
w.Y = 8            // 等价于 to w.Circle.Point.Y = 8
w.Radius = 5       // 等价于 to w.Circle.Radius = 5
w.Spokes = 20
```

### 结构体访问权限控制

如果结构体成员名字是以大写字母开头的，那么该成员就是导出的；一个结构体可能同时包含导出和未导出的成员。

* 首字母大写（导出成员）可以跨包访问；
* 首字母小写（非导出成员）仅能同包内访问。

## 方法

方法是和特殊类型关联的函数。在函数声明时，在关键字 `func` 和 `方法名` 之间增加一个参数，即成为一个方法。该附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。

```go
type user struct {
	name       string
	email      string
}

func (u user) notify() {
	fmt.Println(u.name, u.email)
}

// eg 1. 通过变量调用方法
bill := user{"Bill", "bill@email.com"}
bill.notify()

/* eg 2. 通过指针调用方法
注意：该指针仍然是调用的值接收器声明的方法，
notify 仍然操作的是副本（指针指向的值的副本） */
lisa := &user{"Lisa", "lisa@email.com"}
lisa.notify() // 等价于 (*lisa).notify()
```

关键字 `func` 和 方法名 之间的参数被称为 `接收器`（receiver），其类似于其他语言中的 `this` 和 `self`。Go 可以自定义接收器的名字。

不同于其他语言只能给自定义类型定义方法，Go 能够给任意类型定义方法，包括内置类型，如：数值类型、字符串、slice、map 等，使其附加一些特定行为。只要该类型不是指针或接口即可。

**eg 1. 值接收器**

```go
func (u user) notify() {
	fmt.Println(u.name, u.email)
}
```

**eg 2. 指针接收器**

```go
func (u *user) changeEmail(email string) {
    u.email = email
}
```

* 值接收器使用值的副本来调用方法；
* 指针接收者使用实际值来调用方法。

**Go 语言即允许使用值，也允许使用指针来调用方法，不必严格符合接收器的类型。最终函数内部操作的是参数的副本，还是参数本身，只取决于接收器的类型。**

## 接口

Go 语言提供了另外一种数据类型即接口，它把所有的具有共性的方法定义在一起，任何其他类型只要实现了这些方法就是实现了这个接口。

```go
/* 定义接口 */
type interface_name interface {
   method_name1 [return_type]
   method_name2 [return_type]
   method_name3 [return_type]
   ...
   method_namen [return_type]
}

/* 定义结构体 */
type struct_name struct {
   /* variables */
}

/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1() [return_type] {
   /* 方法实现 */
}
...
func (struct_name_variable struct_name) method_namen() [return_type] {
   /* 方法实现*/
}
```

**注意：**

* 在 Go 中，实现接口的所有方法就隐式地实现了接口，如果只实现部分接口方法，则意味着没有继承这个接口；
* **Go 不支持重载。** 结构体所实现的接口方法需保证方法签名完全一致（方法名、入参、返回值）。

**多态实例：**

```go
// 定义 Phone 接口
type Phone interface {
    call()
}

// 定义结构体 NokiaPhone
type NokiaPhone struct {}

// 结构体 NokiaPhone 实现接口方法 call()
func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}

// 定义结构体 IPhone
type IPhone struct {}

// 结构体 IPhone 实现接口方法 call()
func (iPhone IPhone) call() {
    fmt.Println("I am iPhone, I can call you!")
}

func main() {
    var phone Phone

    phone = new(NokiaPhone)
    phone.call()

    phone = new(IPhone)
    phone.call()
}
```

## 参考资料

* 《Go 语言实战》
* 《Go 语言圣经》
* [Go 语言教程 - 菜鸟](https://www.runoob.com/go/go-interfaces.html)
