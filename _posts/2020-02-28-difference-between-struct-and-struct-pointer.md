---
layout: post
title: "go语言中struct变量和struct指针的区别"
description: "go语言struct结构体"
date: 2020-02-28
tags: [golang, 学习]
categories: [技术, go]
comments: true
---
在学习结构体指针的时候有些疑问, 首先定义一个结构体, 初始化结构体和它的结构体指针:

```go
cat1 := Cat{"samy", 89.0}  
catPointer := &cat1  
  
catPointer.weight = 222  
fmt.Printf("%T \\n", cat1)  
fmt.Printf("%T \\n", catPointer)  
  
fmt.Println(cat1.weight)  
cat1.weight = 111  
fmt.Println(cat1.weight)
```

此时输出:

```
main.Cat 
*main.Cat 
222
111
```

此时变量`cat1`和`catPointer`类型是不同的, 这个我明白. 但是它们的区别是什么呢? 现在它们都可以对结构体的内容做出修改.


感谢两位的回答, 但是我要的答案并不是那两个.
在上述例子中, 我想知道`cat1`和`catPointer`的区别, 为何他们类型不同, 却表现相同.
在go官方教程中有这样一段话:

> To access the field`X`of a struct when we have the struct pointer`p`we could write`(*p).X`. However, that notation is cumbersome, so the language permits us instead to write just`p.X`, without the explicit dereference.

原文链接: https://tour.golang.org/moretypes/4

也就是说, 在go语言操作结构体时候, 用上述例子来说, 就是`cat1`和`catPointer`类型是**不同的**, 但是在使用他们访问结构体的内存时, 会把`catPointer`当作`(*catPointer)`来操作, 那么这两个不同类型的变量就会出现表现相同, 类型不同的情况了.
下面用程序验证,下图可以看出他们两个类型确实不同:
![Snipaste_2020-02-28_02-27-14.png][1]
手动加上`*`之后,再次进行操作:
![Snipaste_2020-02-28_02-46-23.png][2]
答案就很明显了.

[1]: /assets/images/202002/714650719.png
[2]: /assets/images/202002/1472716926.png
