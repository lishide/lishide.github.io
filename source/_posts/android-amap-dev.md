---
title: 高德地图开发：点击 Marker（或气泡）跳转到其它地图，以及开发中问题解决
date: 2018-02-24 09:26:41
tags: [Android, AMap]
---
高德地图，在点击 Marker 上的气泡可以跳转到手机中的地图进行导航。
<!--more-->

1. 设置地图 InfoWindow 点击事件监听，并实现`AMap.OnInfoWindowClickListener`。

```java
aMap?.setOnInfoWindowClickListener(this)
```

2. 处理点击事件

```java
/**
 * 对InfoWindow点击响应事件
 */
override fun onInfoWindowClick(p0: Marker?) {
    try {
        val uri = Uri.parse("geo:$lat,$lng?q=$address")
        val intent = Intent(Intent.ACTION_VIEW, uri)
        this.startActivity(intent)
    } catch (e: Exception) {
        Toast.makeText(getApplication(), "沒有地图应用", Toast.LENGTH_SHORT).show()
    }
}
```

---------

其实，为什么要在点击 marker 或 InfoWindow 的时候才响应事件呢？任何地方都行啊，好吧，真尴尬￣□￣｜｜

在高德地图开发中遇到一个坑是：进入地图时奔溃，检查了所有的`.so`文件，都没问题也没解决，最后发现是混淆代码的问题。这里要吐槽一下，高德地图没有提供他们的代码混淆规则！！！

分享出我自己添加的**Amap ProGuard rules**，不确定是否完全，暂未发现问题。

```bash
################# 高德相关混淆文件 #################
-dontwarn com.amap.api.**
-dontwarn com.autonavi.**
-dontwarn com.a.a.**
-keep class com.amap.api.**  {*;}
-keep class com.autonavi.**  {*;}
-keep class com.a.a.**  {*;}
-keep class com.amap.api.services.**{*;}
```

使用的版本是`com.amap.api:3dmap:5.6.0`，于 2017-12 开发。

