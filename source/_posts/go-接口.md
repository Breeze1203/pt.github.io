---
title: "Go 接口"
date: 2024-11-18 15:41:52
---

接口是定义了合约但并没有实现的类型。举个例子：

``` go
type Logger interface {

  Log(message string)

}
```

那这样做有什么作用呢？其实，接口有助于将代码与特定的实现进行分离。例如，我们可能有各种类型的日志记录器：

``` go
type SqlLogger struct { ... }

type ConsoleLogger struct { ... }

type FileLogger struct { ... }
```

针对接口而不是具体实现的编程会使我们很轻松的修改（或者测试）任何代码都不会产生影响。

你会怎么用？就像任何其它类型一样，它结构可以这样：

``` go
type Server struct {

  logger Logger

}
```

或者是一个函数参数（或者返回值）：

``` go
func process(logger Logger) {

  logger.Log("hello!")

}
```

在像 C# 或者 Java 这类语言中，当类实现接口时，我们必须显式的：

``` java
public class ConsoleLogger : Logger {

  public void Logger(message string) {

    Console.WriteLine(message)

  }

}
```

在 Go 中，下面的情况是隐式发生的。如果你的结构体有一个函数名为 Log 且它有一个 string 类型的参数并没有返回值，那么这个结构体被视为 Logger 。这减少了使用接口的冗长：

``` go
type ConsoleLogger struct {}

func (l ConsoleLogger) Log(message string) {

  fmt.Println(message)

}
```

``` go
package main

import "fmt"

// Animal 接口定义了一个方法 MakeSound
type Animal interface {
    MakeSound()
}

// Dog 结构体
type Dog struct {
    Name string
}

// MakeSound 实现 Animal 接口的方法
func (d *Dog) MakeSound() {
    fmt.Println(d.Name, "says: Woof!")
}

// Cat 结构体
type Cat struct {
    Name string
}

// MakeSound 实现 Animal 接口的方法
func (c *Cat) MakeSound() {
    fmt.Println(c.Name, "says: Meow!")
}

// Bird 结构体
type Bird struct {
    Name string
}

// MakeSound 实现 Animal 接口的方法
func (b *Bird) MakeSound() {
    fmt.Println(b.Name, "says: Tweet!")
}
```

Go 倾向于使用小且专注的接口。Go 的标准库基本上由接口组成。像 io 包有一些常用的接口诸如 io.Reader ， io.Writer ， io.Closer 等。如果你编写的函数只需要一个能调用 Close() 的参数，那么你应该接受一个 io.Closer 而不是像 io 这样的父类型。

接口可以成为其他接口的一部分，也就是说接口也可以与其他接口组成新的接口。例如， io.ReadCLoser 的接口是由 io.Reader 接口和 io.Closer 接口组成的。

最后，接口通常用于避免循环导入。由于它们没有具体的实现，因此它们的依赖是有限的。
