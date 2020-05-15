---
layout:     post
title:      Kotlin 学习系列(一)——Kotlin 基础
subtitle:   《Kotlin 实战》学习笔记
date:       2020-05-13
author:     JonsonHao
header-img: img/home-bg-o.jpg
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

**表达式函数体**可以让前面的函数变得更简单。可以用这个表达式作为完整的函数体，并去掉花括号和 return 语句：

```
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

如果函数体写在花括号中，我们说这个函数有代码块体。如果它直接返回一个表达式，它就有表达式体。

我们还可以进一步简化 max 函数，省掉返回类型：

```
fun max(a: Int, b: Int) = if (a > b) a else b
```

事实上，每个变量和表达式都有类型，每个函数都有返回类型。但作为表达式体函数来说，编译器会分析作为函数体的表达式，并把它的类型作为函数的返回类型，即使没有显示地写出来。这种分析通常被称作**类型推导**。

### 1.2 变量

在 Kotlin 中，变量的声明是以关键字开始，然后是变量名，最后可以加上类型（有时也可以不加）：

```
val question = "The Ultimate Question of Life, the Universe, and Everything"
val answer = 42
//也可以显示指定类型
val answer: Int = 42
```

和表达式函数一样，如果你不指定变量类型，编译器会分析初始化器表达式的值，并把它的类型作为变量。如果变量没有初始化器，需要显式地指定它的类型：

```
val answer: Int
answer = 42
```

声明变量的关键字有两个：
- val —— 不可变引用。使用 val 声明的变量不能在初始化之后再次赋值。它对应的是 Java 的 final 变量。
- var —— 可变引用。这种变量的值可以被改变。这种声明对应的是普通的 Java 变量。

在定义了 val 变量的代码块执行期间，val 变量只能进行唯一一次初始化。但是，如果编译器能确保只有唯一一条初始化语句会被执行，可以根据条件使用不同的值来初始化它：

```
val message: String
if (canPerformOperation()) {
    message = "Success"
} else {
     message = "Failed"
}
```

尽管 val 引用自身是不可变的，但是它指向的对象可能是可变的，例如：

```
val languages = arrayListOf("Java")
languages.add("Kotlin")
```

即使 var 关键字允许变量改变自己的值，但它的类型却是改变不了的，例如：

```
var answer = 42
answer = “no answer”   //错误，类型不匹配
```

### 1.3 字符串格式化

```
val name = if (args.size > 0) args[0] else "Kotlin"
println("Hello, $name")
```

上面的例子中使用了一个新特性，叫做**字符串模版**。Kotlin 中可以让你在字符串字面值中引用局部变量，只需要在变量名称前面加上字符 $。这等价于 Java 中的字符串连接。

如果要在字符串中使用 $ 字符，你要对他进行转义：println("\$x") 会打印 $x，并不会把 x 解释成变量的引用

还可以引用更复杂的表达式，而不是仅限于简单的变量名称，只需要把表达式用花括号括起来：

```
if (args.size > 0) {
    println("Hello, ${args[0]}!")
}
```

还可以直接在双引号中嵌套双引号，只要它们除在某个表达式的范围内：

```
println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
```

## 2. 类和属性

简单的 JavaBean 类 Person，目前它只有一个属性，name

```
//Java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

在 Kotlin 中可以这样写：

```
class Person(val name: String)
```

在 Kotlin 中，public 是默认的可见性，所以可以省略。

### 2.1 属性

想让类的使用者访问到数据，得提供访问器方法：一个 getter，可能还有一个 setter。在 Java 中，字段和其访问器的组合常常被叫做属性。在 Kotlin 中，属性是头等的语言特性，完全代替了字段和访问器方法，在类中声明一个属性和声明一个变量一样：使用 val 和 var 关键字。声明成 val 的属性是只读的，而 var 属性是可变的。

```
class Person (
    //只读属性：生成一个字段和一个简单的 getter
    val name : String,
    //可写属性：一个字段、一个 getter 和一个 setter
    var isMarried: Boolean
)
```

当你声明属性的时候，你就声明了对应的访问器（getter 和 setter）。getter 和 setter 的命名规则有一个例外：如果属性的名字以 is 打头，getter 不会增加任何的前缀；而它的 setter 名称中的 is 会被替换成 set。

```
//在 Kotlin 中使用 Person 类
val person = Person("Bob", true)
println(person.name)
```

在 Java 中，使用 person.setMarried(false) 来表示未婚，而在 Kotlin 中，可以这样写 person.isMarried = false。

### 2.2 自定义访问器

假设你声明这样一个矩形，它能判断自己是否是正方形。不需要一个单独的字段来存储这个信息，因为可以随时通过检查矩形的长宽是否相等来判断：

```
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
    //声明属性的 getter
        get() {
            return height == width
        }
}
```

### 2.3 Kotlin 源码和布局：目录和包

Kotlin 不区分导入的是类还是函数，而且，它允许使用 import 关键字导入任何种类的声明。可以直接导入顶层函数的名称。

## 3. 枚举和 when

### 3.1 枚举

```
//声明一个简单的枚举类
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

enum 是一个软关键字：只有当它出现在 class 前面时才有特殊的意义，在其他地方可以把它当做普通的名称使用

```
//声明一个带属性的枚举类
enum class color (val r: Int, val g: Int, val b: Int) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

这里展示了 Kotlin 语法中唯一必须使用分号的地方：**如果要在枚举类中定义任何方法，就要使用分号把枚举敞亮列表和方法定义分开**。

### 3.2 when

和 if 相似，when 是一个有返回值的表达式，因此可以写一个直接返回 when 表达式的表达式函数。

```
fun getMnemonic(color: Color) = 
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }
```

上面的代码根据传进来的 color 值找到对应的分支。和 Java 不一样的是，你不需要在每个分支都写上 break 语句。也可以把多个值合并到同一个分支，只需要用逗号隔开这些值。

```
fun getWarmth(color: Color) = when(color) {
    Color.RED, Color.ORANGE, COlor.YELLOW -> "warm"
    Color.GREEN -> "neutral"
    Color.BUE, Color.INDIGO, Color.VIOLET -> "cold"
}
```

### 3.3 when 结构中使用任意对象

Kotlin 的 when 结构比 Java 中的 switch 结构强大很多。在 when 结构中的分支条件允许使用任何对象。

```
fun mix(c1: Color, c2: Color) = 
    when(setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }
```

Kotlin 标准函数库中有一个 setOf 函数可以创建出一个 Set。

### 3.4 使用不带参数的 when

```
fun mixOptimized(c1: Color, c2: Color) = 
    when {
        (c1 == RED && c2 == YELLOW) || 
        (c1 == YELLOW && c2 == RED) -> ORANGE
        (c1 == YELLOW && c2 == BLUE) || 
        (c1 == BLUE && c2 == YELLOW) -> GREEN
        (c1 == BLUE && c2 == VIOLET) || 
        (c1 == VIOLET && c2 == BLUE) -> INDIGO
        else -> throw Exception("Dirty color")
     }
```

如果没有给 when 表达式提供参数，分支条件就是任意的布尔表达式。

### 3.5 合并类型：检查和转换

在 Kotlin 中，你要使用 **is** 检查来判断一个变量是否是某种类型。is 检查和 Java 中 instanceOf 相似。但是在 Java 中，如果你已经检查过一个变量是某种类型并且要把它当作这种类型来访问其成员时，在 instanceOf 检查之后还需要显示的加上类型转换。在 Kotlin 中，编译器帮你完成了这些工作，如果你检查过一个变量是某种类型，后面就不再需要转换它，可以把它当作你检查过的类型使用。我们把这种行为称为**智能转换**

只能转换只在变量经过 is 检查且之后不在发生变化的情况下有效。当你对一个属性进行智能转换的时候，这个属性必须是一个 val 属性，而且不能有自定义的访问器。

使用 **as** 关键字来表示到特定类型的显式转换：

```
val n = e as Int
```

### 3.6 代码块作为 if 和 when 的分支

如果 if 、 when 的分支是一个代码块，代码块中的最后一个表达式会被作为结果返回

## 4. 迭代事物：while 循环和 for 循环

### 4.1 while 循环

Kotlin 有 while 循环和 do-while 循环，它们的语法和 Java 中相应的循环没有什么区别：

```
while (condition) {
    ...
}

do {
    ...
} while(condition)
```

### 4.2 迭代数字：区间和数列

在 Kotlin 中没有常规的 Java for 循环。为了替代这种最常见的循环用法，Kotlin 使用了区间的概念。

区间本质上就是两个值之间的间隔，这两个值通常是数字：一个起始值，一个结束值。使用 **.. 运算符**来表示区间：

```
val oneToTen = 1..10
```

注意 Kotlin 的区间是包含的或者闭合的，意味着第二个值始终是区间的一部分

```
//使用 when 实现 Fizz-Buzz 游戏
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz"
    i % 3 == 0 -> "Fizz"
    i % 5 == 0 -> "Buzz"
    else -> "$i"
}

for (i in 1..100) {
    print(fizzBuzz(i))
}
```

也可以使用倒着数
```
//downTo 是递减，step 是步长
for(i in 100 downTo 1 step 2) {
    print(fizzBuzz(i))
}

//输出
Buzz 98 Fizz 94 92 ...
```

使用 until 函数可以创建一个半闭合区间，也就是不包含结束值的区间，例如，循环 for(x in 0 until size) 等同于 for(x in 0..size - 1)。

### 4.3 迭代 map

```
val binaryReps = TreeMap<Char, String>()

for(c in 'A'..'F') {
    val binary = Integer.toBinaryString(c.toInt())
    binaryReps[c] = binary
}

for((letter, binary) in binaryReps) {
    println("$letter = $binary")
}
```

.. 语法不仅可以创建数字区间，还可以创建字符区间。

上面的例子展示了 for 循环允许展开迭代中的集合的元素。

根据键来访问和更新 map 的简明语法。可以使用 map[key] 读取值，并使用 map[key] = value 设置它们，个不需要调用 get 和 set

可以用这样的展开语法在迭代集合的同时跟踪当前项的下标：

```
val list = arrayListOf("10", "11", "1001")
for((index, element) in list.withIndex()) {
    println("$index: $element")
}
```

### 4.4 使用 in 检查集合和区间的成员

使用 **in** 运算符来检查一个值是否在区间中，或者它的逆运算，**!in**，来检查这个值是否不在区间中。

```
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9' 
```

in 运算符和 !in 也适用于 when 表达式。

```
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't konw..."
}
```

in 检查也同样适用于集合：

```
println("Kotlin" in setOf("Java", "Scala"))
```

## 5. Kotlin 中的异常

Kotlin 中异常处理语句的基本形式和 Java 类似，抛出异常的方式也不例外：

```
if (percentage !in 0..100) {
    throw IllegalArgumentException(
        "A percentage value must be between 0 and 100: $percentage")
}
```

Kotlin 中 throw 结构是一个表达式，能作为另一个表达式的一部分使用：

```
val percentage = 
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException(
        "A percentage value must be between 0 and 100: $percentage")
```

### 5.1 try、catch 和 finally

```
fun readNumber(read: BufferReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch(e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}
```

### 5.2 try 作为表达式

```
fun readNumber(read: BufferReader){
    val number = try {
        Integer.parseInt(reader.readLine())
    }
    catch(e: NumberFormatException) {
        null
    }

    println(number)
}
```