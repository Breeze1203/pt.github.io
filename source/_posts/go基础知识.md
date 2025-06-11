---
title: "go基础知识"
date: 2025-06-11 19:23:41
categories: go
tags: go
---

### go命名规范

Go 语言中，任何标识符（变量，常量，函数，自定义类型等）都应该满足以下规律：

- 连续的 <a href="https://golang.org/ref/spec#letter" target="_blank">字符</a> (unicode*letter \| \`*\` .) 或 <a href="https://golang.org/ref/spec#unicode_digit" target="_blank">数字</a>("0" … "9") 组成。

- 以字符或下划线开头。

- 不能和 Go 关键字冲突。

#### Go 关键字:

``` go
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

#### 举例说明**：**

``` go
foo  #合法
foo1 #合法
_foo #合法
变量 #合法
变量1 #合法
_变量 合法

1foo #不合法
1 #不合法
type #不合法
go #不合法
```

### 变量申明

#### 类型声明基本语法

在 Go 语言中，采用的是后置类型的声明方式，形如：

``` go
<命名> <类型>
```

例如：

``` go
x int // 定义 x 为整数类型
```

这么定义不是为了凸显与众不同，而是为了让声明更加清晰易懂，具体可以参考文章<a href="https://blog.golang.org/gos-declaration-syntax" target="_blank">gos-declaration-syntax</a>

#### 变量声明

在 Go 语言中通常我们使用关键字 `var` 来声明变量，例如

``` go
var x int // 表示声明一个名为 x 的整数变量
var b int = 1 // 表示声明一个名为 b 的整数变量，并且附上初始值为 1
var b = 1
```

如果有多个变量同时声明，我们可以采用 `var` 加括号批量声明的方式:

``` go
var (
    a, b int  // 同时声明 a, b 的整数
    c float64
)
```

#### 简短声明方式

变量在声明的时候如果有初始值，我们可以使用 `:=` 的简短声明方式：

``` go
a := 1 // 声明 a 为 1 的整数
b := int64(1)  // 声明 b 为 1 的 64 位整数
```

### 常量定义

常量是指值不能改变的定义，它必须满足如下规则：

- 定义的时候，必须指定值

- 指定的值类型主要有三类： 布尔，数字，字符串， 其中数字类型包含（rune, integer, floating-point, complex), 它们都属于基本数据类型。

- 不能使用 `:=`

例子：

``` go
const a = 64 // 定义常量值为 64 的值

const (
  b = 4
  c = 0.1
)
```

### 基本数据类型

Go 语言中的基本数据类型包含：

``` go
bool      the set of boolean (true, false)

uint8      the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8      the set of all signed  8-bit integers (-128 to 127)
int16      the set of all signed 16-bit integers (-32768 to 32767)
int32      the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64      the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32      the set of all IEEE-754 32-bit floating-point numbers
float64      the set of all IEEE-754 64-bit floating-point numbers

complex64      the set of all complex numbers with float32 real and imaginary parts
complex128      the set of all complex numbers with float64 real and imaginary parts

byte      alias for uint8
rune      alias for int32
uint      either 32 or 64 bits
int      same size as uint
uintptr      an unsigned integer large enough to store the uninterpreted bits of a pointer value

string      the set of string value (eg: "hi")
```

我们可以将基本类型分为三大类：

- 布尔类型

- 数字类型

- 字符串类型

#### 布尔类型

一般我们用于判断条件, 它的取值范围为 `true`, `false`, 声明如下：

``` go
var a bool
var a = true
a := true

const a = true
```

#### 数字类型：

数字类型主要分为有符号数和无符号数，有符号数可以用来表示负数，除此之外它们还有位数的区别，不同的位数代表它们实际存储占用空间，以及取值的范围。

例如：

``` go
var (
  a uint8 = 1
  b int8 = 1
)

var (
    c int = 64
)
```

注意：

- 每种数字类型都有取值范围，超过了取值范围，出现 `overflows` 的错误。

- int，uint 的长度由操作系统的位数决定，在 32 位系统里面，它们的长度未 32 bit, 64 位系统，长度为 64 bit。

#### 字符串

``` go
var a = "hello" //单行字符串
var c = "\"" // 转义符

var b = `hello` //原样输出
var d = `line3  //多行输出
line1
line2
`

var str = "hello, 世界"

b := str[0]  //b is a uint8 type, like byte
fmt.Println(b)  // 104
fmt.Println(string(b)) // h

fmt.Println(len(str)) //12, 查看字符串有多少个字节
fmt.Println(len([]rune(str))) // 8 查看有多少个字符
```

#### 特殊类型

- byte，uint8 别名，用于表示二进制数据的 bytes

- rune，int32 别名, 用于表示一个符号

``` go
var str = "hello, 世界"

for _, char := range str {
  fmt.Printf("%T", char)
}
```
