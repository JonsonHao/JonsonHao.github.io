---
layout:     post
title:      ThreadLocal 原理分析
subtitle:   从源码角度理解ThreadLocal
date:       2020-05-10
author:     JonsonHao
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Handler
---


## 前言

> ThreadLocal 是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定的线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据

在 **Android 的消息机制**中，每个线程的 **Loopera** 实例都是不同的，各个线程之间的 Looper 都在独立的轮询消息队列，它们之间没有任何的交集，互不干扰，这其实就是通过 **ThreadLocal** 来实现的：

```
public static @Nullable Looper myLooper() {
    //从ThreadLocal 中获取 Looper 实例
    return sThreadLocal.get();
}
```

何时可以使用 ThreadLocal？下面借用《Android开发艺术探索》中的一句话来说明：

> 一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用 ThreadLocal

下面通过实际的例子来演示 ThreadLocal 的含义：

```
/**
 * @author: created by linjunhao
 * @date: 2020-03-28 14:56
 * @description:
 */
public class ThreadLocalTest {

    private static ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        mBooleanThreadLocal.set(true);
        System.out.println("[Thread#main]mBooleanThreadLocal=" + mBooleanThreadLocal.get());

        new Thread("Thread#1") {

            @Override
            public void run() {
                mBooleanThreadLocal.set(false);
                System.out.println("[Thread#1]mBooleanThreadLocal=" + mBooleanThreadLocal.get());

            }
        }.start();

        new Thread("Thread#2") {
            @Override
            public void run() {
                System.out.println("[Thread#1]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
            }
        }.start();
    }

}
```

运行结果：

```
[Thread#main]mBooleanThreadLocal=true
[Thread#1]mBooleanThreadLocal=false
[Thread#1]mBooleanThreadLocal=null
```

可以看到，虽然我们在不同线程中调用的是同一个 ThreadLocal 对象，但是它们通过 ThreadLocal 获取到的值却是不一样的。接下来我们看看 ThreadLocal 内部是如何实现的吧，**了解 ThreadLocal 是如何做到同一个实例，维护着不同线程的数据副本的**

## 源码分析

### ThreadLocal

ThreadLocal 是一个泛型类，支持存储各种数据类型，它对外暴露的方法很少，基本只有下面三个：

```
set()
get()
remove()
```

#### set()

首先，我们先看一下 ThreadLocal 是如何把数据存储进去的：

```
// ThreadLocal#set()
public void set(T value) {
    //1.获取当前线程
    Thread t = Thread.currentThread();
    //2.以当前线程作为参数，获取一个 ThreadLocalMap 对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
        //3.map 不为空，则直接把数据存进去
        map.set(this, value);
    else
        //4.map 为空，则创建一个 map
        createMap(t, value);
}
```

从上面的代码可以看到，ThreadLocalMap 应该就是存储数据的容器类。接下来看一下 getMap() 方法：

```
//ThreadLocal#getMap()
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

getMap() 方法直接返回线程的成员变量 **threadLocals**，这个变量是 ThreadLocalMap 类型的。因为 ThreadLocal 和 Thread 是在同一个包下，所以 ThreaLocal 可以直接访问 Thread 类的包访问权限的成员变量 threadLocals，并且 Thread 类中的 threadLocals **默认是 null** 的，在 Thread 类中没有任何赋值的地方，那它在哪里赋值呢？我们看一下 createMap() 方法：

```
//ThreadLocal#createMap()
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

也就是说，threadLocals 只有在 ThreadLocal 中的 createMap() 方法中赋值

以上，set 方法就分析完了，虽然我们还不知道 ThreadLocalMap 作用是什么，但是从 map.set() 可以猜测出，ThreadLocalMap 应该就是一个存储数据的容器，我们就先暂时这么认为，后面会详细了解一下这个类。接下来我们看一下 get() 方法：

#### get()

```
//ThreadLocal#get()
public T get() {
    //1.获取当前线程实例
    Thread t = Thread.currentThread();
    //2.以当前线程为参数，获取 ThreadLocal
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //3.map 不为空，则以当前 ThreadLocal 实例为key参数，从map中获取值
        ThreadLocalMap.Entry e = map.getEntry(this);
        //有找到值则直接返回
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //4.如果map为空或map中取不到值，则返回默认值
    return setInitialValue();
}
```

同样的，get 方法也是先获取到当前线程实例后拿到线程的 ThreadLocalMap 对象，然后从中获取值，这也就更加证明了我们前面的猜测，ThreaLocalMap 就是一个类似 HashMap 的容器类（不过它和 HashMap 不同）。接着，我们往下看 setInitialValue 方法：

```
//ThreadLocal#setInitialValue()
private T setInitialValue() {
    //获取默认值
    T value = initialValue();
    //获取当前线程对象
    Thread t = Thread.currentThread();
    //以当前线程为参数，获取 ThreadLocal
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    //返回默认值
    return value;
}

//ThreadLocal#initialValue()
protected T initialValue() {
    return null;
}
```

setInitialValue() 除了调用 initialValue() 获取默认值然后返回之外，其他代码逻辑基本和 set() 方法一致。initialValue 默认返回是 null，这个方法用户是可以重写的，然后返回你想要返回的默认值。

#### remove()

```
public void remove() {
    //根据当前线程实例获取 ThreadLocalMap
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        // map 不为空就根据 key 移除对应的值
        m.remove(this);
}
```

到这里是不是对 ThreadLocal 的原理有了较为深刻的理解了。**ThreadLocal 是如何维护不同线程的数据副本的？原来，这些数据本来就是存储在各自的线程当中的。**接下来，我们去揭开 ThreadLocalMap 的面纱，验证一下，ThreadLocalMap 是不是一个用于存储数据的容器类：

### ThreadLocalMap

ThreadLocalMap 其实是 ThreaLocal 的静态内部类，看源码可以发现，其内部数据结构其实就是一个**哈希表**，初始容量为**16**，数组元素为 **Entry**：

```
//ThreadLocal#ThreadLocalMap
static class ThreadLocalMap {

    //Entry
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    private Entry[] table;
    //初始容量
    private static final int INITIAL_CAPACITY = 16;
    private int size = 0;

    private void set(ThreadLocal key, Object value) {
        ...
    }

    private Entry getEntry(ThreadLocal key) {
        ...   
    }
}
```

**Entry** 继承了 **WeakReference<ThreadLocal<?>>**，我们可以把它看成一个健值对，健是当前 ThreaLocal 对象，值是存储的对象

### 参考文献

* [带你了解源码中的 ThreadLocal](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650243742&idx=1&sn=27de324a1685ebbb112a239777eb7f52&chksm=886373f1bf14fae7c7515c76d65707dc6f026996c7cbfe3df16a52d86ced2904e154941f744f&scene=38#wechat_redirect)
* [从 ThreadLocal 的实现看散列算法](https://blog.csdn.net/y4x5M0nivSrJaY3X92c/article/details/81124944)
* [Java | ThreadLocal与无锁线程安全](https://www.jianshu.com/p/ed1eae5f7dfd)
* [通俗易懂弄清ThreadLocal内部原理](https://juejin.im/post/5e26e2636fb9a030051f02dd)
* 《Android开发艺术探索》 任玉刚 著

