---
title: "go数组切片"
date: 2024-11-18 15:41:52
---

### **数组**

- 定义：由若干相同类型的元素组成的序列

- 数组的长度是固定的，声明后无法改变

- 数组的长度是数组类型的一部分，eg：元素类型相同但是长度不同的两个数组是不同类型的

- 需要严格控制程序所使用内存时，数组十分有用，因为其长度固定，避免了内存二次分配操作

#### **示例**

``` go
package main

import "fmt"

func main() {
    // 定义长度为 5 的数组
    var arr1 [5]int
    for i := 0; i < 5; i++ {
        arr1[i] = i
    }
    printHelper("arr1", arr1)

    // 以下赋值会报类型不匹配错误，因为数组长度是数组类型的一部分
    // arr1 = [3]int{1, 2, 3}
    arr1 = [5]int{2, 3, 4, 5, 6} // 长度和元素类型都相同，可以正确赋值

    // 简写模式，在定义的同时给出了赋值
    arr2 := [5]int{0, 1, 2, 3, 4}
    printHelper("arr2", arr2)

    // 数组元素类型相同并且数组长度相等的情况下，数组可以进行比较
    fmt.Println(arr1 == arr2)

    // 也可以不显式定义数组长度，由编译器完成长度计算
    var arr3 = [...]int{0, 1, 2, 3, 4}
    printHelper("arr3", arr3)

    // 定义前四个元素为默认值 0，最后一个元素为 -1
    var arr4 = [...]int{4: -1}
    printHelper("arr4", arr4)

    // 多维数组
    var twoD [2][3]int
    for i := 0; i < 2; i++ {
        for j := 0; j < 3; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("twoD: ", twoD)
}

func printHelper(name string, arr [5]int) {
    for i := 0; i < 5; i++ {
        fmt.Printf("%v[%v]: %v\n", name, i, arr[i])
    }

    // len 获取长度
    fmt.Printf("len of %v: %v\n", name, len(arr))

    // cap 也可以用来获取数组长度
    fmt.Printf("cap of %v: %v\n", name, cap(arr))

    fmt.Println()
}
```

在Go语言中，声明和初始化可以一起完成，因此这行代码是有效的，并且符合Go语言的语法规则。

总结起来：

- 变量的声明可以同时包括初始化操作。

- 变量的声明和初始化可以在包级别（全局变量）或函数内部进行。

- 赋值操作是指将一个已经声明的变量更新为新的值，在Go语言中，这种操作必须在函数体内部执行

### 切片

#### **切片组成要素：**

- 指针：指向底层数组

- 长度：切片中元素的长度，不能大于容量

- 容量：指针所指向的底层数组的总容量

#### **常见初始化方式**

- 使用 `make` 初始化

``` go
slice := make([]int, 5)     // 初始化长度和容量都为 5 的切片
slice := make([]int, 5, 10) // 初始化长度为 5, 容量为 10 的切片
```

- 使用简短定义

``` go
slice := []int{1, 2, 3, 4, 5}
```

- 使用数组来初始化切片

``` go
arr := [5]int{1, 2, 3, 4, 5}
slice := arr[0:3] // 左闭右开区间，最终切片为 [1,2,3]
```

- 使用切片来初始化切片

``` go
sliceA := []int{1, 2, 3, 4, 5}
sliceB := sliceA[0:3] // 左闭右开区间，sliceB 最终为 [1,2,3]
```

#### **长度和容量**

``` go
package main

import (
    "fmt"
)

func main() {
    slice := []int{1, 2, 3, 4, 5}
    fmt.Println("len: ", len(slice))
    fmt.Println("cap: ", cap(slice))

    //改变切片长度
    slice = append(slice, 6)
    fmt.Println("after append operation: ")
    fmt.Println("len: ", len(slice))
    fmt.Println("cap: ", cap(slice)) //注意，底层数组容量不够时，会重新分配数组空间，通常为两倍
}
```

以上代码，预期输出如下：

``` go
len:  5
cap:  5
after append operation:
len:  6
cap:  12
```

#### **注意点**

- 多个切片共享一个底层数组的情况

> 对底层数组的修改，将影响上层多个切片的值

``` go
package main

import (
    "fmt"
)

func main() {
    slice := []int{1, 2, 3, 4, 5}
    newSlice := slice[0:3]
    fmt.Println("before modifying underlying array:")
    fmt.Println("slice: ", slice)
    fmt.Println("newSlice: ", newSlice)
    fmt.Println()

    newSlice[0] = 6
    fmt.Println("after modifying underlying array:")
    fmt.Println("slice: ", slice)
    fmt.Println("newSlice: ", newSlice)
}
```

以上代码预期输出如下：

``` go
before modify underlying array:
slice:  [1 2 3 4 5]
newSlice:  [1 2 3]

after modify underlying array:
slice:  [6 2 3 4 5]
newSlice:  [6 2 3]
```

- 使用 `copy` 方法可以避免共享同一个底层数组

> 示例代码如下：

``` go
package main

import (
    "fmt"
)

func main() {
    slice := []int{1, 2, 3, 4, 5}
    newSlice := make([]int, len(slice))
    copy(newSlice, slice)
    fmt.Println("before modifying underlying array:")
    fmt.Println("slice: ", slice)
    fmt.Println("newSlice: ", newSlice)
    fmt.Println()

    newSlice[0] = 6
    fmt.Println("after modifying underlying array:")
    fmt.Println("slice: ", slice)
    fmt.Println("newSlice: ", newSlice)
}
```

以上代码预期输出如下：

``` go
before modifying underlying array:
slice:  [1 2 3 4 5]
newSlice:  [1 2 3 4 5]

after modifying underlying array:
slice:  [1 2 3 4 5]
newSlice:  [6 2 3 4 5]
```

### **小练习**

如何使用 `copy` 函数进行切片部分拷贝？

    // 假设切片 slice 如下:
    slice := []int{1, 2, 3, 4, 5}

    // 如何使用 copy 创建切片 newSlice, 该切片值为 [2, 3, 4]
    newSlice = copy(?,?)

``` go
// 假设切片 slice 如下:
sliceD := []int{1, 2, 3, 4, 5}
// 如何使用 copy 创建切片 newSlice, 该切片值为 [2, 3, 4]
newSliceE := make([]int, 3)
copy(newSliceE, sliceD[1:4])
fmt.Println("newSliceE", newSliceE)
```
