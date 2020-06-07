---
title: 使用 ARouter 实现登录拦截
date: 2020-06-05 15:01:49
tags: [Android, ARouter]
---

# 概述

使用 ARouter 路由框架优雅地实现登录拦截功能。

<!--more-->

# 前言

一个应用中有许多页面，有些页面是无需登录的（游客模式），有些页面是需要登录才能看的，当我们进行页面跳转时会先判断用户是否登录。如果已经登录，则正常跳转；如果没有登录，则跳转到登录页面先登录。这样的操作，大家应该都很熟悉吧。一般情况下，我们的逻辑是这样的：

```java
if (TextUtils.isEmpty(token)) {  // 还没登录，先跳转登录页面
    Intent intent = new Intent(this, LoginActivity.class);
    startActivity(intent);
} else {  // 已经登录了，跳转订单页面
    Intent intent = new Intent(this, OrderActivity.class);
    startActivity(intent);
}
```

上面的做法需要在每一个目标页面重复做登录检查，这样设计的扩展性并不友好。在这里介绍阿里的 ARouter 路由框架，可以更加优雅地实现登录拦截功能。

# 使用方法

## SDK 初始化

SDK 集成步骤：[ARouter](https://github.com/alibaba/ARouter)。

## 拦截器的使用

### 登录拦截器

```java
/**
 * 在跳转过程中处理登录事件，这样就不需要在目标页重复做登录检查
 * 拦截器会在跳转之间执行，多个拦截器会按优先级顺序依次执行
 *
 * @author lishide
 * @date 2020/5/30
 */
@Interceptor(name = "login", priority = 3)
public class LoginInterceptorImpl implements IInterceptor {

    @Override
    public void process(Postcard postcard, InterceptorCallback callback) {
        String path = postcard.getPath();
        if (isNotLogin()) {
            // 如果没有登录
            switch (path) {
                // 不需要登录的直接进入这个页面
                case RouterHub.PATH_LOGIN:
                case RouterHub.PATH_MAIN:
                    callback.onContinue(postcard);
                    break;
                // 需要登录的直接拦截下来
                default:
                    callback.onInterrupt(null);
                    break;
            }
        } else {
            // 如果已经登录不拦截
            callback.onContinue(postcard);
        }

    }

    @Override
    public void init(Context context) {
        //此方法只会走一次
    }

    private boolean isNotLogin() {
        return <判断是否登录>;
    }

}

```

### 路由跳转的回调

```java
/**
 * 路由跳转的回调
 *
 * @author lishide
 * @date 2020/5/30
 */
public class LoginNavigationCallbackImpl implements NavigationCallback {

    /**
     * 找到了
     */
    @Override
    public void onFound(Postcard postcard) {

    }

    /**
     * 找不到了
     */
    @Override
    public void onLost(Postcard postcard) {

    }

    /**
     * 跳转成功了
     */
    @Override
    public void onArrival(Postcard postcard) {

    }

    @Override
    public void onInterrupt(Postcard postcard) {
        String path = postcard.getPath();
        Bundle bundle = postcard.getExtras();
        // 被登录拦截了下来了
        // 需要调转到登录页面，把参数跟被登录拦截下来的路径传递给登录页面，登录成功后再进行跳转被拦截的页面
        ARouter.getInstance().build(RouterHub.PATH_LOGIN)
                .with(bundle)
                .withString(RouterHub.PATH, path)
                .navigation();
    }
}
```

### 启动 Activity

```java
ARouter.getInstance().build(RoutePath.PATH_SECOND)
        .withString("msg", "ARouter 传递过来的需要登录的参数 msg")
        .navigation(this, new LoginNavigationCallbackImpl());
```

这样就能实现页面的登录拦截了。

