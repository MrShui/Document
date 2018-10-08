# Java并发编程——`synchronized`

[博客](http://www.cnblogs.com/dolphin0520/p/3923737.html)

[TOC]

## 什么时候会出现线程安全问题

* 多个线程同时访问一个资源可能会出现线程安全问题
* 这种共享的资源叫做临界变量
* 多个线程执行一个方法，方法内部的局部变量不是临界变量（方法在栈上执行，他在Java中线程私有）

## 如何解决线程安全问题

* 在访问临界资源代码时，在其面加入锁，访问完毕之后释放锁
* `synchronized`、`Lock`

## `synchronized`同步方法或者同步快

* 当一个对象正在访问一个对象的`synchronized`方法，那么其他线程不能访问该对象的其他`synchronized`方法，因为一个对象只有一把锁

### `static`与`synchronized` 

* `static`方法使用`synchronized`，则它的锁是类锁
* 访问非`static synchronized`方法的锁是对象锁
* 对象锁和类锁互不影响

```java
public class Test {
 
    public static void main(String[] args)  {
        final InsertData insertData = new InsertData();
        new Thread(){
            @Override
            public void run() {
                insertData.insert();
            }
        }.start(); 
        new Thread(){
            @Override
            public void run() {
                insertData.insert1();
            }
        }.start();
    }  
}
 
class InsertData { 
    public synchronized void insert(){
        System.out.println("执行insert");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("执行insert完毕");
    }
     
    public synchronized static void insert1() {
        System.out.println("执行insert1");
        System.out.println("执行insert1完毕");
    }
}
```

![img](../img/192113596129599.png)

==对于`synchronized`方法或者`synchronized`代码块，当出现异常的时候，JVM会自动释放当前线程占用的锁，因此不会出现异常导致的死锁现象==