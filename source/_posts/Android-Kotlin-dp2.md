---
title: 【团队分享】Kotlin 及设计模式实践（二）
date: 2018-03-28 10:43:05
tags: [Android, Kotlin, Design Patterns]
---
上一讲中以建造者模式开始，代码使用了 Kotlin 语言，有些写法可能不太理解，本讲先讲解一些常见的关于类与继承、属性和字段及修饰符等相关的知识，然后分享工厂方法模式的简单实现。

<!--more-->

### 1. Kotlin

#### 1.1 类与继承
##### 1.1.1 类
使用关键字 class 声明类，类声明由类名、类头（指定其类型参数、主构造函数等）以及由花括号包围的类体构成。
``` java
class Test {
}

class Empty
```

##### 1.1.2 构造函数
类可以有一个主构造函数以及多个二级构造函数。主构造函数是类头的一部分：跟在类名后面(可以有可选的类型参数)。

``` java
class Person constructor(firstName: String) {
}
```
> 如果主构造函数没有注解或可见性说明，则 `constructor` 关键字是可以省略

主构造函数不能包含任意代码。初始化代码可以放在以 init 做前缀的初始化块内
``` java
class Customer(name: String) {
    init {
        println("Customer initialized with value ${name}")
    }
}
```

声明属性并在主构造函数中初始化,在 Kotlin 中有更简单的语法：
``` java
class Person(val firstName: String, val lastName: String, var age: Int) {
}
```
如果构造函数有注解或可见性声明，则 `constructor` 关键字是不可少的，并且可见性应该在前：
``` java
class Customer public @inject constructor (name: String) {...}
```

##### 1.1.3 二级构造函数
类也可以有二级构造函数，需要加前缀 `constructor`:
``` java
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

##### 1.1.4 创建类的实例
我们可以像使用普通函数那样使用构造函数创建类实例：
``` java
val test = Test()
val customer = Customer("Jobs")
```

> Kotlin 没有 new 关键字

##### 1.1.5 类成员
类可以包含：
-- 构造函数和初始化代码块
-- 函数
-- 属性
-- 内部类
-- 对象声明

##### 1.1.6 继承
``` java
class Example //　隐式继承于 Any
```

声明一个明确的父类，需要在类头后加冒号再加父类
``` java
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```
如果类有主构造函数，则基类可以而且是必须在主构造函数中使用参数立即初始化。
如果类没有主构造函数，则必须在每一个构造函数中用 super 关键字初始化基类，或者在代理另一个构造函数做这件事。注意在这种情形中不同的二级构造函数可以调用基类不同的构造方法：
``` java
class MyView : View {
    constructor(ctx: Context) : super(ctx) {
    }
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx,attrs) {
    }
}
```

##### 1.1.7 复写方法
kotlin 需要把可以复写的成员都明确注解出来，并且重写它们：
``` java
open class Base {
    open fun v() {}
    fun nv() {}
}

class Derived() : Base() {
    override fun v() {}
}
```

标记为override的成员是open的，它可以在子类中被复写。如果你不想被重写就要加 final:
``` java
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

##### 1.1.8 复写属性
复写属性与复写方法类似
``` java
open class Foo {
  open val x: Int get { ... }
}

class Bar1 : Foo() {
  override val x: Int = ...
}
```

##### 1.1.9 抽象类
一个类或一些成员可能被声明成 abstract 。
``` java
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```
#### 1.2 属性和字段
##### 1.2.1 属性声明
使用 var 关键字声明可变属性，用 val 关键字声明只读属性
``` java
var name: String = ...
val result = ...
```

##### 1.2.2 延迟初始化属性
避免非空检查
``` java
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()
    }
}
```

#### 1.3 可见性修饰词
类，对象，接口，构造函数，属性以及它们的 setter 方法都可以有可见性修饰词。( getter与对应的属性拥有相同的可见性)。在 Kotlin 中有四种修饰词：`private`,`protected`,`internal`,以及 `public` 。默认的修饰符是 `public`。

...（常见，略）
internal —— 模块
`internal` 修饰符是指成员的可见性是只在同一个模块中才可见的。


### 2. 工厂方法模式
#### 2.1 定义

工厂方法模式（Factory Method Pattern）又称为工厂模式，也叫虚拟构造器（Virtual Constructor）模式或者多态工厂（Polymorphic Factory）模式，它属于类创建型模式。在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

#### 2.2 结构

Factory：抽象工厂角色，定义创建实例的抽象方法；
ConcreteFactory：具体工厂角色，负责创建特定实例；
Product：抽象产品角色，是所创建的所有对象的父类，负责描述所有实例所共有的公共接口；
ConcreteProduct：具体产品角色，是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

### 3. 简单工厂模式

#### 3.1 定义
简单工厂模式（Simple Factory Pattern）：又称为静态工厂方法（Static Factory Method）模式，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

#### 3.2 结构
Factory：工厂角色，负责实现创建所有实例的内部逻辑；
Product：抽象产品角色，是所创建的所有对象的父类，负责描述所有实例所共有的公共接口；
ConcreteProduct：具体产品角色，是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

### 4 工厂模式的实现

#### 4.1 Product
定义了抽象产品角色，及抽象方法 print。
``` java
abstract class Product {
    abstract fun print()
}
```

#### 4.2 ConcreteProductA 与 ConcreteProductB
定义了两个具体产品角色，分别实现了 print 方法。
``` java
class ConcreteProductA : Product() {
    override fun print() {
        println("print of ConcreteProductA")
    }
}

class ConcreteProductB : Product() {
    override fun print() {
        println("print of ConcreteProductB")
    }
}
```

#### 4.3 抽象 Factory
定义了抽象工厂角色，及抽象方法 factoryMethod。
``` java
abstract class Factory {
    abstract fun factoryMethod(): Product
}
```

#### 4.4 ConcreteFactoryA与ConcreteFactoryB
定义了两个具体工厂角色，分别实现了 factoryMethod 方法。
``` java
class ConcreteFactoryA : Factory() {
    override fun factoryMethod(): Product {
        println("具体工厂A: create ProductA")

        return ConcreteProductA()
    }
}

class ConcreteFactoryB : Factory() {
    override fun factoryMethod(): Product {
        println("具体工厂B: create ProductB")

        return ConcreteProductB()
    }
}
```

#### 4.5 生产产品
不同的具体产品实例，用不同的具体工厂来创建。
``` java
var factory1: Factory = ConcreteFactoryA()
var product1 = factory1.factoryMethod()
product1.print()

factory1 = ConcreteFactoryB()
product1 = factory1.factoryMethod()
product1.print()
```

#### 4.6 简单工厂
像这样拥有多个工厂的方式称之为多工厂方法模式，当工厂只有一个的时候，则可将其简化。
``` java
class SimpleFactory {
    fun createProduct(tag: String): Product? {
        var product: Product? = null
        when (tag) {
            "A" -> {
                product = ConcreteProductA()
                println("create ProductA")
            }
            "B" -> {
                product = ConcreteProductB()
                println("create ProductB")
            }
            else -> {
            }
        }

        return product
    }
}
```
简单工厂角色，实现了根据传入的参数来创建产品的功能。

#### 4.7 简单工厂生产产品

``` java
val factory = SimpleFactory()
var product: Product? = factory.createProduct("A")
if (product != null) {
    product.print()
}
product = factory.createProduct("B")
if (product != null) {
    product.print()
}
product = factory.createProduct("C")
if (product != null) {
    product.print()
}
```
这里加入了产品对象是否为 null 的判断，当用户传入错误的参数时是不能得到想要的产品的。

#### 4.8 输出结果
``` bash
具体工厂A: create ProductA
print of ConcreteProductA
具体工厂B: create ProductB
print of ConcreteProductB
create ProductA
print of ConcreteProductA
create ProductB
print of ConcreteProductB
```




