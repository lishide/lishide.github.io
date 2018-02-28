---
title: Kotlin Learn Note
date: 2018-02-28 09:14:00
tags: Kotlin
---

## 基础
### 基本类型


### 控制流：if、when、for、while
#### If 表达式
在 Kotlin 中， `if` 是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则） ，因为普通的 `if` 就能胜任这个角色。
```groovy
// 传统用法
var max = a
if (a < b) max = b
// With else
var max: Int
if (a > b) {
	max = a
} else {
	max = b
}

// 作为表达式
val max = if (a > b) a else b
```

if 的分支可以是代码块，最后的表达式作为该块的值：
```java
val max = if (a > b) {
	print("Choose a")
	a
} else {
	print("Choose b")
	b
}
```
如果你使用 `if` 作为表达式而不是语句（例如：返回它的值或者把它赋给变量） ，该表达式需要有 `else` 分支。

#### When 表达式
1.简单形式
```java
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个块
    	print("x is neither 1 nor 2")
    }
}
```

2.多分支用相同的方式处理——可以把多个分支条件放在一起，用逗号分隔：
```java
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

3.任意表达式（而不只是常量） 作为分支条件
```java
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

4.检测一个值在（`in`） 或者不在（`!in`） 一个区间或者集合中：
```java
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

5.检测一个值是（`is`） 或者不是（`!is`） 一个特定类型的值
注意： 由于[智能转换](http://)，你可以访问该类型的方法和属性而无需任何额外的检测。
```java
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

6.`when` 也可以用来取代 `if - else if` 链。
```java
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

#### For 循环
语法
```java
for (item in collection) print(item)
//
for (i in array.indices) {
	print(array[i])
}
```

#### While 循环
`while` 和 `do .. while` 照常使用
```java
while (x > 0) {
	x--
}

do {
	val y = retrieveData()
} while (y != null) // y 在此处可见
```

---
### 操作符和特殊符号
Kotlin 支持以下操作符和特殊符号：
- `+` 、 `-` 、 `*` 、 `/` 、 `%` —— 数学操作符
 	- `*` 也用于将数组传递给 vararg 参数
- `=`
	- 赋值操作符
	- 也用于指定参数的默认值
- `+=` 、 `-=` 、 `*=` 、 `/=` 、 `%=` —— 广义赋值操作符
- `++` 、 `--` —— 递增与递减操作符
- `&&` 、 `||` 、 `!` —— 逻辑“与”、“或”、“非”操作符（对于位运算，请使用相应的中缀函数）
- `==` 、 `!=` —— 相等操作符（对于非原生类型会翻译为调用 `equals()` ）
- `===` 、 `!==` —— 引用相等操作符
- `<` 、 `>` 、 `<=` 、 `>=` —— 比较操作符（对于非原生类型会翻译为调用 `compareTo()` ）
- `[` 、 `]` —— 索引访问操作符（会翻译为调用 `get` 与 `set` ）
- `!!` 断言一个表达式非空
- `?.` 执行安全调用（如果接收者非空，就调用一个方法或访问一个属性）
- `?:` 如果左侧的值为空，就取右侧的值（elvis 操作符）
- `::` 创建一个成员引用或者一个类引用
- `..` 创建一个区间
- `:` 分隔声明中的名称与类型
- `?` 将类型标记为可空
- `->`
	- 分隔 lambda 表达式的参数与主体
	- 分隔在函数类型中的参数类型与返回类型声明
	- 分隔 when 表达式分支的条件与代码体
- `@`
	- 引入一个注解
	- 引入或引用一个循环标签
	- 引入或引用一个 lambda 表达式标签
	- 引用一个来自外部作用域的 “this”表达式
	- 引用一个外部超类
- `;` 分隔位于同一行的多个语句
- `$` 在字符串模版中引用变量或者表达式
- `_`
	- 在 lambda 表达式中代替未使用的参数
	- 在解构声明中代替未使用的参数


