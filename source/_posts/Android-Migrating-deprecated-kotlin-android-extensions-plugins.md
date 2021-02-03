---
title: 迁移被废弃的 kotlin-android-extensions 插件
date: 2021-02-03 14:42:31
tags: [Android, Kotlin, Gradle]
---

在Kotlin 1.4.20-M2中，JetBrains废弃了Kotlin Android Extensions编译插件。

在项目中受此影响的功能可能有：
1. 通过 kotlinx.android.synthetic 获取控件 Id
2. @Parcelize

问题 1 推荐的替代方案是使用 ViewBinding，之前我的项目中已经使用这种方式了。

问题 2 解决方案：

添加 kotlin-parcelize 插件

```groovy
plugins {
    ..
    id 'kotlin-parcelize'
}
```

然后更改旧的 import 语句，将：`import kotlinx.android.parcel.Parcelize` 改为：`import kotlinx.parcelize.Parcelize`

例子：

```kotlin
import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
class User(val name: String, val age: Int): Parcelable
```


更多详细说明可见：https://weilu.blog.csdn.net/article/details/109557820
