# JDK源码分析之——ReentrantLock

## 简介

`ReentrantLock` 是 `Lock` 接口的实现类，是一把可以支持重入的锁，静态内部抽象类 `Sync` 继承了 AQS（`AbstractQueuedSynchronizer`），`Sync` 的子类 `FairSync`、`NonfairSync` 分别实现了公平锁和非公平锁。与 `synchronized` 相比，重入锁有着显式的操作过程，开发人员必须手动指定何时加锁，何时释放锁。所以，重入锁对逻辑控制的灵活性要远远好于 `synchronized`。`ReentrantLock` 具有以下特点：

1. 支持重入性，能够对共享资源重复加锁，即获得锁的线程再次获取锁时不需要被阻塞；
2. 支持公平锁和非公平锁；
3. 支持中断响应；
4. 支持锁申请等待限时；
5. 支持锁申请快速失败。

以上特点都会在源码中一一体现，在注释中也做了简单的说明，细心的同学一定会发现哦。

## 1.构造方法

`ReentrantLock` 有两个构造方法，无参构造方法默认是创建非公平锁，入参 boolean 类型的构造方法通过 true 和 false 决定创建什么样的锁，其中 true 创建公平锁，false 创建非公平锁。其中公平锁和非公平锁由静态内部类 `FairSync` 和 `NonfairSync` 实现（具体实现下面会写）,构造方法源码如下：

``` java
/**
 * 无参构造方法默认是创建非公平锁
 */
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * 有参构造方法
 * true创建公平锁，false创建非公平锁
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

## 2.Sync

静态内部抽象类 `Sync` 继承了 AQS，并实现了 AQS 的独占模式，这里只重写了 `tryRelease` 方法，`tryAcquire` 方法都是放在子类中去重写的，具体源码解析如下：

``` java
abstract static class Sync extends AbstractQueuedSynchronizer {

    /**
     * 这里定义为抽象方法的lock方法
     * 是为非公平版本提供快速路径
     */
    abstract void lock();

    /**
     * 在这里实现非公平锁的tryAcquire方法
     * 是因为两种方式的锁对应的tryLock方法都需要调用这个方法
     * 而两种方式的锁真正的tryAcquire方法都将在子类中实现
     */
    final boolean nonfairTryAcquire(int acquires) {
        // 获取当前线程对象，用于后面比较锁的持有对象是否为当前线程
        final Thread current = Thread.currentThread();
        // 获取state值，state在这里表示加锁的次数
        int c = getState();
        if (c == 0) {
            // state为0表示当前锁没有被任何线程持有
            // 通过CAS的方式尝试去修改state的值
            if (compareAndSetState(0, acquires)) {
                // 修改成功表示获得锁
                // 把当前线程对象赋值给exclusiveOwnerThread变量
                // 标记当前持有锁的线程对象
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            // 锁被当前线程持有
            // 就在state加上acquires（重入的概念）
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            
            // 修改state的值
            setState(nextc);
            return true;
        }

        // 获取锁失败
        return false;
    }

    /**
     * 释放锁，其实就是修改state的值
     * 每释放一次锁state就减releases
     * 加了几次锁，就得释放几次锁
     * state == 0时锁才完全释放
     */
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            // 持有锁的不是当前线程支持抛出异常
            throw new IllegalMonitorStateException();
        
        boolean free = false;
        if (c == 0) {
            // 如果state == 0表示锁已经完全释放
            free = true;
            // 将持有锁定的标记置为null
            setExclusiveOwnerThread(null);
        }

        // 修改state的值
        setState(c);
        return free;
    }

    /**
     * 判断持有锁的线程是否为当前线程
     */
    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }
}
```

## 3.NonfairSync

非公平锁的实现类 `NonfairSync`，重写了 `lock` 方法和 `tryAcquire` 方法，源码如下：

``` java
static final class NonfairSync extends Sync {

    /**
     * 非公平锁调用加锁
     */
    final void lock() {
        // 不判断当前锁是否已经被占有
        // 尝试直接获得锁，有可能抢在同步队列中的线程之前获得锁
        // 体现了非公平锁的概念
        if (compareAndSetState(0, 1))
            // 成功获得锁
            // 把当前线程对象赋值给exclusiveOwnerThread变量
            // 标记当前持有锁的线程对象
            setExclusiveOwnerThread(Thread.currentThread());
        else
            // 调用AQS的acquire方法获得锁
            acquire(1);
    }

    /**
     * 非公平锁加锁，其实就是修改state的值
     * 每加一次锁state就加1
     * 具体实现在nonfairTryAcquire方法中
     */
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

## 4.FairSync

公平锁的实现类 `FairSync`，也重写了 `lock` 方法和 `tryAcquire` 方法，源码如下：

``` java
static final class FairSync extends Sync {

    /**
     * 公平锁调用加锁
     * 直接调用AQS的acquire方法获得锁
     */
    final void lock() {
        acquire(1);
    }

    /**
     * 公平锁加锁
     * 没有其它线程在等待锁，或者当前线程是同步队列的第一个
     * 才有资格尝试去获得锁
     */
    protected final boolean tryAcquire(int acquires) {
        // 获取当前线程对象，用于后面比较锁的持有对象是否为当前线程
        final Thread current = Thread.currentThread();
        // 获取state值，state在这里表示加锁的次数
        int c = getState();
        if (c == 0) {
            // state为0表示当前锁没有被任何线程持有
            // 在没有其它线程在等待锁
            // 或者当前线程是同步队列的第一个时
            // 才通过CAS的方式尝试去修改state的值
            // 体现了公平锁的概念
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                // 修改成功表示获得锁
                // 把当前线程对象赋值给exclusiveOwnerThread变量
                // 标记当前持有锁的线程对象
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            // 锁被当前线程持有
            // 就在state加上acquires（重入的概念）
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");

            // 修改state的值
            setState(nextc);
            return true;
        }

        // 获取锁失败
        return false;
    }
}

/**
 * 判断当前线程之前是否有线程在等待锁
 * 如果当前线程之前有一个排队的线程返回true
 * 如果当前线程位于同步队列的开头或队列为空返回false
 */
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

## 5.ReentrantLock的常用方法

### 5.1 加锁

加锁方式有好几种，大致可分为直接加锁、支持中断响应的加锁、支持快速失败的加锁、支持超时的加锁，因为具体加锁的实现都已经由 AQS 和 `ReentrantLock` 的内部类 `Sync`、`NonfairSync` 和 `FairSync` 实现好饿了，所以 `ReentrantLock` 加锁就变得很简单，具体如下：

``` java
/**
 * 直接加锁
 */
public void lock() {
    sync.lock();
}

/**
 * 支持中断响应的加锁
 */
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}

/**
 * 支持快速失败的加锁
 */
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}

/**
 * 支持超时的加锁
 */
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

### 5.2 释放锁

释放锁的方式就一种，也很简单，源码如下：

``` java
/**
 * 释放锁
 */
public void unlock() {
    sync.release(1);
}
```

## 6.总结

重入锁使用 `java.util.concurrent.locks.ReentrantLock` 类来实现的，它完全可以替代 `synchronized` 关键字。与 `synchronized` 相比，重入锁有着显式的操作过程，开发人员必须手动指定何时加锁，何时释放锁。所以，重入锁对逻辑控制的灵活性要远远好于 `synchronized`。重入锁对于一个线程是可以反复进入的，也就是说，一个线程可以多次获得同一把锁，只不过需要注意的是，如果同一个线程多次获得锁，那么在释放锁的时候，也必须释放相同的次数。在 `ReentrantLock` 的实现中，主要包含三个要素：原子状态，原子状态使用 CAS 操作来存储当前锁的状态，判断是否已经被别的线程持有；同步队列，所有没有请求到锁的线程，会进入同步队列进行等待。待有线程释放锁后，系统就能从同步队列中唤醒一个线程，继续工作；阻塞原语 `park()` 和 `unpark()`，用来挂起和恢复线程，没有得到锁的线程将会被挂起。
