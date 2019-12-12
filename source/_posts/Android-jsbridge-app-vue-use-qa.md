---
title: JsBridge 使用和优化
date: 2019-09-13 20:07:55
tags:
- Android
- JSBridge
- Vue
---

现在的 App 开发中，结合 H5 和原生控件混合开发是比较常见的做法，可降低开发成本、解决 UI 适配等。然而在某些场景中，H5 需要与原生做一些数据交换和事件处理。虽然 WebView 提供了相关 Js 调用的方法，但在 Android 早期版本（4.2 版本前）中发现存在严重的漏洞，所以我们这里不再讨论，而是使用现在比较常用的一个第三方框架——[JsBridge](https://github.com/lzyzsd/JsBridge) 来实现。

<!--more-->

# JsBridge 的使用

JsBridge 的基本使用方法可查看作者的介绍，原生和 Js 的配置和使用方法介绍的比较清晰。Android 端可根据需要将 `com.github.lzyzsd.jsbridge.BridgeWebView` 做一个封装，初始化常用的 WebView 设置，以及 WebViewClient 和 WebChromeClient，打造你项目中通用的 WebView。另外，H5 前站现在普遍流行使用 Vue 框架，在接入 JsBridge 方法的时候出现了一些问题，在和前端同事一起探索后，实现了原生和 H5 的双向数据交互，并把 JsBridge 在 Vue 中的使用进行了封装。

# JsBridge 在 Vue 的封装与交互

## JsBridge 工具类

```js
/**
 * 用法：
 * import jsBridge from 'fileName.js'
 *
 * 1、给 App 端发送数据
 * jsBridge.callHandler(eventName, data, callback(reponseData))
 * 参数说明：
 * eventName (string): 必传, 与 App 端约定的事件名
 * data (object): 非必传, 发送给 App 端的数据
 * callback (function): 通信完成后，前端的回调，reponseData，是 App 端返回的数据
 *
 * 2、接受 App 端的数据
 * jsBridge.registerHandler(eventName, callback(data, responseCallback))
 * 参数说明：
 * eventName (string): 必传，与 App 端约定的事件名
 * callback (function): data: 是接受到的数据，responseCallback，通信完成后，传给 App 端的回调
 */

let isAndroid = navigator.userAgent.indexOf('Android') > -1 || navigator.userAgent.indexOf('Adr') > -1;
let isiOS = !!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);

//这是必须要写的，用来创建一些设置
function setupWebViewJavascriptBridge(callback) {
  if (isAndroid) {
    if (window.WebViewJavascriptBridge) {
      callback(WebViewJavascriptBridge)
    } else {
      document.addEventListener(
        'WebViewJavascriptBridgeReady',
        function () {
          callback(WebViewJavascriptBridge)
        },
        false
      );
    }
  }
  if (isiOS) {
    if (window.WebViewJavascriptBridge) {
      return callback(WebViewJavascriptBridge);
    }
    if (window.WVJBCallbacks) {
      return window.WVJBCallbacks.push(callback);
    }
    window.WVJBCallbacks = [callback];
    var WVJBIframe = document.createElement('iframe');
    WVJBIframe.style.display = 'none';
    WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
    document.documentElement.appendChild(WVJBIframe);
    setTimeout(function () {
      document.documentElement.removeChild(WVJBIframe)
    }, 0);
  }
}
setupWebViewJavascriptBridge(function (bridge) {
  if (isAndroid) {
    //安卓端，接受数据时，需要先进行初始化
    bridge.init(function (message, responseCallback) {
      var data = {
        'Javascript Responds': 'Wee!'
      };
      responseCallback(data);
    })
  }
})

export default {
  // 给 App 发送数据
  callHandler(name, data, callback) {
    setupWebViewJavascriptBridge(function (bridge) {
      bridge.callHandler(name, data, callback)
    })
  },
  // 接受 App 端的数据
  registerHandler(name, callback) {
    setupWebViewJavascriptBridge(function (bridge) {
      bridge.registerHandler(name, function (data, responseCallback) {
        callback(data, responseCallback)
      })
    })
  }
}

```

## 引入

```js
import jsBridge from './utils/JSbridge.js'
```

## 组件中使用
使用的方法名与 App 端约定好。

1、给 App 端发送数据
```js
jsBridge.callHandler(eventName, data, callback(reponseData))
```

> 参数说明：
> * eventName (string): 必传, 与 App 端约定的事件名
> * data (object): 非必传, 发送给 App 端的数据
> * callback (function): 通信完成后，前端的回调，reponseData，是 App 端返回的数据

2、接受 App 端的数据
```js
jsBridge.registerHandler(eventName, callback(data, responseCallback))
```

> 参数说明：
> * eventName (string): 必传，与 App 端约定的事件名
> * callback (function): data: 是接受到的数据，responseCallback，通信完成后，传给 App 端的回调


# 问题解决

## 找不到处理 `yy://__QUEUE_MESSAGE__/` 意图的 Activity

异常如下：

`android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.VIEW dat=yy://__QUEUE_MESSAGE__/ flg=0x10000000 }`

在 WebViewClient 的 shouldOverrideUrlLoading 中可以进行拦截 URL 跳转，实现在当前 WebView 加载网页链接，以及通过自定义 Uri 协议格式跳转页面（包括其他 App 的页面）。
Android 端出现此问题是你在这里做了跳转并返回了 true。翻阅 JsBridge 调用原理，其中一步是：Js 将消息内容放在 sendMessageQueue 中，并设置 iframe 的 src 为 `yy://__QUEUE_MESSAGE__/`。所以如果你像上面说的那样进行了处理，这里就会异常，App 端收不到消息。

解决方案是：
处理你已知的要进行加载或跳转的协议，如 Http/Https 协议、自定义 Uri 协议等，返回 true，其他的情况，请：`return super.shouldOverrideUrlLoading(view, url)`，交给系统处理（即：处理 JsBridge 的方法是这种情况）。


# 参考
- [JsBridge使用和原理](https://www.jianshu.com/p/910e058a1d63/)
- [Android JsBridge实战 打造专属你的Hybrid APP](https://www.jianshu.com/p/52071a3d07b4)
- [JSbridge 在Vue的封装与交互](https://www.jianshu.com/p/b03eaa6fb38a)

