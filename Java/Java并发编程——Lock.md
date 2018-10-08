# Java并发编程——`Lock`

[博客](http://www.cnblogs.com/dolphin0520/p/3923167.html)

[TOC]

## `synchronized`的缺陷

不能灵活加锁以及释放锁

## `java.util.concurrent.locks`包下常用的类

### `Lock`

```java
public interface Lock {
    void lock();
    void lookInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time,TimeUnit unit) throws InterruptedException;
    void unlock();
    COndition newCondition();
}
```

#### `lock()`

获取锁，如果锁已被其他线程获取，则进行等待

```java
Lock lock = new ReentrantLock();
lock.lock();
try{
    
}catch(Exception e){
   //处理任务 
}finally{
    lock.unlock()
}
```

#### `tryLock()`

获取锁并返回获取锁的结果，这个方法无论有没有获取到锁都会立即返回结果，不会阻塞等待

#### `tryLock(long time,TimeUnit unit)`

如果获取不到锁，会等待一定时间，如果在一定时间还是没有获取到锁，会返回false不在获取

#### `lockInterruptibly()`

同`lock()`，但是`lock()`获取锁阻塞过程中不可以被中断，而`lockInterruptibly()`可以被中断，中断是抛出异常

==获取锁之后就无法被中断。只有阻塞状态下才有可能被中断==

### `ReentrantLock`

可重入锁

#### `lock()`的正确用法

##### 错误🌰

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by Shui on 2018/7/17.
 */
public class ReentrantLockTest {
    private List<Integer> arrayList = new ArrayList<>();

    public static void main(String arg[]) {
        ReentrantLockTest reentrantLockTest = new ReentrantLockTest();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
    }

    public void insert(Thread thread) {
        Lock lock = new ReentrantLock();//fixbug 
        lock.lock();
        try {
            System.out.println(thread.getName() + "得到了锁");
            for (int i = 0; i < 5; i++) {
                arrayList.add(i);
            }
        } catch (Exception ignored) {

        } finally {
            lock.unlock();
        }
    }
}
```

> Thread-0得到了锁
> Thread-1得到了锁
> 释放了锁
> 释放了锁

`insert()`方法中`Lock`会在每个线程中创建一个锁

##### 正确🌰

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by Shui on 2018/7/17.
 */
public class ReentrantLockTest {
    private List<Integer> arrayList = new ArrayList<>();
    private Lock lock = new ReentrantLock();

    public static void main(String arg[]) {
        ReentrantLockTest reentrantLockTest = new ReentrantLockTest();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
    }

    public void insert(Thread thread) {
        lock.lock();
        try {
            System.out.println(thread.getName() + "得到了锁");
            for (int i = 0; i < 5; i++) {
                arrayList.add(i);
            }
        } catch (Exception ignored) {

        } finally {
            lock.unlock();
            System.out.println("释放了锁");
        }
    }
}

```

> Thread-0得到了锁
> 释放了锁
> Thread-1得到了锁
> 释放了锁

#### `tryLock()`用法

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by Shui on 2018/7/17.
 */
public class ReentrantLockTest {
    private List<Integer> arrayList = new ArrayList<>();
    private Lock lock = new ReentrantLock();

    public static void main(String arg[]) {
        ReentrantLockTest reentrantLockTest = new ReentrantLockTest();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
        new Thread(() -> reentrantLockTest.insert(Thread.currentThread())).start();
    }

    public void insert(Thread thread) {
        if (lock.tryLock()) {
            try {
                System.out.println(thread.getName() + "得到了锁");
                for (int i = 0; i < 5; i++) {
                    arrayList.add(i);
                    Thread.sleep(200);
                }
            } catch (Exception e) {

            } finally {
                lock.unlock();
                System.out.println(thread.getName() + "释放了锁");
            }
        } else {
            System.out.println(thread.getName() + "获取锁失败");
        }
    }
}
```

> Thread-0得到了锁
> Thread-1获取锁失败
> Thread-0释放了锁

#### `lockInterruptibly()`用法

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by Shui on 2018/7/17.
 */
public class ReentrantLockTest {
    private Lock lock = new ReentrantLock();

    public static void main(String arg[]) {
        ReentrantLockTest test = new ReentrantLockTest();
        MyThread thread1 = new MyThread(test);
        MyThread thread2 = new MyThread(test);
        thread1.start();
        thread2.start();

        try {
            Thread.sleep(2000);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread2.interrupt();
    }

    public void insert(Thread thread) throws InterruptedException {
        lock.lockInterruptibly();
        try {
            System.out.println(thread.getName() + "得到了锁");
            long startTime = System.currentTimeMillis();

            for (; ; ) {
                if (System.currentTimeMillis() - startTime >= Integer.MAX_VALUE) {
                    break;
                }
            }
        } catch (Exception e) {

        } finally {
            System.out.println(Thread.currentThread().getName() + "执行finally");
            lock.unlock();
            System.out.println(Thread.currentThread().getName() + "释放了锁");
        }
    }

    static class MyThread extends Thread {
        private ReentrantLockTest test;

        public MyThread(ReentrantLockTest test) {
            this.test = test;
        }

        @Override
        public void run() {
            try {
                test.insert(Thread.currentThread());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(Thread.currentThread().getName() + "被中断");
            }
        }
    }
}
```

> Thread-0得到了锁
> java.lang.InterruptedException
> 	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
> 	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
> 	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
> 	at ReentrantLockTest.insert(ReentrantLockTest.java:28)
> 	at ReentrantLockTest$MyThread.run(ReentrantLockTest.java:57)
> Thread-1被中断

### `ReadWriteLock`

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

* 一个获取读锁，一个获取写锁
* 可以将文件的读写操作分开
* 可以使多个线程同时进行读操作

### `ReentrantReadWriteLock`

#### `synchronized`读操作

```java
public class ReadWriteTest {
    public static void main(String arg[]) {
        final ReadWriteTest test = new ReadWriteTest();

        new Thread(
                () -> test.get(Thread.currentThread())
        ).start();

        new Thread(
                () -> test.get(Thread.currentThread())
        ).start();
    }

    private synchronized void get(Thread thread) {
        long start = System.currentTimeMillis();
        while (System.currentTimeMillis() - start <= 0.3) {
            System.out.println(thread.getName() + "正在进行读操作");
        }
        System.out.println(thread.getName() + "读操作完成");
    }
}
```

> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0读操作完成
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1读操作完成

#### 读写锁

```java
public class ReadWriteTest {
    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public static void main(String arg[]) {
        final ReadWriteTest test = new ReadWriteTest();

        new Thread(
                () -> test.get(Thread.currentThread())
        ).start();

        new Thread(
                () -> test.get(Thread.currentThread())
        ).start();
    }

    private void get(Thread thread) {
        lock.readLock().lock();
        try {
            long start = System.currentTimeMillis();
            while (System.currentTimeMillis() - start <= 1) {
                System.out.println(thread.getName() + "正在进行读操作");
            }
            System.out.println(thread.getName() + "读操作完成");
        } catch (Exception ignored) {

        } finally {
            lock.readLock().unlock();
        }

    }
}
```

> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-1正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-0正在进行读操作
> Thread-1正在进行读操作
> Thread-1读操作完成
> Thread-0读操作完成

threa1 和threa0 可以同时进行读操作

### `Lock`和`synchronized`

* `Lock`是一个接口,`synchronized`是关键字，`synchronized`是内置的语言实现
* `synchronized`在出现异常时，会自动释放所持有的锁，因此不会导致死锁的现象出现；`Lock`在出现异常的时候不会自动释放锁
* `Lock`可以让等待锁的线程相应中断，而`synchronized`不行
* `Lock`可以知道有没有成功获取到锁，`synchronized`不行
* `Lock`可以提高多个线程读操作的效率

从性能上看，大量线程竞争统一资源时，`Lock`的性能要高

## 锁的相关概念

### 可重入锁

* `synchronized`和`ReentrantLock`都是可重入锁
* 可重入性，表明了锁的分配机制，基于线程分配，而不是基于方法分配

```java
class MyClass {
    public synchronized void method1() {
        method2();
    }
     
    public synchronized void method2() {
         
    }
}
```

如果`synchronized`不具有可重入性，那么调用`method1`时会去获取这个对象的锁，`method2`被`method1`调用的时候又会去获取这个对象的锁，但是这个对象的锁被`method1`占用了，那么就会造成死锁

### 可中断锁

* 可以被中断的锁
* `Lock`是可中断锁（`lockInterruptybly()`），`synchronized`不是可中断锁

### 公平锁

* 一个锁被释放，多个线程等待该锁时，等待时间最长的线程优先获取到锁
* `sychronized`是非公平锁
* `Lock`以及读写锁默认是非公平锁，可以通过构造器传参实现公平锁

### 读写锁

* 将对一个资源的访问分为两个锁操作，读锁和写锁

