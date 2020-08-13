---
title: Android：Utils 规范&路由导航工具
date: 2020-06-19 14:11:37
tags: [Android, Util, ARouter]
---

# Utils 规范

1. 使用 Kotlin
   为什么是 Kotlin？因为 Kotlin 方便扩展某一类 util（使用扩展函数）。
   - 强制：必须注释。
   - 建议：放在基础层的 CommonSDK 里。

2. 第三方库提供了单例，可以直接调用时，**一定不要直接用**。 将其调用方法再包一层，降低侵入性。如需更换其他同类框架，在调用函数方面影响小一点（但不是没有影响，因为有些地方还是要改，只是让你改的地方少一点）。

  以 ARouter 为例，ARouter 提供了 `ARouter.getInstance()...` 的用法，请不要偷懒，要自己再包一层。

<!--more-->

## 使用单例模式

可以使用 Kotlin 的 `object` 很容易地声明单例模式。

```kotlin
object Singleton {
    ...
}

// Kotlin 中调用
Singleton.xx()

// Java 中调用
Singleton.INSTANCE.xx()
```

该类将永远只有一个实例，并且该实例（以线程安全的方式首次访问它时创建）具有与该类相同的名称。[文档地址](https://kotlinlang.org/docs/tutorials/kotlin-for-py/objects-and-companion-objects.html)

这种方式和 Java 单例模式的饿汉式一样，不过比 Java 中的实现代码量少很多，其实是个语法糖。

有人说，Utils 推荐使用懒汉式，在使用时再初始化。但是，根据官方文档的意思，`object` 的单例模式是以线程安全的方式首次访问它时创建的，所以这里是可以用 `object` 的单例模式来声明的。

## 工具方法

一般地，都会写多个**静态方法**来实现不同特性的功能。但是在封装一些第三方库，采用这样的方式真的好吗？别人写的那么好的框架，可使用 Builder 来链式调用，却被装在了固定参数的静态方法里。。。

# 实践：导航路由器

下面就结合这几点对路由导航工具类进行一个升级优化。我们可以运用 **Builder 模式**，使用提供给此构建器的参数创建一个**自己的**导航路由器。

项目中使用 ARouter 进行路由导航，我们可以按照类似参数声明创建自己的构建器。

```kotlin
import android.app.Activity
import android.content.Context
import android.os.Bundle
import androidx.annotation.AnimRes
import com.alibaba.android.arouter.facade.Postcard
import com.alibaba.android.arouter.launcher.ARouter

/**
 * Navigation to the route with path.
 * <p>
 * Creates a navigation router with the arguments supplied to this builder.
 * <p>
 * There will only ever be one instance of this class,
 * and the instance (which is created the first time it is accessed,
 * in a thread-safe manner) has got the same name as the class.
 * <p>
 * Used in Kotlin:
 * Nav.setPath("").setContext(activity).go()
 * <p>
 * Used in Java:
 * Nav.INSTANCE.setPath("").setContext(activity).go();
 *
 * @author lishide
 * @date 2020/6/18
 */
object Nav {

    private var path: String? = null
    private var mContext: Context? = null
    private var bundle: Bundle? = null
    private var flag = -1
    private var enterAnim = -1
    private var exitAnim = -1
    private var requestCode = -1

    /**
     * Set the path.
     *
     * @param path Where you go.
     * @return This Builder object to allow for chaining of calls to set methods.
     * @see ARouter.build(String)
     */
    fun setPath(path: String): Nav {
        this.path = path
        return this
    }

    /**
     * Set the context.
     *
     * @param context Activity and so on.
     * @return This Builder object to allow for chaining of calls to set methods.
     */
    fun setContext(context: Context?): Nav {
        mContext = context
        return this
    }

    /**
     * Set the bundle.
     * <p>
     * BE ATTENTION TO THIS METHOD WAS <P>SET, NOT ADD!</P>
     *
     * @param bundle bundle
     * @return This Builder object to allow for chaining of calls to set methods.
     * @see Postcard.with(Bundle)
     */
    fun setBundle(bundle: Bundle?): Nav {
        this.bundle = bundle
        return this
    }

    /**
     * Set special flags controlling how this intent is handled.
     *
     * @param flag Flags of route
     * @return This Builder object to allow for chaining of calls to set methods.
     * @see Postcard.withFlags
     */
    fun setFlag(flag: Int): Nav {
        this.flag = flag
        return this
    }

    /**
     * Set the requestCode.
     *
     * @param requestCode requestCode
     * @return This Builder object to allow for chaining of calls to set methods.
     */
    fun setRequestCode(requestCode: Int): Nav {
        this.requestCode = requestCode
        return this
    }

    /**
     * Set normal transition anim.
     *
     * @param enterAnim enter
     * @param exitAnim  exit
     * @return This Builder object to allow for chaining of calls to set methods.
     * @see Postcard.withTransition
     */
    fun setTransition(@AnimRes enterAnim: Int, @AnimRes exitAnim: Int): Nav {
        this.enterAnim = enterAnim
        this.exitAnim = exitAnim
        return this
    }

    /**
     * Creates an {@link ARouter} with the arguments supplied to this builder.
     * <p>
     * Calling this method navigation to the route with path in postcard.
     */
    fun go() {
        if (path.isNullOrEmpty()) {
            println("path can not be empty")
            return
        }

        val postcard = ARouter.getInstance().build(path)
        if (null != bundle) {
            postcard.with(bundle)
        }
        if (flag != -1) {
            postcard.withFlags(flag)
        }
        if (enterAnim != -1 && exitAnim != -1) {
            postcard.withTransition(enterAnim, exitAnim)
        }
        if (null == mContext) {
            postcard.navigation()
        } else {
            if (requestCode != -1 && mContext is Activity) {
                postcard.navigation(
                    mContext as Activity?, requestCode, LoginNavigationCallbackImpl()
                )
            } else {
                postcard.navigation(mContext, LoginNavigationCallbackImpl())
            }
        }
    }

}
```

- Used in Kotlin:
  `Nav.setPath("").setContext(activity).go()`

- Used in Java:
  `Nav.INSTANCE.setPath("").setContext(activity).go();`

> 可以在方法上加上 `@JvmStatic` 注解，在 Java 中使用就和 Kotlin 中一样了。感觉没必要，就多写个 `.INSTANCE`。

这样就将导航路由器封装好了，最终使用了 ARouter 的导航。如果以后要更换其他的路由框架，只修改最后的一个方法就行了，是不是比以前写若干个静态方法要更方便呢。

仅设置了常用的一些方法，更多的方法在使用到的时候追加即可。

> LoginNavigationCallbackImpl 是一个登录拦截器路由跳转的回调实现，具体功能实现见：[使用 ARouter 实现登录拦截](https://lishide.github.io/2020/06/05/Android-ARouter-Login-Interceptor/)，如不需要，可直接去掉此参数。

类似地，可以给图片加载器 Glide，事件总线 EventBus、MMKV 等 key-value 库等套一层。

