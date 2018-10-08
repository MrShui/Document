# Java并发编程——3个并发类

## `CountDownLatch`

提供一个计数器，计数器倒计完毕，会继续执行因为`await`被阻塞的线程

* `public void await() throw InterruptedException`:线程阻塞，知道倒计数为0
* `public void await(long time,TimeUnit unit)`:同上，等待time时间后线程会被自动唤醒
* `public void countDown()`:将`countDown`减1

## `CyclicBarrier`

回环栅栏

```java
public class CyclicBarrierTest {
    public static void main(String arg[]) {
        int N = 4;
        CyclicBarrier barrier = new CyclicBarrier(N);
        for (int i = 0; i < N; i++) {
            new Writer(barrier).start();
        }
    }

    static class Writer extends Thread {
        private CyclicBarrier cyclicBarrier;

        Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程：" + Thread.currentThread().getName() + "正在写入数据...");
            try {
                Thread.sleep((long) (5000 * Math.random()));
                System.out.println("线程：" + Thread.currentThread().getName() + "写入数据完毕，等待其他线程写入数据完毕");
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("线程：" + Thread.currentThread().getName() + "所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

> 线程：Thread-0正在写入数据...
> 线程：Thread-3正在写入数据...
> 线程：Thread-2正在写入数据...
> 线程：Thread-1正在写入数据...
> 线程：Thread-1写入数据完毕，等待其他线程写入数据完毕
> 线程：Thread-2写入数据完毕，等待其他线程写入数据完毕
> 线程：Thread-0写入数据完毕，等待其他线程写入数据完毕
> 线程：Thread-3写入数据完毕，等待其他线程写入数据完毕
> 线程：Thread-3所有线程写入完毕，继续处理其他任务...
> 线程：Thread-1所有线程写入完毕，继续处理其他任务...
> 线程：Thread-2所有线程写入完毕，继续处理其他任务...
> 线程：Thread-0所有线程写入完毕，继续处理其他任务...

## `Semaphore`

信号量

```java
public class SemaphoreTest {
    public static void main(String arg[]) {
        int N = 8;//工人数量
        Semaphore semaphore = new Semaphore(5);//机器数量
        for (int i = 0; i < N; i++) {
            new Worker(i, semaphore).start();
        }
    }

    static class Worker extends Thread {
        private int num;
        private Semaphore semaphore;

        public Worker(int num, Semaphore semaphore) {
            this.num = num;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人" + this.num + "占用一个机器再生产...");
                Thread.sleep((long) (2000 * Math.random()));
                System.out.println("工人" + this.num + "释放出机器");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

> 工人1占用一个机器再生产...
> 工人4占用一个机器再生产...
> 工人3占用一个机器再生产...
> 工人0占用一个机器再生产...
> 工人2占用一个机器再生产...
> 工人3释放出机器
> 工人5占用一个机器再生产...
> 工人4释放出机器
> 工人6占用一个机器再生产...
> 工人1释放出机器
> 工人7占用一个机器再生产...
> 工人7释放出机器
> 工人0释放出机器
> 工人6释放出机器
> 工人2释放出机器
> 工人5释放出机器

## 应用场景

* `CountDownLatch`用于某个线程等待其他线程执行完毕，继续执行
* `CyclicBarrier`用于一组线程互相等待至某个状态，然后一组线程同时执行
* `Semaphore`和锁类似，控制对某组资源的访问权限