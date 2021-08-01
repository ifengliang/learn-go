## Map
Golang的map使用哈希表作为底层实现，一个哈希表里可以有多个哈希表节点，也即bucket，而每个bucket就保存了map中的一个或一组键值对
### 创建map
1. 内置的make函数可以创建一个map：
```go
package main

import "fmt"

func main() {
	m := make(map[string]int)
	fmt.Println(m) //map[]
}
```
2. 我们也可以用map字面值的语法创建map，同时还可以指定一些最初的key/value：
```go
package main

import "fmt"

func main() {
	m := map[string]int{
		"a": 1,
		"b": 2,
	}
	fmt.Println(m) //map[a:1 b:2]
}
```
### 新增/修改/删除
```go
package main

import "fmt"

func main() {
	m := make(map[string]int)
	m["a"] = 1     // 新增 "a":1的键值对
	fmt.Println(m) // map[a:1]

	m["a"] = 2     // 修改
	fmt.Println(m) // map[a:2]

	delete(m, "a") // 删除键值对,key不存在时也不会出错
	fmt.Println(m) // map[]
}
```
### 判断key是否存在
但是有时候可能需要知道对应的元素是否真的是在map之中, 可以像下面这样测试
```
package main

import "fmt"

func main() {
	m := make(map[string]int)
	m["a"] = 1
	m["b"] = 2
	fmt.Println(m) // map[a:2]

	if v, ok := m["a"]; ok { // 使用ok-idiom判断key是否存在,并返回值
		fmt.Println(v) // 2
	}
}
```

### 遍历map
遍历map中全部的key/value对：
```go
package main

import (
	"fmt"
)

func main() {
	m := make(map[string]int)
	m["a"] = 1
	m["b"] = 2
	m["c"] = 3

	for k, v := range m {
		fmt.Printf("%s\t%d\n", k, v)
	}
}
```
> output:
> a	1
b	2
c	3

如果要按顺序遍历key/value对，我们必须显式地对key进行排序，可以使用sort包的Strings函数对字符串slice进行排序
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	m := make(map[string]int)
	m["c"] = 3
	m["b"] = 2
	m["a"] = 1

	var keys []string
	for k := range m {
		keys = append(keys, k)
	}
	sort.Strings(keys)
	for _, k := range keys {
		fmt.Printf("%s\t%d\n", k, m[k])
	}
}
```
> output
> a	1
b	2
c	3

### map的内存结构

https://blog.csdn.net/weixin_39685836/article/details/110397523?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control