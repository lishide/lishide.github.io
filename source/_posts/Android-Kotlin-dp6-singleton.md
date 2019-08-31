---
title: Kotlin-单例模式（Singleton Pattern）
date: 2019-08-31 19:56:57
tags:
- Kotlin
- Design Patterns
---

# 单例模式(Singleton Pattern)
单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。

## 单例模式的要点
- 一是某个类只能有一个实例；
- 二是它必须自行创建这个实例；
- 三是它必须自行向整个系统提供这个实例。

## 单例的几种实现方式
- 饿汉式单例
- 懒汉式单例
- 线程安全的懒汉式单例
- DCL（double check lock）双重校验锁式单例
- 静态内部类单例
- 枚举单例

## 饿汉式单例
饿汉式单例模式是实现单例模式比较简单的一种方式，它有个特点就是不管需不需要该单例实例，该实例对象都会被实例化。

### Kotlin 实现
kotlin 中的饿汉式实现比较简单，只需要定义一个 `object 对象表达式` 即可，无需手动去设置构造器私有化和提供全局访问点，这一点 Kotlin 编译器全给你做好了。

```kotlin
object Singleton
```

## 懒汉式单例
这种方式实现了懒加载，但是不是线程安全的，可能在多个线程中创建多个不同的实例。

### Kotlin 实现

```kotlin
class KLazilySingleton private constructor() {

    fun doSomething() {
        println("do some thing")
    }

    companion object {
        private var mInstance: KLazilySingleton? = null
            get() {
                return field ?: KLazilySingleton()
            }

        fun getInstance(): KLazilySingleton {
            return requireNotNull(mInstance)
        }
    }

}

// 在 Kotlin 中调用
fun main(args: Array<String>) {
    KLazilySingleton.getInstance().doSomething()
}
```

## 线程安全的懒汉式单例
这种方式具备很好的懒加载，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。

### Kotlin 实现

```kotlin
class KLazilySingleton private constructor() : Serializable {

    fun doSomething() {
        println("do some thing")
    }

    companion object {
        private var mInstance: KLazilySingleton? = null
            get() {
                return field ?: KLazilySingleton()
            }

        @JvmStatic
        @Synchronized  // 添加 synchronized 同步锁
        fun getInstance(): KLazilySingleton {
            return requireNotNull(mInstance)
        }
    }

    // 防止单例对象在反序列化时重新生成对象
    private fun readResolve(): Any {
        return KLazilySingleton.getInstance()
    }
}

// 在 Kotlin 中调用
fun main(args: Array<String>) {
    KLazilySingleton.getInstance().doSomething()
}
```

## DCL（double check lock）双重校验锁式单例

我们知道线程安全的单例模式直接是使用 `synchronized` 同步锁，锁住 `getInstance` 方法，每一次调用该方法的时候都得获取锁，但是如果这个单例已经被初始化了，其实按道理就不需要申请同步锁了，直接返回这个单例类实例即可。于是就有了 DCL 实现单例方式。这种方式采用双锁机制，安全且在多线程情况下能保持高性能。

### Kotlin 实现
在 Kotlin 中有个天然特性可以支持线程安全 DCL 的单例，可以说也是非常非常简单，就仅仅 3 行代码左右，那就是 `Companion Object + lazy 属性代理`，一起来看下吧。

```kotlin
class KLazilyDCLSingleton private constructor() : Serializable {

    fun doSomething() {
        println("do some thing")
    }

    private fun readResolve(): Any {  // 防止单例对象在反序列化时重新生成对象
        return instance
    }

    companion object {
        // 通过 @JvmStatic 注解，使得在 Java 中调用 instance 直接是像调用静态函数一样，
        // 类似 KLazilyDCLSingleton.getInstance()，
        // 如果不加注解，在 Java 中必须这样调用: KLazilyDCLSingleton.Companion.getInstance().
        @JvmStatic
        // 使用 lazy 属性代理，并指定 LazyThreadSafetyMode 为SYNCHRONIZED 模式保证线程安全
        val instance: KLazilyDCLSingleton by lazy(LazyThreadSafetyMode.SYNCHRONIZED) { KLazilyDCLSingleton() }
    }
}

//在 Kotlin 中调用，直接通过 KLazilyDCLSingleton 类名调用 instance
fun main(args: Array<String>) {
    KLazilyDCLSingleton.instance.doSomething()
}
```

## 静态内部类单例
DCL 虽然在一定程度上能解决资源消耗、多余 `synchronized` 同步、线程安全等问题，但是某些情况下还会存在 DCL 失效问题，尽管在 JDK1.5 之后通过具体化 volatile 原语来解决 DCL 失效问题，但是它始终并不是优雅一种解决方式，在多线程环境下一般不推荐 DCL 的单例模式。所以引出静态内部类单例实现，这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

### Kotlin 实现

```kotlin
class KOptimizeSingleton private constructor() : Serializable {

    companion object {
        @JvmStatic
        fun getInstance(): KOptimizeSingleton {  // 全局访问点
            return SingletonHolder.mInstance
        }
    }

    fun doSomething() {
        println("do some thing")
    }

    private object SingletonHolder {
        val mInstance: KOptimizeSingleton = KOptimizeSingleton()
    }

    private fun readResolve(): Any {  // 防止单例对象在反序列化时重新生成对象
        return SingletonHolder.mInstance
    }
}

// 在 Kotlin 中调用
fun main(args: Array<String>) {
    KOptimizeSingleton.getInstance().doSomething()
}
```

## 枚举单例
其实细心的小伙伴就会观察到上面例子中我都会去实现 `Serializable` 接口，并且会去实现 `readResolve` 方法。这是为了反序列化会重新创建对象而使得原来的单例对象不再唯一。通过序列化一个单例对象将它写入到磁盘中，然后再从磁盘中读取出来，从而可以获得一个新的实例对象，即使构造器是私有的，反序列化会通过其他特殊途径创建单例类的新实例。然而为了让开发者能够控制反序列化，提供一个特殊的钩子方法那就是 `readResolve` 方法，这样一来我们只需要在 `readResolve` 直接返回原来的实例即可，就不会创建新的对象。
枚举单例实现，就是为了防止反序列化，因为我们都知道枚举类反序列化是不会创建新的对象实例的。 Java的序列化机制对枚举类型做了特殊处理，一般来说在序列枚举类型时，只会存储枚举类的引用和枚举常量名称，反序列化的过程中，这些信息被用来在运行时环境中查找存在的枚举类型对象，枚举类型的序列化机制保证只会查找已经存在的枚举类型实例，而不是创建新的实例。

### Kotlin 实现

```kotlin
enum class KEnumSingleton {
    INSTANCE;

    fun doSomeThing() {
        println("do some thing")
    }
}

// 在 Kotlin 中调用
fun main(args: Array<String>) {
    KEnumSingleton.INSTANCE.doSomeThing()
}
```

## 总结

最后补充一点，关于在 Kotlin 中使用单例模式的建议：一般大多数情况情况下直接使用 `object对象表达式` 即可，因为它比较简单，生成的字节码也相比于静态内部类那种方式要少得多；如果需要懒汉式加载的话还是比较建议使用 Kotlin 中的 `by lazy + Compaion Object` 那种方式。

