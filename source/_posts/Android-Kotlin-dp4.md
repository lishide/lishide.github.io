---
title: 【团队分享】Kotlin 及设计模式实践（四）
date: 2018-05-02 13:20:00
tags: [Android, Kotlin, Design Patterns]
---

这篇文章搁置了太久，直到今天，还是没有完成。为了发布新文章，反复把这个文档移出，再还原。今天先发布出来，防止丢失，过几天再修改补全文章，不能再拖延啦。

<!--more-->

# 2. 适配器模式
## 2.1 定义
适配器模式(Adapter Pattern) ：将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

## 2.2 结构
适配器模式包含如下角色：

- Target：目标抽象类
- Adapter：适配器类
- Adaptee：适配者类
- Client：客户类

适配器模式有对象适配器和类适配器两种实现：
对象适配器：
![对象适配器](http://p6wpxhpqt.bkt.clouddn.com/img_dp_Adapter.jpg)
类适配器：
![类适配器](http://p6wpxhpqt.bkt.clouddn.com/img_dp_Adapter_classModel.jpg)




