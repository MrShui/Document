# Java并发编程——同步容器

## Java中的同步容器类

### `Vector`

实现`List`接口，类似`ArrayList`，线程安全

### `Stack`

继承`Vector`，线程安全

### `HashTable`

实现了`Map`接口,类似`HashMap`，线程安全

### `Collections`

可以通过`Collections.synchronizedList()`等类似方法创建线程安全的容器

## 同步容器缺陷

* 执行效率低
* 多个线程不能同时读取

```java
public class Test {
    static Vector<Integer> vector = new Vector<>();

    public static void main(String arg[]) {
        while (true) {
            for (int i = 0; i < 10; i++) {
                vector.add(i);
            }

            Thread thread1 = new Thread(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.remove(i);
                }
            });
            Thread thread2 = new Thread(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.get(i);
                }
            });
            thread1.start();
            thread2.start();

            while (Thread.activeCount() > 10) {

            }
        }
    }
}
```

> objc[6503]: Class JavaLaunchHelper is implemented in both /Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/bin/java (0x1067684c0) and /Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/libinstrument.dylib (0x1067f44e0). One of the two will be used. Which one is undefined.
> Exception in thread "Thread-495" Exception in thread "Thread-497" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 8
> 	at java.util.Vector.get(Vector.java:748)
> 	at threadlocal.Test.lambda$main$1(Test.java:24)
> 	at java.lang.Thread.run(Thread.java:745)
> java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 10
> 	at java.util.Vector.get(Vector.java:748)
> 	at threadlocal.Test.lambda$main$1(Test.java:24)
> 	at java.lang.Thread.run(Thread.java:745)
> Exception in thread "Thread-2774227" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 16
> 	at java.util.Vector.get(Vector.java:748)
> 	at threadlocal.Test.lambda$main$1(Test.java:24)
> 	at java.lang.Thread.run(Thread.java:745)

之所以报错，是因为同步容器只是保证了在同一时间只有一个线程可以执行`add()`、`remove()`等方法。但是还是可以在`add()`方法执行的同时执行`remove()`

## `Java ConcurrentModificationException`

### `ConcurrentModificationException`异常出现的原因

```java
public class Test {

    public static void main(String arg[]) {
        List<Integer> list = new ArrayList<>();
        list.add(2);
        for (Integer integer : list) {
            if (integer == 2) {
                list.remove(integer);
            }
        }
    }
}
```

> Exception in thread "main" java.util.ConcurrentModificationException
> 	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
> 	at java.util.ArrayList$Itr.next(ArrayList.java:851)
> 	at threadlocal.Test.main(Test.java:15)

#### 解析

`modCount`为1，而`expectedModCount`为0，因此程序就抛出了`ConcurrentModificationException`异常

#### 修改

##### 单线程

```java
public static void main(String arg[]) {
    List<Integer> list = new ArrayList<>();
    list.add(2);
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
        if (iterator.next() == 2) {
            iterator.remove();
        }
    }

}
```

##### 多线程

```java
public class Test {
    static List<Integer> list = new ArrayList<>();

    public static void main(String arg[]) {
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                Iterator<Integer> iterator = list.iterator();
                while (iterator.hasNext()) {
                    Integer integer = iterator.next();
                    System.out.println(integer);

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                Iterator<Integer> iterator = list.iterator();
                while (iterator.hasNext()) {
                    Integer integer = iterator.next();

                    if (integer == 2) {
                        iterator.remove();
                    }
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

> Exception in thread "Thread-0" java.util.ConcurrentModificationException
> 	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
> 	at java.util.ArrayList$Itr.next(ArrayList.java:851)
> 	at Test$1.run(Test.java:23)
> 	at java.lang.Thread.run(Thread.java:745)

虽然使用了`iterator.remove();`，但是不同线程中的`iterator`不同

##### 解决方法

* 同步
* `CopyOnWriteArrayList`

### `CopyOnWriteArrayList`

写时复制容器

写的时候，加锁将容器copy一份，在copy出来的新容器上操作，操作完毕之后将引用执行新容器

#### `CopyOnWriteMap`

```java
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * 写时复制map
 * Created by Shui on 2018/7/24.
 */
public class CopyOnWriteMap<K, V> implements Map<K, V>, Cloneable {
    private volatile Map<K, V> internalMap;

    public CopyOnWriteMap() {
        internalMap = new HashMap<>();
    }

    @Override
    public V put(K key, V value) {
        synchronized (this) {
            HashMap<K, V> newMap = new HashMap<>(internalMap);
            V val = newMap.put(key, value);
            internalMap = newMap;
            return val;
        }
    }


    @Override
    public V get(Object key) {
        return internalMap.get(key);
    }

    @Override
    public void putAll(Map<? extends K, ? extends V> m) {
        synchronized (this) {
            HashMap<K, V> newMap = new HashMap<>(internalMap);
            newMap.putAll(m);
            internalMap = newMap;
        }
    }

    @Override
    public int size() {
        return internalMap.size();
    }

    @Override
    public boolean isEmpty() {
        return internalMap.size() == 0;
    }

    @Override
    public boolean containsKey(Object key) {
        return internalMap.containsKey(key);
    }

    @Override
    public boolean containsValue(Object value) {
        return internalMap.containsValue(value);
    }

    @Override
    public V remove(Object key) {
        return internalMap.remove(key);
    }

    @Override
    public void clear() {
        internalMap.clear();
    }

    @Override
    public Set<K> keySet() {
        return internalMap.keySet();
    }

    @Override
    public Collection<V> values() {
        return internalMap.values();
    }

    @Override
    public Set<Entry<K, V>> entrySet() {
        return internalMap.entrySet();
    }
}
```

#### 缺点

* 内存占用
* 写入无法立即读取

### `ConcurrentHashMap`