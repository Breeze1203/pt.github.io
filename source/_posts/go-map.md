---
title: "go map"
date: 2024-11-18 15:41:52
---

在 Go 语言里面，map 一种无序的键值对, 它是数据结构 hash 表的一种实现方式，类似 Python 中的字典。

### **语法**

使用关键字 map 来声明形如：

``` go
map[KeyType]ValueType
```

注意点：

- 必须指定 key, value 的类型，插入的纪录类型必须匹配。

- <span class="mark" style="display: inline-block;">key 具有唯一性，插入纪录的 key 不能重复。</span>

- KeyType 可以为基础数据类型（例如 bool, 数字类型，字符串）, 不能为数组，切片，map，它的取值必须是能够使用 `==` 进行比较。

- <span class="mark" style="display: inline-block;">ValueType 可以为任意类型。</span>

- 无序性。

- 线程不安全, 一个 goroutine 在对 map 进行写的时候，另外的 goroutine 不能进行读和写操作，Go 1.6 版本以后会抛出 runtime 错误信息。

### **声明和初始化**

- 使用 var 声明

``` go
var cMap map[string]int  // 只定义, 此时 cMap 为 nil
fmt.Println(cMap == nil)
cMap["北京"] = 1  // 报错，因为 cMap 为 nil
```

- 使用 `make`

``` go
cMap := make(map[string]int)
cMap["北京"] = 1

// 指定初始容量
cMap = make(map[string]int, 100)
cMap["北京"] = 1
```

说明：在使用 make 初始化 map 的时候，可以指定初始容量，这在能预估 map key 数量的情况下，减少动态分配的次数，从而提升性能。

- 简短声明方式

``` go
cMap := map[string]int{"北京": 1}
```

### **map 基本操作**

``` go
cMap := map[string]int{}

cMap["北京"] = 1 //写

code := cMap["北京"] // 读
fmt.Println(code)

code = cMap["广州"]  // 读不存在 key
fmt.Println(code)

code, ok = cMap["广州"]  // 检查 key 是否存在
if ok {
  fmt.Println(code)  
} else {
  fmt.Println("key not exist")  
}

delete(cMap, "北京") // 删除 key
fmt.Println("北京")
```

### **循环和无序性**

    cMap := map[string]int{"北京": 1, "上海": 2, "广州": 3, "深圳": 4}

    for city, code := range cMap {
      fmt.Printf("%s:%d", city, code)
      fmt.Println()
    }

### **线程不安全**

``` go
cMap := make(map[string]int)

var wg sync.WaitGroup
wg.Add(2)

go func() {
    cMap["北京"] = 1
    wg.Done()
}()

go func() {
    cMap["上海"] = 2
    wg.Done()
}()

wg.Wait()
```

在 Go 1.6 之后的版本，多次运行此段代码，你将遇到这样的错误信息：

``` go
fatal error: concurrent map writes

goroutine x [running]:
runtime.throw(0x10c64b6, 0x15)
.....
```

解决之道：

- 对读写操作加锁

- 使用 security map, 例如 `sync.map`

### **map 嵌套**

``` go
provinces := make(map[string]map[string]int)

provinces["北京"] = map[string]int{
  "东城区": 1,
  "西城区": 2,
  "朝阳区": 3,
  "海淀区": 4,
}

fmt.Println(provinces["北京"])
```

### java回顾：

在Java中，键（Key）和值（Value）类型在使用\`Map\`接口及其实现时，主要有以下几个方面的限制或考虑：

1\. **非空性**：大多数\`Map\`实现要求键和值都不能为\`null\`。例如，\`HashMap\`、\`TreeMap\`和\`LinkedHashMap\`都不允许键或值为\`null\`。然而，\`Hashtable\`和\`ConcurrentHashMap\`允许键为\`null\`，但不允许值为\`null\`。

2\. **可哈希性**：键必须提供合适的\`hashCode()\`实现，因为\`Map\`实现依赖于键的\`hashCode()\`方法来快速定位键值对。如果键对象没有恰当的\`hashCode()\`实现，那么\`Map\`的性能可能会受到影响。

3\. **相等性**：键必须正确实现\`equals()\`方法，因为\`Map\`使用\`equals()\`来确定两个键是否相等。如果两个键对象通过\`equals()\`方法比较相等，那么它们应该被认为是同一个键。

4\. **不可变性**：对于键，最好是不可变的，因为如果键对象在插入到\`Map\`之后其\`hashCode()\`或\`equals()\`行为发生变化，可能会导致\`Map\`无法正确地定位或删除键值对。

5\. **性能**：选择适当的键类型可以影响\`Map\`的性能。例如，使用\`Integer\`或\`String\`这样的轻量级对象作为键通常比使用复杂对象更高效。

6\. **类型限制**：某些特定的\`Map\`实现可能有额外的类型限制。例如，\`EnumMap\`的键必须是枚举类型。

7\. **泛型**：Java泛型确保了键和值类型的安全性。例如，\`Map\<String, Integer\>\`明确表示键是\`String\`类型，值是\`Integer\`类型。

8\. **线程安全**：如果你的应用程序需要在多线程环境中使用\`Map\`，那么你可能需要使用线程安全的\`Map\`实现，如\`Hashtable\`、\`ConcurrentHashMap\`或通过\`Collections.synchronizedMap()\`包装的\`Map\`。
