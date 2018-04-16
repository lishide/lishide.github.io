---
title: 【团队分享】Kotlin 及设计模式实践（一）
date: 2018-03-19 11:13:38
tags: [Android, Kotlin, Design Patterns]
---
今天给大家分享 Kotlin 入门的一些基本语法概念以及其独特的优秀特性和最近一段时间在实际开发中使用到的一些习惯用法，至于基础知识，例如变量、类定义、函数等这些不在本次分享中详细阐述。有 Java 开发基础，甚至说有某一种开发语言的基础，都是比较容易上手的。通过下面的几点分享，大家先领略一下 Kotlin 的简洁和优雅。结合一种较为常用且简单的设计模式——建造者模式进行本次的讲解，作为抛砖引玉，可能经验不足，但希望我们团队能够固定有这样的分享学习活动，激活团队的氛围，传递更多的知识与技能。

<!--more-->

# 1. Kotlin

## 1.1 （对 Android 开发而言）不用 `findViewById`，提高开发效率
可以直接使用控件的 id 进行操作，减少原先在 Java 中大量的 `findViewById` 的代码。
``` xml
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```
IDE 提示导入类似这个包 `import kotlinx.android.synthetic.main.activity_main.*`，则可直接使用控件的 id 来操作。
``` java
recyclerView.layoutManager = LinearLayoutManager(this)
recyclerView.adapter = mAdapter
```

## 1.2 if 表达式
在 Kotlin 中， `if` 是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则） ，因为普通的 `if` 就能胜任这个角色。
### 1.2.1 用法
```groovy
// 传统用法
var max = a
if (a < b) max = b

// 带 else
var max: Int
if (a > b) {
	max = a
} else {
	max = b
}

// 作为表达式
val max = if (a > b) a else b
```

### 1.2.2 if 的分支可以是代码块，最后的表达式作为该块的值：
```java
val max = if (a > b) {
	println("Choose a")
	a
} else {
	println("Choose b")
	b
}
```
如果你使用 `if` 作为表达式而不是语句（例如：返回它的值或者把它赋给变量） ，该表达式需要有 `else` 分支。

## 1.3 when 表达式
`when` 取代了类 C 语言的 `switch` 操作符。

### 1.3.1 简单形式
```java
when (x) {
    1 -> println("x == 1")
    2 -> println("x == 2")
    else -> { // 注意这个块
    	println("x is neither 1 nor 2")
    }
}
```

### 1.3.2 多分支用相同的方式处理
可以把多个分支条件放在一起，用逗号分隔：
```java
when (x) {
    0, 1 -> println("x == 0 or x == 1")
    else -> println("otherwise")
}
```

### 1.3.3 任意表达式（而不只是常量） 作为分支条件
```java
//调用
whenExpressionTest(2)

private fun whenExpressionTest(x: Int) {
    val s = "2"
    when (x) {
        parseInt(s) -> println("s encodes x")
        else -> println("s does not encode x")
    }
}
```

### 1.3.4 `when` 也可以用来取代 `if - else if` 链
如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：
```java
private fun whenNoParamTest(x: String, y: Int) {
    when {
        x.isEmpty() -> println("x is empty")
        // Kotlin 中，使用 str1 == str2，比较的只是字符串的内容，此处返回的是 true
        x == "test" -> println("x's text is test")
        y == -1 -> println("y is -1")
        else -> println("otherwise")
    }
}
//调用
whenNoParamTest("test", 1)
```
> Q：如果上述调用的方法中参数改为`whenNoParamTest("test", -1)`，满足了多项条件，运行结果如何？为什么？
> A：会执行满足条件了第一个，其他的不被执行。

## 1.4 for 循环
### 1.4.1 语法
``` java
for (item in collection) print(item)
```

### 1.4.2 区间上迭代
``` java
// 整型区间
for (i in 1..4) print(i) // 输出“1234”
// 倒序迭代数字，步长为2
for (i in 8 downTo 1 step 2) {
    println(i)  // 换行输出“8642”
}
// 不包括其结束元素的区间
for (i in 1 until 10) {   // i in [1, 10) 排除了 10
     println(i)  // 换行输出“123456789”
}
```
> **区间表达式**在后期分享中详细说明。

### 1.4.3 通过索引遍历一个数组或者一个 list
``` java
val array = arrayOf("a", "b", "c")

for (i in array.indices) {
    println(array[i])
}
```

## 1.5 While 循环
`while` 和 `do .. while` 和其它语言没什么区别

## 1.6 空安全
许多编程语言（包括 Java）中最常见的异常之一，就是访问空引用的成员会导致空引用异常（`NullPointerException`）。Kotlin 的类型系统旨在消除来自代码空引用的危险。
### 1.6.1 可空类型与非空类型

Kotlin 类型系统区分一个引用可以容纳 `null` 还是不能容纳，如：
``` java
// String 类型的常规变量不能容纳 null
var a: String = "abc"
a = null // 编译错误
// 允许为空，写作 String?
var b: String? = "abc"
b = null // ok
```
调用它的方法或访问它的属性：
``` java
val l = a.length // OK，不会导致 NPE
val l = b.length // 错误：变量“b”可能为空
```
但是我们还是需要访问该属性，对吧？有几种方式可以做到，请看**1.6.2 安全调用**。

### 1.6.2 安全调用
#### 1.6.2.1 在条件中检查 null
``` java
val l = if (b != null) b.length else -1
```

#### 1.6.2.2 安全的调用
``` java
b?.length
```

#### 1.6.2.3 Elvis 操作符，写作 `?:`
``` java
val l = b?.length ?: -1
```

#### 1.6.2.4 `!!` 操作符（非空断言运算符）
``` java
val l = b!!.length
```
>  如果值为空，就会抛出一个 NPE 异常

另外还有通过**安全的类型转换**或其他方式来避免这些异常的发生，涉及到 Kotlin 的扩展函数，本次暂不做讲解。

# 2. 建造者模式

## 2.1 定义
建造者模式(Builder Pattern)：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

## 2.2 结构
Product：产品角色；
Director：指挥者，利用具体建造者创建产品；
AbstractBuilder：抽象建造者，定义抽象的建造方法；
ConcreteBuilder：具体建造者，实现建造方法；

## 2.3 Builder 模式的简单实现
以手机的组装为例，并把此过程简化为设置机型、设置操作系统、设置屏幕等几个部分，然后通过 Director 和具体的 Builder 来构建手机对象。示例如下：

### 2.3.1 手机抽象类，即 Product 角色
``` java
abstract class Mobile {
    private var mBrand: String? = null
    private var mMemory: Int = 0
    private var mScreen: Float = 1f
    protected var mOS: String? = null

    fun setBrand(brand: String) {
        mBrand = brand
    }

    fun setMemory(memory: Int) {
        mMemory = memory
    }

    fun setScreen(screen: Float) {
        mScreen = screen
    }

    abstract fun setOS()

    override fun toString(): String {
        return "Mobile(mBrand=$mBrand, mMemory=$mMemory, mScreen=$mScreen, mOS=$mOS)"
    }

}
```

### 2.3.2 具体的 Mobile 类，Honor
``` java
class HonorMobile : Mobile() {

    override fun setOS() {
        mOS = "Android 8.0"
    }
}
```

### 2.3.3 抽象 Builder 类，定义了5个抽象方法，用于设置产品属性及获取实例
``` java
abstract class Builder {
    // 设置品牌
    abstract fun buildBrand(brand: String)
    // 设置内存
    abstract fun buildMemory(memory: Int)
    // 设置屏幕
    abstract fun buildScreen(screen: Float)
    // 设置系统
    abstract fun buildOS()
    // 创建 Mobile
    abstract fun create(): Mobile
}
```

### 2.3.4 具体的 Builder 类，HonorBuilder，实现产品的创建
``` java
class HonorBuilder : Builder() {
    private val mMobile = HonorMobile()
    override fun buildBrand(brand: String) {
        mMobile.setBrand(brand)
    }

    override fun buildMemory(memory: Int) {
        mMobile.setMemory(memory)
    }

    override fun buildScreen(screen: Float) {
        mMobile.setScreen(screen)
    }

    override fun buildOS() {
        mMobile.setOS()
    }

    override fun create(): Mobile {
        return mMobile
    }
}
```

### 2.3.5 Director 类，负责构造 Mobile，通过设置的建造者，创建产品实例
``` java
class Director(builder: Builder) {
    private var mBuilder: Builder? = null

    init {
        mBuilder = builder
    }

    /**
     * 构建对象
     */
    fun construct(brand: String, memory: Int, screen: Float) {
        mBuilder?.buildBrand(brand)
        mBuilder?.buildMemory(memory)
        mBuilder?.buildScreen(screen)
        mBuilder?.buildOS()
    }
}
```

### 2.3.6 具体建造者，实现产品的创建
``` java
val build = HonorBuilder()
val aDirector: Director = Director(build)
aDirector.construct("Honor 9", 4, 5.15f)

tvBuild2.setOnClickListener {
    tvRes.text = build.create().toString()
}
```

注意：
现实开发中，Director 常会被省略，直接使用 Builder 来进行对象的组装（链式调用）,大致如下：

``` java
build.buildBrand("Honor V10")
build.buildMemory(6)
build.buildScreen(5.99f)
build.buildOS()

tvBuild2.setOnClickListener {
    tvRes.text = build.create().toString()
}
```

# 3. 总结
此次分享中的部分内容可能出现的有些突然，不太理解某些类及变量的声明等等，我会汇总大家的意见，在以后的讲解中继续分享。下次讲解关于类、函数、对象相关的知识。

几点感悟：
- 持续学习
- 有计划地小结与分享