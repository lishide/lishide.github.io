---
title: 2018 年 8 月 Android 面试总结（亲历）
date: 2018-08-18 10:36:12
tags: [Android, 面试]
---
# 前言
在这个酷热的 8 月，笔者经历了一次找工作。整理一下遇到的一些面试题，由于记忆比较零散，有一些也忘记了，这里记录其中的两家的面试题，各家问的有重复的问题，就不一一列举了。

> 相关题目的答案会在后期逐渐完善

<!--more-->

# 面试
## 丁香园-丁香医生高级 Android 开发面经（2018.07.31）

- 自我介绍，之前项目介绍

- 离职原因

- kotlin语言

- kotlin 相比 Java 的优势

- kotlin 高阶函数

- kotlin 中的 when，when 在什么情况可以省略 else 分支，什么情况不可缺少？
when 取代了类 C 语言的 switch 操作符。when会对所有的分支进行检查直到有一个条件满足。when 可以用做表达式或声明。如果有分支可以用同样的方式处理的话，分支条件可以连在一起。when 也可以用来取代 if - else if 链。
如果把 when 做为表达式的话 else 分支是强制的，除非编译器可以证明分支条件已经覆盖所有可能性。

- kotlin 的 Anko 库，为什么推出？
Anko是JetBrains开发的一个强大的库。它主要的目的是用来替代以前XML的方式来使用代码生成UI布局。Anko包含了很多的非常有帮助的函数和属性来避免让你写很多的模版代码。通过查看Anko源码学习kotlin语言是一种不错的方法。Anko能帮助我们简化代码，比如，实例化Intent，Activity之间的跳转，Fragment的创建，数据库的访问，Alert的创建等等。

- kotlin 不用 findViewById，直接用控件 Id 做操作的内部原理
https://blog.csdn.net/hust_twj/article/details/80290362

- Retrofit 优点、涉及到的设计模式
概念：Retrofit是一个基于RESTful的HTTP网络请求框架的封装，其中网络请求的本质是由OKHttp完成的，而Retrofit仅仅负责网络请求接口的封装。
原理：App应用程序通过Retrofit请求网络，实际上是使用Retrofit接口层封装请求参数，Header、URL等信息，之后由OKHttp完成后续的请求，在服务器返回数据之后，OKHttp将原始的结果交给Retrofit，最后根据用户的需求对结果进行解析。
涉及到的设计模式：建造者模式、工厂方法模式、代理模式、外观模式、策略模式、单例模式、装饰器模式、适配器模式，责任链模式、装饰器模式的不同

- Retrofit 提供了哪些拦截器

- OkHttp 请求网络的原理（源码）
进行通信的原理主要是通过dispatcher不断从requestQueue中取出请求call，根据是否已经缓存调用Cache或NetWork这两类数据获取接口之一，从内存缓存或者服务器中获取请求的数据。分为同步和异步请求，同步请求通过call.execute()直接返回当前的response，而异步请求会将当前的call.enqueue添加到请求队列中，通过回调的方式来获取最后的结果。

- 哪些途径关注最新技术动态

- 组件化，以及你认为的组件化的应用场景

- 热更新

- AIDL（了解）
AIDL是一个缩写，全称是Android Interface Definition Language，也就是Android接口定义语言。
为了实现进程间通信，尤其是在涉及多进程并发情况下的进程间通信。


- GC算法（懵13，复习了四种引用，却略过了 GC 机制）
标记/清除算法
复制算法
标记/整理算法
分代收集算法

- View 事件分发，涉及到的设计模式（责任链模式）
当一个事件点击后，系统需要将这个事件传递给一个具体的View去处理，这个事件的传递过程就是分发过程。
事件分发的顺序
即 事件传递的顺序：Activity -> ViewGroup -> View
事件分发由三个方法协作完成：
    dispatchTouchEvent() :分发（传递）点击事件
    onInterceptTouchEvent():判断是否拦截了某个事件(只存在于ViewGroup；普通的View是没有该方法)
    onTouchEvent():处理点击事件
https://www.jianshu.com/p/38015afcdb58
https://blog.csdn.net/qq_30379689/article/details/73698192

- Handler 相关知识点，理解，为什么不能在子线程更新 UI 等

- RxJava，线程调度
subscribeOn 用于指定 subscribe() 时所发生的线程，从源码角度可以看出，内部线程调度是通过 ObservableSubscribeOn来实现的。
ObservableSubscribeOn 的核心源码在 subscribeActual 方法中，通过代理的方式使用 SubscribeOnObserver 包装 Observer 后，设置 Disposable 来将 subscribe 切换到 Scheduler 线程中。
observeOn 方法用于指定下游 Observer 回调发生的线程。

- EventBus

- 路由（讲了我用过的协议跳转；了解阿里的 arouter，大概说了下，可能说的不太对）

- Dagger2 依赖注入
一款基于Java注解来实现的完全在编译阶段完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。

- 内存优化，怎么分析

- 性能优化，一个原生 App 界面卡顿，可能的原因（掉帧）

- CPU、GPU、屏幕

- 熟悉 groovy 吗？写一个批量将...（忘了）。回答的是AS 的 gradle 配置使用的语言，写那个不会。。

- Java 多态

还有个别几个问题，忘了。

## 目前接受 Offer 的公司

### 1. 电面（半小时）
- ArrayList 和 LinkedList

- 数组和链表区别
https://blog.csdn.net/melody_day/article/details/53517550

- service保活

- 同一手机上安装不同版本的apk

- 组件化
（1）概念：
组件化：是将一个APP分成多个module，每个module都是一个组件，也可以是一个基础库供组件依赖，开发中可以单独调试部分组件，组件中不需要相互依赖但是可以相互调用，最终发布的时候所有组件以lib的形式被主APP工程依赖打包成一个apk。
（2）由来：
1、	APP版本迭代，新功能不断增加，业务变得复杂，维护成本高
2、	业务耦合度高，代码臃肿，团队内部多人协作开发困难
3、	Android编译代码卡顿，单一工程下代码耦合严重，修改一处需要重新编译打包，耗时耗力。
4、	方便单元测试，单独改一个业务模块，不需要着重关注其他模块。

- 路由

### 2. 笔试（半小时）
- 斐波那切数列（黄金分割数列）算法
```java
int fib (int n) {
    int a = 0;
    int b = 1;
    int c;
    for (int i = 2; i <= n; i++) {
        c = a + b;
        a = b;
        b = c;
    }
	return c;
}
```

- 熟悉的三种算法，用代码或语言描述其中一种
冒泡排序、快速排序、选择排序、插入排序等
冒泡排序：
```java
for (int i = 0; i < arr.length - 1; i++) { // 外层循环控制排序趟数
    for (int j = 0; j < arr.length - 1 - i; j++) { // 内层循环控制每一趟排序多少次
        if (arr[j] > arr[j + 1]) {
            int temp = arr[j];
            arr[j] = arr[j + 1];
            arr[j + 1] = temp;
        }
    }
}
```

- Activity 生命周期

- Activity 任务栈和使用方法

- 多渠道打包
https://blog.csdn.net/Android_Study_OK/article/details/75371940

- 用过的设计模式及其使用场景

### 3. 面试（一小时）
- 内存优化

- MVP
M（Model）
数据层，和MVC中的M一样，用来放数据的处理（比如网络请求，缓存等）。
V(View)
负责UI具体实现展现。比如Presenter派发过来一个动作是showDialog显示进度命令，那么我们这个View就负责实现具体UI。
P(Presenter)
负责处理业务逻辑代码，处理Model数据，然后将处理完的数据分发到View层。

- RxJava 操作符

- 大概介绍项目，亮点功能

- 数据库（项目中用到的）

- EventBus

- 加载一个运营活动页，怎么做

- Dagger2 依赖注入
一款基于Java注解来实现的完全在编译阶段完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。

### 4. Android 组长面（一小时）
- 自我介绍

- 项目介绍

- fragment切换

- 自定义view

- 6.0、7.0、8.0适配

- 项目中一些第三方库

- 某框架使用中的问题

- 事件分发

- 常用的设计模式

- 单例模式中的 volatile

- git 命令

- 离职原因

### 5.主管面（半小时）

- 自我介绍

- 离职原因

- 工作中职责

### 6. HR（微信）
- 谈薪资
- 入职相关事项
- offer

# 总结
最后，说几点个人看法。

- 平时的积累很重要，要善于积累。
- 开发有一定年限，一定需要懂得深层原理（源码）。
- 持续学习，保持求知欲。
- 不卑不亢，尽最大努力发挥自己了解的，不懂的地方虚心请教。

希望大家都会找到满意的工作。



