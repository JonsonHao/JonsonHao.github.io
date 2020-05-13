---
layout:     post
title:      Kotlin 学习系列(一)——Kotlin
subtitle:   《Kotlin 实战》学习笔记
date:       2020-05-13
author:     JonsonHao
header-img: img/post-bg-kotlin1.jpg
catalog: true
tags:
    - Kotlin
---

![Kotlin基础](https://tva1.sinaimg.cn/large/007S8ZIlgy1ger6m4r31cj30u00wuwpb.jpg)

## 1. 函数和变量

从一个最经典的例子开始：一个打印 “Hello, World!” 的程序：

```
fun main(args: Array<String>) {
    println("Hello, World!")
}
```

我们可以从上面的一小段代码中发现：

- 关键字 fun 用来声明一个函数
- 参数类型写在它的名称后面
- 函数可以定义在文件的最外层，不需要把它放在类中
- 数组就是类
- 使用 println 代替了 System.out.println。
- 和许多其他现代语言一样，可以省略每行代码结尾的分号

### 1.1 函数

我们已经从上面的例子中直到该如何创建一个没有返回值的函数了。如果函数有返回值的，我们需要把返回值放在参数列表的后面：

```
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

总结一下，函数的声明以关键字 fun 开始，函数名称紧随其后，接下来是用括号括起来的参数列表。参数列表后面跟着函数的返回类型，他们之间用冒号隔开。

> 在 Kotlin 中，if 是表达式，而不是语句。语句和表达式的区别在于，表达式有值，并且能作为另一个表达式的一部分使用；而语句总是包围着它的代码块中的顶层元素，并且没有自己的值。在 Java 中，所有的控制结构都是语句。而在 Kotlin 中，除了循环（for、while 和 do/while）以外大多数控制结构都是表达式。

>另一方面，Java 中的赋值操作是表达式，在 Kotlin 中反而变成了语句。这有助于避免比较和赋值之间的混淆，而这种混淆是常见的错误

### 1.2 变量
### 1.3 字符串格式化
## 2. 类和属性
### 2.1 属性
### 2.2 自定义访问器
### 2.3 Kotlin 源码和布局：目录和包
## 3. 枚举和 when
### 3.1 枚举
### 3.2 when
### 3.3 when 结构中使用任意对象
### 3.4 使用不带参数的 when
### 3.5 合并类型：检查和转换
### 3.6 用 when 代替 if
### 3.7 代码块作为 if 和 when 的分支
## 4. 迭代事物：while 循环和 for 循环
### 4.1 while 循环
### 4.2 迭代数字：区间和数列
### 4.3 迭代 map
### 4.4 使用 in 检查集合和区间的成员
## 5. Kotlin 中的异常
### 5.1 try、catch 和 finally
### 5.2 try 作为表达式