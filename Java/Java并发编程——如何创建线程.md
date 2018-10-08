# Java并发编程——如何创建线程

[读此博客笔记]: https://www.cnblogs.com/dolphin0520/p/3920373.html	"博客笔记"

[TOC]

## Java中关于应用程序和进程相关的概念

* Java是单线程编程模型。不主动创建线程的话，只会创建一个主线程
* JVM实例在创建的时候，同时会创建很多其他的线程（如垃圾回收器线程）

## Java中如何创建线程

* 继承`Thread`类
* 实现`Runnable`接口

## Java中如何创建进程

### 进程类

```java
public abstract class Procress {
    //获取进程的输出流
    abstract public OutputStream getOutputStream();
    
    //获取进程的输入流
    abstract public InputStream getInputStream();
    
    //获取进程的错误流
    abstract public InputStream getErrorStream();
    
    //让进程等待
    abstract public int waitFor throws InterruptedException;
    
    //获取进程的退出标志
    abstract public int exitValue();
    
    //摧毁线程
    abstract public void destroy();
}
```

### ProgressBuilder创建

```java
ProcessBuilder processBuilder = new ProcessBuilder("ifconfig");
Process process = processBuilder.start();
```

### Runtime创建

```java
String cmd = "ifconfig";
Progress progress = Runtime.getRuntime().exec(cmd);
```

