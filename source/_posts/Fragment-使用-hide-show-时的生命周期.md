---
title: Fragment 使用 hide/show 时的生命周期
date: 2018-03-10 13:13:29
tags: [Android, Fragment]
---

当你第一眼看到这个标题的时候，肯定很惊讶。没错，我也觉得标题可能不规范，不过真的解决我在使用 Fragment 时遇到的坑，这里分享出来，供大家参考，一起交流。
<!--more-->

先来描述一下我的 APP 中使用到 Fragment 的功能和遇到的问题，第一个 Fragment 里是一个视频播放控件，希望在切换到其他 Fragment 的时候，暂停视频；再次回到这个 Fragment 时恢复播放。由于我使用的是 hide 和 show 的方式进行的 Fragment 之间的切换，每个 Fragment 只被初始化一次。那么问题来了，我尝试按照网上说的 Fragment 的生命周期，在 onPause 或 onStop 方法中暂停，在 onResume 中恢复播放，可是发现压根没走这些方法，onPause 等方法不会调用，onResume 只在刚进入是调用了一次，也没法做到让视频暂停。

话说回来，Fragment 的使用越来越普遍了，掌握它的生命周期以及注意事项是非常有必要的，首先

> All subclasses of Fragment must include a public empty constructor. The framework will often re-instantiate a fragment class when needed, in particular during state restore, and needs to be able to find this constructor to instantiate it.

也就是说每个继承 Fragment 的类都必须要有公开的构造方法，以便 fragment能在需要的时候还原原来的状态。感觉很难理解？通俗的说就是：SDK还原 fragment 数据的时候，肯定先通过调用 XXX.newInstance() 方法，获取到 fragment 的实例对象。这就是为什么要提供一个公开的构造方法的原因了！

其次，生命周期是必须了解的，这个就没必要废话了，网上搜下到处都是。

重点来了，跟我使用 Fragment 遇到的问题一样，这里再举一个 Fragment 使用时常遇到的情况，在一个 Activity 中通过菜单选项的点击来切换不同的Fragment，通常是需要保存 Fragment 的状态的，比如编辑个人信息模块时点击其他菜单，返回时你编辑的信息应该要保存的，而不是再次初始化。这时就该使用 Fragment 的 hide/show 方法了。
很快你就会发现 Fragment 的生命周期怎么不走了？

这时此方法 onHiddenChanged 派上用场了，当 Fragment 隐藏时，该方法会调用传入参数为true表示该 Fragment 被隐藏了，当 Fragment 调用了 show 方法后，该方法传入的参数为 false，表示该 Fragment 正在显示！

所以总结起来，如果使用 hide/show 方法来控制 Fragment 的使用时，原本需要在 onResume 以及 onPause 方法做的事情就可以迁移到 onHiddenChanged 时进行管理，如：
``` java
if (hidden) {// 不在最前端界面显示

} else {// 重新显示到最前端

}
```
这样就能完美实现当前 Fragment 在隐藏和显示时分别需要做的事了，我的 APP 视频暂停与播放的问题就解决了。
附上我的 APP 中使用的代码，这样就能理解了：
``` java
@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    if (hidden) {  //不在最前端界面显示
        mVideoView.pause();
    } else {  //重新显示到最前端
        mVideoView.start();
    }
}
```

关于`add()`, `show()`, `hide()`, `replace()`方法的正确使用，网上还有更多介绍，我这里简单做一个我遇到的问题以及解决方案的总结，分享给大家。

下面贴出完整的生命周期：

![img_fragment_llifecycle](img_fragment_llifecycle.png)
