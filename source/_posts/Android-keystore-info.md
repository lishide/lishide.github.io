---
title: Android 获取 SHA1 等 keystore 的信息
date: 2018-03-15 19:29:31
tags: [Android, keystore, SHA1]
---
keystore 文件为 Android 签名证书文件

获取 keystore 相关信息命令：`keytool -list -v -keystore key_name`
<!--more-->

##### 1. 发布版
使用 apk 对应的 keystore，命令为：`keytool -v -list -keystore apk.keystore   //这个是自己打包生成的jks`

##### 2. 开发版
使用 debug.keystore，命令为：`keytool -list -v -keystore debug.keystore`
> 打开命令行，输入 `cd .android` 进入**.android**，输入上面的命令。

一定要记得加上 `-v` 参数，不然只能看到 SHA1，没有 MD5。