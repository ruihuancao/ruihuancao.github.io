---
title: Android跨进程通信－AIDL
date: 2015-04-13 20:31:46
categories:
- 开发
- Android
tags:
- IPC
---

IPC,即进程间通讯(Inter-Process Communication),在开发过程需要在应用进程中实现和另外一个进程进行通信。

<!--more-->

## 序列化
在应用底层通信，数据的传输形式是简单的字节序列形式传递，那么就需要把我们使用的对象数据转化成字节序列，这个过程就是序列化。反序列化正好相反，将字节序列回复到对象。用于序列化的接口有：Serializable接口和Parcelable接口。
Serializable：Java提供了一个序列化接口，实现比较简单。
Parcelable：是Android中提供的一个序列化接口，需要自己实现序列化规则，高效。

## AIDL
AIDL全称：Android Interface Definition Language,即Android接口定义语言。由于不同的进程不能共享内存，所以为了解决进程间通讯的问题，Android使用一种接口定义语言来公开服务的接口，本质上，AIDL非常像一个接口，通过公开接口，让别的进程调用该接口，从而实现进程间的通讯。

### 服务端实现
- 1 新建服务端接口IMyService.aidl文件
```
// IMyService.aidl
package com.memento.android.aidl;

import com.memento.android.aidl.TestInfo;

// Declare any non-default types here with import statements

interface IMyService {

    void serviceSayHi();

    void serviceAddTestData(in TestInfo testInfo);
}
```

- 2 新建服务端自定义类型的TestInfo.aidl
```
// TestInfo.aidl
package com.memento.android.aidl;

parcelable TestInfo;
```
- 3 自定义类型，实现Parcelable序列化。该文件和上面的aidl文件同名，同包下
```
package com.memento.android.aidl;

import android.os.Parcel;
import android.os.Parcelable;

public final class TestInfo implements Parcelable {
    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }


    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.value);
    }

    public TestInfo() {
    }

    protected TestInfo(Parcel in) {
        this.value = in.readString();
    }

    public static final Parcelable.Creator<TestInfo> CREATOR = new Parcelable.Creator<TestInfo>() {
        @Override
        public TestInfo createFromParcel(Parcel source) {
            return new TestInfo(source);
        }

        @Override
        public TestInfo[] newArray(int size) {
            return new TestInfo[size];
        }
    };
}
```
- 4 Service 实现, 示例添加了打印日志的线程，为了现实Service是否运行
```

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Log;

import com.memento.android.aidl.IMyService;
import com.memento.android.aidl.TestInfo;

public class MyService extends Service {

    private static final String TAG = MyService.class.getSimpleName();

    private IMyService.Stub mBinder = new IMyService.Stub() {
        @Override
        public void serviceSayHi() throws RemoteException {
            Log.d(TAG, "Server say hi");
        }

        @Override
        public void serviceAddTestData(TestInfo testInfo) throws RemoteException {
            Log.d(TAG, "Server data is:"+testInfo.getValue());
        }
    };

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        new Thread(){
            @Override
            public void run() {
                super.run();
                for (;;){
                    try{
                        Log.d(TAG, "server is runing...");
                        sleep(2000);
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }

    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "Server is runing......");
        return mBinder;
    }
}
```
- 5 配置
```
<service
           android:name=".ui.service.MyService"
           android:enabled="true"
           android:exported="true">
           <intent-filter>
               <category android:name="android.intent.category.DEFAULT" />
               <action android:name="com.memento.android.MyService" />
           </intent-filter>
       </service>
```

### 客户端
可以做成客户端sdk，提供出去
- 1 创建同包名工程
- 2 拷贝上面的aidl包下文件
- 3 调用
```
Intent intentService = new Intent();
       intentService.setAction("com.memento.android.MyService");
       intentService.setPackage("com.memento.android");
       bindService(intentService, new ServiceConnection() {
           @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               mIMyService = IMyService.Stub.asInterface(service);
               Log.d(TAG, "mIMyService is connected");
               try {
                   mIMyService.serviceSayHi();
                   TestInfo testInfo = new TestInfo();
                   testInfo.setValue("this is client");
                   mIMyService.serviceAddTestData(testInfo);
               } catch (RemoteException e) {
                   e.printStackTrace();
               }
           }

           @Override
           public void onServiceDisconnected(ComponentName name) {
               Log.d(TAG, "mIMyService is Disconnected");
           }
       }, BIND_AUTO_CREATE);
```


## 问题
- 1 自定义类型无法找到： android studio 默认编译没有将aidl目录配置进去
如果
```
sourceSets{
    main{
        aidl.srcDirs = ['src/main/aidl']
        java.srcDirs = ['src/main/java','src/main/aidl']
    }
}
```

## 总结
可以看到aidl实现进程间通信就这样实现了，起始aidl是利用了Binder来进行的跨进程通信。Binder是Android中的一种跨进程通讯方式，其底层实现原理比较复杂，我还没看太懂呢。。。。
