---
title: go语言基础--函数、指针、结构体、接口
date: 2017-07-06 11:10:36
tags: go语言基础
categories: go语言学习笔记
---
# Go函数
是时候讨论一下Go的函数定义了。

## 什么是函数

函数，简单来讲就是一段将`输入数据`转换为`输出数据`的`公用代码块`。当然有的时候函数的返回值为空，那么就是说输出数据为空。而真正的处理过程在函数内部已经完成了。
<!--more-->
想一想我们为什么需要函数，最直接的需求就是代码中有太多的重复代码了，为了代码的可读性和可维护性，将这些重复代码重构为函数也是必要的。

## 函数定义

先看一个例子
```go
	package main

	import (
		"fmt"
	)

	func slice_sum(arr []int) int {
		sum := 0
		for _, elem := range arr {
			sum += elem
		}
		return sum
	}

	func main() {
		var arr1 = []int{1, 3, 2, 3, 2}
		var arr2 = []int{3, 2, 3, 1, 6, 4, 8, 9}
		fmt.Println(slice_sum(arr1))
		fmt.Println(slice_sum(arr2))
	}
```
在上面的例子中，我们需要分别计算两个切片的元素和。如果我们把计算切片元素的和的代码分别为两个切片展开，那么代码就失去了简洁性和一致性。假设你预想实现同样功能的代码在拷贝粘贴的过程中发生了错误，比如忘记改变量名之类的，到时候debug到崩溃吧。因为这时很有可能你就先入为主了，因为模板代码没有错啊，是不是。所以函数就是这个用处。

我们再仔细看一下上面的函数定义：

首先是关键字`func`，然后后面是`函数名称`，`参数列表`，最后是`返回值列表`。当然如果函数没有参数列表或者返回值，那么这两项都是可选的。其中返回值两边的括号在只声明一个返回值类型的时候可以省略。

## 命名返回值

Go的函数很有趣，你甚至可以为返回值预先定义一个名称，在函数结束的时候，直接一个return就可以返回所有的预定义返回值。例如上面的例子，我们将sum作为命名返回值。
```go
	package main

	import (
		"fmt"
	)

	func slice_sum(arr []int) (sum int) {
		sum = 0
		for _, elem := range arr {
			sum += elem
		}
		return
	}

	func main() {
		var arr1 = []int{1, 3, 2, 3, 2}
		var arr2 = []int{3, 2, 3, 1, 6, 4, 8, 9}
		fmt.Println(slice_sum(arr1))
		fmt.Println(slice_sum(arr2))
	}
```
这里要注意的是，如果你定义了命名返回值，那么在函数内部你将不能再重复定义一个同样名称的变量。比如第一个例子中我们用`sum:=0`来定义和初始化变量sum，而在第二个例子中，我们只能用`sum=0`初始化这个变量了。因为`:=`表示的是定义并且初始化变量。

## 实参数和虚参数

可能你听说过函数的实参数和虚参数。其实所谓的`实参数就是函数调用的时候传入的参数`。在上面的例子中，实参就是`arr1`和`arr2`，而`虚参数就是函数定义的时候表示函数需要传入哪些参数的占位参数`。在上面的例子中，虚参就是`arr`。`实参和虚参的名字不必是一样的。即使是一样的，也互不影响。`因为虚参是函数的内部变量。而实参则是另一个函数的内部变量或者是全局变量。它们的作用域不同。如果一个函数的虚参碰巧和一个全局变量名称相同，那么函数使用的也是虚参。例如我们再修改一下上面的例子。
```go
	package main

	import (
		"fmt"
	)

	var arr = []int{1, 3, 2, 3, 2}

	func slice_sum(arr []int) (sum int) {
		sum = 0
		for _, elem := range arr {
			sum += elem
		}
		return
	}

	func main() {
		var arr2 = []int{3, 2, 3, 1, 6, 4, 8, 9}
		fmt.Println(slice_sum(arr))
		fmt.Println(slice_sum(arr2))
	}
```

在上面的例子中，我们定义了全局变量arr并且初始化值，而我们的slice_sum函数的虚参也是arr，但是程序同样正常工作。

## 函数多返回值

记不记得你在java或者c里面需要返回多个值时还得去定义一个对象或者结构体的呢？在Go里面，你不需要这么做了。Go函数支持你返回多个值。

其实函数的多返回值，我们在上面遇见过很多次了。那就是`range`函数。这个函数用来迭代数组或者切片的时候返回的是两个值，一个是数组或切片元素的索引，另外一个是数组或切片元素。在上面的例子中，因为我们不需要元素的索引，所以我们用一个特殊的忽略返回值符号`下划线(_)`来忽略索引。

假设上面的例子我们除了返回切片的元素和，还想返回切片元素的平均值，那么我们修改一下代码。
```go
	package main

	import (
		"fmt"
	)

	func slice_sum(arr []int) (int, float64) {
		sum := 0
		avg := 0.0
		for _, elem := range arr {
			sum += elem
		}
		avg = float64(sum) / float64(len(arr))
		return sum, avg
	}

	func main() {
		var arr1 = []int{3, 2, 3, 1, 6, 4, 8, 9}
		fmt.Println(slice_sum(arr1))
	}
```

很简单吧，当然我们还可以将上面的参数定义为命名参数
```go
	package main

	import (
		"fmt"
	)

	func slice_sum(arr []int) (sum int, avg float64) {
		sum = 0
		avg = 0.0
		for _, elem := range arr {
			sum += elem
		}
		avg = float64(sum) / float64(len(arr))
		//return sum, avg
		return
	}

	func main() {
		var arr1 = []int{3, 2, 3, 1, 6, 4, 8, 9}
		fmt.Println(slice_sum(arr1))
	}
```

在上面的代码里面，将`return sum, avg`给注释了而直接使用`return`。其实这两种返回方式都可以。

## 变长参数

想一想我们的fmt包里面的Println函数，它怎么知道你传入的参数个数呢？
```go
	package main

	import (
		"fmt"
	)

	func main() {
		fmt.Println(1)
		fmt.Println(1, 2)
		fmt.Println(1, 2, 3)
	}
```
这个要归功于Go的一大特性，支持可变长参数列表。

首先我们来看一个例子
```go
	package main

	import (
		"fmt"
	)

	func sum(arr ...int) int {
		sum := 0
		for _, val := range arr {
			sum += val
		}
		return sum
	}
	func main() {
		fmt.Println(sum(1))
		fmt.Println(sum(1, 2))
		fmt.Println(sum(1, 2, 3))
	}
```

在上面的例子中，我们将原来的切片参数修改为可变长参数，然后使用range函数迭代这些参数，并求和。
从这里我们可以看出至少一点那就是`可变长参数列表里面的参数类型都是相同的`（*如果你对这句话表示怀疑，可能是因为你看到Println函数恰恰可以输出不同类型的可变参数，这个问题的答案要等到我们介绍完Go的接口后才行*）。

另外还有一点需要注意，那就是`可变长参数定义只能是函数的最后一个参数`。比如下面的例子：
```go
	package main

	import (
		"fmt"
	)

	func sum(base int, arr ...int) int {
		sum := base
		for _, val := range arr {
			sum += val
		}
		return sum
	}
	func main() {
		fmt.Println(sum(100, 1))
		fmt.Println(sum(200, 1, 2))
		fmt.Println(sum(300, 1, 2, 3))
	}
```

这里不知道你是否觉得这个例子其实和那个切片的例子很像啊，在哪里呢？
```go
	package main

	import (
		"fmt"
	)

	func sum(base int, arr ...int) int {
		sum := base
		for _, val := range arr {
			sum += val
		}
		return sum
	}
	func main() {
		var arr1 = []int{1, 2, 3, 4, 5}
		fmt.Println(sum(300, arr1...))
	}
```
呵呵，就是把切片“啪，啪，啪”三个耳光打碎了，传递过去啊！:-P


## 闭包函数

曾经使用python和javascript的时候就在想，如果有一天可以把这两种语言的特性做个并集该有多好。

这一天终于来了，Go支持闭包函数。

首先看一个闭包函数的例子。所谓闭包函数就是将整个函数的定义一气呵成写好并赋值给一个变量。然后用这个变量名作为函数名去调用函数体。

我们将刚刚的例子修改一下：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var arr1 = []int{1, 2, 3, 4, 5}
		
		var sum = func(arr ...int) int {
			total_sum := 0
			for _, val := range arr {
				total_sum += val
			}
			return total_sum
		}
		fmt.Println(sum(arr1...))
	}
```
从这里我们可以看出，其实闭包函数也没有什么特别之处。因为Go不支持在一个函数的内部再定义一个嵌套函数，所以使用闭包函数能够实现在一个函数内部定义另一个函数的目的。

这里我们需要注意的一个问题是，闭包函数对它外层的函数中的变量具有`访问`和`修改`的权限。例如：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var arr1 = []int{1, 2, 3, 4, 5}
		var base = 300
		var sum = func(arr ...int) int {
			total_sum := 0
			total_sum += base
			for _, val := range arr {
				total_sum += val
			}
			return total_sum
		}
		fmt.Println(sum(arr1...))
	}
```

这个例子，输出315，因为total_sum加上了base的值。
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var base = 0
		inc := func() {
			base += 1
		}
		fmt.Println(base)
		inc()
		fmt.Println(base)
	}
```

在上面的例子中，闭包函数修改了main函数的局部变量base。

最后我们来看一个闭包的示例，生成偶数序列。
```go
	package main

	import (
		"fmt"
	)

	func createEvenGenerator() func() uint {
		i := uint(0)
		return func() (retVal uint) {
			retVal = i
			i += 2
			return
		}
	}
	func main() {
		nextEven := createEvenGenerator()
		fmt.Println(nextEven())
		fmt.Println(nextEven())
		fmt.Println(nextEven())
	}
```

这个例子很有意思的，因为我们定义了一个`返回函数定义`的函数。而所返回的函数定义就是`在这个函数的内部定义的闭包函数`。这个闭包函数在外层函数调用的时候，每次都生成一个新的偶数（加2操作）然后返回闭包函数定义。

其中`func() uint`就是函数createEvenGenerator的返回值。在createEvenGenerator中，这个返回值是return返回的闭包函数定义。
```go
	func() (retVal uint) {
        	retVal = i
        	i += 2
        	return
    	}
```
因为createEvenGenerator函数返回的是一个函数定义，所以我们再把它赋值给一个代表函数的变量，然后用这个代表闭包函数的变量去调用函数执行。

## 递归函数

每次谈到递归函数，必然绕不开阶乘和斐波拉切数列。

阶乘
```go
	package main

	/**
	    n!=1*2*3*...*n
	*/
	import (
		"fmt"
	)

	func factorial(x uint) uint {
		if x == 0 {
			return 1
		}
		return x * factorial(x-1)
	}

	func main() {
		fmt.Println(factorial(5))
	}
```

如果x为0，那么返回1，因为0!=1。如果x是1，那么f(1)=1*f(0)，如果x是2，那么f(2)=2*f(1)=2*1*f(0)，依次推断f(x)=x*(x-1)*...*2*1*f(0)。

从上面看出所谓递归，就是在函数的内部重复调用一个函数的过程。需要注意的是这个函数必须能够一层一层分解，并且有出口。上面的例子出口就是0。

斐波拉切数列

求第N个斐波拉切元素
```go
	package main

	/**
		f(1)=1
		f(2)=2
		f(n)=f(n-2)+f(n-1)
	*/
	import (
		"fmt"
	)

	func fibonacci(n int) int {
		var retVal = 0
		if n == 1 {
			retVal = 1
		} else if n == 2 {
			retVal = 2
		} else {
			retVal = fibonacci(n-2) + fibonacci(n-1)
		}
		return retVal

	}
	func main() {
		fmt.Println(fibonacci(5))
	}
```

斐波拉切第一个元素是1，第二个元素是2，后面的元素依次是前两个元素的和。

其实对于递归函数来讲，只要知道了函数的出口，后面的不过是让计算机去不断地推断，一直推断到这个出口。理解了这一点，递归就很好理解了。


## 异常处理

当你读取文件失败而退出的时候是否担心文件句柄是否已经关闭？抑或是你对于try...catch...finally的结构中finally里面的代码和try里面的return代码那个先执行这样的问题痛苦不已？

一切都结束了。一门完美的语言必须有一个清晰的无歧义的执行逻辑。

好，来看看Go提供的异常处理。

*defer*
```go
	package main

	import (
		"fmt"
	)

	func first() {
		fmt.Println("first func run")
	}
	func second() {
		fmt.Println("second func run")
	}

	func main() {
		defer second()
		first()
	}
```
Go语言提供了关键字`defer`来在函数运行结束的时候运行一段代码或调用一个清理函数。上面的例子中，虽然second()函数写在first()函数前面，但是由于使用了defer标注，所以它是在main函数执行结束的时候才调用的。

所以输出结果
```
	first func run
	second func run
```
`defer`用途最多的在于释放各种资源。比如我们读取一个文件，读完之后需要释放文件句柄。
```go
	package main

	import (
		"bufio"
		"fmt"
		"os"
		"strings"
	)

	func main() {
		fname := "D:\\Temp\\test.txt"
		f, err := os.Open(fname)
		defer f.Close()
		if err != nil {
			os.Exit(1)
		}
		bReader := bufio.NewReader(f)
		for {
			line, ok := bReader.ReadString('\n')
			if ok != nil {
				break
			}
			fmt.Println(strings.Trim(line, "\r\n"))
		}
	}
```

在上面的例子中，我们按行读取文件，并且输出。从代码中，我们可以看到在使用os包中的Open方法打开文件后，立马跟着一个defer语句用来关闭文件句柄。这样就保证了该文件句柄在main函数运行结束的时候或者异常终止的时候一定能够被释放。而且由于紧跟着Open语句，一旦养成了习惯，就不会忘记去关闭文件句柄了。


*panic* & *recover*

>当你周末走在林荫道上，听着小歌，哼着小曲，很是惬意。突然之间，从天而降瓢泼大雨，你顿时慌张（panic）起来，没有带伞啊，淋着雨感冒就不好了。于是你四下张望，忽然发现自己离地铁站很近，那里有很多卖伞的，心中顿时又安定了下来（recover），于是你飞奔过去买了一把伞（defer）。

好了，panic和recover是Go语言提供的用以处理异常的关键字。`panic用来触发异常`，而`recover用来终止异常并且返回传递给panic的值`。（注意`recover并不能处理异常`，而且`recover只能在defer里面使用，否则无效`。）

先瞧个小例子
```go
	package main

	import (
		"fmt"
	)

	func main() {
		fmt.Println("I am walking and singing...")
		panic("It starts to rain cats and dogs")
		msg := recover()
		fmt.Println(msg)
	}
```
看看输出结果

```
	runtime.panic(0x48d380, 0xc084003210)
        C:/Users/ADMINI~1/AppData/Local/Temp/2/bindist667667715/go/src/pkg/runtime/panic.c:266 	+0xc8
	main.main()
        D:/JemyGraw/Creation/Go/freebook_go/func_d1.go:9 +0xea
	exit status 2
```
咦？怎么没有输出recover获取的错误信息呢？

这是因为在运行到panic语句的时候，程序已经异常终止了，后面的代码就不运行了。

那么如何才能阻止程序异常终止呢？这个时候要使用defer。因为`defer一定是在函数执行结束的时候运行的。不管是正常结束还是异常终止`。

修改一下代码
```go
	package main

	import (
		"fmt"
	)

	func main() {
		defer func() {
			msg := recover()
			fmt.Println(msg)
		}()
		fmt.Println("I am walking and singing...")
		panic("It starts to rain cats and dogs")
	}
```

好了，看下输出
```go
	I am walking and singing...
	It starts to rain cats and dogs
```
小结：

panic触发的异常通常是运行时错误。比如试图访问的索引超出了数组边界，忘记初始化字典或者任何无法轻易恢复到正常执行的错误。

# Go指针
不要害怕，Go的指针是好指针。

## 定义

所谓`指针其实你可以把它想像成一个箭头，这个箭头指向（存储）一个变量的地址`。

因为这个箭头本身也需要变量来存储，所以也叫做指针变量。

Go的指针`不支持那些乱七八糟的指针移位`。`它就表示一个变量的地址`。看看这个例子：
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x int
		var x_ptr *int

		x = 10
		x_ptr = &x

		fmt.Println(x)
		fmt.Println(x_ptr)
		fmt.Println(*x_ptr)
	}
```

上面例子输出`x的值`，`x的地址`和`通过指针变量输出x的值`，而`x_ptr就是一个指针变量`。
```
	10
	0xc084000038
	10
```
认真理清楚这两个符号的意思。

**&** `取一个变量的地址`

**\*** `取一个指针变量所指向的地址的值`


考你一下，上面的例子中，如何输出x_ptr的地址呢？
```go
	package main

	import (
		"fmt"
	)

	func main() {
		var x int
		var x_ptr *int

		x = 10
		x_ptr = &x

		fmt.Println(&x_ptr)
	}
```
此例看懂，指针就懂了。

永远记住一句话，`所谓指针就是一个指向（存储）特定变量地址的变量`。没有其他的特别之处。

再变态一下，看看这个：
```
	package main

	import (
		"fmt"
	)

	func main() {
		var x int
		var x_ptr *int

		x = 10
		x_ptr = &x

		fmt.Println(*&x_ptr)
	}
```
1. x_ptr 是一个`指针变量`，它`指向(存储)x的地址`；
2. &x_ptr 是`取这个指针变量x_ptr的地址`，这里可以设想`有另一个指针变量x_ptr_ptr(指向)存储`这个`x_ptr指针的地址`；
3. *&x_ptr 等价于`*x_ptr_ptr`就是`取这个x_ptr_ptr指针变量`所`指向(存储)`的`地址所对应的变量的值` ，也就是`x_ptr的值`，也就是`指针变量x_ptr指向(存储)的地址`，也就是`x的地址`。 这里可以看到，其实`*&`这两个运算符在一起就相互抵消作用了。

## 用途

`指针的一大用途就是可以将变量的指针作为实参传递给函数，从而在函数内部能够直接修改实参所指向的变量值。`

Go的变量传递都是值传递。
```go
	package main

	import (
		"fmt"
	)

	func change(x int) {
		x = 200
	}
	func main() {
		var x int = 100
		fmt.Println(x)
		change(x)
		fmt.Println(x)
	}
```

上面的例子输出结果为
```
	100
	100
```
很显然，change函数`改变的`仅仅是`内部变量x`的`值`，而`不会改变`传递进去的`实参`。其实，也就是说Go的函数一般关心的是输出结果，而输入参数就相当于信使跑到函数门口大叫，你们这个参数是什么值，那个是什么值，然后就跑了。你函数根本就不能修改它的值。不过如果是传递的实参是指针变量，那么函数一看，小子这次你地址我都知道了，哪里跑。那么就是下面的例子：
```go
	package main

	import (
		"fmt"
	)

	func change(x *int) {
		*x = 200
	}
	func main() {
		var x int = 100
		fmt.Println(x)
		change(&x)
		fmt.Println(x)
	}
```

上面的例子中，change函数的虚参为`整型指针变量`，所以在main中调用的时候`传递的是x的地址`。然后在change里面使用`*x=200`修改了这个x的地址的值。所以`x的值就变了`。这个输出是：
```
	100
	200
```

## new

new这个函数挺神奇，因为它的用处太多了。这里还可以通过new来`初始化一个指针`。上面说过指针指向(存储)的是一个变量的地址，但是指针本身也需要地址存储。先看个例子：
```go
	package main

	import (
		"fmt"
	)

	func set_value(x_ptr *int) {
		*x_ptr = 100
	}
	func main() {
		x_ptr := new(int)
		set_value(x_ptr)
		//x_ptr指向的地址
		fmt.Println(x_ptr)
		//x_ptr本身的地址
		fmt.Println(&x_ptr)
		//x_ptr指向的地址值
		fmt.Println(*x_ptr)
	}
```

上面我们定义了一个x_ptr变量，然后用`new申请`了一个`存储整型数据的内存地址`，然后将这个`地址赋值`给`x_ptr指针变量`，也就是说`x_ptr指向（存储）的是一个可以存储整型数据的地址`，然后用set_value函数将`这个地址中存储的值`赋值为100。所以第一个输出是`x_ptr指向的地址`，第二个则是`x_ptr本身的地址`，而`*x_ptr`则是`x_ptr指向的地址中存储的整型数据的值`。

```
	0xc084000040
	0xc084000038
	100
```
## 小结

好了，现在用个例子再来回顾一下指针。

交换两个变量的值。
```go
	package main

	import (
		"fmt"
	)

	func swap(x, y *int) {
		*x, *y = *y, *x
	}
	func main() {
		x_val := 100
		y_val := 200
		swap(&x_val, &y_val)
		fmt.Println(x_val)
		fmt.Println(y_val)
	}
```

很简单吧，这里利用了Go提供的`交叉赋值`的功能，另外由于是使用了指针作为参数，所以在swap函数内，x_val和y_val的值就被交换了。

# Go结构体和指针

基本上到这里的时候，就是上了一个台阶了。Go的精华特点即将展开。

## 结构体定义

上面我们说过Go的指针和C的不同，结构体也是一样的。Go是一门删繁就简的语言，一切令人困惑的特性都必须去掉。

简单来讲，Go提供的`结构体`就是把`使用各种数据类型定义`的`不同变量组合起来`的`高级数据类型`。闲话不多说，看例子:
```go
	type Rect struct {
		width float64
		length float64
	}
```
上面我们定义了一个矩形结构体，首先是关键是`type`表示要`定义一个新的数据类型了`，然后是新的数据类型名称`Rect`，最后是`struct`关键字，表示这个高级数据类型是结构体类型。在上面的例子中，因为`width和length的数据类型相同`，还可以写成如下格式：

```go
	type Rect struct {
		width, length float64
	}
```

好了，来用结构体干点啥吧，计算一下矩形面积。
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func main() {
		var rect Rect
		rect.width = 100
		rect.length = 200
		fmt.Println(rect.width * rect.length)
	}
```
从上面的例子看到，其实结构体类型和基础数据类型使用方式差不多，唯一的区别就是结构体类型可以通过`.`来访问内部的成员。包括`给内部成员赋值`和`读取内部成员值`。

在上面的例子中，我们是用var关键字先定义了一个Rect变量，然后对它的成员赋值。我们也可以使用初始化的方式来给Rect变量的内部成员赋值。
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func main() {
		var rect = Rect{width: 100, length: 200}

		fmt.Println(rect.width * rect.length)
	}
```

当然`如果你知道结构体成员定义的顺序`，也可以不使用`key:value`的方式赋值，`直接按照结构体成员定义的顺序给它们赋值`。
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func main() {
		var rect = Rect{100, 200}

		fmt.Println("Width:", rect.width, "* Length:",
			rect.length, "= Area:", rect.width*rect.length)
	}
```
输出结果为

	Width: 100 * Length: 200 = Area: 20000

## 结构体参数传递方式

我们说过，`Go函数的参数传递方式是值传递`，这句话`对结构体也是适用的`。

```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func double_area(rect Rect) float64 {
		rect.width *= 2
		rect.length *= 2
		return rect.width * rect.length
	}
	func main() {
		var rect = Rect{100, 200}
		fmt.Println(double_area(rect))
		fmt.Println("Width:", rect.width, "Length:", rect.length)
	}
```
上面的例子输出为:
```
	80000
	Width: 100 Length: 200
```
也就说虽然在double_area函数里面我们将结构体的宽度和长度都加倍，但仍然没有影响main函数里面的rect变量的宽度和长度。


## 结构体组合函数

上面我们在main函数中计算了矩形的面积，但是我们觉得矩形的面积如果能够作为矩形结构体的“内部函数”提供会更好。这样我们就可以直接说这个矩形面积是多少，而不用另外去取宽度和长度去计算。现在我们看看结构体“内部函数”定义方法：

```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func (rect Rect) area() float64 {
		return rect.width * rect.length
	}

	func main() {
		var rect = Rect{100, 200}

		fmt.Println("Width:", rect.width, "Length:", rect.length,
			"Area:", rect.area())
	}
```
咦？这个是什么“内部方法”，根本没有定义在Rect数据类型的内部啊？

确实如此，我们看到，虽然main函数中的rect变量可以直接调用函数area()来获取矩形面积，但是area()函数确实没有定义在Rect结构体内部，这点和C语言的有很大不同。`Go使用组合函数的方式来为结构体定义结构体方法`。我们仔细看一下上面的area()函数定义。

首先是关键字`func`表示这是一个函数，第二个参数是`结构体类型和实例变量`，第三个是`函数名称`，第四个是`函数返回值`。这里我们可以看出area()函数和普通函数定义的`区别就在于`area()函数`多了一个结构体类型限定`。这样一来Go就知道了这是一个为结构体定义的`方法`。

这里需要注意一点就是`定义在结构体上面的函数(function)`一般叫做`方法(method)`。

## 结构体和指针

我们在指针一节讲到过，`指针的主要作用就是在函数内部改变传递进来变量的值`。对于上面的计算矩形面积的例子，我们可以修改一下代码如下：
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func (rect *Rect) area() float64 {
		return rect.width * rect.length
	}

	func main() {
		var rect = new(Rect)
		rect.width = 100
		rect.length = 200
		fmt.Println("Width:", rect.width, "Length:", rect.length,
			"Area:", rect.area())
	}
```

上面的例子中，使用了new函数来创建一个结构体指针rect，也就是说rect的类型是\*Rect，结构体遇到指针的时候，你`不需要使用*去访问结构体的成员`，直接使用`.`引用就可以了。所以上面的例子中我们直接使用`rect.width=100` 和`rect.length=200`来设置结构体成员值。因为这个时候rect是结构体指针，所以我们定义area()函数的时候结构体限定类型为`*Rect`。

其实在计算面积的这个例子中，我们不需要改变矩形的宽或者长度，所以定义area函数的时候结构体限定类型仍然为`Rect`也是可以的。如下：
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func (rect Rect) area() float64 {
		return rect.width * rect.length
	}

	func main() {
		var rect = new(Rect)
		rect.width = 100
		rect.length = 200
		fmt.Println("Width:", rect.width, "Length:", rect.length,
			"Area:", rect.area())
	}
```
这里Go足够聪明，所以rect.area()也是可以的。

至于`使不使用结构体指针和使不使用指针的出发点是一样的`，那就是`你是否试图在函数内部改变传递进来的参数的值`。再举个例子如下：
```go
	package main

	import (
		"fmt"
	)

	type Rect struct {
		width, length float64
	}

	func (rect *Rect) double_area() float64 {
		rect.width *= 2
		rect.length *= 2
		return rect.width * rect.length
	}

	func main() {
		var rect = new(Rect)
		rect.width = 100
		rect.length = 200
		fmt.Println(*rect)
		fmt.Println("Double Width:", rect.width, "Double Length:", rect.length,
			"Double Area:", rect.double_area())
		fmt.Println(*rect)
	}
```
这个例子的输出是：
```
	{100 200}
	Double Width: 200 Double Length: 400 Double Area: 80000
	{200 400}
```

## 结构体内嵌类型

我们可以在一个`结构体内部定义另外一个结构体类型的成员`。例如iPhone也是Phone，我们看下例子：
```go
	package main

	import (
		"fmt"
	)

	type Phone struct {
		price int
		color string
	}

	type IPhone struct {
		phone Phone
		model string
	}

	func main() {
		var p IPhone
		p.phone.price = 5000
		p.phone.color = "Black"
		p.model = "iPhone 5"
		fmt.Println("I have a iPhone:")
		fmt.Println("Price:", p.phone.price)
		fmt.Println("Color:", p.phone.color)
		fmt.Println("Model:", p.model)
	}
```
输出结果为
```
	I have a iPhone:
	Price: 5000
	Color: Black
	Model: iPhone 5
```
在上面的例子中，我们在结构体IPhone里面定义了一个Phone变量phone，然后我们可以像正常的访问结构体成员一样访问phone的成员数据。但是我们原来的意思是`“iPhone也是(is-a)Phone”`，而这里的结构体IPhone里面定义了一个phone变量，给人的感觉就是`“iPhone有一个(has-a)Phone”`，挺奇怪的。当然Go也知道这种方式很奇怪，所以支持如下做法：

```go
	package main

	import (
		"fmt"
	)

	type Phone struct {
		price int
		color string
	}

	type IPhone struct {
		Phone
		model string
	}

	func main() {
		var p IPhone
		p.price = 5000
		p.color = "Black"
		p.model = "iPhone 5"
		fmt.Println("I have a iPhone:")
		fmt.Println("Price:", p.price)
		fmt.Println("Color:", p.color)
		fmt.Println("Model:", p.model)
	}
```
输出结果为
```
	I have a iPhone:
	Price: 5000
	Color: Black
	Model: iPhone 5
```
在这个例子中，我们定义IPhone结构体的时候，`不再定义Phone变量`，`直接把结构体Phone类型定义在那里`。然后IPhone就可以`像访问直接定义在自己结构体里面的成员一样访问Phone的成员`。

上面的例子中，我们演示了结构体的内嵌类型以及内嵌类型的成员访问，除此之外，假设结构体A内部定义了一个内嵌结构体B，那么A同时也可以调用所有定义在B上面的函数。

```go
	package main

	import (
		"fmt"
	)

	type Phone struct {
		price int
		color string
	}

	func (phone Phone) ringing() {
		fmt.Println("Phone is ringing...")
	}

	type IPhone struct {
		Phone
		model string
	}

	func main() {
		var p IPhone
		p.price = 5000
		p.color = "Black"
		p.model = "iPhone 5"
		fmt.Println("I have a iPhone:")
		fmt.Println("Price:", p.price)
		fmt.Println("Color:", p.color)
		fmt.Println("Model:", p.model)

		p.ringing()
	}
```
输出结果为：
```
	I have a iPhone:
	Price: 5000
	Color: Black
	Model: iPhone 5
	Phone is ringing...
```

## 接口

我们先看一个例子，关于Nokia手机和iPhone手机都能够打电话的例子。
```go
	package main

	import (
		"fmt"
	)

	type NokiaPhone struct {
	}

	func (nokiaPhone NokiaPhone) call() {
		fmt.Println("I am Nokia, I can call you!")
	}

	type IPhone struct {
	}

	func (iPhone IPhone) call() {
		fmt.Println("I am iPhone, I can call you!")
	}
	func main() {
		var nokia NokiaPhone
		nokia.call()

		var iPhone IPhone
		iPhone.call()
	}
```
我们定义了NokiaPhone和IPhone，它们都有各自的方法call()，表示自己都能够打电话。但是我们想一想，是手机都应该能够打电话，所以这个不算是NokiaPhone或是IPhone的独特特点。否则iPhone不可能卖这么贵了。

再仔细看一下`接口的定义`，首先是关键字`type`，然后是`接口名称`，最后是关键字`interface`表示这个类型是接口类型。`在接口类型里面，我们定义了一组方法`。

Go语言提供了一种接口功能，它把所有的具有共性的方法定义在一起，`任何其他类型只要实现了这些方法就是实现了这个接口`，`不一定非要显式地声明`要去实现哪些接口啦。比如上面的手机的call()方法，就完全可以定义在接口Phone里面，而NokiaPhone和IPhone只要实现了这个接口就是一个Phone。
```go
	package main

	import (
		"fmt"
	)

	type Phone interface {
		call()
	}

	type NokiaPhone struct {
	}

	func (nokiaPhone NokiaPhone) call() {
		fmt.Println("I am Nokia, I can call you!")
	}

	type IPhone struct {
	}

	func (iPhone IPhone) call() {
		fmt.Println("I am iPhone, I can call you!")
	}

	func main() {
		var phone Phone

		phone = new(NokiaPhone)
		phone.call()

		phone = new(IPhone)
		phone.call()

	}
```
在上面的例子中，我们定义了一个接口Phone，接口里面有一个方法call()，仅此而已。然后我们在main函数里面定义了一个Phone类型变量，并分别为之赋值为NokiaPhone和IPhone。然后调用call()方法，输出结果如下：
```
	I am Nokia, I can call you!
	I am iPhone, I can call you!
```
以前我们说过，`Go语言式静态类型语言，变量的类型在运行过程中不能改变`。但是在上面的例子中，phone变量好像先定义为Phone类型，然后是NokiaPhone类型，最后成为了IPhone类型，真的是这样吗？

原来，在Go语言里面，`一个类型A只要实现了接口X所定义的全部方法`，那么`A类型的变量`也是`X类型的变量`。在上面的例子中，NokiaPhone和IPhone都实现了Phone接口的call()方法，所以它们都是Phone，这样一来是不是感觉正常了一些。

我们为Phone添加一个方法sales()，再来熟悉一下接口用法。
```go
	package main

	import (
		"fmt"
	)

	type Phone interface {
		call()
		sales() int
	}

	type NokiaPhone struct {
		price int
	}

	func (nokiaPhone NokiaPhone) call() {
		fmt.Println("I am Nokia, I can call you!")
	}
	func (nokiaPhone NokiaPhone) sales() int {
		return nokiaPhone.price
	}

	type IPhone struct {
		price int
	}

	func (iPhone IPhone) call() {
		fmt.Println("I am iPhone, I can call you!")
	}

	func (iPhone IPhone) sales() int {
		return iPhone.price
	}

	func main() {
		var phones = [5]Phone{
			NokiaPhone{price: 350},
			IPhone{price: 5000},
			IPhone{price: 3400},
			NokiaPhone{price: 450},
			IPhone{price: 5000},
		}

		var totalSales = 0
		for _, phone := range phones {
			totalSales += phone.sales()
		}
		fmt.Println(totalSales)

	}
```
输出结果：

	14200

上面的例子中，我们定义了一个手机数组，然后计算手机的总售价。可以看到，由于NokiaPhone和IPhone都实现了sales()方法，所以它们都是Phone类型，但是计算售价的时候，Go会知道调用哪个对象实现的方法。

接口类型还可以作为结构体的数据成员。

假设有个败家子，iPhone没有出的时候，买了好几款Nokia，iPhone出来后，又买了好多部iPhone，老爸要来看看这小子一共花了多少钱。
```go
	package main

	import (
		"fmt"
	)

	type Phone interface {
		sales() int
	}

	type NokiaPhone struct {
		price int
	}

	func (nokiaPhone NokiaPhone) sales() int {
		return nokiaPhone.price
	}

	type IPhone struct {
		price int
	}

	func (iPhone IPhone) sales() int {
		return iPhone.price
	}

	type Person struct {
		phones []Phone
		name   string
		age    int
	}

	func (person Person) total_cost() int {
		var sum = 0
		for _, phone := range person.phones {
			sum += phone.sales()
		}
		return sum
	}

	func main() {
		var bought_phones = [5]Phone{
			NokiaPhone{price: 350},
			IPhone{price: 5000},
			IPhone{price: 3400},
			NokiaPhone{price: 450},
			IPhone{price: 5000},
		}

		var person = Person{name: "Jemy", age: 25, phones: bought_phones[:]}

		fmt.Println(person.name)
		fmt.Println(person.age)
		fmt.Println(person.total_cost())
	}
```
这个例子纯为演示接口作为结构体数据成员，如有雷同，纯属巧合。这里面我们定义了一个Person结构体，结构体内部定义了一个手机类型切片。另外我们定义了Person的total_cost()方法用来计算手机花费总额。输出结果如下：
```
	Jemy
	25
	14200
```
## 小结

Go的结构体和接口的实现方法可谓删繁就简，去除了很多别的语言令人困惑的地方，而且学习难度也不大，很容易上手。不过由于思想比较独到，也有可能会有人觉得功能太简单而无用，这个就各有看法了，不过在逐渐的使用过程中，我们会慢慢领悟到这种设计所带来的好处，以及所避免的问题。