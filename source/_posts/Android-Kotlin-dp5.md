---
title: Kotlin 及设计模式实践（五）
date: 2019-07-12 20:54:26
tags:
- Android
- Kotlin
- Design Patterns
---

本讲的 Kotlin 知识部分记录区间/范围、类型检查和转换的简要笔记。

<!--more-->

# Kotlin

## 区间/范围（Ranges）

range 表达式是通过 rangeTo 函数形成的。rangeTo 函数拥有形如 `..` 的操作符，该操作符是用 `in` 和 `!in` 实现的。 Range 可以对任何可比较的类型做操作，但对整数基本类型是优化过的。

### 基本写法

- `0..100` 表示 [0,100]
- `1 until 100` 表示 [0,100)
- `i in 0..100` 判断 i 是否在区间 [0,100] 中

### 一些实用的函数

- **rangeTo()**

  `rangeTo()` 函数仅仅是调用 *Range 的构造函数

- **downTo()**

  倒序迭代

- **reversed()**

  返回反向的级数

- **step()**

  使用指定步数的迭代

- **until**

  不包括其结束元素的区间

### 使用例子

```kotlin
// Checking if value of comparable is in range. Optimized for number primitives.
if (i in 1..10) println(i)

if (x in 1.0..3.0) println(x)

if (str in "island".."isle") println(str)

// Iterating over arithmetical progression of numbers. Optimized for number primitives (as indexed for-loop in Java).
for (i in 1..4) print(i) // prints "1234"

for (i in 4..1) print(i) // prints nothing

for (i in 4 downTo 1) print(i) // prints "4321"

for (i in 1..4 step 2) print(i) // prints "13"

for (i in (1..4).reversed()) print(i) // prints "4321"

for (i in (1..4).reversed() step 2) print(i) // prints "42"

for (i in 4 downTo 1 step 2) print(i) // prints "42"

for (x in 1.0..2.0) print("$x ") // prints "1.0 2.0 "

for (x in 1.0..2.0 step 0.3) print("$x ") // prints "1.0 1.3 1.6 1.9 "

for (x in 2.0 downTo 1.0 step 0.3) print("$x ") // prints "2.0 1.7 1.4 1.1 "

for (str in "island".."isle") println(str) // error: string range cannot be iterated over

```

## 类型检查和转换

### `is` 与 `!is` 操作符

Kotlin 在运行时通过使用 `is` 操作符或其否定形式 `!is` 来检查对象是否符合给定类型。

`is` 运算符类似 Java 中的 `instanceof`：

```java
"abc" instanceof String
```

`is` 运算符可以检查对象 A 是否与特定的类型 X 兼容（此对象 A 是 X 类型或者派生于 X 类型），还可以用来检查一个对象（变量）是否属于某数据类型。

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
}
else {
    print(obj.length)
}
```

### 智能转换

在大多数情况下，不需要在 Kotlin 中使用显式转换操作符，因为编译器跟踪不可变值的 `is` 检查，并在需要时自动插入（安全的）转换：

```kotlin
if (x is String) {
  print(x.length) // x 自动转换为字符串
}
```

编译器足够聪明，能够知道如果反向检查导致返回，那么该转换是安全的：

```kotlin
if (x !is String) return
print(x.length) // x 自动转换为 String
```

或者在 `||` 和 `&&` 的右侧：

```kotlin
// `||` 右侧的 x 自动转换为字符串
if (x !is String || x.length == 0) return

// `&&` 右侧的 x 自动转换为字符串
if (x is String && x.length > 0) {
    print(x.length) // x 自动转换为字符串
}
```

这样的转换用于 `when 表达式` 和 `whie 循环` 中也一样：

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is Array<Int> -> print(x.sum())
}
```

### “不安全的”转换操作符

如果转换是不可能的，转换操作符会抛出一个异常。因此，我们称之为*不安全的*。 Kotlin 中的不安全转换由中缀操作符 ***as*** 完成：

```kotlin
val x: String = y as String
```

注意，***null*** 不能转换为 `String` 因该类型不是可空的， 即如果 `y` 为空，上面的代码会抛出一个异常。 为了匹配 Java 转换语义，我们必须在转换右边有可空类型，就像：

```kotlin
val x: String? = y as String?

```



### “安全的”（可空）转换操作符

为了避免抛出异常，可以使用*安全*转换操作符 ***as?***，它可以在失败时返回 ***null***：

```kotlin
val x: String? = y as? String

```

请注意，尽管事实上 ***as?*** 的右边是一个非空类型的 `String`，但是其转换的结果是可空的。



