# JDK源码分析之——ThreadLocal

## 简介
ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储，其实意思差不多。可能很多朋友都知道ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。  

ThreadLocal的存储结构如下：

![10019_1](../img/10019_1.png)

## 源码分析

### ThreadLocalMap
ThreadLocalMap类是ThreadLocal的静态内部类。每个Thread维护一个ThreadLocalMap映射表，这个映射表的key是ThreadLocal实例本身，value是真正需要存储的Object。这样的设计主要有以下几点优势：

1. 这样设计之后每个ThreadLocalMap的Entry数量变小了：之前是Thread的数量，现在是ThreadLocal的数量，能提高性能；

2. 当Thread销毁之后对应的ThreadLocalMap也就随之销毁了，能减少内存使用量。

ThreadLocalMap就是以ThreadLocal对象为key的简易版的HashMap，通过静态内部类Entry来存储key和value，使用线性探测的方式来解决哈希冲突的问题，并且提供了基本的元素增删改查和扩容的方法。这里要讲的是ThreadLocalMap的静态内部类Entry，Entry继承了WeakReference并使用ThreadLocal的弱引用作为key，先来看一下Entry的定义：

``` java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

弱引用的对象在jvm下一次进行gc的时候就会被回收，对象内存空间就会被释放，于是就会产生这样一个问题：

ThreadLocalMap使用ThreadLocal的弱引用作为key，如果外部没有ThreadLocal对象的强引用，只是作为局部变量来使用的话，那么在下一次gc时ThreadLocal对象就会被回收，ThreadLocalMap中对应的Entry就会出现key为null的情况，这些key对应的value也将无法再访问，但是value却有一个来自当前线程的强引用，因此只有等这个线程销毁时，value才能真正的被回收。但是当我们在使用线程池的情况时，核心线程是一直在运行的，线程对象不会被回收，于是就可能出现内存泄漏的问题。

对象之间的引用结构图如下：

![10019_2](../../../silentao_blog/source/img/backend/10019/10019_2.png)

ThreadLocalMap使用了线性探测来处理哈希冲突的问题，并在这个过程中将那些key为null的Entry和对应的value清理掉了，具体实现在expungeStaleEntry方法中，可以看源码：

``` java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 将key为空的Entry和对应的value置为null
    // 以便gc时把对应的对象回收掉
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    // 同时map的数量减1
    size--;

    // 线性探测只到为null
    // 期间会清理key为null的Entry
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
            (e = tab[i]) != null;
            i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            // 将key为空的Entry和对应的value置为null
            // 以便gc时把对应的对象回收掉
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            // 调整Entry的位置
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

expungeStaleEntry方法执行完后，key为null的Entry到value对象就不会存在强引用了，在gc时Entry和value对象就会被回收，自然就不存在内存泄漏的问题。查看expungeStaleEntry方法的调用位置可以知道，在ThreadLocalMap的getEntry、set和remove方法中，都会调用expungeStaleEntry方法。当我不需要获取value也不需要set value时，还是可能会出现内存泄漏问题，这是可能就需要手动的调用ThreadLocal的remove方法来防止内存泄漏。

其实，最好的方式就是将ThreadLocal变量定义成private static的，这样的话ThreadLocal的生命周期就更长，由于一直存在ThreadLocal的强引用，所以ThreadLocal也就不会被回收，也就能保证任何时候都能根据ThreadLocal的弱引用访问到Entry的value值，然后remove它，可以防止内存泄露。

### initialValue

initialValue()方法是protected类型的，很显然是建议在子类重载该函数的，所以通常该方法都会以匿名内部类的形式被重载，以指定初始值。当调用get()方法的时候，若是与当前线程关联的ThreadLocal值已经被设置过，则不会调用initialValue()方法；否则，会调用initialValue()方法来进行初始值的设置。通常initialValue()方法每个线程只会调用一次，除非调用了remove()方法之后又调用get()方法，此时，与当前线程关联的ThreadLocal值处于没有设置过的状态（其状态体现在源码中，就是线程的ThreadLocalMap对象是否为null），initialValue()方法仍会被调用。源码如下：

``` java
protected T initialValue() {
    return null;
}
```

### get

获取与当前线程关联的ThreadLocal值，废话不多说，直接看源码：


``` java
public T get() {
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 根据当前线程对象，获取线程对于的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null) {// map不为空
        // 以当前ThreadLocal对象的引用获取map中对于的value
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            // value不为空直接返回
            return result;
        }
    }
     
    // 对象线程的ThreadLocal对象进行初始化
    return setInitialValue();
}
 
// 获取当前线程的ThreadLocalMap对象
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
 
// ThreadLocal初始化方法
private T setInitialValue() {
    // 通过调用子类重载的initialValue获取ThreadLocal关联的对象
    T value = initialValue();
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取当前线程的ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // 以当前TheadLocal对象为key，将value set到map中
        map.set(this, value);
    else
        // map为空，则创建map
        createMap(t, value);
    return value;
}
 
// 给当前线程对象创建ThreadLocalMap对象
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

ThreadLocal的实现离不开ThreadLocalMap类，get其实就是去当前线程的ThreadLocalMap里获取以ThreadLocal对象为key对应Entry的值，大致过程如下：

1. 对应的值已经存在就直接返回；
2. 否则调用initialValue方法获取初始值；
3. 如果ThreadLocalMap不为空就将初始值以ThreadLocal为key存入ThreadLocalMap对象中；
4. 否则就创建当前对象的ThreadLocalMap对象并存入初始值；
5. 最后返回初始值。

### set

设置与当前线程关联的ThreadLocal值，依然不多废话，直接看源码：

``` java
public void set(T value) {
    // 先获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取当前线程的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
            // 以当前TheadLocal对象为key，将value set到map中
        map.set(this, value);
    else
            // map为空，则创建map
        createMap(t, value);
}
```

### remove

将与当前线程关联的ThreadLocal值删除，源码如下：

``` java
public void remove() {
    // 获取当前线程的ThreadLocalMap对象
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        // 调用ThreadLocalMap的remove方法删除与当前线程关联的ThreadLocal对象
        m.remove(this);
}
```

## 总结

①ThreadLocal并不解决线程间共享数据的问题；
②ThreadLocal通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题；
③每个线程持有一个Map并维护了ThreadLocal对象与具体实例的映射，该Map由于只被持有它的线程访问，故不存在线程安全以及锁的问题；
④ThreadLocal 适用于变量在线程间隔离且在方法间共享的场景。