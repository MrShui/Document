# Java并发编程——阻塞队列

## 几种主要的阻塞队列

### `ArrayBlockingQueue`

* 基于数组实现
* 构造方法需要传入容量
* 可以指定公平性和非公平性，默认情况下为非公平（*公平性指的是等待时间长的线程优先执行*）
* 有界队列

### `LinkedBlockingQueue`

* 基于链表实现
* 默认容量为，`Integer.MAX_VALUE`
* 非公平
* 无界队列

### `PriorityBlockingQueue`

* 按线程优先级排列，优先级高的优先出队
* 无界队列

#### `DelayQueue`

* 基于`PriorityBlockingQueue`
* 元素延时的时间到了，才会出队

## 阻塞队列和非阻塞队列中的方法

### 非阻塞队列

* `add(e)`、`remove()`:添加删除元素，添加失败，抛出异常
* `offer(e)`、`poll()`:添加删除元素，添加失败，返回`false`
* `peek()`:取元素看一眼

### 阻塞队列

* `put(e)`、`take()`:存取元素，若队列满或者空则等待
* `offer`、`poll`：存取元素，等待一段时间操作不成功就会返回`false`

## 应用

生产者消费者