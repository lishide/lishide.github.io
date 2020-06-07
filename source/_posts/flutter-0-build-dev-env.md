---
title: Flutter 初尝：搭建 Flutter 开发环境
date: 2020-04-24 21:00:05
tags: [Flutter, Android Studio]
---

# Flutter 初尝：搭建 Flutter 开发环境

[《Flutter实战》](https://book.flutterchina.club/)
[Flutter中文网](https://flutterchina.club/)

<!--more-->

## 获取 Flutter SDK

学习和使用可按照 [Flutter 中文网](https://book.flutterchina.club/) 教程进行，但是在安装过程中发现，使用 flutter 项目下载的 release 包安装后会报以下错误：

```bash
Error: The Flutter directory is not a clone of the GitHub project.
The flutter tool requires Git in order to operate properly;
to set up Flutter, run the following command:
git clone -b beta https://github.com/flutter/flutter.git
```

解决方案：
1. 在 flutter 目录执行 `git init`。**不推荐**
2. 按官方推荐 `git clone -b beta https://github.com/flutter/flutter.git` 使用 beta 分支。
> 这个仓库的拉取真的「看命」，有时候下载超慢，刚开始下载了 N 多次都失败了，今早来下载速度非常快。祝你好运！

以上仅是我遇到的情况，不代表官网的步骤有问题。主要步骤还是按照官网介绍进行哦。

总结一下安装流程（macOS）。

- 下载 SDK
终端路径切换至 `Applications`，运行：
`git clone -b beta https://github.com/flutter/flutter.git` 。

- 配置 flutter

  终端输入：`open ~/.bash_profile`，配置 Flutter 路径
  ```text
  export PUB_HOSTED_URL=https://pub.flutter-io.cnexport
  FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cnexport
  PATH=$PATH:/Applications/flutter/bin
  ```

- 更新配置的环境变量 `source ~/.bash_profile`

- 验证

  终端输入 `flutter -h`，没有提示该命令找不到，则配置成功。会自动安装一些 Flutter 依赖等。也有可能需要重启系统，那就重启后再试。

- 运行 `flutter doctor` 命令

  命令行输入 `flutter doctor`，安装 Futter 剩余依赖项。

  有报红的，按照提示一个一个的解决。

  在 Windows 上运行出现 `Unkonwn operating system. Cannot install Dart SDK.`

  解决办法：到 flutter/bin 目录，打开命令行，运行 `flutter.bat doctor`，而非 `flutter doctor` 则会出现下载相关依赖的操作，以后就可以用 `flutter doctor` 了。

## IDE 配置与使用

接下来再进行 IDE 的安装与配置，安装 Android Studio、安装 Xcode、安装 Flutter 插件，这些过程就不赘述了，相信它们已经存在于你的电脑上了。也可以使用 VS Code 编辑器进行开发和调试，同样安装 Flutter 插件即可。

创建 Flutter 应用，连接真机或模拟器设备，就可以运行和调试 Flutter 应用了。**热重载**是超棒的体验！

