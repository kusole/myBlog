---
title: go语言基础--数据类型、变量、控制
date: 2017-07-06 10:43:48
tags: go语言基础
categories: go语言学习笔记
comment: false	
---
# Go语言内置基础数据类型

在自然界里面，有猫，有狗，有猪。有各种动物。每种动物都是不同的。  
比如猫会喵喵叫，狗会旺旺叫，猪会哼哼叫。。。  
Stop!!!  
好了，大家毕竟不是幼儿园的小朋友。介绍到这里就可以了。
<!--more-->
论点就是每个东西都有自己归属的类别(Type)。  
那么在Go语言里面，每个变量也都是有类别的，这种类别叫做`数据类型(Data Type)`。  
Go的数据类型有两种：一种是`语言内置的数据类型`，另外一种是`通过语言提供的自定义数据类型方法自己定义的自定义数据类型`。

先看看语言`内置的基础数据类型`

## 数值型(Number)

数值型有`三种`，一种是`整数类型`，另外一种是`带小数的类型`(一般计算机里面叫做`浮点数类型`)，还有一种`虚数类型`。  

整数类型不用说了，和数学里面的是一样的。和数学里面不同的地方在于计算机里面`正整数和零`统称为`无符号整型`，而`负整数`则称为`有符号整型`。  

Go的内置整型有`uint8`, `uint16`, `uint32`, `uint64`, `int8`, `int16`, `int32`和`int64`。其中`u`开头的类型就是`无符号整型`。无符号类型能够表示正整数和零。而有符号类型除了能够表示正整数和零外，还可以表示负整数。
另外还有一些别名类型，比如`byte`类型，这个类型和`uint8`是一样的，表示`字节类型`。另外一个是`rune类型`，这个类型和`int32`是一样的，用来表示`unicode的代码点`，就是unicode字符所对应的整数。

Go还定义了三个`依赖系统`的类型，`uint`，`int`和`uintptr`。因为在32位系统和64位系统上用来表示这些类型的位数是不一样的。

*对于32位系统*

uint=uint32  
int=int32  
uintptr为32位的指针  

*对于64位系统*

uint=uint64  
int=int64  
uintptr为64位的指针  

至于类型后面跟的数字8，16，32或是64则表示用来表示这个类型的位不同，`位越多，能表示的整数范围越大`。
比如对于用N位来表示的整数，如果是`有符号的整数`，能够表示的整数范围为`-2^(N-1) ~ 2^(N-1)－1`；如果是`无符号的整数`，则能表示的整数范围为`0 ～ 2^N`。

Go的浮点数类型有两种，`float32`和`float64`。float32又叫`单精度浮点型`，float64又叫做`双精度浮点型`。其`最主要的区别就是小数点后面能跟的小数位数不同`。

另外Go还有两个其他语言所没有的类型，`虚数类型`。`complex64`和`complex128`。

对于数值类型，其所共有的操作为`加法(＋)`，`减法(－)`，`乘法(＊)`和`除法(/)`。另外对于`整数类型`，还定义了`求余运算(%)`

求余运算为整型所独有。如果对浮点数使用求余，比如这样
```go
    package main

    import (
        "fmt"
    )

    func main() {
        var a float64 = 12
        var b float64 = 3

        fmt.Println(a % b)
    }
```

编译时候会报错

    invalid operation: a % b (operator % not defined on float64)

所以，这里我们可以知道所谓的`数据类型有两层意思`，一个是定义了`该类型所能表示的数`，另一个是定义了`该类型所能进行的操作`。
简单地说，对于一只小狗，你能想到的一定是狗的面貌和它会汪汪叫，而不是猫的面容和喵喵叫。


## 字符串类型(String)

字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由`单个字节`连接起来的。（对于汉字，通常由多个字节组成）。这就是说，传统的字符串是由字符组成的，而`Go的字符串不同`，是`由字节组成`的。这一点需要注意。

字符串的表示很简单。用(双引号"")或者(``号)来描述。

    "hello world"

或者

    `hello world`

唯一的区别是，**双引号之间的转义字符会被转义，而``号之间的转义字符保持原样不变**。
```go
    package main

    import (
        "fmt"
    )

    func main() {
        var a = "hello \n world"
        var b = `hello \n world`

        fmt.Println(a)
        fmt.Println("----------")
        fmt.Println(b)
    }
```
输出结果为
```go
    hello 
     world
    ----------
    hello \n world
```
字符串所能进行的一些基本操作包括:  
（1）`获取字符长度`  
（2）`获取字符串中单个字节`  
（3）`字符串连接`  
```go
    package main

    import (
        "fmt"
    )

    func main() {
        var a string = "hello"
        var b string = "world"

        fmt.Println(len(a))
        fmt.Println(a[1])
        fmt.Println(a + b)
    }
```
输出如下 
```
    5
    101
    helloworld
```
这里我们看到a[1]得到的是一个整数，这就证明了上面`"Go的字符串是由字节组成的这句话"`。我们还可以再验证一下。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var a string = "你"
		var b string = "好"
		fmt.Println(len(a))
		fmt.Println(len(b))
		fmt.Println(len(a + b))
		fmt.Println(a[0])
		fmt.Println(a[1])
		fmt.Println(a[2])
	}
```
输出
```
    3
    3
    6
    228
    189
    160
```
我们开始的时候，从上面的三行输出知道，"你"和"好"分别是用三个字节组成的。我们依次获取a的三个字节，输出，得到结果。


## 布尔型(Bool)

布尔型是表示`真值`和`假值`的类型。可选值为`true`和`false`。

所能进行的操作如下：
`&& and 与`
`|| or 或`
`!  not 非`

Go的布尔型取值`就是true`或`false`。`任何空值(nil)或者零值(0, 0.0, "")都不能作为布尔型来直接判断`。
```go
	package main

	import (
    	"fmt"
	)

	func main() {
    	var equal bool
    	var a int = 10
    	var b int = 20
    	equal = (a == b)
    	fmt.Println(equal)
	}
```
输出结果

    false

下面是错误的用法
```go
	package main

	import (
    	"fmt"
	)

	func main() {
    	if 0 {
        	fmt.Println("hello world")
    	}
    	if nil {
        	fmt.Println("hello world")
    	}
    	if "" {
     		fmt.Println("hello world")
    	}
	}
```
编译错误
```
    ./t.go:8: non-bool 0 (type untyped number) used as if condition
    ./t.go:11: non-bool nil used as if condition
    ./t.go:14: non-bool "" (type untyped string) used as if condition
```

上面介绍的是Go语言内置的基础数据类型。

# 变量和常量定义
现在我们讨论一下Go语言的变量定义。
 
## 变量定义 

所谓的变量就是一个拥有指定`名称`和`类型`的`数据存储位置`。  
在上面我们使用过变量的定义，现在我们来仔细看一个例子。  
 ```go 
	package main

	import (
		"fmt"
	)

	func main() {
		var x string = "hello world"
		fmt.Println(x)
	}
```
变量的定义首先使用`var`关键字，然后指定变量的名称`x`，再指定变量的类型`string`，在本例中，还对变量`x`进行了赋值，然后在命令行输出该变量。Go这种变量定义的方式和其他的语言有些不同，但是在使用的过程中，你会逐渐喜欢的。当然上面的变量定义方式还可以如下，即先定义变量，再赋值。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x string
		x = "hello world"
		fmt.Println(x)
	}
```
或者是直接赋值，让Go语言推断变量的类型。如下：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = "hello world"
		fmt.Println(x)
	}
```
当然，上面变量的定义还有一种`快捷方式`。如果你知道变量的初始值，完全可以像下面这样定义变量，完全让`Go来推断语言的类型`。这种定义的方式连关键字`var`都省略掉了。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		x := "hello world"
		fmt.Println(x)
	}
```
注意：上面这种使用`:=`方式定义变量的方式`只能用在函数内部`。
```go
	package main

	import (
		"fmt"
	)

	x:="hello world"
	func main() {
		y := 10
		fmt.Println(x)
		fmt.Println(y)
	}
```
对于上面的变量定义x是无效的。会导致编译错误：


	./test_var_quick.go:7: non-declaration statement outside function body


不过我们对上面的例子做下修改，比如这样是可以的。也就是使用var关键字定义的时候，如果给出初始值，就不需要显式指定变量类型。
```go
	package main

	import (
		"fmt"
	)

	var x = "hello world"

	func main() {
		y := 10
		fmt.Println(x)
		fmt.Println(y)
	}
```

`变量`之所以称为变量，就是因为`它们的值在程序运行过程中可以发生变化`，但是`它们的变量类型是无法改变的`。因为`Go语言是静态语言`，并`不支持`程序运行过程中`变量类型发生变化`。比如如果你强行将一个字符串值赋值给定义为int的变量，那么会发生编译错误。即使是强制类型转换也是不可以的。`强制类型转换只支持同类的变量类型`。比如数值类型之间强制转换。

下面我们看几个例子：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x string = "hello world"
		fmt.Println(x)
		x = "i love go language"
		fmt.Println(x)
	}
```
本例子演示变量的值在程序运行过程中发生变化，结果输出为
```
	hello world
	i love go language
```
我们尝试不同类型的变量之间转换
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x string = "hello world"
		fmt.Println(x)
		x = 11
		fmt.Println(x)
	}
```
在本例子中，如果试图将一个数值赋予字符串变量x，那么会发生错误：

	./test_var.go:10: cannot use 11 (type int) as type string in assignment

上面的意思就是无法将整型数值11当作字符串赋予给字符串变量。

但是同类的变量之间是可以强制转换的，如浮点型和整型之间的转换。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x float64 = 32.35
		fmt.Println(x)
		fmt.Println(int(x))
	}
```
输出的结果为
```
	32.35   
	32
```

## 变量命名

上面我们看了一些变量的使用方法，那么定义一个变量名称，有哪些要求呢？
这里我们要注意，`Go的变量名称必须以字母或下划线(_)开头，后面可以跟字母，数字，或者下划线(_)`。除此之外，Go语言并不关心你如何定义变量。我们通用的做法是定义一个用户友好的变量。假设你需要定义一个狗狗的年龄，那么使用dog_age作为变量名称要好于用x来定义变量。

## 变量作用域

现在我们再来讨论一下变量的作用域。所谓作用域就是可以有效访问变量的区域。比如很简单的，你不可能在一个函数func_a里面访问另一个函数func_b里面定义的局部变量x。所以变量的作用域目前分为两类，一个是`全局变量`，另一个是`局部变量`。下面我们看个全局变量的例子：
```go
	package main

	import (
		"fmt"
	)

	var x string = "hello world"

	func main() {
		fmt.Println(x)
	}
```
这里变量x定义在main函数之外，但是main函数仍然可以访问x。全局变量的作用域是该包中所有的函数。
```go
	package main

	import (
		"fmt"
	)

	var x string = "hello world"

	func change() {
		x = "i love go"
	}
	func main() {
		fmt.Println(x)
		change()
		fmt.Println(x)
	}
```
在上面的例子用，我们用了change函数改变了x的值。输出结果如下：
```
	hello world
	i love go
```

我们再看一下局部变量的例子。
```go
	package main

	import (
		"fmt"
	)

	func change() {
		x := "i love go"
	}
	func main() {
		fmt.Println(x)
	}
```
该例子中main函数试图访问change函数中定义的局部变量x，结果发生了下面的错误(未定义的变量x)：

	./test_var.go:11: undefined: x



## 常量

Go语言也支持常量定义。所谓`常量就是在程序运行过程中保持值不变的变量定义`。常量的定义和变量类似，只是用`const`关键字替换了var关键字，另外常量在定义的时候`必须有初始值`。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		const x string = "hello world"
		const y = "hello world"
		fmt.Println(x)
		fmt.Println(y)
	}
```
这里有一点需要注意，变量定义的类型推断方式`:=`不能够用来定义常量。因为常量的值是在编译的时候就已经确定的，但是变量的值则是运行的时候才使用的。这样常量定义就无法使用变量类型推断的方式了。

常量的值在运行过程中是无法改变的，强制改变常量的值是无效的。

```go
	package main

	import (
		"fmt"
	)

	func main() {
		const x string = "hello world"
		fmt.Println(x)
		x = "i love go language"
		fmt.Println(x)
	}
```
比如上面的例子就会报错

	./test_var.go:10: cannot assign to x

我们再看一个Go包math里面定义的常量Pi，用它来求圆的面积。
```go
	package main

	import (
		"fmt"
		"math"
	)

	func main() {
		var radius float64 = 10
		var area = math.Pow(radius, 2) * math.Pi
		fmt.Println(area)
	}
```

## 多变量或常量定义

Go还提供了一种`同时定义多个变量或者常量`的快捷方式。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var (
			a int     = 10
			b float64 = 32.45
			c bool    = true
		)
		const (
			Pi   float64 = 3.14
			True bool    = true
		)

		fmt.Println(a, b, c)
		fmt.Println(Pi, True)
	}
```
# 程序控制结构
虽然剧透可耻，但是为了体现Go语言的设计简洁之处，必须要先剧透一下。

Go语言的控制结构关键字只有

`if..else if..else`，`for` 和 `switch`。

而且在Go中，为了避免格式化战争，对程序结构做了统一的强制的规定。看下下面的例子。

请比较一下A程序和B程序的不同之处。

**A程序**
```go
	package main

	import (
		"fmt"
	)

	func main() {
		fmt.Println("hello world")
	}
```
**B程序**
```go
	package main

	import (
		"fmt"
	)

	func main() 
	{
		fmt.Println("hello world")
	}
```
还记得我们前面的例子中，`{}`的格式是怎么样的么？在上面的两个例子中只有A例的写法是对的。因为在Go语言中，强制了`{}`的格式。如果我们试图去编译B程序，那么会发生如下的错误提示。

	./test_format.go:9: syntax error: unexpected semicolon or newline before {

## if..else if..else

if..else if..else 用来判断一个或者多个条件，然后根据条件的结果执行不同的程序块。举个简单的例子。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var dog_age = 10

		if dog_age > 10 {
			fmt.Println("A big dog")
		} else if dog_age > 1 && dog_age <= 10 {
			fmt.Println("A small dog")
		} else {
			fmt.Println("A baby dog")
		}
	}
```
上面的例子判断狗狗的年龄如果`(if)`大于10就是一个大狗；否则判断`(else if)`狗狗的年龄是否小于等于10且大于1，这个时候狗狗是小狗狗。否则`(else)`的话（就是默认狗狗的年龄小于等于1岁），那么狗狗是Baby狗狗。

在上面的例子中，我们还可以发现Go的if..else if..else语句的判断条件一般都不需要使用`()`。当然如果你还是愿意写，也是对的。另外如果为了将某两个或多个条件绑定在一起判断的话，还是需要括号`()`的。

比如下面的例子也是对的。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		const Male = 'M'
		const Female = 'F'

		var dog_age = 10
		var dog_sex = 'M'

		if (dog_age == 10 && dog_sex == 'M') {
			fmt.Println("dog")
		}
	}
```
但是如果你使用Go提供的格式化工具来格式化这段代码的话，Go会智能判断你的括号是否必须有，否则的话，会帮你去掉的。你可以试试。

	go fmt test_bracket.go


然后你会发现，咦？！果真被去掉了。

另外因为每个判断条件的结果要么是true要么是false，所以可以使用`&&`，`||`来连接不同的条件。使用`!`来对一个条件取反。

## switch

switch的出现是为了解决某些情况下使用if判断语句带来的繁琐之处。

例如下面的例子：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		//score 为 [0,100]之间的整数
		var score int = 69

		if score >= 90 && score <= 100 {
			fmt.Println("优秀")
		} else if score >= 80 && score < 90 {
			fmt.Println("良好")
		} else if score >= 70 && score < 80 {
			fmt.Println("一般")
		} else if score >= 60 && score < 70 {
			fmt.Println("及格")
		} else {
			fmt.Println("不及格")
		}
	}
```
在上面的例子中，我们用if..else if..else来对分数进行分类。这个只是一般的情况下if判断条件的数量。如果if..else if..else的条件太多的话，我们可以使用switch来优化程序。比如上面的程序我们还可以这样写：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		//score 为 [0,100]之间的整数
		var score int = 69

		switch score / 10 {
		case 10:
		case 9:
			fmt.Println("优秀")
		case 8:
			fmt.Println("良好")
		case 7:
			fmt.Println("一般")
		case 6:
			fmt.Println("及格")
		default:
			fmt.Println("不及格")
		}
	}
```
关于switch的几点说明如下：

(1) switch的判断条件可以为任何数据类型。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var dog_sex = "F"
		switch dog_sex {
		case "M":
			fmt.Println("A male dog")
		case "F":
			fmt.Println("A female dog")
		}
	}
```

(2) 每个case后面跟的是一个完整的程序块，该程序块`不需要{}`，也`不需要break结尾`，因为每个case都是独立的。

(3) 可以为switch提供一个默认选项default，在上面所有的case都没有满足的情况下，默认执行default后面的语句。


## for

for用在Go语言的循环条件里面。比如说要你输出1...100之间的自然数。最笨的方法就是直接这样。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		fmt.Println(1)
		fmt.Println(2)
		...
		fmt.Println(100)
	}
```
这个不由地让我想起一个笑话。
>以前一个地主的儿子学习写字，只学了三天就把老师赶走了。因为在这三天里面他学写了一，二，三。他觉得写字真的太简单了，不就是画横线嘛。于是有一天老爹过寿，让他来记送礼的人名单。直到中午还没有记完，老爹很奇怪就去问他怎么了。他哭着说，“不知道这个人有什么毛病，姓什么不好，姓万”。

哈哈，回来继续。我们看到上面的例子也是如地主的儿子那样就不好了。所以，我们必须使用循环结构。我们用for的循环语句来实现上面的例子。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var i int = 1

		for ; i <= 100; i++ {
			fmt.Println(i)
		}
	}
```
在上面的例子中，首先初始化变量i为1，然后在for循环里面判断是否小于等于100，如果是的话，输出i，然后再使用i++来将i的值自增1。上面的例子，还有一个更好的写法，就是将i的定义和初始化也放在for里面。如下：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		for i := 1; i <= 100; i++ {
			fmt.Println(i)
		}
	}
```
在Go里面没有提供while关键字，如果你怀念while的写法也可以这样：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var i int = 1

		for i <= 100 {
			fmt.Println(i)
			i++
		}
	}
```
或许你会问，如果我要死循环呢？是不是`for true`？呵呵，不用了，直接这样。
```go
	for{
		...
	}
```

以上就是Go提供的全部控制流程了。

再复习一下，Go只提供了：

**if**
```go
	if ...{
		...
	}else if ...{
		...
	}else{
		...
	}
```
**switch**
```go	
	switch(...){
	case ...:
			 ...
	case ...:
			 ...
	...
	
	default:
			  ...
	}
```
**for**
```go	
	for ...; ...; ...{
		...
	}
	
	for ...{
		...
	}
	
	for{
		...
	}
```
# 数组，切片和字典

在上面的章节里面，我们讲过Go内置的基本数据类型。现在我们来看一下Go内置的高级数据类型，数组，切片和字典。

## 数组(Array)

数组是一个具有`相同数据类型`的元素组成的`固定长度`的`有序集合`。比如下面的例子

	var x [5]int
	
表示数组x是一个整型数组，而且数值的长度为5。

`Go提供了几种不同的数组定义方法。`

`最基本的方式就是使用var关键字来定义，然后依次给元素赋值`。`对于没有赋值的元素，默认为零值`。比如对于整数，零值就是0，浮点数，零值就是0.0，字符串，零值就是""，对象零值就是nil。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x [5]int
		x[0] = 2
		x[1] = 3
		x[2] = 3
		x[3] = 2
		x[4] = 12
		var sum int
		for _, elem := range x {
			sum += elem
		}
		fmt.Println(sum)
	}
```
在上面的例子中，我们首先使用`var`关键字来声明，然后给出数组名称`x`，最后说明数组为整型数组，长度为5。然后我们使用索引方式给数组元素赋值。在上面的例子中，我们还使用了一种遍历数组元素的方法。该方法利用Go语言提供的内置函数range来遍历数组元素。`range函数可以用在数组，切片和字典上面`。当`range来遍历数组的时候返回数组的索引和元素值`。在这里我们是对数组元素求和，所以我们对索引不感兴趣。在Go语言里面，`当你对一个函数返回值不感兴趣的话，可以使用下划线(_)来替代它`。另外这里如果我们真的定义了一个索引，在循环结构里面却没有使用索引，Go语言编译的时候还是会报错的。所以用下划线来替代索引变量也是唯一之举了。最后我们输出数组元素的和。

还有一种方式，如果知道了数组的初始值。可以像下面这样定义。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = [5]int{1, 2, 3, 4}
		x[4] = 5

		var sum int
		for _, i := range x {
			sum += i
		}
		fmt.Println(sum)
	}
```
当然，即使你不知道数组元素的初始值，也可以使用这样的定义方式。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = [5]int{}
		x[0] = 1
		x[1] = 2
		x[2] = 3
		x[3] = 4
		x[4] = 5

		var sum int
		for _, i := range x {
			sum += i
		}
		fmt.Println(sum)
	}
```
`在这里我们需要特别重视数组的一个特点，就是数组是有固定长度的。`

但是如果我们有的时候也可以不显式指定数组的长度，而是使用`...`来替代数组长度，Go语言会自动计算出数组的长度。不过这种方式定义的数组一定是有初始化的值的。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = [...]string{
			"Monday",
			"Tuesday",
			"Wednesday",
			"Thursday",
			"Friday",
			"Saturday",
			"Sunday"}

		for _, day := range x {
			fmt.Println(day)
		}
	}
```
在上面的例子中，还需要注意一点就是如果将数组元素定义在不同行上面，那么最后一个元素后面必须跟上`}`或者`,`。上面的例子也可以是这样的。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = [...]string{
			"Monday",
			"Tuesday",
			"Wednesday",
			"Thursday",
			"Friday",
			"Saturday",
			"Sunday",
		}

		for _, day := range x {
			fmt.Println(day)
		}
	}
```
`Go提供的这种可以自动计算数组长度的方法在调试程序的时候特别方便，假设我们注释掉上面数组x的最后一个元素，我们甚至不需要去修改数组的长度。`
	
## 切片(Slice)

在上面我们说过数组是有固定长度的有序集合。这也就是说一旦数组长度定义，你将无法在数组里面多添加哪怕一个元素。数组的这种特点有的时候会成为很大的缺点，尤其是当数组的元素个数不确定的情况下。

所以`切片`诞生了。

切片和数组很类似，甚至你可以理解成数组的子集。但是`切片有一个数组所没有的特点，那就是切片的长度是可变的`。

严格地讲，切片有`容量(capacity)`和`长度(length)`两个属性。

首先我们来看一下切片的定义。切片有两种定义方式，一种是先声明一个变量是切片，然后使用内置函数make去初始化这个切片。另外一种是通过取数组切片来赋值。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = make([]float64, 5)
		fmt.Println("Capcity:", cap(x), "Length:", len(x))
		var y = make([]float64, 5, 10)
		fmt.Println("Capcity:", cap(y), "Length:", len(y))

		for i := 0; i < len(x); i++ {
			x[i] = float64(i)
		}
		fmt.Println(x)

		for i := 0; i < len(y); i++ {
			y[i] = float64(i)
		}
		fmt.Println(y)
	}
```
输出结果为
```
	Capcity: 5 Length: 5
	Capcity: 10 Length: 5
	[0 1 2 3 4]
	[0 1 2 3 4]
```
上面我们首先用make函数定义切片x，这个时候x的容量是5，长度也是5。然后使用make函数定义了切片y，这个时候y的容量是10，长度是5。然后我们再分别为切片x和y的元素赋值，最后输出。

所以使用make函数定义切片的时候，有`两种方式`，一种`只指定长度，这个时候切片的长度和容量是相同的`。另外一种是`同时指定切片长度和容量`。虽然切片的容量可以大于长度，但是`赋值的时候要注意最大的索引仍然是len(x)－1`。否则会报索引超出边界错误。

另外一种是通过数组切片赋值，采用`[low_index:high_index]`的方式获取数值切片，其中切片元素`包括low_index的元素`，但是`不包括high_index的元素`。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var arr1 = [5]int{1, 2, 3, 4, 5}
		var s1 = arr1[2:3]
		var s2 = arr1[:3]
		var s3 = arr1[2:]
		var s4 = arr1[:]
		fmt.Println(s1)
		fmt.Println(s2)
		fmt.Println(s3)
		fmt.Println(s4)
	}
```
输出结果为
```
	[3]
	[1 2 3]
	[3 4 5]
	[1 2 3 4 5]
```


在上面的例子中，我们还省略了low_index或high_index。如果省略了low_index，那么等价于从索引0开始；如果省略了high_index，则默认high_index等于len(arr1)，即切片长度。

这里为了体现切片的长度可以变化，我们看一下下面的例子：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var arr1 = make([]int, 5, 10)
		for i := 0; i < len(arr1); i++ {
			arr1[i] = i
		}
		fmt.Println(arr1)

		arr1 = append(arr1, 5, 6, 7, 8)
		fmt.Println("Capacity:", cap(arr1), "Length:", len(arr1))
		fmt.Println(arr1)
	}
```
输出结果为
```
	[0 1 2 3 4]
	Capacity: 10 Length: 9
	[0 1 2 3 4 5 6 7 8]
```
这里我们初始化arr1为容量10，长度为5的切片，然后为前面的5个元素赋值。然后输出结果。然后我们再使用Go内置方法append来为arr1追加四个元素，这个时候再看一下arr1的容量和长度以及切片元素，我们发现切片的长度确实变了。

另外我们再用`append`方法给arr1多追加几个元素，试图超过arr1原来定义的容量大小。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var arr1 = make([]int, 5, 10)
		for i := 0; i < len(arr1); i++ {
			arr1[i] = i
		}

		arr1 = append(arr1, 5, 6, 7, 8, 9, 10)
		fmt.Println("Capacity:", cap(arr1), "Length:", len(arr1))
		fmt.Println(arr1)
	}
```
输出结果为
```
	Capacity: 20 Length: 11
	[0 1 2 3 4 5 6 7 8 9 10]
```
我们发现arr1的长度变为11，因为元素个数现在为11个。另外我们发现arr1的容量也变了，变为原来的两倍。这是因为`Go在默认的情况下，如果追加的元素超过了容量大小，Go会自动地重新为切片分配容量，容量大小为原来的两倍`。

上面我们介绍了，可以`使用append函数给切片增加元素`，现在我们再来介绍一个`copy函数用来从一个切片拷贝元素到另一个切片`。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		slice1 := []int{1, 2, 3, 4, 5, 6}
		slice2 := make([]int, 5, 10)
		copy(slice2, slice1)
		fmt.Println(slice1)
		fmt.Println(slice2)
	}
```
输出结果
```
	[1 2 3 4 5 6]
	[1 2 3 4 5]
```
在上面的例子中，我们将slice1的元素拷贝到slice2，因为slice2的长度为5，所以最多拷贝5个元素。

总结一下，数组和切片的区别就在于`[]`里面是否有数字或者`...`。因为数值长度是固定的，而切片是可变的。


## 字典(Map)

字典是一组`无序的`，`键值对`的`集合`。

字典也叫做`关联数组`，因为数组通过`索引`来查找元素，而字典通过`键`来查找元素。当然，很显然的，字典的键是不能重复的。如果试图赋值给同一个键，后赋值的值将覆盖前面赋值的值。

字典的定义也有两种，一种是`初始化数据`的定义方式，另一种是`使用神奇的make函数`来定义。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x = map[string]string{
			"A": "Apple",
			"B": "Banana",
			"O": "Orange",
			"P": "Pear",
		}

		for key, val := range x {
			fmt.Println("Key:", key, "Value:", val)
		}
	}
```
输出结果为
```
	Key: A Value: Apple
	Key: B Value: Banana
	Key: O Value: Orange
	Key: P Value: Pear
```
在上面的例子中，我们定义了一个string:string的字典，其中`[]`之间的是键类型，右边的是值类型。另外我们还看到了`range函数，此函数一样神奇，可以用来迭代字典元素，返回key:value键值对`。当然如果你对键或者值不感兴趣，一样可以使用`下划线(_)`来忽略返回值。

```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x map[string]string

		x = make(map[string]string)

		x["A"] = "Apple"
		x["B"] = "Banana"
		x["O"] = "Orange"
		x["P"] = "Pear"

		for key, val := range x {
			fmt.Println("Key:", key, "Value:", val)
		}
	}
```
上面的方式就是使用了make函数来初始化字典，`试图为未经过初始化的字典添加元素会导致运行错误`，你可以把使用make函数初始化的那一行注释掉，然后看一下。

当然上面的例子中，我们可以把定义和初始化合成一句。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		x := make(map[string]string)

		x["A"] = "Apple"
		x["B"] = "Banana"
		x["O"] = "Orange"
		x["P"] = "Pear"

		for key, val := range x {
			fmt.Println("Key:", key, "Value:", val)
		}
	}
```
现在我们再来看一下字典的数据访问方式。如果你访问的元素所对应的键存在于字典中，那么没有问题，如果不存在呢？

这个时候会返回零值。对于字符串零值就是""，对于整数零值就是0。但是对于下面的例子：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		x := make(map[string]int)

		x["A"] = 0
		x["B"] = 20
		x["O"] = 30
		x["P"] = 40

		fmt.Println(x["C"])
	}
```
在这个例子中，很显然不存在键C，但是程序的输出结果为0，这样就和键A对应的值混淆了。

Go提供了一种方法来解决这个问题：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		x := make(map[string]int)

		x["A"] = 0
		x["B"] = 20
		x["O"] = 30
		x["P"] = 40

		if val, ok := x["C"]; ok {
			fmt.Println(val)
		}
	}
```
上面的例子中，我们可以看到事实上使用`x["C"]`的返回值有两个，一个是值，另一个是是否存在此键的bool型变量，所以我们看到ok为true的时候就输出键C的值，如果ok为false，那就是字典中不存在这个键。

现在我们再来看看`Go提供的内置函数delete，这个函数可以用来从字典中删除元素`。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		x := make(map[string]int)

		x["A"] = 10
		x["B"] = 20
		x["C"] = 30
		x["D"] = 40

		fmt.Println("Before Delete")
		fmt.Println("Length:", len(x))
		fmt.Println(x)

		delete(x, "A")

		fmt.Println("After Delete")
		fmt.Println("Length:", len(x))
		fmt.Println(x)
	}
```
输出结果为
```
	Before Delete
	Length: 4
	map[A:10 B:20 C:30 D:40]
	After Delete
	Length: 3
	map[B:20 C:30 D:40]
```
我们在删除元素前查看一下字典长度和元素，删除之后再看一下。这里面我们还可以看到`len函数也可以用来获取字典的元素个数`。当然如果你试图删除一个不存在的键，那么程序也不会报错，只是不会对字典造成任何影响。

最后我们再用一个稍微复杂的例子来结束字典的介绍。

我们有一个学生登记表，登记表里面有一组学号，每个学号对应一个学生，每个学生有名字和年龄。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var facebook = make(map[string]map[string]int)
		facebook["0616020432"] = map[string]int{"Jemy": 25}
		facebook["0616020433"] = map[string]int{"Andy": 23}
		facebook["0616020434"] = map[string]int{"Bill": 22}

		for stu_no, stu_info := range facebook {
			fmt.Println("Student:", stu_no)
			for name, age := range stu_info {
				fmt.Println("Name:", name, "Age:", age)
			}
			fmt.Println()
		}
	}
```
输出结果为
```
	Student: 0616020432
	Name Jemy Age 25

	Student: 0616020433
	Name Andy Age 23

	Student: 0616020434
	Name Bill Age 22
```
当然我们也可以用初始化的方式定义字典：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var facebook = map[string]map[string]int{
			"0616020432": {"Jemy": 25},
			"0616020433": {"Andy": 23},
			"0616020434": {"Bill": 22},
		}

		for stu_no, stu_info := range facebook {
			fmt.Println("Student:", stu_no)
			for name, age := range stu_info {
				fmt.Println("Name:", name, "Age:", age)
			}
			fmt.Println()
		}
	}
```

输出结果是一样的。

