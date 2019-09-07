---
title: Android 端使用 Mob 快速集成分享
date: 2019-09-06 23:42:08
tags:
- Android
- 第三方 SDK
---

本文记录 Android 端使用 Mob 快速集成分享的步骤。前半部分主要同官方的集成文档，后半部分是自己进行的封装，降低侵入性，以及更方便地使用。

相关链接：
- [ShareSDK 集成文档](http://wiki.mob.com/sdk-share-android-3-0-0/)

<!--more-->

# 集成准备

1. 注册应用申请 Mob 的 AppKey 和 AppSecret；

2. 需要申请第三方平台的 key，如微信、QQ 等；

> 注：使用 ShareSDK Gradle 集成方式，不需要在 AndroidMainfest.xml 下面配置任何权限和 Activity。

# 集成配置

1、打开项目根目录的 build.gradle，在 buildscrip–>dependencies 模块下面添加 classpath ‘com.mob.sdk:MobSDK:2018.0319.1724’，如下所示：

```groovy
buildscript {
    repositories {
        ...
    }
    dependencies {
        ...
        classpath "com.mob.sdk:MobSDK:2018.0319.1724"
    }
}
```

2、在使用到 Mob 产品的 module 下面的 build.gradle 文件里面添加引用

```groovy
apply plugin: 'com.mob.sdk'
```

3、然后添加 MobSDK 方法，配置 mob 的 key 和秘钥 （与第 2 步是一个 gradle 中；注意：MobSDK 方法是配置到文件根目录，与 android 并列，不要配置到 android 里面哦），Gradle 集成方式可以在 Mob 产品的 module 下面的 build.gradle 文件里面配置 ShareSDK 各个社交平台的 key 信息。

```groovy
// 社会化分享 ShareSdk 配置
MobSDK {
    appKey "申请 Mob 的 appkey"
    appSecret "申请 Mob 的 AppSecret"
    ShareSDK {
        devInfo {
            Wechat {
                id "0"
                sortId "0"
                appId "申请 Wechat 的 appkey"
                appSecret "申请 Wechat 的 AppSecret"
                bypassApproval "false"
                enable "true"
            }
            WechatMoments {
                id "1"
                sortId "1"
                appId "申请 Wechat 的 appkey"
                appSecret "申请 Wechat 的 AppSecret"
                bypassApproval "false"
                enable "true"
            }
            QQ {
                id "2"
                sortId "2"
                appId "appId"
                appKey "appKey"
                shareByAppClient "true"
                enable "true"
            }
            QZone {
                id "3"
                sortId "3"
                appId "appId"
                appKey "appKey"
                shareByAppClient "false"
                enable "true"
            }
        }
    }
}
```
> 配置字段请查看官方说明。

# 添加代码

1.初始化 MobSDK

如果您没有在`AndroidManifest`中设置`appliaction`的类名，MobSDK 会将这个设置为`com.mob.MobApplication`，但如果您设置了，请在您自己的 Application 类中调用：`MobSDK.init(this);`以初始化 MobSDK。

并且在Manifest清单文件中配置：tools:replace="android:name"，如下所示：
```xml
<application
   android:name = ".MyApplication"
   tools:replace="android:name">
```
2、分享代码调用
添加配置后，即可调用授权、获取资料、分享等操作，如一键分享的代码：

```java
//java
private void showShare() {
    OnekeyShare oks = new OnekeyShare();
    // title标题，微信、QQ和QQ空间等平台使用
    oks.setTitle(getString(R.string.share));
    // titleUrl QQ和QQ空间跳转链接
    oks.setTitleUrl("http://sharesdk.cn");
    // text是分享文本，所有平台都需要这个字段
    oks.setText("我是分享文本");
    // imagePath是图片的本地路径，确保SDcard下面存在此张图片
    oks.setImagePath("/sdcard/test.jpg");
    // url在微信、Facebook等平台中使用
    oks.setUrl("http://sharesdk.cn");
    // 启动分享GUI
    oks.show(this);
}
```

到此就基本集成完成了，即可实现分享了。不过我们还可以做一些封装，往下看。

# 通用分享工具的封装
如果你的应用在某一阶段已知并确定做分享某一种类型，如分享超链接，则可以将分享功能做一个进一步封装。

下面以常用的 **分享超链接** 为例，做一个通用分享工具的封装，方便更方便地调用。

## 分享内容实体

```java
public class ShareContent {

    private String title;
    private String titleUrl;
    private String text;
    private String imgUrl;
    private String url;

    public ShareContent(String title, String titleUrl, String text, String imgUrl, String url) {
        this.title = title;
        this.titleUrl = titleUrl;
        this.text = text;
        this.imgUrl = imgUrl;
        this.url = url;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getTitleUrl() {
        return titleUrl;
    }

    public void setTitleUrl(String titleUrl) {
        this.titleUrl = titleUrl;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public String getImgUrl() {
        return imgUrl;
    }

    public void setImgUrl(String imgUrl) {
        this.imgUrl = imgUrl;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public static class Builder {
        String title;
        String titleUrl;
        String text;
        String imgUrl;
        String url;

        public Builder() {
        }

        /**
         * @param title 标题
         */
        public Builder setTitle(String title) {
            this.title = title;
            return this;
        }

        /**
         * @param titleUrl 放在文本里面的url
         */
        public Builder setTitleUrl(String titleUrl) {
            this.titleUrl = titleUrl;
            return this;
        }

        /**
         * @param text 文本内容
         */
        public Builder setText(String text) {
            this.text = text;
            return this;
        }

        /**
         * @param imgUrl 分享图片的网络图片地址，因为目前shareSdk不支持Drawable中的图片分享，所以先都统一用网络图片
         */
        public Builder setImgUrl(String imgUrl) {
            this.imgUrl = imgUrl;
            return this;
        }

        /**
         * @param url 分享的链接
         */
        public Builder setUrl(String url) {
            this.url = url;
            return this;
        }

        public ShareContent build() {
            return new ShareContent(title, titleUrl, text, imgUrl, url);
        }
    }

}
```

## 分享接口
提供分享内容
> 需要分享功能的实体实现此接口。

```java
public interface IShareable {

    ShareContent getShareContent();

}
```

## 分享器接口

```java
public interface IShare {

    void share(Context context, IShareable shareable);

    void share(Context context, IShareable shareable, PlatformActionListener callback);

    void share(Context context, ShareContent content);

    void share(Context context, final ShareContent content, PlatformActionListener callback);

    void share(Context context, final ShareContent content, String platform, PlatformActionListener callback);
}
```

## 通用分享工具类

```java
public class CommonShare implements IShare {

    @Override
    public void share(Context context, IShareable shareable) {
        share(context, shareable, null);
    }

    @Override
    public void share(Context context, IShareable shareable, PlatformActionListener callback) {
        ShareContent content = shareable.getShareContent();
        if (content != null) {
            share(context, content, callback);
        } else {
            Timber.e("The shareable return a null object");
        }
    }

    @Override
    public void share(Context context, final ShareContent content) {
        share(context, content, null);
    }

    @Override
    public void share(Context context, final ShareContent content, PlatformActionListener callback) {
        share(context, content, null, callback);
    }

    @Override
    public void share(Context context, final ShareContent content, String platform, PlatformActionListener callback) {
        if (content == null) {
            Timber.e("The shareContent is null");
            return;
        }

        OnekeyShare oks = new OnekeyShare();
        oks.disableSSOWhenAuthorize();
        if (platform != null) {
            oks.setPlatform(platform);
        }

        // title标题，微信、QQ和QQ空间等平台使用
        oks.setTitle(content.getTitle());
        // titleUrl QQ和QQ空间跳转链接
        oks.setTitleUrl(content.getTitleUrl());
        // text是分享文本，所有平台都需要这个字段
        oks.setText(content.getText());
        // 各平台都会用到
        oks.setImageUrl(content.getImgUrl());
        // url在微信、Facebook等平台中使用
        oks.setUrl(content.getUrl());
        if (callback != null) {
            oks.setCallback(callback);
        }

        oks.setShareContentCustomizeCallback((platform1, paramsToShare) -> {
            if ("Wechat".equals(platform1.getName()) || "WechatMoments".equals(platform1.getName())) {
                paramsToShare.setShareType(Platform.SHARE_WEBPAGE);//分享网页，既图文分享
            } else if ("SinaWeibo".equals(platform1.getName())) {
                paramsToShare.setText(content.getText() + " " + content.getTitleUrl());
            }
        });

        // 设置是否是直接分享
        oks.setSilent(false);
        // 启动分享GUI
        oks.show(context);
    }

}
```

## 使用

### 实例化 `ShareContent`
1、直接创建其对象
```java
ShareContent shareContent = new ShareContent.Builder()
        .setTitle(title)
        .setTitleUrl(url)
        .setText(text)
        .setImgUrl(imageUrl)
        .setUrl(url)
        .build();
```

2、需要分享功能的实体实现 `IShareable`，提供分享内容，如商品详情实体类等。

### 调用通用分享工具

```java
new CommonShare().share(context, shareContent);
```

这样，一行代码就实现了分享器的调用，是不是很简单，重点是不需要直接去操作 SDK 的调用方法，降低侵入性。

## 问题排查与解决
如果你按照文档接入了 SDK，运行程序，成功实现各个平台的分享，那么恭喜你，成功了。

但是，如果在调试中发现分享失败，请检查集成步骤，参考官方问题说明进行解决。

大多数分享的错误都出现参数上，可以将 Mob 的这些参数放在你的代码中测试：
```java
oks.setTitle("分享标题--Title");
oks.setTitleUrl("http://mob.com");
oks.setText("分享测试文--Text");
oks.setImageUrl("http://f1.sharesdk.cn/imgs/2014/02/26/owWpLZo_638x960.jpg");
```

## 总结

Mob 的分享 SDK 的集成还是很简单的，开发的很多 App 中都使用到了，接入很简单，官方给的文档也很清晰（主要还是集成步骤少，很多它都帮我们做了，无需再配置权限、注册回调页面等）。

后面做的封装其实也是很基本的东西，但把这些 **第三方库提供的方法包装一层** 再使用是一个好习惯！写的多了，你就懂了~

