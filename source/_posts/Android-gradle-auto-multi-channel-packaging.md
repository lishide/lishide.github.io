---
title: Android 自动化脚本多渠道打包
date: 2019-12-11 18:03:29
tags:
- Android
- gradle
- VasDolly
---

# Android 自动化脚本多渠道打包

## 相关的参考技术文档
- [Android 自动化脚本多渠道加固、打包](https://juejin.im/post/5db037d4f265da4d2b34f557?utm_source)
- [VasDolly](https://github.com/Tencent/VasDolly)
- [VasDolly 命令行说明地址](https://github.com/Tencent/VasDolly/blob/master/command/README.md)
- [VasDolly 实现原理](https://github.com/Tencent/VasDolly/wiki/VasDolly%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)

## 技术方案

1. 使用 VasDolly 多渠道打包；
2. 其他市场直接上架，需要加固的市场让运营的同事手动加固一下。

<!--more-->

## 集成 VasDolly

### 添加对 VasDolly Plugin 的依赖

在根工程的 `build.gradle` 中，添加对打包 Plugin 的依赖：

```groovy
dependencies {
    classpath 'com.leon.channel:plugin:2.0.3'
}
```

### 引用 VasDolly Plugin

在主 App 工程的 `build.gradle` 中，添加对 VasDolly Plugin 的引用：

```groovy
apply plugin: 'channel'
```

### 添加对 VasDolly helper 类库的依赖

在主 App 工程的 `build.gradle` 中，添加读取渠道信息的 helper 类库依赖：

```groovy
dependencies {
    api 'com.leon.channel:helper:2.0.3'
}
```
> 以下流程可略过，在下面的**自动化脚本多渠道打包**流程中一起操作。

### 配置渠道列表

指定渠道文件 `channel.txt`，一行一个渠道信息。

### 通过 Gradle 生成多渠道包

（略）

### 通过命令行生成渠道包、读取渠道信息

从 `V1.0.5` 版本开始支持命令行，具体使用文档可参考 `command` 目录下的 [README](https://github.com/Tencent/VasDolly/blob/master/command/README.md)。

#### 读取渠道信息

通过 helper 类库中的 `ChannelReaderUtil` 类读取渠道信息。

```java
String channel = ChannelReaderUtil.getChannel(getApplicationContext());
```
如果没有渠道信息，那么这里返回 `null`，开发者需要自己判断。

> 更详细的集成文档和参数说明请查看 VasDolly 文档。

## 自动化脚本多渠道打包

### 新增目录结构

在项目根目录下新增一个 `vasdolly` 文件夹，里面包含一下几个文件：
- channel.txt：指定渠道文件，一行一个渠道信息。
- multi-channel.gradle：多渠道打包命令脚本
- VasDolly.jar：命令行工具，可在 VasDolly 仓库中获取。
- README.md：本说明文档，非必须。

### 编辑 channel.txt

添加渠道信息，一行一个，例如：

```text
official
yingyongbao
huawei
xiaomi
ali
qihoo360
oppo
vivo
```

### multi-channel.gradle

```groovy
ext {
    jarPath = "${project.rootDir}/vasdolly/VasDolly.jar"
    channelsPath = "${project.rootDir}/vasdolly/channel.txt"

    /**
     * 从 app/build 文件夹中找出 release 文件
     * 只能匹配出以 apk 结尾并且包含 release 字符串的 apk 文件
     */
    findReleaseApkPath = { String appBuildOutputPath ->
        def appBuildOutput = new File(appBuildOutputPath)
        def apkFile = null
        appBuildOutput.eachFile {
            if (it.name.endsWith(".apk") && it.name.contains("release")) {
                apkFile = it
            }
        }
        return apkFile
    }

    /**
     * 多渠道打包
     * apk -> 原有 release 包的文件
     * outputPath -> 多渠道打包后文件输出路径
     */
    buildMultipleChannels = { File apk, File outputPath ->
        println(outputPath)
        if (apk == null || !apk.exists()) {
            throw new FileNotFoundException("没有找到 APK 文件")
        }
        if (!outputPath.exists()) {
            outputPath.mkdirs()
        }

        def cmd = "java -jar ${jarPath} put -c ${channelsPath} ${apk} ${outputPath}"
        println cmd
        cmd.execute().waitForProcessOutput(System.out, System.err)
    }
}

```

### app 的 build.gradle 中添加

```groovy
apply from: "../vasdolly/multi-channel.gradle"

/**
 * 开启多渠道打包的任务
 * 这个任务会依赖 assembleRelease 打出来的 apk 包
 */
task assembleMultiChannelRelease() {
    group 'multipleChannels'
    dependsOn('assembleRelease')

    doLast {
        // 路径根据你的项目实际情况改变，不一定都是 app，比如多 module 的项目
        def appBuildOutputPath = "${project.rootDir}/app/build/outputs/apk/release/"
        def outputChannelsFilePath = appBuildOutputPath + "channels/"
        buildMultipleChannels(findReleaseApkPath(appBuildOutputPath), new File(outputChannelsFilePath))
    }
}
```

任务名 `assembleMultiChannelRelease` 和 apk 输出路径 `appBuildOutputPath` 可根据项目的实际情况改变，以避免在 **多 module / 多 productFlavors** 情况下出现的路径错误和冲突。

### 代码中获取渠道名称

```kotlin
fun getChannelName(ctx: Activity): String {
    return try {
        ChannelReaderUtil.getChannel(ctx.application)
    } catch (e: Exception) {
        e.printStackTrace()
        "official"
    }
}
```

### 编译打包

最终我们执行 `./gradlew assembleMultiChannelRelease`

记得 clean：`./gradlew clean`，可组合使用。

即，App 发布版多渠道打包命令（例子）为：

`./gradlew clean assembleMultiChannelRelease`

查看结果，在目标文件夹下即可找到生成的多渠道包。

至此便完成了**自动化脚本多渠道打包**的功能，通过一个命令即可快速生成多渠道包。



