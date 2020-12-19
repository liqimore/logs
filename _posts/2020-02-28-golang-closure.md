---
layout: post
title: "go语言中对闭包的理解和实例演示"
description: "go语言闭包"
date: 2020-02-28
tags: [golang闭包]
categories: [技术, go]
comments: true
---


go中的函数闭包(Function Closures)对于我来说比较难理解, 在之前的开发中也没有用到其他语言的闭包特性, 所以特意认真学习了一下. 下面我会详细解释我对go中闭包的理解和一个实例用法.

简单来说, 闭包在go中的实现方法就是在函数中嵌套另一个子函数, 如下代码片段所示(摘自官方教程):

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
```

可以看到, 创建了一个名为`adder()`的函数, 函数中有一个局部变量`sum`, 闭包的**第一个特性就是可以直接访问父函数的变量**, 所以在内部函数中直接使用了`sum`.

<!--more-->

```go
ad := adder()
fmt.Println(ad(10))
```

这样子就是对闭包特性的一次使用. 函数也是一个变量存在到内存中的, 所以可以把`ad`作为函数的reference, 在println语句中, 是`ad`第一次调用, 这里通过函数的reference传入的值实际上是传到了`adder()`中的匿名函数里, 即`x = 10`.

我在初次学习中对这里有疑惑, 不理解`ad := adder()`, 其实这句话可以根据字面意思理解, 即把`adder()`的返回值赋给`ad`, 由于`adder()`的返回值是一个匿名函数, 那么我们就拿到了匿名函数的reference, 下面使用`ad(10)`传入值也就理所当然了.

下面我用一个更好的例子来解释:

```go
func minusValue(a, b int) func(int) int {
	fmt.Println("this is from minusValue, and a is", a, "b is", b)
	sum := a + b
	return func(para int) int {
		sum += para
		fmt.Println("this is from inner func and sum is", sum)
		return sum
	}
}
```

我直接使用`minusValue(1,2)`调用它, 那么控制台只会输出`this is from minusValue, and a is 1 b is 2`. 这里大家都理解. 当我用下面的方法调用, 就体现出了闭包的另一个特性, **闭包匿名函数中返回的变量只要还有reference在用, 那么它就会一直存在到内存中**, 请看下面的调用和结果输出:

```go
	f := minusValue(1,2)
	f(10)
	f(10)
	fmt.Println("----------------")
	n := minusValue(10,20)
	n(100)
```

输出:
![Snipaste_2020-02-28_22-09-13.jpg][1]

这就是闭包的用法以及特性, 那么在实际的生产中, 也有着很多作用(我也是才知道), 例如([来源][2]):
编写一个程序，具体要求如下：

> 编写一个函数 makeSuffix(suffix string) ，可以接收一个文件后缀名（比如.jpg），并返回一个闭包；
> 调用闭包，可以传入一个文件名，如果该文件名没有指定的后缀（比如 .jpg），则返回 文件名.jpg，如果有 .jpg后缀，则返回源文件名；
> strings.HasSuffix，该函数可以判断某个字符串是否有指定的后缀。

答案:

```go
package main

import (
	"fmt"
	"strings"
)
func makesuffix(suffix string) func(string) string {
	return func(name string) string {
		//如果name没有指定的后缀，则加上，否则就返回原来的名字
		if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
	}
}

func main() {
	f2 :=makesuffix(".jpg")
	fmt.Println("文件名处理后=", f2("winter"))
	fmt.Println("文件名处理后=", f2("bird.jpg"))
}
```

通过使用闭包的特性, 预先"设置"一个要处理的后缀, 之后通过不断调用闭包的reference, 就可以达到所需要求.

[1]: /assets/images/202002/2249658084.jpg
[2]: https://blog.csdn.net/cui_yonghua/article/details/93645557
