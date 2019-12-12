---
title: Android 加快编译速度
date: 2019-10-25 22:51:26
tags:
- Android
- gradle
- Kotlin
---

加快编译速度，做以下几点配置。

<!--more-->

## 开启 gradle 并行编译，开启 daemon，调整 jvm 内存大小
```groovy
# When set to true the Gradle daemon is to run the build.
org.gradle.daemon=true
# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
# Default value: -Xmx10248m -XX:MaxPermSize=256m
org.gradle.jvmargs=-Xmx4096m -XX:MaxPermSize=1024m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
org.gradle.parallel=true
# Enables new incubating mode that makes Gradle selective when configuring projects.
# Only relevant projects are configured which results in faster builds for large multi-projects.
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:configuration_on_demand
org.gradle.configureondemand=true
```

## 开启 gradle 缓存
```groovy
org.gradle.caching=true
```

## 启用构建缓存
```groovy
# 启用并设置「构建缓存」的目录 (使用 `./gradlew cleanBuildCache` 指令清除 cache 内容)
android.enableBuildCache=true
android.buildCacheDir=buildCacheDir/
```
Build Cache 默认的存储目录`~/.android/build-cache`。为了方便管理（如，缓存过多时手动清除），上述配置的第二行就自己指定了缓存的存储目录。

将缓存目录添加到`.gitignore`：
```groovy
buildCacheDir.lock
buildCacheDir/
```

关于更多 Build Cache 的内容请参考官方说明：https://developer.android.com/studio/build/build-cache.html
如果无法访问请看这里：https://developer.android.google.cn/studio/build/build-cache.html


## 禁用 PNG 图片优化处理
### 在 Root Project 的 build.gradle 文件中添加下面一个函数, 来判断是否是打 debug 包, 如下:
```groovy
//是否是执行Debug相关task (通用函数, 可供子module调用)
def isDebug() {
    def taskNames = gradle.startParameter.taskNames
    for (tn in taskNames) {
        if((tn.contains("install") || tn.contains("assemble")) && tn.contains("Debug")) {
             return true
        }
    }
    return false
}
```
### 在主 module 的 build.gradle 文件中的 android {} 块中添加下面配置:

```groovy
android {

    // 其他配置省略 ...

    //如果是构建debug包, 则禁用 "png cruncher" (默认cruncherEnabled=true, 禁用以加速构建)
    def enableCruncher = { ->
        return !isDebug()
    }

    aaptOptions { //禁用cruncher, 以加速编译
        cruncherEnabled = enableCruncher()
        cruncherProcesses = 0
    }

}
```

### Debug 关闭 crunchPng 优化
buildTypes 的 debug 中增加
```groovy
crunchPngs false //关闭crunchPng优化, 以加快构建
```

## Dex 配置项优化
```groovy
// 优化编译速度
// 优化`transformClassDexBuilderForDebug`的速度
dexOptions {
    dexInProcess true
    preDexLibraries true
    maxProcessCount 8
}
```

## 开启增量编译
compileOptions 中增加
```groovy
incremental = true  //开启增量编译
```

## 跳过 Tests 和 Lint 相关的 Task
```groovy
//跳过 Lint 和 Test 相关的 task, 以加速编译
if (isDebug()) {
    gradle.taskGraph.whenReady {
        tasks.each { task ->
            if (task.name.contains("Test") || task.name.contains("Lint")) {
                task.enabled = false
            }
        }
    }
}
```

## Kotlin 相关编译优化
```groovy
# 开启kotlin的增量和并行编译
kotlin.incremental=true
kotlin.incremental.java=true
kotlin.incremental.js=true
kotlin.caching.enabled=true
# 开启kotlin并行编译
kotlin.parallel.tasks.in.project=true
# 优化kapt
# 并行运行kapt1.2.60版本以上支持
kapt.use.worker.api=true
# 增量编译 kapt1.3.30版本以上支持
kapt.incremental.apt=true
# kapt avoiding 如果用kapt依赖的内容没有变化，会完全重用编译内容，省掉`app:kaptGenerateStubsDebugKotlin`的时间
kapt.include.compile.classpath=false
```

## 主 Module 中添加 kapt 闭包
```groovy
// 优化编译速度 如果有用到kapt添加如下配置
kapt {
    useBuildCache = true
    javacOptions {
        option("-Xmaxerrs", 500)
    }
}
```

至此便完成了 Gradle 的编译速度的优化，包括 Kotlin 下编译，编译速度应该有一个很大的提升，希望你也可以通过设置提高工作效率。可能还有一些诸如 MultiDex 方面的优化，以后在优化中不断探索。

