---
title: Gradle：修改 .gradle 默认目录
date: 2021-11-19 17:40:28
tags:
- Android
- gradle
---

Gradle：修改 .gradle 默认目录

<!--more-->

1、将 `C:\Users\xxx\.gradle` 的默认目录复制到 `D:/目标文件夹/.gradle`

2、在 Windows 的环境变量中新建一个环境变量设置（如下），设置完成之后，点击确定，关闭设置窗口，重启计算机。

```bash
变量名：GRADLE_USER_HOME
变量值：D:\Toolbox\.gradle
```


