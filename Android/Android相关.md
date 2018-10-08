# andrid相关

## webview常见的一些坑

### js注入漏洞

api16及以前，通过`addJavaInterface()`，通过反射执行任意的Java对象方法

### `WebViewClinet.onPageFinish()`

回调的时候无法判断页面是否真的加载完成了，有可能加载的网页产生跳转的时候也会回调此方法

应该使用`WebChromeClient.onProgressChanged()`

### 后台耗电

`webview`会自己开启一些线程，如果没有正确销毁`webview`，这些残存的线程会一直在后台运行

### 闪屏

硬件加速导致，暂时关闭硬件加速

### `webview`防止内存泄漏

给`webview`一个父容器，通过`addView()`的方式新建`webView`

```java
if (mWebViewContainer != null && mWebView != null) {
	mWebViewContainer.removeView(mWebView);
	mWebView.removeAllViews();
	mWebView.destroy();
}
```

## `IPC`方式

#### 使用`Bundle`

#### 使用文件共享

#### 使用`Messenger`

底层`Messenger`本质是个`AIDL`

* `private final Messenger mMessenger = new Messenger(new MessagerHandler());`
* `service`中的`onBind()`中返回`mMessenger.getBinder()`
* `server`端通知`client`端可以从`message`中取出`replyTo`，此`replyTo`是从`client`端传递过来的`messenger`

#### 使用`AIDL`

##### 使用

* 服务端
  * 创建`AIDL`文件(接口文件，和传递的实体类)
  * 创建`service`，继承`AIDL`生成的`Stub`抽象类
* 客户端
  * 复制服务端`AIDL`文件到客户端中
  * 绑定服务端`AIDL`服务，将服务端返回的`Binder`对象转成`AIDL`接口所属的类型
  * 调用`AIDL`接口方法

==接口需要使用RemoteCallbackList==

##### 支持的数据类型

* 基本数据类型
* `String`和`CharSequence`
* `List`:只支持`ArrayList`
* `Map`：只支持`HashMap`
* `Parcelable`
* `AIDL`接口

#### 使用`ContentProvider`

#### 使用`Socket`

## `Binder`机制

粘合了两个进程的粘合剂

* 机制、模型：Binder是一种android中的IPC方式
* 模型结构、组成：Binder是一种虚拟的物理设备驱动。
* Android实现：继承了IBinder接口

> 进程空间分为内核空间和用户空间
>
> * 用户空间：不可共享
> * 内核空间：可共享

### 传统的IPC方式需要copy两次

![ç¤ºæå¾](http://upload-images.jianshu.io/upload_images/944365-12935684e8ec107c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用户空间（客户端）->内核缓存区->用户空间（服务端）==两次copy==

### Binder通过内存映射的形式实现了copy一次的IPC

![ç¤ºæå¾](http://upload-images.jianshu.io/upload_images/944365-c10d6032f91a103f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Client：客户端进程
* Server：服务端进程
* Binder驱动：一种虚拟设备驱动，连接Client进程、Server进程、ServiceManager进程的桥梁（Binder驱动持有没有Server进程在内核空间中的Binder实体，并给Client进程提供Binder实体的引用）
  * 传递进程间的数据==通过内存映射==
  * 实现线程控制，采用Binder的线程池，并由Binder驱动自身管理
* ServiceManager：管理service的注册与查询（将字符形式的Binder名字转换成client中对改Binder的引用）==类似于路由器==

![ç¤ºæå¾](http://upload-images.jianshu.io/upload_images/944365-65a5b17426aed424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## `Activity`生命周期和启动模式

* `onPause()`会在下一个界面的`onCreate()`之前调用，因此`onPause()`中不要执行耗时操作
* 启动模式
  * standard
  * singleTop
  * singleTask：如果新启动的Activity在相应任务栈中已经存在，则会把任务栈中此Activity上面的其他Activity出栈
  * singleInstance

