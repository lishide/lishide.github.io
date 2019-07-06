---
title: 解决 Android 的 webview 不支持 input type=file 标签
date: 2019-06-15 12:23:32
tags: [Android, WebView]
---
# 解决 Android 的 webview 不支持 `input type=file` 标签

项目中遇到 H5 的 `input type="file"` 标签在 Android 的 webview 中失效，查了一下是由于安全原因将其屏蔽了。重写 webview 的 WebChromeClient 可以解决。

<!--more-->

## WebView 设置 WebChromeClient

重写 WebChromeClient 中关于文件选择的方法，onShowFileChooser 和 openFileChooser。

```java
mWebView.setWebChromeClient(new WebChromeClient() {

    // For 3.0+ Devices (Start)
    // onActivityResult attached before constructor
    protected void openFileChooser(ValueCallback uploadMsg, String acceptType) {
        mUploadMessage = uploadMsg;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/*");
        startActivityForResult(Intent.createChooser(i, "File Browser"), FILE_CHOOSER_RESULT_CODE);
    }

    // For Lollipop 5.0+ Devices
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    @Override
    public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
        if (uploadMessage != null) {
            uploadMessage.onReceiveValue(null);
            uploadMessage = null;
        }
        uploadMessage = filePathCallback;
        Intent intent = fileChooserParams.createIntent();
        try {
            startActivityForResult(intent, REQUEST_SELECT_FILE);
        } catch (ActivityNotFoundException e) {
            uploadMessage = null;
            Toast.makeText(getContext(), "Cannot Open File Chooser", Toast.LENGTH_LONG).show();
            return false;
        }
        return true;
    }

    //For Android 4.1 only
    protected void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
        mUploadMessage = uploadMsg;
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        intent.setType("image/*");
        startActivityForResult(Intent.createChooser(intent, "File Browser"), FILE_CHOOSER_RESULT_CODE);
    }

    protected void openFileChooser(ValueCallback<Uri> uploadMsg) {
        mUploadMessage = uploadMsg;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/*");
        startActivityForResult(Intent.createChooser(i, "File Chooser"), FILE_CHOOSER_RESULT_CODE);
    }

});
```



## 选择结果的回调

在 onActivityResult 中获取对应的选取文件的返回结果

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        if (requestCode == REQUEST_SELECT_FILE) {
            if (uploadMessage == null) {
                return;
            }
            uploadMessage.onReceiveValue(WebChromeClient.FileChooserParams.parseResult(resultCode, data));
            uploadMessage = null;
        }
    } else if (requestCode == FILE_CHOOSER_RESULT_CODE) {
        if (null == mUploadMessage) {
            return;
        }
        // Use MainActivity.RESULT_OK if you're implementing WebView inside Fragment
        // Use RESULT_OK only if you're implementing WebView inside an Activity
        Uri result = data == null || resultCode != WebActivity.RESULT_OK ? null : data.getData();
        mUploadMessage.onReceiveValue(result);
        mUploadMessage = null;
    } else {
        Toast.makeText(getContext(), "选择图片失败", Toast.LENGTH_LONG).show();
    }

}
```

可以解决 3.1、4.1、5.0 以上的版本的问题。
