---
title: Android 开发中常见的图片选择、裁剪及权限管理等第三方功能库的升级优化
date: 2018-05-22 16:38:37
tags: [Android, album, permission]
---
# 1. 前言
近期开发中有个需求涉及到多张图片选择、裁剪及上传功能，原项目中有类似的库和第三方框架，花了很长时间把相关的大部分功能加入进来，并未完全实现需求，然而最后发现有问题，查找解决方法，发现该开源的 Issues 中很多人都提到了我遇到的几个问题，尚未解决，该库也停止维护；现有的第三方框架的裁剪样式也不符合 App 要求。在一系列问题寻求解决方案无果后，决定弃用以前使用的框架，使用实现此功能更优秀的框架。
后台同事有其他的工作，这块需求的 API 尚未开发，有时间留给我们做优（tian）化（keng），在跟另一位 Android 同事提议和沟通后，决定把图片选择、图片裁剪、权限管理的第三方工具进行优化和升级，一方面是为此项目做优化，避免以后问题越来越多；另一方面作为学习和知识储备，选择和会使用更加优秀的轮子。

<!--more-->

# 2. 升级优化
## 2.1 图片选择
严大的库：[Album](https://github.com/yanzhenjie/Album)

### 2.1.1 Gradle
`'com.yanzhenjie:album:2.1.1'`

### 2.1.2 简单使用
#### 2.1.2.1 初始化
```java
Album.initialize(AlbumConfig.newBuilder(this)
        .setAlbumLoader(MediaLoader())
        .setLocale(Locale.getDefault())
        .build()
```

#### 2.1.2.2 图片加载器
```java
public class MediaLoader implements AlbumLoader {

    @Override
    public void load(ImageView imageView, AlbumFile albumFile) {
        load(imageView, albumFile.getPath());
    }

    @Override
    public void load(ImageView imageView, String url) {
        Glide.with(imageView.getContext())
                .load(url)
                .into(imageView);
    }
}
```

#### 2.1.2.3 调用
```java
private fun selectImage() {
    Album.image(this)
            .singleChoice()
            .camera(true)
            .columnCount(2)
            .widget(
                    Widget.newDarkBuilder(this)
                            .title(R.string.choose_album)
                            .statusBarColor(ContextCompat.getColor(this, R.color.albumColorPrimaryBlack))
                            .toolBarColor(ContextCompat.getColor(this, R.color.albumColorPrimaryBlack))
                            .build()
            )
            .onResult { result ->
                // do something
            }
            .start()
}
```

`onResult` 中返回图片相关的信息，根据需要做处理。上述示例是 Album 的简单使用，也是我在项目中做头像选择时的代码，简单吧！Album 不仅可以选择图片，还可以选择视频（单个或多个）、拍摄照片、拍摄视频、画廊等，支持在 Activity、Fragment 中调用，`.with()` 中传 **this**，即当前 Activity 或 Fragment。

## 2.2 图片裁剪
还是严大的库：[Durban](https://github.com/yanzhenjie/Durban)
### 2.2.1 Gradle
`'com.yanzhenjie:durban:1.0.1'`

### 2.2.2 简单使用
```java
private fun cropImage(imagePathList: String) {
    val cropDirectory = Constants.sdPath(this) + Constants.APP_CROP_PATH
    val file = File(cropDirectory)
    // 判断文件目录是否存在
    if (!file.exists()) {
        file.mkdirs()
    }

    Durban.with(this)
            .statusBarColor(ContextCompat.getColor(this, R.color.albumColorPrimaryBlack))
            .toolBarColor(ContextCompat.getColor(this, R.color.albumColorPrimaryBlack))
            .navigationBarColor(ContextCompat.getColor(this, R.color.albumColorPrimaryBlack))
            // Image path list/array.
            .inputImagePaths(imagePathList)
            // Image output directory.
            .outputDirectory(cropDirectory)
            // Image size limit.
            .maxWidthHeight(1000, 1000)
            // Aspect ratio.
            .aspectRatio(1f, 1f)
            // Output format: JPEG, PNG.
            .compressFormat(Durban.COMPRESS_JPEG)
            // Compress quality, see Bitmap#compress(Bitmap.CompressFormat, int, OutputStream)
            .compressQuality(90)
            // Gesture: ROTATE, SCALE, ALL, NONE.
            .gesture(Durban.GESTURE_ALL)
            .controller(Controller.newBuilder()
                    .enable(false)
                    .rotation(true)
                    .rotationTitle(true)
                    .scale(true)
                    .scaleTitle(true)
                    .build())
            .requestCode(REQUEST_CODE_SELECT)
            .start()
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (data != null && requestCode == REQUEST_CODE_SELECT) {
        val images = Durban.parseResult(data)
        // accept the cropping results
    }
}

companion object {
    const val REQUEST_CODE_SELECT = 123
}
```

在 `onActivityResult` 回调中接收裁剪后的结果，inputImagePaths()方法的参数传入了一张图片的路径，还可以是路径 list，也可以是路径数组，同样支持在 Activity、Fragment 中调用。

## 2.3 权限管理
又是严大的库：[AndPermission](https://github.com/yanzhenjie/AndPermission)
之前使用的是 1.x 的版本，这次升级到 2.x，变动有点大，但变得功能健壮和使用简洁了，还是决定有必要升级一下。支持在 Activity、Fragment 中调用，提供用户拒绝权限后再次申请权限的处理。

### 2.3.1 Gradle
`'com.yanzhenjie:permission:2.0.0-rc5'`

### 2.3.2 简单使用
```java
private fun requestPermission(vararg permission: String) {
    AndPermission.with(this)
            .runtime()
            .permission(*permission)
            .rationale(RuntimeRationale())
            .onGranted {
                // success, do something
            }
            .onDenied { permissions ->
                if (AndPermission.hasAlwaysDeniedPermission(this, permissions)) {
                    PermissionHandleUtil.showSettingDialog(this, permissions)
                }
            }
            .start()
}

// 调用
requestPermission(*Permission.Group.STORAGE, Permission.CAMERA)
```

`requestPermission()` 中传入需要的权限，是可变参数。Kotlin 可同时传入已有字符串和字符串数组，字符串数组前加 `*`，就是这么6（关于语法，此处不做详解）。
`RuntimeRationale` 和 `PermissionHandleUtil` 中包含权限处理的提示 dialog，我将其封装成了工具类，在作者的 simple 中可以找到，此处不再列出。

# 3. 总结
结合以上三个开源库，就顺利完成了权限管理、图片选择、裁剪的整个流程。以上只是我在某个功能上用到的，每个框架都有更多可选的配置，满足很多需求，方便高效。

又安利了一波严大的开源库，哈哈哈！

- 学习并实践优秀开源库，避免重复造轮子
- 选择成熟的开源项目，降低风险