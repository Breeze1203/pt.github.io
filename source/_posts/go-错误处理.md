---
title: "Go 错误处理"
date: 2025-06-11 19:23:38
categories: go
tags: go
---

Go 语言通过内置的错误接口提供了非常简单的错误处理机制。

error 类型是一个接口类型，这是它的定义：

``` go
type error interface {
    Error() string
}
```

我们可以在编码中通过实现 error 接口类型来生成错误信息。

函数通常在最后的返回值中返回错误信息。使用 errors.New 可返回一个错误信息：

``` go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 实现
}
```

在下面的例子中，我们在调用 Sqrt 的时候传递的一个负数，然后就得到了 non-nil 的 error 对象，将此对象与 nil 比较，结果为 true，所以 fmt.Println(fmt 包在处理 error 时会调用 Error 方法)被调用，以输出错误，请看下面调用的示例代码：

``` go
result, err:= Sqrt(-1)

if err != nil {
   fmt.Println(err)
}
```

### **实例**

## 实例

``` go
package main

import (
    "fmt"
)

// 定义一个DivideError结构
type DivideError struct {
    divide  int
    divider int
}

// 实现Error接口
func (de *DivideError) Error() string {
    str := "Cannot proceed,the divider is zero.divide: %divider: 0"
    return fmt.Sprintf(str, de.divide)
}

// 定义int 类型除法运算函数
func Divide(varDivide int, varDivider int) (result int, errorMsg string) {
    if varDivider == 0 {
        dData := DivideError{
            divide:  varDivide,
            divider: varDivider,
        }
        errorMsg = dData.Error()
        return
    } else {
        return varDivide / varDivider, ""
    }
}

func main() {
    // 正常情况
    if result, errorMsg := Divide(100, 10); errorMsg == "" {
        fmt.Println("100/10 = ", result)
    }
    // 当除数为零的时候会返回错误信息
    if _, errorMsg := Divide(100, 0); errorMsg != "" {
        fmt.Println("errorMsg is: ", errorMsg)
    }
}
```

执行以上程序，输出结果为：

    100/10 =  10
    errorMsg is:  
        Cannot proceed, the divider is zero.
        dividee: 100
        divider: 0
