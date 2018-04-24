---
title: 【团队分享】Kotlin 及设计模式实践（三）
date: 2018-04-13 15:40:35
tags: [Android, Kotlin, Design Patterns]
---
本讲的 Kotlin 知识部分主要讲解接口、数据类及常见函数的使用，第二部分讲解抽象工厂模式的简单实现。

<!--more-->

# 1. Kotlin
## 1.1 接口
Kotlin 的接口与 Java 8 类似，既包含抽象方法的声明，也包含实现。与抽象类不同的是，接口无法保存状态。

### 1.1.1 接口定义
使用关键字 `interface` 来定义接口
```java
interface MyInterface {
    fun bar()
    fun foo() {
      // 可选的方法体
    }
}
```

### 1.1.2 实现接口
一个类或者对象可以实现一个或多个接口。
```java
class Child : MyInterface {
    override fun bar() {
        // 方法体
    }
}
```

### 1.1.3 接口中的属性
可以在接口中定义属性。在接口中声明的属性要么是抽象的，要么提供访问器的实现。
```java
interface MyInterface {
    val prop: Int // 抽象的

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 28
}
```

### 1.1.4 解决覆盖冲突
实现多个接口时，可能会遇到同一方法继承多个实现的问题。例如
```java
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```
上例中，接口 A 和 B 都定义了方法 `foo()` 和 `bar()`。 两者都实现了 `foo()`, 但是只有 B 实现了 `bar()` (`bar()` 在 A 中没有标记为抽象， 因为**没有方法体时默认为抽象**）。因为 C 是一个实现了 A 的具体类，所以必须要重写 `bar()` 并实现这个抽象方法。
然而，如果我们从 A 和 B 派生 D，我们需要实现我们从多个接口继承的所有方法，并指明 D 应该如何实现它们。这一规则既适用于继承单个实现（`bar()`）的方法也适用于继承多个实现（`foo()`）的方法。

## 1.2 数据类
### 1.2.1 定义
只保存数据的类，用 data 标注:
```java
data class User(val name: String, val age: Int)
```
编译器会自动根据主构造函数中声明的所有属性添加如下方法：
- `equals()`/`hashCode` 函数
- `toString` 格式是 "User(name=john, age=42)"
- [compontN()functions](http://kotlinlang.org/docs/reference/multi-declarations.html) 对应按声明顺序出现的所有属性
- `copy()` 函数

> 如果在类中明确声明或从基类继承了这些方法，编译器不会自动生成。

为确保这些生成代码的一致性，并实现有意义的行为，数据类要满足下面的要求：
- 注意如果构造函数参数中没有 `val` 或者 `var` ，就不会在这些函数中出现；
- 主构造函数应该至少有一个参数；
- 主构造函数的所有参数必须标注为 `val` 或者 `var` ；
- 数据类不能是 abstract，open，sealed，或者 inner ；
- 从1.1开始数据类可以继承其它类
- 在 JVM 中如果构造函数是无参的，则所有的属性必须有默认的值;
```java
data class User(val name: String = "", val age: Int = 0)
```

### 1.2.2 复制
对一些属性做修改但想要其他部分不变。
实现（编译器自动添加的 copy 函数）：
```java
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
```
使用：
```java
val kotlin = User(name = "kotlin", age = 1)
val olderKotlin = kotlin.copy(age = 2)
```



## 1.3 函数
### 1.3.1 函数声明
使用 `fun` 关键字声明函数：
``` java
fun sum(x: Int): Int {
    return 2 * x
}
```
### 1.3.2 函数用法
调用函数使用传统的方法：
```java
val result = sum(2)
```
#### 1.3.2.1 参数
使用 `name: type` 表示法定义，参数用逗号隔开，每个参数必须有显式类型：
```java
fun powerOf(x: Int, y: Int) {
//……
}
```

#### 1.3.2.2 默认参数
函数参数可以有默认值，当省略相应的参数时使用默认值。与其他语言相比，这可以减少重载数量：
```java
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
//……
}
```
默认值通过类型后面的 = 及给出的值来定义。
覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：
```java
open class A {
    open fun foo(i: Int = 10) { …… }
}

class B : A() {
    override fun foo(i: Int) { …… }  // 不能有默认值
}
```
如果一个默认参数在一个无默认值的参数之前，那么该默认值只能通过使用**命名参数**调用该函数来使用：
```java
fun foo(bar: Int = 0, baz: Int) { /* …… */ }

foo(baz = 1) // 使用默认值 bar = 0
```

#### 1.3.2.3 命名参数
可以在调用函数时使用命名的函数参数。当一个函数有大量的参数或默认参数时会非常方便。
给定以下函数：
```java
fun reformat(str: String,
             isA: Boolean = true,
             isB: Boolean = true,
             isC: Boolean = false,
             word: Char = ' ') {
……
}
```
我们可以使用默认参数来调用它：
```java
reformat(str)
```
然而，当使用非默认参数调用它时，该调用看起来就像：
```java
reformat(str, true, true, false, '_')
```
使用命名参数我们可以使代码更具有可读性：
```java
reformat(str,
    isA = true,
    isB = true,
    isC = false,
    word = '_'
)
```
并且如果我们不需要所有的参数：
```java
reformat(str, word = '_')
```

#### 1.3.2.4 返回 Unit 的函数
如果一个函数不返回任何有用的值，它的返回类型是 `Unit`。
```java
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 或者 `return` 是可选的
}
```
Unit 返回类型声明也是可选的。上面的代码等同于：
```java
fun printHello(name: String?) {
    ……
}
```

#### 1.3.2.5 单表达式函数
当函数返回单个表达式时，可以省略花括号并且在 = 符号之后指定代码体即可：
```java
fun sum(x: Int): Int = x * 2
```
当返回值类型可由编译器推断时，显式声明返回类型是可选的：
```java
fun sum(x: Int) = x * 2
```

### 1.3.3 函数作用域
在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样创建一个类来保存一个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

#### 1.3.3.1 局部函数
Kotlin 支持局部函数，即一个函数在另一个函数内部。

#### 1.3.3.2 成员函数
成员函数是在类或对象内部定义的函数：
```java
class Sample() {
    fun foo() { print("Foo") }
}
```

成员函数以点表示法调用：
```java
Sample().foo() // 创建类 Sample 实例并调用 foo
```

# 2. 抽象工厂模式
## 2.1 定义
抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为 Kit 模式，属于对象创建型模式。

## 2.2 结构
AbstractFactory：抽象工厂，定义创建实例的抽象方法；
ConcreteFactory：具体工厂，实现具体逻辑；
AbstractProduct：抽象产品，定义产品的抽象方法；
Product：具体产品，实现具体逻辑；

![抽象工厂模式-类图](http://p6wpxhpqt.bkt.clouddn.com/img_dp_AbatractFactory.jpg)

## 2.3 抽象工厂模式的简单实现
### 2.3.1 AbstractProduct
定义两个抽象产品角色，分别有其抽象方法 print。
```java
abstract class AbstractProductA {
    abstract fun printA()
}
abstract class AbstractProductB {
    abstract fun printB()
}
```

### 2.3.2 ConcreteProduct

定义了四个具体产品角色，分别实现了父类对应的 printA 和 printB 方法。

A1 和 B1 属于同一个产品族的不同产品等级的两种产品，A2 和B2 类似。其中：

产品等级——同一类产品的产品等级相同，如海尔冰箱、西门子冰箱等；

产品族——同一家工厂生产的不同产品等级的产品，如海尔冰箱、海尔洗衣机等。
```java
class ConcreteProductA1 : AbstractProductA() {
    override fun printA() {
        println("printA of ConcreteProductA1")
    }
}

class ConcreteProductA2 : AbstractProductA() {
    override fun printA() {
        println("printA of ConcreteProductA2")
    }
}

class ConcreteProductB1 : AbstractProductB() {
    override fun printB() {
        println("printB of ConcreteProductB1")
    }
}

class ConcreteProductB2 : AbstractProductB() {
    override fun printB() {
        println("printB of ConcreteProductB2")
    }
}
```

### 2.3.3 AbstractFactory
定义抽象工厂角色，及抽象方法
```java
abstract class AbstractFactory {
    abstract fun createProductA(): AbstractProductA
    abstract fun createProductB(): AbstractProductB
}
```

### 2.3.4 ConcreteFactory
定义了两个具体工厂角色，分别实现
```java
class ConcreteFactory1 : AbstractFactory() {
    override fun createProductA(): AbstractProductA {
        println("create ProductA1")
        return ConcreteProductA1()
    }

    override fun createProductB(): AbstractProductB {
        println("create ProductB1")
        return ConcreteProductB1()
    }
}

class ConcreteFactory2 : AbstractFactory() {
    override fun createProductA(): AbstractProductA {
        println("create ProductA2")
        return ConcreteProductA2()
    }

    override fun createProductB(): AbstractProductB {
        println("create ProductB2")
        return ConcreteProductB2()
    }
}
```

### 2.3.5 AbstractFactoryPattern
不同产品族的具体产品实例，用不同的具体工厂来创建。
```java
var factory: AbstractFactory = ConcreteFactory1()
var productA = factory.createProductA()
var productB = factory.createProductB()
productA.printA()
productB.printB()

factory = ConcreteFactory2()
productA = factory.createProductA()
productB = factory.createProductB()
productA.printA()
productB.printB()
```
### 2.3.6 输出结果
```bash
----------抽象工厂模式 start----------
create ProductA1
create ProductB1
printA of ConcreteProductA1
printB of ConcreteProductB1
create ProductA2
create ProductB2
printA of ConcreteProductA2
printB of ConcreteProductB2
----------抽象工厂模式   end----------
```


