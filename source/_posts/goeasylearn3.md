---
title: go语言基础--并行计算、包、测试
date: 2017-07-06 11:40:38
tags: go语言基础
categories: go语言学习笔记
---
# Go并行计算

如果说Go有什么让人一见钟情的特性，那大概就是并行计算了吧。

做个题目

>如果我们列出10以下所有能够被3或者5整除的自然数，那么我们得到的是3，5，6和9。这四个数的和是23。
那么请计算1000以下（不包括1000）的所有能够被3或者5整除的自然数的和。

<!--more-->
这个题目的一个思路就是：

(1) 先计算1000以下所有能够被3整除的整数的和A，  
(2) 然后计算1000以下所有能够被5整除的整数和B，
(3) 然后再计算1000以下所有能够被3和5整除的整数和C， 
(4) 使用A+B-C就得到了最后的结果。  

按照上面的方法，传统的方法当然就是一步一步计算，然后再到第(4)步汇总了。

但是一旦有了Go，我们就可以让前面三个步骤并行计算，然后再在第(4)步汇总。

并行计算涉及到一个新的`数据类型chan`和一个新的`关键字go`。

先看例子：

``` go
package main

import (
	"fmt"
	"time"
)

func get_sum_of_divisible(num int, divider int, resultChan chan int) {
	sum := 0
	for value := 0; value < num; value++ {
		if value%divider == 0 {
			sum += value
		}
	}
	resultChan <- sum
}

func main() {
	LIMIT := 10
	resultChan := make(chan int, 3)
	t_start := time.Now()
	go get_sum_of_divisible(LIMIT, 3, resultChan)
	go get_sum_of_divisible(LIMIT, 5, resultChan)
	//这里其实那个是被3整除，哪个是被5整除看具体调度方法，不过由于是求和，所以没关系
	sum3, sum5 := <-resultChan, <-resultChan

	//单独算被15整除的
	go get_sum_of_divisible(LIMIT, 15, resultChan)
	sum15 := <-resultChan

	sum := sum3 + sum5 - sum15
	t_end := time.Now()
	fmt.Println(sum)
	fmt.Println(t_end.Sub(t_start))
}
```

(1) 在上面的例子中，我们首先定义了一个普通的函数get_sum_of_divisible，这个函数的`最后一个参数是一个整型chan类型`，这种类型，你可以把它当作一个先进先出的队列。你可以`向它写入数据`，也可以`从它读出数据`。它`所能接受的数据类型`就是`由chan关键字后面的类型所决定`的。在上面的例子中，我们使用`<-`运算符将函数计算的结果写入channel。channel是go提供的用来协程之间通信的方式。本例中main是一个协程，三个get_sum_of_divisible调用是协程。要在这四个协程间通信，必须有一种可靠的手段。

(2) 在main函数中，我们使用go关键字来开启并行计算。并行计算是由goroutine来支持的，`goroutine`又叫做`协程`，你可以把它看作为比线程更轻量级的运算。开启一个协程很简单，就是`go关键字`后面`跟上所要运行的函数`。

(3) 最后，我们要从channel中取出并行计算的结果。使用`<-`运算符从channel里面取出数据。

在本例中，我们为了演示go并行计算的速度，还引进了time包来计算程序执行时间。在同普通的顺序计算相比，并行计算的速度是非同凡响的。

好了，上面的例子看完，我们来详细讲解Go的并行计算。

## Go Routine 协程

所谓协程，就是Go提供的轻量级的独立运算过程，比线程还轻。创建一个协程很简单，就是go关键字加上所要运行的函数。看个例子：
```go
	package main

	import (
		"fmt"
	)

	func list_elem(n int) {
		for i := 0; i < n; i++ {
			fmt.Println(i)
		}
	}
	func main() {
		go list_elem(10)
	}
```
上面的例子是创建一个协程遍历一下元素。但是当你运行的时候，你会`发现什么都没有输出`！`为什么呢？`
因为上面的`main函数`在`创建完协程后`就`立刻退出`了，所以`协程`还`没有来得及运行`呢！修改一下：
```go
	package main

	import (
		"fmt"
	)

	func list_elem(n int) {
		for i := 0; i < n; i++ {
			fmt.Println(i)
		}
	}
	func main() {
		go list_elem(10)
		var input string
		fmt.Scanln(&input)
	}
```
这里，我们在main函数创建协程后，要求用户输入任何数据后才退出，这样协程就有了运行的时间，故而输出结果：
```
	0
	1
	2
	3
	4
	5
	6
	7
	8
	9
```
其实在开头的例子里面，我们的main函数事实上也被阻塞了，因为`sum3, sum5, sum15 := <-resultChan, <-resultChan, <-resultChan`这行代码在channel里面没有数据或者数据个数不符的时候，都会阻塞在那里，直到协程结束，写入结果。

不过既然是并行计算，我们还是得看看协程是否真的并行计算了。
```go
	package main

	import (
		"fmt"
		"math/rand"
		"time"
	)

	func list_elem(n int, tag string) {
		for i := 0; i < n; i++ {
			fmt.Println(tag, i)
			tick := time.Duration(rand.Intn(100))
			time.Sleep(time.Millisecond * tick)
		}
	}
	func main() {
		go list_elem(10, "go_a")
		go list_elem(20, "go_b")
		var input string
		fmt.Scanln(&input)
	}
```
输出结果
```
	go_a 0
	go_b 0
	go_a 1
	go_b 1
	go_a 2
	go_b 2
	go_b 3
	go_b 4
	go_a 3
	go_b 5
	go_b 6
	go_a 4
	go_a 5
	go_b 7
	go_a 6
	go_a 7
	go_b 8
	go_b 9
	go_a 8
	go_b 10
	go_b 11
	go_a 9
	go_b 12
	go_b 13
	go_b 14
	go_b 15
	go_b 16
	go_b 17
	go_b 18
	go_b 19
```
在上面的例子中，我们让两个协程在每输出一个数字的时候，随机Sleep了一会儿。如果是并行计算，那么输出是无序的。从上面的例子中，我们可以看出两个协程确实并行运行了。


## Channel通道

Channel提供了`协程之间`的`通信方式`以及`运行同步机制`。

假设训练定点投篮和三分投篮，教练在计数。
```go
	package main

	import (
		"fmt"
		"time"
	)

	func fixed_shooting(msg_chan chan string) {
		for {
			msg_chan <- "fixed shooting"
			fmt.Println("continue fixed shooting...")
		}
	}

	func count(msg_chan chan string) {
		for {
			msg := <-msg_chan
			fmt.Println(msg)
			time.Sleep(time.Second * 1)
		}
	}

	func main() {
		var c chan string
		c = make(chan string)

		go fixed_shooting(c)
		go count(c)

		var input string
		fmt.Scanln(&input)
	}
```
输出结果为：
```
	fixed shooting
	continue fixed shooting...
	fixed shooting
	continue fixed shooting...
	fixed shooting
	continue fixed shooting...
```
我们看到在fixed_shooting函数里面我们将消息传递到channel，然后输出提示信息"continue fixed shooting..."，而在count函数里面，我们从channel里面取出消息输出，然后间隔1秒再去取消息输出。这里面我们可以考虑一下，如果我们不去从channel中取消息会出现什么情况？我们把main函数里面的`go count(c)`注释掉，然后再运行一下。发现程序再也不会输出消息和提示信息了。这是因为channel中根本就没有信息了，因为`如果你要向channel里面写信息`，`必须有配对的取信息的一端`，否则是不会写的。

我们再把三分投篮加上。
```go
	package main

	import (
		"fmt"
		"time"
	)

	func fixed_shooting(msg_chan chan string) {
		for {
			msg_chan <- "fixed shooting"
		}
	}

	func three_point_shooting(msg_chan chan string) {
		for {
			msg_chan <- "three point shooting"
		}
	}

	func count(msg_chan chan string) {
		for {
			msg := <-msg_chan
			fmt.Println(msg)
			time.Sleep(time.Second * 1)
		}
	}

	func main() {
		var c chan string
		c = make(chan string)

		go fixed_shooting(c)
		go three_point_shooting(c)
		go count(c)

		var input string
		fmt.Scanln(&input)
	}
```
输出结果为：
```
	fixed shooting
	three point shooting
	fixed shooting
	three point shooting
	fixed shooting
	three point shooting
```
我们看到程序交替输出定点投篮和三分投篮，这是因为写入channel的信息必须要读取出来，否则尝试再次写入就失败了。

在上面的例子中，我们发现`定义一个channel信息变量`的方式就是多加一个`chan`关键字。并且你能够`向channel写入数据`和`从channel读取数据`。这里我们还可以设置channel通道的方向。

## Channel通道方向

所谓的`通道方向`就是`写`和`读`。如果我们如下定义

	c chan<- string //那么你只能向channel写入数据
	
而这种定义

	c <-chan string //那么你只能从channel读取数据
	
`试图向只读chan变量写入数据或者试图从只写chan变量读取数据都会导致编译错误。`

如果是默认的定义方式

	c chan string //那么你既可以向channel写入数据也可以从channnel读取数据
	
	
## 多通道(Select)

如果上面的投篮训练现在有两个教练了，各自负责一个训练项目。而且还在不同的篮球场，这个时候很显然，我们一个channel就不够用了。修改一下：
```go
	package main

	import (
		"fmt"
		"time"
	)

	func fixed_shooting(msg_chan chan string) {
		for {
			msg_chan <- "fixed shooting"
			time.Sleep(time.Second * 1)
		}
	}

	func three_point_shooting(msg_chan chan string) {
		for {
			msg_chan <- "three point shooting"
			time.Sleep(time.Second * 1)
		}
	}

	func main() {
		c_fixed := make(chan string)
		c_3_point := make(chan string)

		go fixed_shooting(c_fixed)
		go three_point_shooting(c_3_point)

		go func() {
			for {
				select {
				case msg1 := <-c_fixed:
					fmt.Println(msg1)
				case msg2 := <-c_3_point:
					fmt.Println(msg2)
				}
			}

		}()

		var input string
		fmt.Scanln(&input)
	}
```
其他的和上面的一样，唯一不同的是我们将定点投篮和三分投篮的消息写入了不同的channel，那么main函数如何知道从哪个channel读取消息呢？使用select方法，select方法依次检查每个channel是否有消息传递过来，如果有就取出来输出。如果同时有多个消息到达，那么select闭上眼睛随机选一个channel来从中读取消息，如果没有一个channel有消息到达，那么select语句就阻塞在这里一直等待。

在某些情况下，比如学生投篮中受伤了，那么就轮到医护人员上场了，教练在一般看看，如果是重伤，教练就不等了，就回去了休息了，待会儿再过来看看情况。我们可以给select加上一个case用来判断是否等待各个消息到达超时。
```go
	package main

	import (
		"fmt"
		"time"
	)

	func fixed_shooting(msg_chan chan string) {
		var times = 3
		var t = 1
		for {
			if t <= times {
				msg_chan <- "fixed shooting"
			}
			t++
			time.Sleep(time.Second * 1)
		}
	}

	func three_point_shooting(msg_chan chan string) {
		var times = 5
		var t = 1
		for {
			if t <= times {
				msg_chan <- "three point shooting"
			}
			t++
			time.Sleep(time.Second * 1)
		}
	}

	func main() {
		c_fixed := make(chan string)
		c_3_point := make(chan string)

		go fixed_shooting(c_fixed)
		go three_point_shooting(c_3_point)

		go func() {
			for {
				select {
				case msg1 := <-c_fixed:
					fmt.Println(msg1)
				case msg2 := <-c_3_point:
					fmt.Println(msg2)
				case <-time.After(time.Second * 5):
					fmt.Println("timeout, check again...")
				}
			}

		}()

		var input string
		fmt.Scanln(&input)
	}
```
在上面的例子中，我们让投篮的人在几次过后挂掉，然后教练就每次等5秒出来看看情况（累死丫的，:-P），因为我们对等待的时间不感兴趣就不用变量存储了，直接`<-time.After(time.Second*5)`，或许你会奇怪，为什么各个channel消息都没有到达，select为什么不阻塞？就是因为这个time.After，虽然它没有显式地告诉你这是一个channel消息，但是记得么？main函数也是一个channel啊！哈哈！至于time.After的功能实际上让main阻塞了5秒后返回给main的channel一个时间。所以我们在case里面把这个时间消息读出来，select就不阻塞了。

输出结果如下：
```
	fixed shooting
	three point shooting
	fixed shooting
	three point shooting
	fixed shooting
	three point shooting
	three point shooting
	three point shooting
	timeout, check again...
	timeout, check again...
	timeout, check again...
	timeout, check again...
```
这里select还有一个`default的选项`，如果你指定了default选项，那么当select发现`没有消息到达`的时候`也不会阻塞`，直接开始转回去再次判断。

## Channel Buffer通道缓冲区

我们定义chan变量的时候，还可以指定它的`缓冲区大小`。一般我们`定义的channel都是同步的`，也就是说接受端和发送端彼此等待对方ok才开始。但是如果你给一个channel`指定了一个缓冲区`，那么`消息的发送和接受式异步的`，`除非channel缓冲区已经满了`。

	c:=make(chan int, 1)
	
我们看个例子：
```go
	package main

	import (
		"fmt"
		"strconv"
		"time"
	)

	func shooting(msg_chan chan string) {
		var group = 1
		for {
			for i := 1; i <= 10; i++ {
				msg_chan <- strconv.Itoa(group) + ":" + strconv.Itoa(i)
			}
			group++
			time.Sleep(time.Second * 10)
		}
	}

	func count(msg_chan chan string) {
		for {
			fmt.Println(<-msg_chan)
		}
	}

	func main() {
		var c = make(chan string, 20)
		go shooting(c)
		go count(c)

		var input string
		fmt.Scanln(&input)
	}
```
输出结果为：
```
	1:1
	1:2
	1:3
	1:4
	1:5
	1:6
	1:7
	1:8
	1:9
	1:10
	2:1
	2:2
	2:3
	2:4
	2:5
	2:6
	2:7
	2:8
	2:9
	2:10
	3:1
	3:2
	3:3
	3:4
	3:5
	3:6
	3:7
	3:8
	3:9
	3:10
	4:1
	4:2
	4:3
	4:4
	4:5
	4:6
	4:7
	4:8
	4:9
	4:10
```
你可以尝试运行一下，每次都是一下子输出10个数据。然后等待10秒再输出一批。


## 小结

并行计算这种特点最适合用来开发网站服务器，因为一般网站服务都是高并发的，逻辑十分复杂。而使用Go的这种特性恰是提供了一种极好的方法。

# 使用包和测试管理项目

Go天生就是为了支持良好的项目管理体验而设计的。

## 包

在软件工程的实践中，我们会遇到很多功能重复的代码，比如去除字符串首尾的空格。高质量软件产品的特点就是它的部分代码是可以重用的，比如你不必每次写个函数去去除字符串首尾的空格。

我们上面讲过变量，结构体，接口和函数等，事实上所谓的包，就是把一些用的多的这些变量，结构体，接口和函数等统一放置在一个逻辑块中。并且给它们起一个名字，这个名字就叫做包名。

例如我们上面用的最多的fmt包，这个包提供了很多格式化输出的函数，你可以在自己的代码中引用这个包，来做格式化输出，而不用你自己每次去写个函数。一门成熟的语言都会提供齐全的基础功能包供人调用。

使用包有三个好处

1. 可以减少函数名称重复，因为不同包中可以存在名称相同的函数。否则得话，你得给这些函数加上前缀或者后缀以示区别。
2. 包把函数等组织在一起，方便你查找和重用。比如你想用Println()函数输出一行字符串，你可以很方便地知道它在fmt包中，直接引用过来用就可以了。
3. 使用包可以加速程序编译。因为包是预编译好的，你改动自己代码得时候，不必每次去把包编译一下。

## 创建包

我们现在来举个例子，用来演示Go的项目管理。

首先我们在目录`/Users/jemy/JemyGraw/GoLang`下面创建文件夹`pkg_demo`。然后在`pkg_demo`里面创建`src`文件夹
。然后再在`main`文件夹里面创建`main.go`文件。另外为了演示包的创建，我们在`src`目录下面创建文件夹`net.duokr`，然后再在`net.duokr`文件夹里面创建`math`文件夹，这个文件夹名称就是这个文件夹下面go文件的包名称。然后我们再创建一个`math_pkg.go`文件，之所以取这个名字而不是`math.go`只是为了说明这个文件名称和包名不需要一致。然后我们还创建了一个`math_pkg_test.go`文件作为包的测试用例文件。整体结构如下：

	.
	└── src
	    ├── main
	    │   ├── build.sh
	    │   └── main.go
	    └── net.duokr
	        └── math
	            ├── math_pkg.go
	            └── math_pkg_test.go


其中build.sh是我们为了编译这个项目而写的脚本，因为编译项目需要几条命令，把它写在脚本文件中方便使用。另外为了能够让build.sh能够执行，使用`chmod +x build.sh`为它赋予可执行权限。build.bat是Windows下面的编译脚本。
我们来看一下`math_pkg.go`的定义：
```go
	package math

	func Add(a, b int) int {
		return a + b
	}
	func Subtract(a, b int) int {
		return a - b
	}
	func Multiply(a, b int) int {
		return a * b
	}

	func Divide(a, b int) int {
		if b == 0 {
			panic("Can not divided by zero")
		}
		return a / b
	}
```
首先是包名，然后是几个函数定义，这里我们会发现这些`函数定义首字母都是大写`，`Go规定了只有首字母大写的函数才能从包导出使用，即其他调用这个包中函数的代码只能调用那些导出的函数`。

我们再看一下`main.go`的定义：
```go
	package main

	import (
		"fmt"
		math "net.duokr/math"
	)

	func main() {
		var a = 100
		var b = 200

		fmt.Println("Add demo:", math.Add(a, b))
		fmt.Println("Substract demo:", math.Subtract(a, b))
		fmt.Println("Multiply demo:", math.Multiply(a, b))
		fmt.Println("Divide demo:", math.Divide(a, b))
	}
```
在main.go里面，我们使用import关键字引用我们自定义的包math，引用的方法是从main包平行的文件夹net.duokr开始，后面跟上包名math。这里面我们给这个长长的包名起了一个别名就叫math。然后分别调用math包里面的函数。

最后我们看一下我们的编译脚本：
```sh
	export GOPATH=$GOPATH:/Users/jemy/JemyGraw/GoLang/pkg_demo
	export GOBIN=/Users/jemy/JemyGraw/GoLang/pkg_demo/bin
	go build net.duokr/math
	go build main.go
	go install main
```
第一行，我们将项目路径加入GOPATH中，这样待会儿编译main.go的时候才能找到我们自定义的包；

第二行，我们设置本项目的安装目录，第五行的命令将编译好的文件放到这个目录下面；

第三行，我们编译我们的自定义包；

第四行，我们编译我们main.go文件；

第五行，将编译好的文件安装到指定目录下。

这里还有一个Windows下面的编译脚本build.bat：
```sh
	@echo off
	set GOPATH=GOPATH;C:\JemyGraw\GoLang\pkg_demo
	set GOBIN=C:\JemyGraw\GoLang\pkg_demo\bin
	go build net.duokr\math
	go build main.go
	go install main
```
好了，运行脚本编译一下，在main文件夹和bin文件夹下面都会生成一个可执行文件。

这个时候文件夹结构为：

	.
	├── bin
	│   └── main
	├── pkg
	│   └── darwin_386
	│       └── net.duokr
	│           └── math.a
	└── src
	    ├── main
	    │   ├── build.bat
	    │   ├── build.sh
	    │   ├── main
	    │   └── main.go
	    └── net.duokr
	        └── math
	            ├── math_pkg.go
	            └── math_pkg_test.go

运行一下，输出结果为：
```
	Add demo: 300
	Substract demo: -100
	Multiply demo: 20000
	Divide demo: 0
```
好了，包的使用介绍完毕，我们再来看一下测试用例怎么写。

## 测试

在上面的例子中，我们发现我们自定义的包下面还有一个math_pkg_test.go文件，这个文件包含了本包的一些测试用例。而且Go会把以`_test.go`结尾的文件当作是测试文件。

测试怎么写，当然是用assert来判断程序的运行结果是否和预期的相同了。

我们来看看这个math包的测试用例。
```
	package math

	import (
		"testing"
	)

	func TestAdd(t *testing.T) {
		var a = 100
		var b = 200

		var val = Add(a, b)
		if val != a+b {
			t.Error("Test Case [", "TestAdd", "] Failed!")
		}
	}

	func TestSubtract(t *testing.T) {
		var a = 100
		var b = 200

		var val = Subtract(a, b)
		if val != a-b {
			t.Error("Test Case [", "TestSubtract", "] Failed!")
		}
	}

	func TestMultiply(t *testing.T) {
		var a = 100
		var b = 200

		var val = Multiply(a, b)
		if val != a*b {
			t.Error("Test Case [", "TestMultiply", "] Failed!")
		}
	}

	func TestDivideNormal(t *testing.T) {
		var a = 100
		var b = 200

		var val = Divide(a, b)
		if val != a/b {
			t.Error("Test Case [", "TestDivideNormal", "] Failed!")
		}
	}
```
将路径切换到测试文件所在目录，运行`go test`命令，go会自动测试所有的测试用例。

在上面的例子中，测试用例的特点是以函数名以`Test`开始，而且具有唯一参数`t *testing.T`。

## 小结

Go提供的包和用例测试是构建优秀的软件产品的基础，只要我们不断学习，努力去做，一定可以做的很好。
	

