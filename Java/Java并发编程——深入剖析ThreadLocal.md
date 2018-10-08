# Java并发编程——深入剖析`ThreadLocal`

* android `handler`相关就用到了`ThreadLocal`
* 线程本地变量，线程本地存储
* `ThreadLocal`为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量

## 深入解析`ThreadLocal`

```java
public T get();
public void set(T value);
public void remove();
protected T initialValue();
```

* `get()`:获取`ThreadLocal`在当前线程中保存的变量副本
* `set()`:用来设置当前线程中变量的副本
* `remove()`:移除当前线程中的变量副本
* `initialValue()`:延迟加载

### 🌰

```java
public class Test {
    ThreadLocal<Long> longLocal = new ThreadLocal<>();
    ThreadLocal<String> stringLocal = new ThreadLocal<>();

    public void set() {
        longLocal.set(Thread.currentThread().getId());
        stringLocal.set(Thread.currentThread().getName());
    }

    public long getLong() {
        return longLocal.get();
    }

    public String getString() {
        return stringLocal.get();
    }

    public static void main(String arg[]) throws InterruptedException {
        final Test test = new Test();
        test.set();

        System.out.println(test.getLong());
        System.out.println(test.getString());

        Thread thread1 = new Thread(() -> {
            test.set();

            System.out.println(test.getLong());
            System.out.println(test.getString());
        });
        thread1.start();
        thread1.join();

        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}
```

> 1
> main
> 11
> Thread-0
> 1
> main

==使用`get()`之前必须先`set()`，否则需要重写`initValue()`==

### `initValue()`用法

```java
public class Test {
    ThreadLocal<Long> longLocal = new ThreadLocal<Long>() {
        @Override
        protected Long initialValue() {
            return Thread.currentThread().getId();
        }
    };
    ThreadLocal<String> stringLocal = new ThreadLocal<String>() {
        @Override
        protected String initialValue() {
            return Thread.currentThread().getName();
        }
    };

    public long getLong() {
        return longLocal.get();
    }

    public String getString() {
        return stringLocal.get();
    }

    public static void main(String arg[]) throws InterruptedException {
        final Test test = new Test();
        System.out.println(test.getLong());
        System.out.println(test.getString());

        Thread thread1 = new Thread(() -> {
            System.out.println(test.getLong());
            System.out.println(test.getString());
        });
        thread1.start();
        thread1.join();

        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}
```

> 1
> main
> 11
> Thread-0
> 1
> main

## 应用场景

### 解决数据库连接

```java
private static ThreadLocal<Connection> connectionHolder
        = new ThreadLocal<Connection>() {
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};

public static Connection getConnection() {
    return connectionHolder.get();
}
```

### `Session`管理

```java
private static final ThreadLocal threadSession = new ThreadLocal();
 
public static Session getSession() throws InfrastructureException {
    Session s = (Session) threadSession.get();
    try {
        if (s == null) {
            s = getSessionFactory().openSession();
            threadSession.set(s);
        }
    } catch (HibernateException ex) {
        throw new InfrastructureException(ex);
    }
    return s;
}
```

### android中的`handler`机制

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
}

/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

