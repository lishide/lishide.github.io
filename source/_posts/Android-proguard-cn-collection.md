---
title: Android-proguard-cn-collection
date: 2018-06-06 11:46:00
tags: [Android, ProGuard]
---
# 前言
某一个夜深人静的加班夜，也许你在做代码混淆......ProGuard 是一个比较枯燥且没有成就感的技术，但 Android 项目没有了 ProGuard 还真就不行。在 Android 中使用 ProGuard 可以起到压缩、优化、混淆、预检的作用，保证我们开发出的 APK 更健壮。

<!--more-->

近期的一次发版中，同事反应一个发生在某米手机（没有黑~）上的奔溃 bug，手头的测试真机数量有限且均无此问题。使用云测试进行调试，上传了 release 和 debug 版的 apk，发现那款机型的确出现问题。经过分析，问题定位在了代码混淆问题上，因为想到自己之前的一次发版中也曾有过这个问题。

又一次遇到这样的问题，觉得有必要好好掌握一下正确地写代码混淆规则的姿势了，而不只是在使用第三方和轮子时，单纯地 copy 开源库的 ProGuard。

# 使用
## 编写 ProGuard 文件
受《App 研发录》启发，收集和整理了 Android 开发中国内项目常用库的混淆规则，按照包老师的思路，是一个三步走的过程。

- 基本混淆
- 针对 App 的量身定制
- 针对第三方 jar 包的解决方案

整理了一个类似模板的项目吧，希望对大家有用，轻松搞定混淆。并已开源到 GitHub——[Android-proguard-cn-collection](https://github.com/lishide/Android-proguard-cn-collection)，收集整理中，如果你有补充，期待你的 PR。

### 基本混淆
最基本的配置信息，任何 App 都要使用，基本不用动。（代码及注释均来自《App 研发录》）

```groovy
# # 1. 基本混淆======================================================================================
# ## 1.1 基本指令-------------------------------------------------------------
# 代码混淆压缩比，在 0~7 之间，默认为 5，一般不需要改
-optimizationpasses 5

# 混淆时不使用大小写混合，混淆后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共的库的类
-dontskipnonpubliclibraryclasses

# 指定不去忽略非公共的库的类的成员
-dontskipnonpubliclibraryclassmembers

# 不做预校验，preverify 是 proguard 的 4 个步骤之一
# Android 不需要 preverify，去掉这一步可加快混淆速度
-dontpreverify

# 有了 verbose 这句话，混淆后就会生成映射文件
# 包含有类名 -> 混淆后类名的映射关系
# 然后使用 printmapping 指定映射文件的名称
-verbose
-printmapping proguardMapping.txt

# 指定混淆时采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不改变
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

# 保护代码中的 Annotation 不被混淆，这在 JSON 实体映射时非常重要，比如 fastJson
-keepattributes *Annotation*

# 避免混淆泛型，这在 JSON 实体映射时非常重要，比如 fastJson
-keepattributes Signature

# 抛出异常时保留代码行号，在异常分析中可以方便定位
-keepattributes SourceFile,LineNumberTable
#---------------------------------------------------------------------------

# ## 1.2 需要保留的东西-------------------------------------------------------
# 保留所有的本地 native 方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留了继承自 Activity、Application 这些类的子类
# 因为这些子类都有可能被外部调用
# 比如说，第一行就保证了所有 Activity 的子类不要被混淆
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.android.vending.licensing.ILicensingService

# 保留在 Activity 中的方法参数是 view 的方法，
# 从而我们在 layout 里面编写 onClick 就不会被影响
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

# 枚举类不能被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留自定义控件（继承自 View）不被混淆
-keep public class * extends android.view.View {
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留 Parcelable 序列化的类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留 Serializable 序列化的类不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 对于 R（资源）下的所有类及其方法，都不能被混淆
-keep class **.R$* {
    *;
}

# 对于带有回调函数 onXXEvent 的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
}
#---------------------------------------------------------------------------
#===================================================================================================
```

### 针对 App 的量身定制
这部分做部分修改，保留你的项目中的实体类、自定义 View、内嵌类、WebView、反射等不被混淆，如果没有使用到的，忽略即可。

```groovy
# # 2. 针对 App 的量身定制============================================================================
# ## 2.1 保留实体类和成员不被混淆-----------------------------------------------
# 例如（替换成你的包名结构）：
-keep class com.yourpackage.app.model.entity.** {
    public void set*(***);
    public *** get*();
    public *** is*();
}
#---------------------------------------------------------------------------

# ## 2.2 内嵌类--------------------------------------------------------------

#---------------------------------------------------------------------------

# ## 2.3 webview------------------------------------------------------------
-keepclassmembers class fqcn.of.javascript.interface.for.Webview {
   public *;
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, jav.lang.String);
}
#---------------------------------------------------------------------------

# ## 2.4 对 JavaScript 的处理------------------------------------------------
# 保留 JS 方法不被混淆
#-keepclassmembers class com.example.xxx.MainActivity$JSInterface1 {
#    <methods>;
#}
#---------------------------------------------------------------------------

# ## 2.5 处理反射------------------------------------------------------------

#---------------------------------------------------------------------------

# ## 2.6 对于自定义 View 的解决方案--------------------------------------------
-keep class com.yourpackage.a.** { *; }
#---------------------------------------------------------------------------
#===================================================================================================
```

### 针对第三方 jar 包的解决方案
项目中使用到很多第三方提供的 SDK，如下列出了针对 `android-support-v4.jar` 和第三方 jar 包的解决方案。根据使用到的第三方库，查找对应的 ProGuard，加入到混淆规则中。

```groovy
# # 3. 针对第三方 jar 包的解决方案=====================================================================
# ## 3.1 针对 android-support-v4.jar 的解决方案-------------------------------
-libraryjars libs/android-support-v4.jar
-dontwarn android.support.v4.**
-keep class android.support.v4.** { *; }
-keep interface android.support.v4.app.** { *; }
-keep public class * extends android.support.v4.**
-keep public class * extends android.app.Fragment
#---------------------------------------------------------------------------

# ## 3.2 其他的第三方 jar 包的解决方案-----------------------------------------

#---------------------------------------------------------------------------
#===================================================================================================
```

## 在项目中指定混淆文件
将 Module 下的 build.gradle 中 minifyEnabled 设置为 true。

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

以上三部分混淆规则，可以写在一个 ProGuard 文件中，即创建 Android 工程后默认生成的 `proguard-rules.pro`，proguardFiles 指定使用`'proguard-rules.pro'`（默认就是这样）。

或者，可以写多个 ProGuard 文件，分别指定，这种方式更直观。
```groovy
proguardFile 'proguard-a.pro'
proguardFile 'proguard-b.pro'
proguardFile 'proguard-c.pro'
...
```

# 总结

- 如果需要代码混淆，请尽早做
- 测试工作要基于混淆包进行，才能尽早发现问题
- 发版前，要额外测试推送、分享、支付等功能

# 感谢
包建强老师的《App 研发录》
