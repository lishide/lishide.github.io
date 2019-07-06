---
title: Kotlin 使用 Parcelize 注解简化 Parcelable 的书写
date: 2019-07-06 12:21:26
tags:
- Android
- Kotlin
- Parcelize
---

Parcelize 是 Kotlin 在 1.1.4 中新增加的功能。

<!--more-->

# Parcelize 注解

在新版的 Kotlin 插件中，包含了一个自动 `Parcelable` 实现生成器。只需要在主函数中，声明序列化的属性并添加一个 `@Parcelize` 注解，它将自动为我们创建 `writeToParcel()` 和 `createFromParcel()` 等模板代码。

编译完以后，实际上是跟以前实现方式的结果是一致的。

其实并没有什么高深的地方，但是这一点可以节约我们的代码量，而且提高开发效率。

# 配置
## Kotlin 版本

`1.1.4` 及以上

## Gradle 配置
`@Parcelize` 是一个实验室功能，所以还需要在 `Gradle` 中，增加 `experimental` 配置。

```groovy
androidExtensions {
    experimental = true
}
```

## 解决 Lint 错误

直接使用 `@Parcelize` 你将面临一个 Lint 的错误提示，只需要增加 `@SuppressLint("ParcelCreator")` 忽略它就可以了。

> 新版本已经支持。

# 示例：数据类

```java
@Parcelize
data class User(val name: String, val age: Int) : Parcelable
```
