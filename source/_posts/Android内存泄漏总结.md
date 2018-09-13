---
title: Android内存泄露总结
date: 2016-09-12 20:10:13
categories:
- Android
tags:
- Android
---
开发过程中如何避免内存泄露，以及内存泄露查看方法总结。内存泄露简单说就是，该被释放的对象没有被释放，存在引用无法被GC回收.
<!--more-->

## Java内存分配策略
java内存分配策略有3种：静态分配,栈式分配,和堆式分配，对应内存存储空间，静态存储区（方法区）、栈区和堆区。

- 静态存储区（方法区）：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
- 栈区 ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
- 堆区 ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。

栈存储的是基本类型的变量和对象的引用（指向堆中对象的内存首地址）
堆中存放的是成员变量（对象的实体），由java垃圾回收期管理。

## Java内存管理
Java的内存管理就是对象的分配和释放问题。
Java使用有向图的方式进行内存管理。如果某个对象与这个根顶点不可达,那么我们认为这个对象不再被引用，可以被 GC 回收。

## Java中的内存泄露
存在一些被分配的对象，已经没有使用，但是在有向图中可达。意味着存在引用,不在使用的对象无法别GC回收。

## Android中常见的内存泄漏
- 集合类泄漏
经常在做缓存的时候常使用map的全局变量，如果只添加对象，没有相应的删除机制，添加进去的对象则一致无法被回收，内存泄露。
- 单例造成的内存泄漏
在做数据层时，常建立单例引用。单例在整个程序运行期间一致存在，建立时需要传入一个Context,
如果传入一个ApplicationContext,它在程序运行期间一致存在，没有问题。
如果传入Activity Context，及时Activity生命周期结束，由于存在单例引用，导致GC无法回收，内存泄露。
```
public class DataManager {
    private volatile static DataManager instance;
    // 如果传入Activity导致其被单例引用，无法被回收内存泄露
    // 需要传入Applicaltion Context
    private Context mContext;
    private DataManager(Context context){
        this.mContext = context;
    }
    public static DataManager getInstance(Context context){
        synchronized (DataManager.class){
            if(instance == null){
                instance = new DataManager(context);
            }
        }
        return instance;
    }
}
```

- 非静态内部类
非静态内部类默认持有外部类引用，静态变量在整个应用期间可用，这就导致一直持有Activity，Activity无法被回收。
```
public class MainActivity extends AppCompatActivity{
    private static Test mTest = null;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mTest = new Test();
    }

    class Test{

    }
}
```

- 匿名内部类
如果runnable线程与Activity生命周期不一致，则内存泄漏，runnable默认持有Activity引用，Activity结束，线程未结束，无法回收。
```
public class MainActivity extends AppCompatActivity{
    Runnable runnable = new Runnable() {
        @Override
        public void run() {

        }
    };
}
```

- Handler造成的内存泄漏
这个很常见，解决办法，静态内部类 + WeakReference
Java对引用的分类有 Strong reference, SoftReference, WeakReference, PhatomReference 四种
除了强引用，后3中都可以被GC回收
```
public class MainActivity extends AppCompatActivity{

    private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            test();
        }
    };

    private void test(){}

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
            }
        }, 1000*60*10);
    }
}
```
- 资源未关闭或未取消注册的内存泄露
io操作，BraodcastReceiver，数据库操游标等

## 内存泄露解决办法
### 工具
- Android Studio Memory Monitor
可以查看内存情况，触发GC，快速定位卡顿，Crash，内存泄露等其他问题，都可以查看。
例：
查看内存变化，打开一个页面然后关闭，手动触发GC，查看内存变化，如此重复几次，一直显著增加，则内存泄露。
Heap Viewer 在Memory中点击Dump Java Heap 就可以查看分配情况。
Allocation Tracker 记录一段时间内的分配情况，有助于分析问题
- LeakCanary
开源项目，检测内存泄露，很好用，自行google。

这些查找方法没什么特别，都是在寻找异常的对象，然后查看它的引用，找出存在问题。有的问题看一下就知道，有时候根本不需要去查看，比如上面的常见问题，其他的复杂问题，需要根据日志或者异常点定点查找，更加快速。但更主要的是了解为什么发生内存泄露，从而在写代码的时候就尽量去避免问题产生。

## 开发过程中解决问题方法
要掌握的主要还是解决问题的方法，而不是特定的问题。解决问题通常都能找到解决方案，花时间的主要还是定位问题
- 查看日志。常见的异常如空指针等可以直接定位问题，日志搜索过滤ANR等信息
- 搜索引擎。google，[社区](http://stackoverflow.com)
- 查看源码和官方文档。这是解决问题最直接有效的方式。

实际中一般是多种方式组合能很快解决
