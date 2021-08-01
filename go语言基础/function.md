## 函数
函数是执行特定任务的代码块。在Go中，一个函数可以返回多个值
## 函数声明
函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。
```go
func funcName(parameter-list) (result-list) {
    function body//逻辑处理
    return value1, value2
}
```
## 函数的使用
```go
package main

import "fmt"

func main() {
	m := max(1, 2) // 函数调用
	fmt.Println(m) // 2
}

// 定义一个函数,返回两个int值中大的那个
func max(num1, num2 int) int {
	var result int  // 定义一个局部变量用来存放返回值
	if num1 > num2 {
		result = num1
	} else {
		result = num2
	}
	return result
}
```
### 函数命名规则
- 通常是动词和介词加上名词，例如scanWords
- 避免不必要的缩写，printError要⽐printErr更好⼀些
- 避免使⽤类型关键字，⽐如buildUserStruct看上去会很别扭
- 避免歧义，不能有多种⽤途解释造成误解
- 避免与内置函数同名，这会导致误⽤
- 避免使⽤数字，除⾮特定专有名词
- 统⼀使⽤camel/pascalcase拼写风格
### 函数参数
形参 vs 实参
形参：定义函数时，用于接收外部传入的数据
实参：调用函数时，传给形参的实际的数据

值传递 vs 引用传递
- 值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
- 所谓引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。
- Go中函数调用只有值传递,不管是指针、引⽤类型，还是其他类型参数，都是值拷贝传递。区别⽆⾮是拷贝⽬标对象，还是拷贝指针⽽已

变长参数
变参本质上就是⼀个切⽚。只能接收⼀到多个同类型参数，且必须放在列表尾部
```go
package main

import "fmt"

func test(str string, a ...int) {
	fmt.Printf("%T,%v\n", a, a)
}

func main() {
	test("abc", 1, 2, 3, 4)
}
```
> output
> []int,[1 2 3 4]

### 返回值
一个函数可以返回多个值
```go
package main

import "fmt"

func test() (int, int) {
	return 1, 2
}

func main() {
	x, y := test()
	fmt.Printf("%d,%d\n", x, y)
}
```
> output
> 1,2

隐式返回
```go
package main

import "fmt"

func sum(x, y int) (z int) { // 这里z可以当作函数的局部变量使用
	z = x + y // 这里z只是赋值
	return    // 相当于 return z
}

func main() {
	sum := sum(1, 2)
	fmt.Printf("%d", sum) // 3
}
```

同名遮蔽
```go
package main

import "fmt"

func sum(x, y int) (z int) {
	{
		z := x + y // 新定义的同名局部变量，同名遮蔽
		return     // 错误：z is shadowed during return(改成return z)
	}
	return
}

func main() {
	sum := sum(1, 2)
	fmt.Printf("%d", sum)
}
```
## 匿名函数
匿名函数是指没有定义名字的函数
```go
package main

import "fmt"

func main() {
	sum := func(x, y int) int { // 函数没有名称
		return x + y
	}(1,2) // 传递参数
	fmt.Printf("%d", sum) //3
}
```

## 闭包
闭包是函数和其引⽤环境的组合体
```go
package main

import "fmt"

func test(x int) func() {
	// 返回一个函数，而且函数引用了上下文环境变量x
	// 当该函数在main中运行是，它依然可以正确读到x的值
	return func() {
		fmt.Println(x)
	}
}

func main() {
	f := test(1)
	f() // 调用返回的函数，打印x的值：1
}
```
## 延迟函数
defer语句用于延迟函数的调用，每次defer都会把一个函数压入栈中，函数返回前再把延迟的函数取出并执行

为了方便描述，我们把创建defer的函数称为主函数，defer语句后面的函数称为延迟函数。

defer规则
- 规则一：延迟函数的参数在defer语句出现时就已经确定下来了
官方给出一个例子，如下所示：
```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```
defer语句中的fmt.Println()参数i值在defer出现时就已经确定下来，实际上是拷贝了一份。后面对变量i的修改不会影响fmt.Println()函数的执行，仍然打印"0"。
> 对于指针类型参数，规则仍然适用，只不过延迟函数的参数是一个地址值，这种情况下，defer后面的语句对变量的修改可能会影响延迟函数。

- 规则二：延迟函数执行按后进先出顺序执行，即先出现的defer最后执行
defer类似于入栈操作，执行defer类似于出栈操作。

- 规则三：延迟函数可能操作主函数的具名返回值
定义defer的函数，即主函数可能有返回值，返回值有没有名字没有关系，defer所作用的函数，即延迟函数可能会影响到返回值。

若要理解延迟函数是如何影响主函数返回值的，只要明白函数是如何返回的就足够了。

函数返回过程
关键字return不是一个原子操作，实际上return只代理汇编指令ret，即将跳转程序执行。比如语句return i，实际上分两步进行，即将i值存入栈中作为返回值，然后执行跳转，而defer的执行时机正是跳转前，所以说defer执行时还是有机会操作返回值的。

举个实际的例子进行说明这个过程：
```go
func deferFuncReturn() (result int) {
    i := 1

    defer func() {
       result++
    }()

    return i
}
```
该函数的return语句可以拆分成下面两行：
```go
result = i
return
```
而延迟函数的执行正是在return之前，即加入defer后的执行过程如下：
```go
result = i
result++
return
```
所以上面函数实际返回i++值。

关于主函数有不同的返回方式，但返回机制就如上机介绍所说，只要把return语句拆开都可以很好的理解，下面分别举例说明

主函数拥有匿名返回值，返回字面值

一个主函数拥有一个匿名的返回值，返回时使用字面值，比如返回"1"、"2"、"Hello"这样的值，这种情况下defer语句是无法操作返回值的。

一个返回字面值的函数，如下所示：
```go
func foo() int {
    var i int

    defer func() {
        i++
    }()

    return 1
}
```
上面的return语句，直接把1写入栈中作为返回值，延迟函数无法操作该返回值，所以就无法影响返回值。

主函数拥有匿名返回值，返回变量

一个主函数拥有一个匿名的返回值，返回使用本地或全局变量，这种情况下defer语句可以引用到返回值，但不会改变返回值。

一个返回本地变量的函数，如下所示：
```go
func foo() int {
    var i int

    defer func() {
        i++
    }()

    return i
}
```
上面的函数，返回一个局部变量，同时defer函数也会操作这个局部变量。对于匿名返回值来说，可以假定仍然有一个变量存储返回值，假定返回值变量为"anony"，上面的返回语句可以拆分成以下过程：
```go
anony = i
i++
return
```
由于i是整型，会将值拷贝给anony，所以defer语句中修改i值，对函数返回值不造成影响。

主函数拥有具名返回值

主函声明语句中带名字的返回值，会被初始化成一个局部变量，函数内部可以像使用局部变量一样使用该返回值。如果defer语句操作该返回值，可能会改变返回结果。

一个影响函返回值的例子：
```go
func foo() (ret int) {
    defer func() {
        ret++
    }()

    return 0
}
```
上面的函数拆解出来，如下所示：
```
ret = 0
ret++
return
```
函数真正返回前，在defer中对返回值做了+1操作，所以函数最终返回1。

在处理其他资源时，也可以采用defer机制，比如对文件的操作：
```go
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return ReadAll(f)
}
```
或是处理互斥锁
```go
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```