---
date: 2023-01-01
title: Go 陷阱之 interface 方法调用规则
modify: 2023-01-01
---

# 概述

接口方法调用时，调用方必须和接口方法定义的接收者类型相同或者可以通过推导得到。

**具体的规则**:

- 接收者为值的方法，可以通过值类型变量调用
- 接收者为值的方法，可以通过指针类型变量调用，因为指针可以被解引用得到值类型
- 接收者为指针的方法，可以通过指针类型变量调用
- **接收者为指针的方法，不可以通过值类型变量调用**

# 示例

## 接收者和调用方都是值类型

```go
package main

import (
	"fmt"
)

type Person interface {
	Name() string
	Age() int
}

type Martian struct {
}

// 接收者为值类型
func (m Martian) Name() string {
	return "martian"
}

// 接收者为值类型
func (m Martian) Age() int {
	return 0
}

func main() {
	var m Person = Martian{} // 调用方为值类型
	fmt.Printf("name is %s, age is = %d\n", m.Name(), m.Age())
}

// $ go run main.go
// 输出如下
/**
  name is martian, age is = 0
*/
```

## 接收者值类型，调用方指针类型

```go
package main

import (
	"fmt"
)

type Person interface {
	Name() string
	Age() int
}

type Martian struct {
}

// 接收者为值类型
func (m Martian) Name() string {
	return "martian"
}

// 接收者为值类型
func (m Martian) Age() int {
	return 0
}

func main() {
	var m Person = &Martian{} // 调用方为指针类型
	fmt.Printf("name is %s, age is = %d\n", m.Name(), m.Age())
}

// $ go run main.go
// 输出如下
/**
  name is martian, age is = 0
*/
```

## 接收者和调用方都是指针类型

```go
package main

import (
	"fmt"
)

type Person interface {
	Name() string
	Age() int
}

type Martian struct {
}

// 接收者为指针类型
func (m *Martian) Name() string {
	return "martian"
}

// 接收者为指针类型
func (m *Martian) Age() int {
	return 0
}

func main() {
	var m Person = &Martian{} // 调用方为指针类型
	fmt.Printf("name is %s, age is = %d\n", m.Name(), m.Age())
}

// $ go run main.go
// 输出如下
/**
  name is martian, age is = 0
*/
```

## 接收者指针类型，调用方值类型

```go
package main

import (
	"fmt"
)

type Person interface {
	Name() string
	Age() int
}

type Martian struct {
}

// 接收者为指针类型
func (m *Martian) Name() string {
	return "martian"
}

// 接收者为指针类型
func (m *Martian) Age() int {
	return 0
}

func main() {
	var m Person = Martian{} // 调用方为值类型
	fmt.Printf("name is %s, age is = %d\n", m.Name(), m.Age())
}

// $ go run main.go
// 输出如下
/**
  cannot use Martian{} (value of type Martian)
      as type Person in variable declaration:
  Martian does not implement Person
      (Age method has pointer receiver)
*/
```

# 小结

|      | 接收值为值 | 接收者为指针 |
|------|-------|--------|
| 值变量  | true  | false  |
| 指针变量 | true  | true   |

四种组合中，只有 `接收者类型为指针类型，变量为值类型` 时无法通过编译，这是为什么呢？

**因为编译器不会无缘无故创建一个新的指针，退一步说，即使编译器可以创建指针指向变量，但是因为 Go 语言传递参数方式为 `值传递`，
此时新指针指向的变量，并不是调用该方法的变量，而是经过复制传递的参数变量**。

# 扩展阅读

- [Go 方法基础](../introduction/methods.md)