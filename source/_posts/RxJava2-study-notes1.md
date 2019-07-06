---
title: RxJava2 学习
date: 2019-07-05 19:54:19
tags: [RxJava2]
---

RxJava2 学习笔记。
<!--more-->

# 1. 概述

RxJava —— 一个在 Java VM 上使用可观察序列来组成异步的、基于事件的程序库。

# 2. 知识点

- 资源 1：[简书——RxJava2.x 专题](https://www.jianshu.com/c/299d0a51fdd4)
- 资源 2：[RxJava2 基本概念大全](https://www.e-learn.cn/content/java/1947154)

## 2.1 RxJava2 教程（一）

### RxJava2 基础

- 观察者模式的四大要素
	1. `Observable` 被观察者
	2. `Observer` 观察者
	3. `subscribe` 订阅
	4. 事件

> 只有当上游和下游建立连接之后，上游才会开始发送事件。也就是调用了`subscribe()` 方法之后才开始发送事件。

- ObservableEmitter
Emitter 是发射器的意思，就是用来发出事件的，它可以发出三种类型的事件：

	- `onNext(T value)` 发出 next 事件
	- `onComplete()` 发出 complete 事件
	- `onError(Throwable error)` 发出 error 事件


- Disposable

	- 一次性，用完即可丢弃的
	- 用它来切断 Observer（观察者）与 Observable（被观察者）之间的连接，当调用它的`dispose()`方法时，它就会将 Observer（观察者）与 Observable（被观察者）之间的连接切断，从而导致 Observer（观察者）收不到事件。
	- 把它理解成两根管道之间的一个机关，当调用它的`dispose()`方法时，它就会将两根管道切断，从而导致下游收不到事件。

## 2.2 RxJava2 教程（二）

### 线程调度
#### 调度方法

- `subscribeOn()` 指定的是上游发送事件的线程（调度被观察者运行的线程）

- `observeOn()` 指定的是下游接收事件的线程（调度观察者运行的线程）

多次指定上游的线程只有第一次指定的有效，也就是说多次调用`subscribeOn()` 只有第一次的有效，其余的会被忽略。

多次指定下游的线程是可以的，也就是说每调用一次`observeOn()`，下游的线程就会切换一次。

#### 线程选项（结合 RxAndroid）

- `Schedulers.io()` 代表 io 操作的线程，执行耗时操作。通常用于网络、读写文件等 io 密集型的操作
- `Schedulers.computation()` 代表 CPU 计算密集型的操作，例如需要大量计算的操作
- `Schedulers.newThread()` 代表一个常规的新线程
- `AndroidSchedulers.mainThread()`  代表 Android 的主线程

## 2.3 RxJava2 教程（三）

### 变换操作符

- map

  `map` 是 RxJava 中最简单的一个变换操作符了，它的作用就是对上游发送的每一个事件应用一个函数，使得每一个事件都按照指定的函数去变化。

- flatMap

  `flatMap` 将一个发送事件的上游 Observable 变换为多个发送事件的 Observables，然后将它们发射的事件合并后放进一个单独的 Observable 里。

- concatMap

  `concatMap` 和 flatMap 的作用几乎一样，只是它的结果是严格按照上游发送的顺序来发送的（保证顺序）。

### 实践

需求：如果是一个新用户，必须先注册，等注册成功之后再自动登录该怎么做呢？

分析：这是一个嵌套的网络请求，首先需要去请求注册，待注册成功回调了再去请求登录的接口。

解决：优雅的解决嵌套请求，用 `flatMap` 转换

``` java
public interface Api {
    @GET
    Observable<LoginResponse> login(@Body LoginRequest request);

    @GET
    Observable<RegisterResponse> register(@Body RegisterRequest request);
}
```

注册和登录返回的都是 Observable，使用 `flatMap` 操作符转换为另一个 Observable：

``` java
api.register(new RegisterRequest())            //发起注册请求
    .subscribeOn(Schedulers.io())               //在IO线程进行网络请求
    .observeOn(AndroidSchedulers.mainThread())  //回到主线程去处理请求注册结果
    .doOnNext(new Consumer<RegisterResponse>() {
        @Override
        public void accept(RegisterResponse registerResponse) throws Exception {
          //先根据注册的响应结果去做一些操作
        }
    })
    .observeOn(Schedulers.io())                 //回到IO线程去发起登录请求
    .flatMap(new Function<RegisterResponse, ObservableSource<LoginResponse>>() {
        @Override
        public ObservableSource<LoginResponse> apply(RegisterResponse registerResponse) throws Exception {
          return api.login(new LoginRequest());
        }
    })
    .observeOn(AndroidSchedulers.mainThread())  //回到主线程去处理请求登录的结果
    .subscribe(new Consumer<LoginResponse>() {
        @Override
        public void accept(LoginResponse loginResponse) throws Exception {
          Toast.makeText(MainActivity.this, "登录成功", Toast.LENGTH_SHORT).show();
        }
    }, new Consumer<Throwable>() {
        @Override
        public void accept(Throwable throwable) throws Exception {
          Toast.makeText(MainActivity.this, "登录失败", Toast.LENGTH_SHORT).show();
        }
    });
```

## 2.4 RxJava2 教程（四）

### zip

## 2.5 RxJava2 教程（五）

Backpressure；上下游流速不均衡的源头，线程同步/异步情况

## 2.6 RxJava2 教程（六）

解决上下游流速不均衡的问题（使用 `Observable`）：

- 一是从数量上进行治理, 减少发送进水缸里的事件

  - `.sample(2, TimeUnit.SECONDS);` // 进行 sample 采样


- 二是从速度上进行治理, 减缓事件发送进水缸的速度

  - `Thread.sleep(2000);` // 发送事件之后延时 2 秒


## 2.7 RxJava2 教程（七）

### Flowable

## 2.8 RxJava2 教程（八）

Flowable；

BackpressureStrategy

## 2.9 RxJava2 教程（九）

Flowable，如何正确的去实现一个完整的响应式拉取

## 2.10 RxJava2 教程（十）

RxJavaPlugins、Retrofit 自定义异常处理
