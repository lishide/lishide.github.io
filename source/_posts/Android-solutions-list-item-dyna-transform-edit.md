---
title: 列表动态增减并移动，以及 item 中使用 EditText 的问题扫坑
date: 2018-08-10 12:19:03
tags: [Android, RecyclerView, EditText]
---
# 概述
项目中需要一个图文混排的编辑功能，相关信息由用户动态地随意添加。于是使用 RecyclerView 的多布局来实现，记录一下遇到的一些问题的解决方法，主要有以下技术点：动态增加 Item、动态删除 Item、上移 Item、图片选择功能、EditText 复用问题的解决等。

<!--more-->

# 解决方案
先来看下需要实现的图文混排的编辑页面
![图文混排的编辑页面](http://p6wpxhpqt.bkt.clouddn.com/img_art_rv_item_multi_dtl.png)

从图中可以看出此页面一共有 3 种布局类型，分别是文字输入 Item、图片选择 Item，以及使用两个 TextView 来制作按钮的菜单 Item（用来动态添加以上两种 Item）。

RecyclerView 的适配器用的是平时常用的 [BaseRecyclerViewAdapterHelper](https://github.com/CymChad/BaseRecyclerViewAdapterHelper)，创建三种对应的布局文件，并加入到适配器（`addItemType`）。布局文件比较简单就不贴出了，下面主要讲解一下用到的几个功能点和问题的解决。

## 动态增加 Item
增加 Item 的功能，调用 `list.add()` 方法即可，第一个参数传入 `list.size - 1` 是为了让每次增加的 Item 在**菜单 Item**的前一个位置。
``` java
list.add(list.size - 1, BmGoodsAddDtlEntity(BmGoodsAddDtlEntity.TYPE_LIST_TEXT))
```

## 动态删除 Item
删除 Item 也很简单，移除当前位置的 List：
``` java
list.removeAt(position)
```

## 上移 Item
上移Item，调用 `moveUpItem(position)` 方法，交换集合中两个元素。
``` java
/**
 * 上移 Item
 * @param position
 */
private fun moveUpItem(position: Int) {
    if (position == 0) {
        toast(getString(R.string.msg_already_top))
    } else if (position > 0 && position <= list.size - 1) {
        swap(list, position, position - 1)
    }
}

/**
 * 集合中两个元素的交换操作
 * @param list
 * @param oldPosition
 * @param newPosition
 * @param <T>
 */
private fun <T> swap(list: MutableList<T>?, oldPosition: Int, newPosition: Int) {
    if (null == list) {
        throw IllegalStateException("The list can not be empty...")
    }
    val tempElement = list[oldPosition]

    // 向前移动，前面的元素需要向后移动
    if (oldPosition < newPosition) {
        for (i in oldPosition until newPosition) {
            list[i] = list[i + 1]
        }
        list[newPosition] = tempElement
    }
    // 向后移动，后面的元素需要向前移动
    if (oldPosition > newPosition) {
        for (i in oldPosition downTo newPosition + 1) {
            list[i] = list[i - 1]
        }
        list[newPosition] = tempElement
    }
}
```

## 图片选择功能
图片选择相关功能见我的另一篇博文：
[Android 开发中常见的图片选择、裁剪及权限管理等第三方功能库的升级优化](https://lishide.github.io/2018/05/22/Android-improve-frame-album-permission/)

## EditText 复用问题的解决
### 问题
RecyclerView 中使用 EditText 滚动后数据消失，错乱。

### 解决办法
首先想到需要为 EditText 添加 TextWatcher 监听器，但实际上这里有问题的。
（网上查了资料，最终顺利解决了问题）afterTextChanged 方法会被多次的调用，其根本原因是EditText 的重新复用，并且重新绘制。当重绘之后，该回调函数没有获取到填充的数据，还是原来复用的数据。
每次填充数据之前先移除 TextWatcher 监听器，然后为 EditText 填充数据，最后再为 EditText 添加 TextWatcher 监听器，取得输入框里的值保存到数据实体中。
关键代码如下：

```java
val etText = helper.getView<EditText>(R.id.etText)

// 第1步：为了避免TextWatcher在第2步被调用，提前将他移除。
if (etText.tag is TextWatcher) {
    etText.removeTextChangedListener(etText.tag as TextWatcher)
}

// 第2步：移除TextWatcher之后，设置EditText的Text。
etText.setText(item?.dtlEntity?.text)

val watcher = object : TextWatcher {
    override fun afterTextChanged(s: Editable?) {
        item?.dtlEntity?.text = if (s?.isNotEmpty() == true) s.toString() else ""
    }

    override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
    }

    override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
    }
}
etText.addTextChangedListener(watcher)
etText.tag = watcher
```

# 总结
实现一个类似的多布局动态生成和操作 Item 的列表，大概用到了以上技术点吧。关键代码即上面所述，作为一个记录和分享，方便以后参考。最后的 EditText 复用问题的解决同样适用于一条 Item 上有多个输入框的情况，重复上述操作即可。取值的话，遍历列表，根据需求取出相应值。

欢迎吐槽，互相交流。
