# Java并发编程——`Thread`类的使用

[博客](http://www.cnblogs.com/dolphin0520/p/3920357.html)

[TOC]

## 线程的状态

* 创建（new）
* 就绪（runnable）
* 运行（running）
* 阻塞（blocked）
* time waiting
* waiting
* 消亡（dead）

![img](../img/061045374695226.png)

## 上下文切换

* 存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行（记录程序计数器、CPU寄存器等状态）
* 上下文切换有系统开销

## `Thread`类中的方法

|    方法     |                             作用                             |
| :---------: | :----------------------------------------------------------: |
|   `start`   |                         启动一个线程                         |
|    `run`    | 由系统调用，当线程获得了CPU执行时间，便会进入`run`方法执行具体任务 |
|   `sleep`   |  让线程睡眠，即阻塞，交出CPU，让CPU执行其他任务；不会释放锁  |
|   `yield`   | 交出CPU执行权限，让具有相同优先级的线程有获取CPU执行时间的机会。当前线程处于就绪状态，不是阻塞状态 |
|   `join`    | `join()`:等待执行此方法所处的当前线程执行完毕，执行调用此方法的线程<br>`join(long millis)`,`join(long millis, int naoseconds)`:等待一定时间执行调用此方法的线程<br>调用此方法会让线程阻塞，并释放锁 |
| `interrupt` | 中断一个处于阻塞状态的线程，并抛出异常。中断处于非阻塞状态的线程，需要特殊处理 |

### 使用中断非阻塞状态的线程

#### 使用`interrupt`(==不推荐使用==)

```java
public static void main(String arg[]) {
    MyThread thread = new MyThread();
    thread.start();
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    thread.interrupt();
}

static class MyThread extends Thread {
    @Override
    public void run() {
        int i = 0;
        while (!isInterrupted() && i < Integer.MAX_VALUE) {
            i++;
        }
    }
}
```

#### 使用`isStop`标记循环结束（==推荐使用==）

```java
public static void main(String arg[]) {
    MyThread thread = new MyThread();
    thread.start();
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    isStop = true;
}

static class MyThread extends Thread {
    @Override
    public void run() {
        int i = 0;
        while (isStop && i < Integer.MAX_VALUE) {
            i++;
        }
    }
}
```

### 线程属性相关方法

#### `getId`

获取线程id

#### `getName`、`setName`

获取线程名称、设置线程名称

#### `getPriority`、`setPriority`

获取线程优先级，设置线程优先级

#### `setDaemon`、`isDaemon`

* 设置线程是否成为守护进程，判断线程是否为守护进程
* 守护线程依赖于创建他的线程，创建它的线程销毁，守护线程也会被销毁
* 在JVM中，垃圾回收器线程就是守护进程

### Thread方法调用图解

![img](../img/061046391107893.png)