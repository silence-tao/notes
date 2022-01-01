# JDK源码分析之——FutureTask

## 简介

FutureTask是提供了一个可取消的异步计算的过程类，FutureTask实现了Future的基本方法，提供了start和cancel操作，可以查询计算是否已经完成，并且可以获取计算的结果。结果只可以在计算完成之后获取，当计算没有完成的时候get方法会阻塞，一旦计算已经完成，那么计算就不能再次启动或是取消。一个FutureTask可以用来包装一个Callable或是一个Runnable对象。因为FurtureTask实现了Runnable方法，所以一个FutureTask可以提交（submit）给一个Excutor执行（excution）。

FutureTask可用于异步获取执行结果或取消执行任务的场景。通过传入Runnable或者Callable的任务给FutureTask，直接调用其run方法或者放入线程池执行，之后可以在外部通过FutureTask的get方法异步获取执行结果，因此，FutureTask非常适合用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。另外，FutureTask还可以确保即使调用了多次run方法，它都只会执行一次Runnable或者Callable任务，或者通过cancel取消FutureTask的执行等。

在获取任务执行结果时，FutureTask内部维护了一个由WaitNode类实现的简单链表，它保存了所有等待返回数据的线程，并在结果返回之前将这些线程挂起，只有等任务执行完成或者等待超时时才会唤醒这些线程。

## 1.状态
FutureTask内部维护了一个用volatile修饰的int型成员state来表示状态，其中共有7种状态，具体如下源码所示：

```` java
// 状态
private volatile int state;
// 新建状态
private static final int NEW          = 0;
// 正在执行状态
private static final int COMPLETING   = 1;
// 完成状态
private static final int NORMAL       = 2;
// 异常状态
private static final int EXCEPTIONAL  = 3;
// 取消状态
private static final int CANCELLED    = 4;
// 正在中断状态
private static final int INTERRUPTING = 5;
// 中断状态
private static final int INTERRUPTED  = 6;
````

可能发生的状态转换：

* NEW -> COMPLETING -> NORMAL
* NEW -> COMPLETING -> EXCEPTIONAL
* NEW -> CANCELLED
* NEW -> INTERRUPTING -> INTERRUPTED

## 2.源码分析

### 2.1 构造方法
FutureTask提供了两个构造方法，分别可以包装Callable和Runable的对象，并将状态初始化为NEW。因为Runable没有返回，所以包装Runable时，会将Runable封装在RunnableAdapter类里，RunnableAdapter的call方法返回的就是构造方法传进来的result，具体代码如下：

``` java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    // 将状态初始化为NEW
    this.state = NEW;
}

public FutureTask(Runnable runnable, V result) {
    // 将Runable包装为RunnableAdapter类
    this.callable = Executors.callable(runnable, result);
    // 将状态初始化为NEW
    this.state = NEW;
}
```

### 2.2 内部类WaitNode

FutureTask使用内部类WaitNode构建了一个简单的单链表用来记录等待任务执行结果的线程，并将这个类对象实例放在waiters成员变量上，定义如下：

``` java
// 等待结果的线程
private volatile WaitNode waiters;

static final class WaitNode {
    // 等待的线程
    volatile Thread thread;
    // 下一个节点
    volatile WaitNode next;
    // 构造方法，将当前线程赋值给thread
    WaitNode() {
        thread = Thread.currentThread();
    }
}
```

### 2.3 run

run方法是线程执行任务的关键方法，具体任务执行逻辑的调用和返回值的获取及异常捕获都在这个方法进行，具体过程如下：

1. 如果当前不是新建状态，直接return；
2. 将当前线程通过CAS赋值给runner成员变量，以便调用cancel方法时可以对线程发起中断，这里保证了任务只会执行一次；
3. 然后调用callable对象的call方法执行任务，这里会再次校验状态是否为新建状态；
4. 获取call方法的返回值，并通过set方法赋值到outcome成员变量中，这里会有NEW -> COMPLETING -> NORMAL的状态变化；
5. 如果call异常，捕获异常后通过setException方法标记异常，这里会有NEW -> COMPLETING -> EXCEPTIONAL的状态变化；
6. 到这里已经执行完了，最后将runner置为null，如果状态是正在中断或中断状态，调用handlePossibleCancellationInterrupt方法执行线程让步，确保来自cancel（true）的任何中断仅在运行或runAndReset时才传递给任务。


具体源码如下：

``` java
/**
 * 任务执行方法
 */
public void run() {
    // 1.判断是否为新建状态
    // 2.将当前线程通过CAS赋值给runner成员变量，保证了任务只会执行一次
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                        null, Thread.currentThread()))
        // 不满足则直接返回
        return;
    try {
        // 当前任务对应的callable对象
        Callable<V> c = callable;
        // double check state
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 执行任务
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                // 捕获异常
                result = null;
                ran = false;
                // 标记异常
                setException(ex);
            }
            if (ran)
                // 将结果赋值到outcome成员变量中
                set(result);
        }
    } finally {
        // runner必须为非空，直到任务执行完成
        // 以确保任务只执行一次
        runner = null;
        // 必须在runner为空后重新读取状态，以防止中断被遗漏
        int s = state;
        if (s >= INTERRUPTING)
            // 如果正在中断或中断状态
            // 调用handlePossibleCancellationInterrupt执行线程让步
            // 确保来自cancel（true）的任何中断仅在运行或runAndReset时才传递给任务
            handlePossibleCancellationInterrupt(s);
    }
}
```

#### 2.3.1 set

任务执行完成结果赋值及成功状态转换，并唤醒所有正在等待任务结果的线程，这里会有NEW -> COMPLETING -> NORMAL的状态变化，源码如下：

``` java
/**
 * 任务执行完成结果赋值及成功状态转换
 */
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        // 将结果赋值给outcome
        outcome = v;
        // 标记为成功状态
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL);
        // 唤醒并移除所有等待结果的线程
        finishCompletion();
    }
}

/**
 * 唤醒并移除所有等待结果的线程
 */
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        // 将waiters变量置为null
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            // 循环遍历单链表
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    // 唤醒等待线程
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```

#### 2.3.2 setException

如果call异常，捕获异常后通过setException方法标记异常，这里会有NEW -> COMPLETING -> EXCEPTIONAL的状态变化，源码如下：

``` java
/**
 * 标记异常
 */ 
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        // 将异常信息赋值给outcome
        outcome = t;
        // 标记为异常状态
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL);
        // 唤醒并移除所有等待结果的线程
        finishCompletion();
    }
}
```

#### 2.3.2 handlePossibleCancellationInterrupt

handlePossibleCancellationInterrupt方法执行线程让步，确保来自cancel（true）的任何中断仅在运行或runAndReset时才传递给任务，源码如下：

``` java
/**
 * 确保来自cancel（true）的任何中断仅在运行或runAndReset时才传递给任务
 */
private void handlePossibleCancellationInterrupt(int s) {
    // It is possible for our interrupter to stall before getting a
    // chance to interrupt us.  Let's spin-wait patiently.
    if (s == INTERRUPTING)
        while (state == INTERRUPTING)
            // 等待待处理的中断
            Thread.yield();
}
```

### 2.4 get

get方法是用来获取任务执行的结果，有两个重载的方法，一个带超时时间，一个不带超时时间。两个方法在任务执行完成之前都会将线程挂起，带超时时间的get方法会在超时后抛出TimeoutException，不带超时时间的get方法会一直被挂起，两个方法都会响应中断并抛出InterruptedException，线程被唤醒后会调用report方法获取并返回任务执行结果，源码如下：

``` java
/**
 * 不带超时时间
 */
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        // 任务还未执行完成
        // 挂起当前线程，没有超时时间
        s = awaitDone(false, 0L);
    // 获取并返回任务执行结果
    return report(s);
}

/**
 * 带超时时间
 */
public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        // 任务还未执行完成并且挂起超时，抛出TimeoutException
        throw new TimeoutException();
    // 获取并返回任务执行结果
    return report(s);
}
```

#### 2.4.1 awaitDone

等待任务完成，或者中断任务或等待超时时中止，源码如下：

``` java
/**
 * 等待和超时
 */
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    // 等待结果节点
    WaitNode q = null;
    // 标记是否已将当前等待节点加入等待链表waiters
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            // 线程中断则移除当前等待节点
            // 并抛出InterruptedException
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            // 当前状态 > COMPLETING表示任务已经完成或者被取消或者异常或者被中断
            if (q != null)
                // 将等待节点thread置为null
                // 以便等待节点可以被回收
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING)
            // 任务正在执行
            // 线程让步以待任务执行完成
            Thread.yield();
        else if (q == null)
            // 创建当前的等待节点
            q = new WaitNode();
        else if (!queued)
            // 将当前等待节点加入到等待链表waiters头部
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                    q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                // 等待已经超时，移除节点
                removeWaiter(q);
                return state;
            }
            // 线程挂起
            LockSupport.parkNanos(this, nanos);
        }
        else
            // 线程挂起
            LockSupport.park(this);
    }
}

/**
 * 将超时或者被中断的等待节点移除
 */
private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null;
        retry:
        for (;;) {
            // 重新开始遍历
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                s = q.next;
                if (q.thread != null)
                    // 将thread不为null的节点作为先驱节点
                    pred = q;
                else if (pred != null) {
                    // 先驱节点不为null
                    // 将thread为null的节点移除
                    pred.next = s;
                    if (pred.thread == null)
                        // 先驱节点的thread为null
                        // 重新遍历
                        continue retry;
                }
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                        q, s))
                    // 当前节点的thread为null并且pred为null
                    // 就将下一个节点s通过CAS修改waiters的头结点
                    // 然后重新遍历
                    continue retry;
            }
            break;
        }
    }
}
```

#### 2.4.2 report

获取并返回任务执行结果，源码如下：

``` java
/**
 * 获取并返回任务执行结果
 */
@SuppressWarnings("unchecked")
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        // 如果状态等于NORMAL表示任务执行完成
        // 返回正常结果
        return (V)x;
    if (s >= CANCELLED)
        // 状态 >= CANCELLED表示任务被取消或者任务被中断
        // 抛出CancellationException
        throw new CancellationException();
    // 否则任务执行异常
    throw new ExecutionException((Throwable)x);
}
```

### 2.5 cancel

取消任务执行，源码如下：

``` java
/**
 * 取消任务执行
 * @param mayInterruptIfRunning true：表示中断任务；false：表示取消任务
 */
public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW &&
            UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
                mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        // 非新建状态
        // 或者修改状态失败都返回false
        return false;
    try {
        if (mayInterruptIfRunning) {
            // 中断任务
            try {
                Thread t = runner;
                if (t != null)
                    // 添加中断标记
                    t.interrupt();
            } finally {
                // 修改中断状态
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        // 唤醒并移除所有等待结果的线程
        finishCompletion();
    }
    return true;
}
```

## 3.总结

1. FutureTask是通过LockSupport来阻塞线程、唤醒线程；

2. 对于多线程访问成员变量waiters、state，都采用CAS来操作。