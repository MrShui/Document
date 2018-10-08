#  Java并发编程——生产者消费者

* 生产者持续生产，直到缓冲区满，阻塞；缓冲区不满后，继续生产
* 消费者持续消费，直到缓冲区空，阻塞；缓冲区不为空之后，继续消费
* 生产者和消费者都可以有多个

## 相关基类

```java
public interface Consumer extends Runnable {
    void consume() throw InterruptedException;
}
```

```java
public interface Producer extends Runnable {
    void produce() throw InterruptedException;
}
```

```java
public abstract class AbsConsumer implements Consumer {
    @override
    public void run() {
        try {
            consume();
        }catch(InterruptedException e) {
            e.printStrack();
        }
    }
}
```

```java
public abstract class AbsProducer implements Producer {
    @override
    public void run() {
        try {
            produce();
        } catch(InterruptedException e) {
            e.printStrack();
        }
    }
}
```

```java
public interface Model {
    Runnable newProducerRunnable();
    
    Runnable newConsumerRunnable();
}
```

## `BlockingQueue`实现

```java
public class BlockingQueueModel implements Model {
    prvate final BlockingQueue<Task> queue;
    private final AtomicInteger atomInt = new AtomicInteger(0);
    
    public BlickingQueueModel(int cap) {
        queue = new LinkedBlockingQueue<>(cap);
    }
    
    @override
    public Runnable newProducerRunnable() {
        return new ProducerImpl();
    }
    
    @override
    public Runnable newConsunerRunnable() {
        return new ConsumerImpl();
    }
    
    class ProducerImpl extends AbsProducer {
        @override
        public void produce() throw InterruptedException {
            Thread.sleep(500+500*Math.random());
            Task task = new Task(atomInt.incrementAndGet());
            queue.add(task);
            System.out.println("produce: "+task.no);
        }
    }
    
    class ConsumerImpl extends AbsConsumer {
        Thread.sleep(1000*Math.random());
        Task task = queue.take();
        System.out.println("consume: "+task.no);
    }
    
    public static void main(String[] arg) {
        Model model = new BlockingQueueModel();
        for(int i = 0;i < 3 ; i++) {
        	new Thread(model.newProducerRunnable()).start();    
        }
        
        for(int i = 0; i < 5 ; i++) {
        	new Thread(model.newConsumerRunnable()).start();            
        }
    }
}
```

## `wait`、`notify`实现

```java
public class WaitModel implements Model {
    private final Queue<Task> queue = new LinkedList();
    private final int cap;
    private final int AtomicInteger atomInt = new AtomicInteger(0);
    
    public WaitModel(int cap) {
        this.cap = cap;
    }
    
    @override
    public Runable newConsumerRunnable() {
        return ConsumerImpl();
    }
    
    @override
    public Runnable newProducerRunnable() {
        return ProducerImpl();
    }
    
    class ConsumerImpl extends AbsConsumer {
        @override
        public void consume() throw InterruptedException {
            while(queue.size == 0) {
                Task task = queue.pull();
                System.out.println("consume: "+ task.no);
            }
        }
    }
    
    class ProducerImpl extends AbsProducer {
        @ovveride
        public void produce() throw InterruptedException {
            while(queue.size == 3) {
                Task task = new Task(atomInt.incrementAndGet());
                queue.offer(task);
                System.out.println("produce: "+task.no);
            }
        }
    }
    
    public static void main(String[] arg) {
        
    }
}
```

