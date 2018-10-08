# Java并发编程——线程池

## `ThreadPoolExecutor`

* `corePoolSize`:核心池大小，当有任务时，就会创建线程，当线程数量达到了核心池大小的时候，就会把任务放入到缓存队列中
* `maximumPoolSize`:线程池最大线程数，表示线程中最多创建多少个线程
* `keepAliveTime`:表示线程没有执行任务时最多保持多长时间会终止==线程池中，线程数量大于`corePoolSize`时才会生效==
* `unit`:`keepAliveTime`的单位
  * `TimeUnit.DAYS`:天
  * `TimeUnit.HOURS`:小时
  * `TimeUnit.MINUTES`:分钟
  * `TimeUnit.SECONDS`:秒
  * `TimeUnit.MILLISECONDS`:毫秒
  * `TimeUnit.MICROSECONDS`:微秒
  * `TimeUnit.NANOSECONDS`:纳秒
* `wordQueue`:阻塞队列，用来存储等待执行的任务
* `ThreadFactory`:线程工厂，主要用来创建线程
* `handler`:拒绝任务时的策略

## 新提交一个任务时

* `poolSize<corePoolSize`:创建一个新的进程
* `poolSize==corePoolSize`:将新的任务缓存到阻塞队列中
* 阻塞队列达到上限&&`poolSize<maximumPoolSize`:新增线程处理任务
* `poolSize==maximumPoolSize`:拒绝任务策略

