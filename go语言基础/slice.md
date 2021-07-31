# 数组和切片
## 数组
数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，因此在Go语言中很少直接使用数组
数组的每个元素可以通过索引下标来访问，索引下标的范围是从0开始到数组长度减1的位置。内置的len函数将返回数组中元素的个数。
```go
package main

import (
	"fmt"
)

func main() {
	array := [5]int{1, 2, 3}
	fmt.Println(array[0]) // 1
	fmt.Println(array) // [1 2 3 0 0 ]
}
```

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。
```go
package main

import (
	"fmt"
)

func main() {
	q := [...]int{1, 2, 3}
	fmt.Printf("%T\n", q) // "[3]int"
}
```

数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定
```go
r := [...]int{99: -1}
```
定义了一个含有100个元素的数组r，最后一个元素被初始化为-1，其它元素都是用0初始化

## Slice
Slice（切片）代表变长的序列，序列中每个元素都有相同的类型
slice的底层确实引用一个数组对象。一个slice由三个部分构成：指针、长度和容量
指针指向第一个slice元素对应的底层数组元素的地址
长度对应slice中元素的数目
容量一般是从slice的开始位置到底层数据的结尾位置

多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠

![slice01](https://books.studygolang.com/gopl-zh/images/ch4-01.png)

slice之间不能比较，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较：
```go
func equal(x, y []string) bool {
    if len(x) != len(y) {
        return false
    }
    for i := range x {
        if x[i] != y[i] {
            return false
        }
    }
    return true
}
```
slice唯一合法的比较操作是和nil比较，例如：
```go
if summer == nil { /* ... */ }
```

内置的append函数用于向slice追加元素：
```go
var runes []rune
for _, r := range "Hello, 世界" {
    runes = append(runes, r)
}
fmt.Printf("%q\n", runes) // "['H' 'e' 'l' 'l' 'o' ',' ' ' '世' '界']"
```

每次调用append函数，必须先检测slice底层数组是否有足够的容量来保存新添加的元素。如果有足够空间的话，直接扩展slice（依然在原有的底层数组之上），将新添加的y元素复制到新扩展的空间，并返回slice,如果没有足够的增长空间的话，append函数则会先分配一个足够大的slice用于保存新的结果，先将输入的x复制到新的空间，然后添加y元素。结果z和输入的x引用的将是不同的底层数组。

slice并不是一个纯粹的引用类型，它实际上是一个类似下面结构体的聚合类型：
```go
type IntSlice struct {
    ptr      *int
    len, cap int
}
```