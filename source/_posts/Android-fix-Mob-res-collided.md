---
title: Android：解决 Mob-ShareSDK v3.7.4 资源冲突问题
date: 2020-05-30 21:23:09
tags: [Android, Mob-SDK]
---

# 问题描述

2020.05.29 下午 16 点多突然编译报错，提示资源冲突：

`Entry name 'res/drawable-xhdpi-v4/ssdk_country_back_arrow.png' collided`

<!--more-->

# 解决

原因是 ShareSdk 和 OneKeyShare 资源冲突，强制指定下使用的 SDK 版本，使用上一个版本。

```groovy
MobSDK {
    ...
    configurations.all {
        resolutionStrategy.force 'cn.sharesdk:OneKeyShare:3.7.3'
    }
}
```

MobSDK 是通过插件的形式接入的： `apply plugin: 'com.mob.sdk'`，最新的编译都用到它的最新版本。希望 Mob 团队早点解决此问题，发布下个版本。

