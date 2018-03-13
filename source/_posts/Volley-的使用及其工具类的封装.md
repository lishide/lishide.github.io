---
title: Volley 的使用及其工具类的封装
date: 2018-03-13 12:53:34
tags:
- Android
- Volley
---
【来自本人2016年简书文章，将其同步至个人博客】

	August 26, 2016 1:10 PM Power By lishide
## 一. Volley 简介

> Volley 的中文翻译为“齐射、并发”，是在 2013 年的 Google 大会上发布的一款 Android 平台网络通信库，具有网络请求的处理、小图片的异步加载和缓存等功能，能够帮助 Android App 更方便地执行网络操作，而且更快速高效。
    在 Google IO 的演讲上，其配图是一幅发射火弓箭的图，有点类似流星。这表示，Volley 特别适合数据量不大但是通信频繁的场景。

### Volley 有如下的优点：
 - 自动调度网络请求；
 - 高并发网络连接；
 - 通过标准的 HTTP cache coherence（高速缓存一致性）缓存磁盘和内存透明的响应；
 - 支持指定请求的优先级；
 - 网络请求 cancel 机制。我们可以取消单个请求，或者指定取消请求队列中的一个区域；
 - 框架容易被定制，例如，定制重试或者回退功能；
 - 包含了调试与追踪工具；

> * Volley 不适合用来下载大的数据文件。因为 Volley 会保持在解析的过程中所有的响应。对于下载大量的数据操作，请考虑使用 DownloadManager 。
> * 在 volley 推出之前我们一般会选择比较成熟的第三方网络通信库，如：android-async-http、retrofit、okhttp 等。他们各有优劣，可有所斟酌地选择选择更适合项目的类库。
> * 附：
Volley 的 GitHub 地址：https://github.com/mcxiaoke/android-volley

## 二、使用

 - Eclipse

``` bash
把 Volley 添加到项目中最简便的方法是 Clone 仓库，然后把它设置为一个 library project。
（1）clone 代码：
git clone https://android.googlesource.com/platform/frameworks/volley

（2）将代码编译成 jar 包：
android update project -p . ant jar

如无意外，将获得 volley.jar 包。

（3）添加 volley.jar 到你的项目中
```

 - AndroidStudio using Gradle build add dependent (recommended)

``` groovy
compile 'com.mcxiaoke.volley:library:1.0.19'
```

----------

#### 使用 Volley 框架实现网络数据请求主要有以下三个步骤：
 1. 创建 RequestQueue 对象，定义网络请求队列；
 2. 创建 XXXRequest 对象( XXX 代表 String，JSON，Image 等等)，定义网络数据请求的详细过程；
 3. 把 XXXRequest 对象添加到 RequestQueue 中，开始执行网络请求。

### 2.1 创建 RequestQueue 对象

一般而言，网络请求队列都是整个 App 内使用的全局性对象，因此最好写入 Application 类中：

``` java
public class MyApplication extends Application{
    // 建立请求队列
    public static RequestQueue queue;

   @Override
   public void onCreate() {
       super.onCreate();
       queue = Volley.newRequestQueue(getApplicationContext());
   }

   public static RequestQueue getHttpQueue() {
      return queue;
   }
}
```

修改 AndroidManifest.xml 文件，使 App 的 Application 对象为我们刚定义的 MyApplication，并添加 INTERNET 权限：

``` xml
<uses-permission android:name="android.permission.INTERNET" />
<application
    android:name=".MyApplication"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/AppTheme" >
</application>
```

### 2.2 创建 XXXRequest 对象并添加到请求队列中

Volley 提供了`JsonObjectRequest`、`JsonArrayRequest`、`StringRequest`等 Request 形式

### 2.3 把 XXXRequest 对象添加到 RequestQueue 中，开始执行网络请求。

``` java
// 设置该请求的标签
request.setTag("listGet");

// 将请求添加到队列中
MyApplication.getHttpQueue().add(request);
```

### 2.4 关闭请求

#### 关闭特定标签的网络请求：

``` java
// 网络请求标签为"listGet"
public void onStop() {
    super.onStop();
    MyApplication.getHttpQueues.cancelAll("listGet");
}
```

#### 取消这个队列里的所有请求：

在 activity 的 onStop() 方法里面，取消所有的包含这个 tag 的请求任务。

``` java
@Override
protected void onStop() {
    super.onStop();
    mRequestQueue.cancelAll(this);
}
```

对 Volley 的 GET 和 POST 请求进行了封装，见 `VolleyRequestUtil.java` 和 `VolleyListenerInterface.java` 。你在使用过程中也可以添加相应的参数，完成自己所要实现的功能。

## 三、Volley 的 GET 和 POST 请求工具类的封装

### 3.1 GET 和 POST 请求的封装

目前，VolleyRequestUtil 工具类只包含了两个函数，分别获取 GET 和 POST 请求。
VolleyRequestUtil.java：

``` java
public class VolleyRequestUtil {

    public static StringRequest stringRequest;
    public static Context context;
    public static int sTimeOut = 30000;

    /*
    * 获取GET请求内容
    * 参数：
    * context：当前上下文；
    * url：请求的url地址；
    * tag：当前请求的标签；
    * volleyListenerInterface：VolleyListenerInterface接口；
    * timeOutDefaultFlg：是否使用Volley默认连接超时；
    * */
    public static void RequestGet(Context context, String url, String tag,
                                  VolleyListenerInterface volleyListenerInterface,
                                  boolean timeOutDefaultFlg) {
        // 清除请求队列中的tag标记请求
        MyApplication.getQueue().cancelAll(tag);
        // 创建当前的请求，获取字符串内容
        stringRequest = new StringRequest(Request.Method.GET, ConstUtils.BASEURL + url,
                volleyListenerInterface.responseListener(), volleyListenerInterface.errorListener());
        // 为当前请求添加标记
        stringRequest.setTag(tag);
        // 默认超时时间以及重连次数
        int myTimeOut = timeOutDefaultFlg ? DefaultRetryPolicy.DEFAULT_TIMEOUT_MS : sTimeOut;
        stringRequest.setRetryPolicy(new DefaultRetryPolicy(myTimeOut,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        // 将当前请求添加到请求队列中
        MyApplication.getQueue().add(stringRequest);
        // 重启当前请求队列
        MyApplication.getQueue().start();
    }

    /*
    * 获取POST请求内容（请求的代码为Map）
    * 参数：
    * context：当前上下文；
    * url：请求的url地址；
    * tag：当前请求的标签；
    * params：POST请求内容；
    * volleyListenerInterface：VolleyListenerInterface接口；
    * timeOutDefaultFlg：是否使用Volley默认连接超时；
    * */
    public static void RequestPost(Context context, String url, String tag,
                                   final Map<String, String> params,
                                   VolleyListenerInterface volleyListenerInterface,
                                   boolean timeOutDefaultFlg) {
        // 清除请求队列中的tag标记请求
        MyApplication.getQueue().cancelAll(tag);
        // 创建当前的POST请求，并将请求内容写入Map中
        stringRequest = new StringRequest(Request.Method.POST, ConstUtils.BASEURL + url,
                volleyListenerInterface.responseListener(), volleyListenerInterface.errorListener()) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                return params;
            }
        };
        // 为当前请求添加标记
        stringRequest.setTag(tag);
        // 默认超时时间以及重连次数
        int myTimeOut = timeOutDefaultFlg ? DefaultRetryPolicy.DEFAULT_TIMEOUT_MS : sTimeOut;
        stringRequest.setRetryPolicy(new DefaultRetryPolicy(myTimeOut,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        // 将当前请求添加到请求队列中
        MyApplication.getQueue().add(stringRequest);
        // 重启当前请求队列
        MyApplication.getQueue().start();
    }
}
```

### 3.2 Volley 请求（成功或失败）的监听事件封装

封装 Volley 请求（成功或失败）的监听事件。
VolleyListenerInterface.java：

``` java
public abstract class VolleyListenerInterface {
    public Context mContext;
    public static Response.Listener<String> mListener;
    public static Response.ErrorListener mErrorListener;

    public VolleyListenerInterface(Context context, Response.Listener<String> listener,
                                   Response.ErrorListener errorListener) {
        this.mContext = context;
        this.mErrorListener = errorListener;
        this.mListener = listener;
    }

    // 请求成功时的回调函数
    public abstract void onMySuccess(String result);

    // 请求失败时的回调函数
    public abstract void onMyError(VolleyError error);

    // 创建请求的事件监听
    public Response.Listener<String> responseListener() {
        mListener = new Response.Listener<String>() {
            @Override
            public void onResponse(String s) {
                Log.e("Volley Response", "response == " + s);
                onMySuccess(s);
            }
        };
        return mListener;
    }

    // 创建请求失败的事件监听
    public Response.ErrorListener errorListener() {
        mErrorListener = new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError volleyError) {
                onMyError(volleyError);
                Toast.makeText(mContext,
                        mContext.getString(R.string.toast_networkError), Toast.LENGTH_LONG).show();
            }
        };
        return mErrorListener;
    }
}
```

### 3.3 Volley 文件上传类的封装

封装文件上传至服务器的工具类。
PostUploadRequest.java：

``` java
public class PostUploadRequest extends Request<JSONObject> {

    private final String prefix = "--";
    private final String end = "\r\n";
    private final String boundary = "--------------" + System.currentTimeMillis();
    private final String mimeType = "multipart/form-data;boundary=" + boundary;
    private Response.Listener<JSONObject> mListener;
    private Map<String, String[]> fileMap;

    public PostUploadRequest(String url, Map<String, String[]> fileMap,
                             Response.Listener<JSONObject> mListener,
                             Response.ErrorListener listener) {
        super(Method.POST, url, listener);
        this.mListener = mListener;
        this.fileMap = fileMap;
        setShouldCache(false);
        setRetryPolicy(new DefaultRetryPolicy(5000, 
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
    }

    @Override
    protected Response<JSONObject> parseNetworkResponse(NetworkResponse response) {
        try {
            String je = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            return Response.success(new JSONObject(je), HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException var3) {
            return Response.error(new ParseError(var3));
        } catch (JSONException var4) {
            return Response.error(new ParseError(var4));
        }
    }

    @Override
    protected void deliverResponse(JSONObject jsonObject) {
        mListener.onResponse(jsonObject);
    }

    @Override
    public String getBodyContentType() {
        return mimeType;
    }

    @Override
    public byte[] getBody() throws AuthFailureError {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            DataOutputStream ds = new DataOutputStream(bos);
            buildTextPart(ds);
            buildFilePart(ds);
            ds.write((prefix + boundary + prefix + end).getBytes("UTF-8"));
            ds.flush();
            return bos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private void buildFilePart(DataOutputStream ds) {
        StringBuilder sb = new StringBuilder();
        Object[] key_arr = fileMap.keySet().toArray();
        for (Object name : key_arr) {
            String[] val = fileMap.get(name.toString());
            String path = val[0];
            String filename = val[1];
            try {
                sb.append(prefix);
                sb.append(boundary);
                sb.append(end);
                sb.append("Content-Disposition: form-data;name=\"");
                sb.append(name.toString());
                sb.append("\";filename=\"");
                sb.append(filename);
                sb.append("\"" + end);//filename是文件名，如xxx.jpg，file是服务器传递参数的名字
                sb.append("Content-Type: application/octet-stream;charset=UTF-8" + end);
                sb.append(end);
                ds.write(sb.toString().getBytes("UTF-8"));
                /* 取得文件的FileInputStream */
                FileInputStream fStream = new FileInputStream(path);//path是文件本地地址
                /* 设置每次写入1024bytes */
                int bufferSize = 1024;
                byte[] buffer = new byte[bufferSize];
                int length;
                /* 从文件读取数据至缓冲区 */
                while ((length = fStream.read(buffer)) != -1) {
                    /* 将资料写入DataOutputStream中 */
                    ds.write(buffer, 0, length);
                }
                ds.write(end.getBytes("UTF-8"));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void buildTextPart(DataOutputStream ds) {
        try {
            Map<String, String> map = getParams();
            Object[] key_arr = map.keySet().toArray();
            for (Object name : key_arr) {
                String val = map.get(name.toString());
                ds.write((prefix + boundary + end).getBytes("UTF-8"));
                ds.write(("Content-Disposition: form-data;name=\"" + name.toString() + "\"" + end).getBytes("UTF-8"));
                ds.write(("Content-Type: text/plain;charset=UTF-8" + end).getBytes("UTF-8"));
                ds.write((end).getBytes("UTF-8"));
                ds.write((val + end).getBytes("UTF-8"));
            }
        } catch (IOException | AuthFailureError e) {
            e.printStackTrace();
        }
    }

}
```

## 四、VolleyRequestUtil 的使用

 - **用 GET 方式请求网络资源：**

``` java
VolleyRequestUtil.RequestGet(this, url, "tag", 
    new VolleyListenerInterface(this, VolleyListenerInterface.mListener,
            VolleyListenerInterface.mErrorListener) {
    // Volley请求成功时调用的函数
    @Override
    public void onMySuccess(String result) {
        Toast.makeText(this, s, Toast.LENGTH_LONG).show();
    }

    // Volley请求失败时调用的函数
    @Override
    public void onMyError(VolleyError error) {
        // ...
    }
});
```

 - **用 POST 方式请求网络资源：**

``` java
VolleyRequestUtil.RequestPOST(this, url, "tag", 
    new VolleyListenerInterface(this, VolleyListenerInterface.mListener,
            VolleyListenerInterface.mErrorListener) {
    // Volley请求成功时调用的函数
    @Override
    public void onMySuccess(String result) {
        Toast.makeText(MainActivity.this, result, Toast.LENGTH_LONG).show();
    }

    // Volley请求失败时调用的函数
    @Override
    public void onMyError(VolleyError error) {
        // ...
    }
});
```

## 五、PostUploadRequest 的使用

用于上传文件的框架，封装于 Volley。

``` java
/**
 * 上传文件分两步：
 * 1.调用此工具类，将文件传输到服务器（从本地选择文件的过程未列出）
 * 2.调用修改文件名的接口，修改数据库中相应的字段，完成上传文件操作
 */
private void uploadFile() {
    String upLoadServerUri = "";
    HashMap<String, String[]> map = new HashMap<>();
    map.put("uploadedfile", new String[]{filename, cutnameString});
    MyApplication.getQueue().add(new PostUploadRequest(upLoadServerUri,
            map, new Response.Listener<JSONObject>() {
        @Override
        public void onResponse(JSONObject jsonObject) {
            updatePic();
        }
    }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError volleyError) {
            Toast.makeText(mContext, "文件上传失败", Toast.LENGTH_SHORT).show();
        }
    }) {
        @Override
        protected Map<String, String> getParams() throws AuthFailureError {
            HashMap<String, String> map = new HashMap<String, String>();
                //map.put("token", "");
            return map;
        }
    });
}

private void updatePic() {
    Map<String, String> params = new HashMap<>();
    params.put("id", "1");
    params.put("pic", cutnameString);
    VolleyRequestUtil.RequestPost(this, url, "tag", params,
            new VolleyListenerInterface(this, VolleyListenerInterface.mListener,
                    VolleyListenerInterface.mErrorListener) {
                // Volley请求成功时调用的函数
                @Override
                public void onMySuccess(String response) {

                }
                // Volley请求失败时调用的函数
                @Override
                public void onMyError(VolleyError error) {

                }
            });
}
```

**Volley 也提供了图片的缓存和优化，（ `com.android.volley.toolbox.NetworkImageView`） 自定义图片控件，本人在开发中未使用。该 Volley 的封装中，暂未考虑到图片和数据缓存。有一些地方封装得仍不够抽象，有待完善。非常欢迎各位能提出修改建议，一起进步！**
**Demo 源码请看 GitHub： [MyVolley](https://github.com/lishide/MyVolley)。**