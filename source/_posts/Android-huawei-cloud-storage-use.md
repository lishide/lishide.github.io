---
title: Android 端-华为云存储的集成与使用
date: 2019-03-12 23:28:59
tags:
---
Android 端-华为云存储的集成与使用

<!--more-->

# 1. 创建应用
申请账号，创建应用，获得所需的 ak、sk 等相关 Key。

# 2. Android 端集成

## 2.1 UseCase 类
``` java
package com.xxx.commonsdk.huawei;

import dagger.internal.Preconditions;
import io.reactivex.Observable;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.disposables.CompositeDisposable;
import io.reactivex.disposables.Disposable;
import io.reactivex.functions.Action;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;

public abstract class UseCase<T, Params> {

    private final CompositeDisposable disposables;

    UseCase() {
        this.disposables = new CompositeDisposable();
    }

    abstract Observable<T> buildUseCaseObservable(Params params);

    public void execute(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
                        Action onComplete, Params params) {
        final Observable<T> observable = this.buildUseCaseObservable(params)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread());
        addDisposable(observable.subscribe(onNext, onError, onComplete));
    }

    public void dispose() {
        if (!disposables.isDisposed()) {
            disposables.dispose();
        }
    }

    private void addDisposable(Disposable disposable) {
        Preconditions.checkNotNull(disposable);
        Preconditions.checkNotNull(disposables);
        disposables.add(disposable);
    }
}
```

## 2.2 文件上传类

``` java
package com.xxx.commonsdk.huawei;

import com.obs.services.ObsClient;
import com.obs.services.model.PutObjectRequest;
import com.obs.services.model.PutObjectResult;
import com.vea.atoms.mvp.commonsdk.http.Api;

import io.reactivex.Observable;
import timber.log.Timber;

import javax.inject.Inject;

import java.io.File;

public class UploadFile extends UseCase<PutObjectResult, UploadFile.Param> {

    private final ObsClient obsClient;

    @Inject
    UploadFile(ObsClient obsClient) {
        this.obsClient = obsClient;
    }

    @Override
    Observable<PutObjectResult> buildUseCaseObservable(Param param) {
        File file = new File(param.getPath());
        PutObjectRequest request = new PutObjectRequest(Api.bucketName, file.getName());
        request.setFile(file);
        request.setProgressListener(status -> {
            Timber.i("AverageSpeed:%s", status.getAverageSpeed());
            Timber.i("TransferPercentage:%s", status.getTransferPercentage());
        });
        return Observable.create(emitter -> {
            PutObjectResult result = obsClient.putObject(request);
            emitter.onNext(result);
            emitter.onComplete();
        });
    }

    public static class Param {
        public Param(String path) {
            this.path = path;
        }

        private String path;

        public String getPath() {
            return path;
        }

        public void setPath(String path) {
            this.path = path;
        }
    }
}

```

# 2.3 初始化
```java
@ActivityScope
@Provides
static ObsClient provideObsClient() {
    ObsConfiguration config = new ObsConfiguration();
    config.setSocketTimeout(30000);
    config.setConnectionTimeout(10000);
    config.setEndPoint(Api.endPoint);
    ObsClient obsClient = new ObsClient(Api.ak, Api.sk, config);
    ObsBucket obsBucket = new ObsBucket();
    obsBucket.setBucketName(Api.bucketName);
    // 设置日志的级别。默认为LogConfigurator.WARN
    LogConfigurator.setLogLevel(LogConfigurator.DEBUG);
    // 设置保留日志文件的个数。默认为10
    LogConfigurator.setLogFileRolloverCount(5);
    // 设置每个日志文件的大小，单位:字节。默认为不限制
    LogConfigurator.setLogFileSize(1024 * 1024 * 10);
    // 设置日志文件存放的目录。默认存放在SD卡的logs目录下
    LogConfigurator.setLogFileDir(Environment.getExternalStorageDirectory().getAbsolutePath());
    // 开启日志
    LogConfigurator.enableLog();
    return obsClient;
}
```

华为云存储相关 Key
```java
/**
* 华为云存储相关 Key
*/
String endPoint = "https://url";
String ak = "AKAKAKAKAKAKAKAK";
String sk = "sksksksksksksksksksksksksksk";
String bucketName = "bucketName-resource-1";

```

## 2.4 使用

```java
public void uploadAvatar(String path) {
    uploadFile.execute(o -> {
        //o 为： PutObjectResult
        // o.getObjectUrl() 为上传后的文件名，继续后续的操作，如提交到服务器等
        editUser("", o.getObjectUrl());
    }, Throwable::printStackTrace, () -> {
    }, new UploadFile.Param(path));
}
```

# 3. 总结
感觉华为云存储文档不是太好，当时做的时候遇到了许多坑，还请教了别人。做个记录，以备参考使用。



