---
title: Android 开发和调试常用命令
date: 2019-06-07 15:23:44
tags: [Android, ADB, Gradle]
---
# Android 开发和调试常用命令

Android 开发和调试中使用到的常用命令。

<!--more-->

## ADB

- 安装 APK

  `adb install xxx.apk`

- 清除已经安装的APK并安装新的APK

  `adb install -r xxx.apk`

- 卸载 APK

  `adb uninstall [package_name]`

- adb pull - 将设备中的文件放到本地

  `adb pull /sdcard/tmp.txt D:\`

- adb push - 将本地文件放到设备中

  `adb push D:\tmp.txt /sdcard`

- adb shell screencap - 截屏操作

  `adb shell screencap -p /sdcard/tmp.png`

- adb shell screenrecord - 录屏操作

  `adb shell screenrecord /sdcard/tmp.mp4`

- adb logcat - 查看当前日志信息

  `用法1：adb logcat -s tag`

  `用法2：adb logcat |findstr pname/pid/keyword`

- 清除指定 APP 的缓存

  `adb shell pm clear package_name`

- 输出指定包名 APP 的安装位置

  `adb shell pm path package_name`

- 输出手机中所有的包名

  `adb shell pm list packages`

- 查看指定包名的内存信息

  `adb shell dumpsys meminfo package_name`

## Gradle

Mac、Linux下是 `./gradlew`，windows下是 `gradlew`

- 删除 ProjectName/app 目录下的 build 文件夹

  `./gradlew clean`

- 检查依赖并编译打包

  `./gradlew build`

- 编译并打 Debug 包

  `./gradlew assembleDebug`

- 编译并打 Huawei 的 debug 包，其他类似

  `./gradlew assembleHuaweiDebug`

- 编译并打 Release 的包

  `./gradlew assembleRelease`

- 编译并打 Huawei 的 Release 包，其他类似

  `./gradlew assembleHuaweiRelease`

- Release 模式打包并安装

  `./gradlew installRelease`

- 卸载 Release 模式包

  `./gradlew uninstallRelease`

- 指定 Module 打包
项目中有许多的可以直接独立运行的 Module，如果在 Gradle 中将签名文件配置好了，那么就不需要普通的手动点击 `Generate Signed APK`，使用 `Terminal`更加方便。

    ```bash
    ./gradlew :<ModuleName>:assembleRelease
    // 示例：
    ./gradlew :sampleApp:assembleRelease
    ```
`[error]command not found`，通常是因为 gradle 没有加入到`PATH`中。将`gradle`改为`./gradlew`即可。( `./`表示同级目录）


