# Java并发编程——`volatile`关键字解析

[读此博客笔记]: https://www.cnblogs.com/dolphin0520/p/3920373.html	"博客笔记"



## 一、内存模型相关

> 运行程序过程中的临时数据会被放入到主存（物理内存）中，由于cpu执行的速度太过，主存的读写会降低cpu指令的运行效率，所以cpu中会存在告诉缓存（工作内存）
>
> 程序运行时，cpu会将主存中数据复制一份到高速缓存中，执行完毕后会将最新的数据刷新到主存中
>
> 如果变量在多个cpu都存在缓存，就会导致缓存不一致问题

### 解决缓存不一致

#### 加锁（早期）

通过加锁，使得只有一个cpu可以使用共享变量

#### 协议

Intel的MESI协议。cpu写数据的时候，如果发现该数据在其他cpu中也有缓存，则会将其他cpu中的缓存置为无效状态。当cpu发现当前变量缓存失效，就会去主存中读取最新的数据

![cpu缓存](../img/cpu缓存.png)

## 二、并发的三个概念

### 原子性

要么全部执行，要么全部不执行，类似于数据库中的事务

### 可见性

多个线程访问同一个变量，一个线程修改了这个变量，其他线程能够立即看到修改后的值

### 有序性

程序按照代码的先后顺序执行。*编译期、cpu、运行时都可能对指令进行重排序（重排序）*

## 三、Java内存模型（`JMM`）

> * `JMM`存在缓存不一致和重排序问题
> * `JMM`规定所有的变量都是存在主存中
> * 每个线程都有自己的工作内存，线程对变量的操作都必须在各自的工作线程中，而不能直接对主存操作
> * 一个线程不能访问另一个线程的工作内存

#### 原子性

```java
x = 10;       //语句1
y = x;        //语句2
x++;          //语句3
x = x + 1;    //语句4
```

* 语句1具有原子性
* 语句2不具有原子性，因为先读在写，复合在一起就不具有原子性
* 语句3、4不具有原子性

**貌似64位的`double`、`long`的读写操作是非原子的，需要使用`volatile`**

#### 可见性

* 普通的共享变量什么时候从工作内存中被写入到主存中是不确定的
* `volatile`可以保证变量的可见性，被`volatile`修饰的变量的修改会被立即更新到主存中，当其他线程需要读取改变量，则会从主存中读取最新的值
* `synchronized`和`Lock`也能保证可见性。它们会保证同一时间只有获取锁的线程执行同步代码块，释放锁之后将数据更新到主存中

#### 有序性

* 重排序在单线程执行环境中不会出错，多线程中会存在问题

* `volatile`可以保证有序性
* `synchronized`和`Lock`保证有序性

## 四、深入剖析`volatile`关键字

### `volatile`的两层语义

* 保证可见性
* 保证有序性，进制指令重排序

```java
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
 
//线程2
stop = true;
```

用上面代码中断线程执行有问题。

线程2中修改`stop`值为`true`，还没有来得及更改主存中的值为`true`，就会导致线程1没有停止

`stop`关键字使用`volatile`关键字之后

* 线程2修改`stop`值后会立即写入到主存中
* 线程1读取`stop`值，发现工作内存中的`stop`值失效了，然后回去主存中读取

### `volatile`可以保证原子性吗

```java
public class Test {
    public volatile int inc = 0;
     
    public void increase() {
        inc++;
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

上面的结果会是小于10000，因为`volatile`可以保证可见性，但是不能保证操作的原子性

某个时刻`inc`为10

`thread1`执行`increase()`时，读取到`inc`然后被阻塞了

`thread2`执行`increase()`时，读取到`inc`，值为10，执行之后为11

`thread1`运行之后，inc被改为11

两个线程写入主存中，最后inc结果为11

#### 使用`synchronized`关键字

```java
public class Test {
    public int inc = 0;
     
    public synchronized void increase() {
        inc++;
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

#### 使用`Lock`

```java
public class Test {
    public int inc = 0;
     
 	public void increase() {
        inc++;
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

#### 使用`AtomicInteger`

```java
public class Test {
    public AtomicInteger inc = new AtomicInteger();
     
    public void increase() {
        inc.getAndIncrement();
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

### `volatile`可以保证有序性吗

```java
//x、y为非volatile变量
//flag为volatile变量
 
x = 2;        //语句1
y = 0;        //语句2
flag = true;  //语句3
x = 4;         //语句4
y = -1;       //语句5
```

* 语句3的执行不会再语句1、语句2之前，不会再语句4、语句5之后
* 语句1、语句2之间，以及语句4、语句5之间的执行顺序无法保证
* 语句1、2的执行结果对语句3、4、5可见

### `volatile`的原理和实现机制

> “观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”——《深入理解Java虚拟机》

lock前缀为内存屏障，也叫内存栅栏

* 内存屏障会保证指令排序不会吧内存屏障后面的指令排到内存屏障的前面，也不会吧内存屏障前面的指令排到内存屏障的后面
* 它会强制将对缓存的修改操作写入到主存中
* 如果是写操作，会导致其他cpu中的缓存行失效

## 五、`volatile`使用场景

### 使用条件

* 对变量的写入操作不依赖与变量的当前值
* 该变量没有包含在具有其他变量的不定式中

### 使用场景

#### 状态标记量

```java
volatile boolean flag = false;
while(!flag){
    doSomething();
}

public void setFlag(boolean flag){
    this.flag = flag;
}
```

#### `double check`

。。。

