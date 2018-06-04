---
title: Android 发布 Lib 到 JitPack
date: 2018-03-01 08:57:16
tags: [Android, GitHub, JitPack]
---
# 前言

转载自 [赵彦军博客](http://www.cnblogs.com/zhaoyanjun/p/5942616.html)

最近封装了几个 Android 的  Library，想把它们放到仓库，方便在以后的项目中直接以依赖的方式引用。参考了几个博客，今天这篇博文**总结**和**推荐**一个最简单方便的方式 —— JitPack。
<!--more-->

# 步骤
## 2.1 在 Android Studio 里面配置 Jitpack 插件

在项目的根目录下的 build.gradle 文件里面添加

 - 在 dependencies 下面添加
``` groovy
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
```

 - 在 repositories 下面添加
```  groovy
maven { url "https://jitpack.io" }
```
如图所示
![Jitpack 插件配置](http://p6wpxhpqt.bkt.clouddn.com/img_aj1_build.png)

Ps：在配置 JitPack 插件的时候，需要用到插件的版本号，目前为止最新版是 1.5。[查看最新版本号，插件的 GitHub 仓库](https://github.com/dcendents/android-maven-gradle-plugin)

## 2.2 配置 Library Module 的 build.gradle

在 Library module 的 build.gradle 文件里面添加
``` groovy
apply plugin: 'com.github.dcendents.android-maven'
```

有的教程说还有这条，加了好像没有被用到 ...
``` groovy
group = 'com.github.YourUsername'  //替换成你的 Github 的用户名
```

如图所示
![配置 Library Module](http://p6wpxhpqt.bkt.clouddn.com/img_aj2_lib.png)

**Sync Now**

## 2.3 将项目提交到 GitHub 上

## 2.4 登录 GitHub 发布一个发行版本

点击 release -> 点击创建新的 release 版本 -> 书写版本号和 release 信息

![release.gif](http://p6wpxhpqt.bkt.clouddn.com/img_aj3_release.gif)


## 2.5 获取引用方式
进入 JitPack 网站 [https://jitpack.io/](https://jitpack.io/)，输入项目的 GitHub 地址：https: // github . com/ username / repo，点击 Look up，下面就显示出了所有的 release 版本。
![release 版本](http://p6wpxhpqt.bkt.clouddn.com/img_aj4_look.png)

点击 Get it 查看引用方式
![Get it](http://p6wpxhpqt.bkt.clouddn.com/img_aj5_get.png)

比如：
``` groovy
compile 'com.github.lishide:NoHttpConnecter:v1.0.0'
```

---

**到此，就结束了，对，就是这么简单。**


