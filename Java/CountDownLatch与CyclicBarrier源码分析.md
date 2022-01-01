# JDK源码分析之——CountDownLatch与CyclicBarrier

CountDownLatch与CyclicBarrier都是我们在并发编程中常用的工具，它们提供了一种控制并发流程的手段，今天我们通过源码分析来看看它的真面目。

## 1.CountDownLatch

CountDownLatch中count down是倒数的意思，latch则是门闩的含义，它通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。

### 1.1 简介

CountDownLatch在内部自定义了AQS框架（对AQS不熟悉可以先看一下Java并发之AQS详解），内部类Sync继承了AbstractQueuedSynchronizer并重写了共享模式下的tryAcquireShared-tryReleaseShared方法。AbstractQueuedSynchronizer中维护了一个volatile类型的整数state，volatile可以保证多线程环境下该变量的修改对每个线程都可见，并且由于该属性为整型，因而对该变量的修改也是原子的。CountDownLatch正是通过这个state来标识需要倒数的次数，当创建一个CountDownLatch对象时，所传入的整数n就会赋值给state属性，当countDown()方法调用时，该线程就会尝试对state减一，而调用await()方法时，当前线程就会判断state属性是否为0，如果为0，则继续往下执行，如果不为0，则使当前线程进入等待状态，直到某个线程将state属性置为0，其就会唤醒在await()方法中等待的线程。

### 1.2 源码分析

#### 1.2.1 构造方法

``` java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

#### 1.2.2 Sync类

Sync类继承了AbstractQueuedSynchronizer，并重写了共享模式下的tryAcquireShared-tryReleaseShared方法，源码如下：

``` java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;
 
    Sync(int count) {
        setState(count);
    }
 
    int getCount() {
        return getState();
    }
 
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }
 
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();// 获取当前state属性的值
            if (c == 0)// 如果state为0，则说明当前计数器已经计数完成，直接返回
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;// 设置成功后返回当前是否为最后一个设置state的线程
        }
    }
}
```

①tryAcquireShared方法：AQS的acquireSharedInterruptibly方法会调用tryAcquireShared方法，tryAcquireShared判断当前state是否为0，如果为0则返回1，对于AQS来说就是获取到了资源，主线程可以继续执行，而这里可以表示其它线程都执行完毕了，主线程可以继续执行；不为0说明还有线程没有执行完，主线程需要继续等待；

②tryReleaseShared方法：AQS的releaseShared方法会调用tryReleaseShared方法，tryReleaseShared会获取当前的state值，并减1，表示当前线程执行完毕。这里使用了CAS的方式修改state的值，保证了线程安全，如果当前线程是最后一个修改state的线程，那么主线程就可以被唤醒，继续执行。

#### 1.2.3 countDown()

调用此方法的线程会尝试对AQS的state减一，表示当前线程完成了自己的事情。需要注意的是，此方法并没有规定一个线程只能调用一次，当同一个线程调用多次countDown()方法时，每次都会使计数器减一，源码如下：

``` java
public void countDown() {
    sync.releaseShared(1);
}
```

 这里是直接调用AQS的releaseShared方法，目的是在AQS里释放一个资源，其实这里就是将state减一，AQS的releaseShared方法会回调Sync类的tryReleaseShared方法，执行state减一操作，如果当前线程是最后一个操作state的线程，那么就会唤醒等待的主线程，releaseShared方法源码如下：

 ``` java
 public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        //唤醒在等待的主线程
        doReleaseShared();
        return true;
    }
    return false;
}
 ```

 #### 1.2.4 await()

调用此方法的主线程会进入同步队列，在其它线程未完成即state还不为0的时候，主线程会被挂起，直到其它线程执行完成后才会被唤醒，源码如下：

``` java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

这里直接调用AQS的acquireSharedInterruptibly方法，这是一个会响应中断的方法，如果线程被中断了，就会抛出InterruptedException。AQS的acquireSharedInterruptibly方法会回调Sync类的tryAcquireShared判断state是否为0，不为0则进入同步队列，并被挂起。acquireSharedInterruptibly方法源码如下：

``` java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

await(long timeout, TimeUnit unit)方法是await()方法的重载方法，会返回一个boolean值，功能和await基本相似。只不过主线程调用这个方法时，会挂起timeout时间，如果这段时间内其它线程完成了任务，返回true，如果超过了timeout还没有完成，则返回false，至于后续如何操作，取决于主线程。

### 1.3 小结

CountDownLatch实质上就是一个AQS计数器，通过AQS来实现线程的等待与唤醒。

## 2.CyclicBarrier

CyclicBarrier是一个同步工具类，它允许一组线程互相等待，直到到达某个公共屏障点。与CountDownLatch不同的是该barrier在释放等待线程后可以重用，所以称它为循环（Cyclic）的屏障（Barrier）。

CyclicBarrier支持一个可选的Runnable命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次。若在继续所有参与线程之前更新共享状态，此屏障操作很有用。

### 2.1 简介

当前线程等待直到所有线程都调用了该屏障的await()方法，如果当前线程不是将到达的最后一个线程，将会被阻塞。解除阻塞的情况有以下几种：

①最后一个线程调用await()；
②当前线程被中断；
③其他正在该CyclicBarrier上等待的线程被中断；
④其他正在该CyclicBarrier上等待的线程超时；
⑤其他某个线程调用该CyclicBarrier的reset()方法。

如果当前线程在进入此方法时已经设置了该线程的中断状态或者在等待时被中断，将抛出InterruptedException，并且清除当前线程的已中断状态。

如果在线程处于等待状态时barrier被reset()或者在调用await()时 barrier 被损坏，将抛出 BrokenBarrierException 异常。

如果任何线程在等待时被中断，则其他所有等待线程都将抛出 BrokenBarrierException 异常，并将 barrier 置于损坏状态。如果当前线程是最后一个将要到达的线程，并且构造方法中提供了一个非空的屏障操作（barrierAction），那么在允许其他线程继续运行之前，当前线程将运行该操作。如果在执行屏障操作过程中发生异常，则该异常将传播到当前线程中，并将 barrier 置于损坏状态。

对于失败的同步尝试，CyclicBarrier 使用了一种要么全部要么全不 (all-or-none) 的破坏模式：如果因为中断、失败或者超时等原因，导致线程过早地离开了屏障点，那么在该屏障点等待的其他所有线程也将通过BrokenBarrierException（如果它们几乎同时被中断，则用 InterruptedException）以反常的方式离开。

### 2.2 源码分析

#### 2.2.1 类成员分析

直接看源码，分析在注释中：

``` java
//静态内联类，表示当前屏障所属的代，可用于实现重置功能
private static class Generation {
    //当前的屏障是否破坏
    boolean broken = false;
}
 
//重入锁，靠这个锁来保证线程安全
private final ReentrantLock lock = new ReentrantLock();
/** Condition to wait on until tripped */
private final Condition trip = lock.newCondition();
//需要拦截的线程数
private final int parties;
//当所有线程到达屏障时，需要执行的屏障操作
private final Runnable barrierCommand;
//当前的所属的代（Generation），每当屏障失效或者开闸之后都会自动替换掉，从而实现重置的功能
private Generation generation = new Generation();
//还需要阻塞的线程数（即parties-当前阻塞的线程数），当新建generation或generation被破坏时，count会被重置。因为对Count的操作都是在获取锁之后，所以不需要其他同步措施。
private int count;
```

#### 2.2.2 构造方法

有两个构造方法，一个带barrierAction操作，一个不带，源码如下：
``` java
//parties表示屏障拦截的线程数量，当屏障撤销时，先执行barrierAction，然后在释放所有线程
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
//默认为空
public CyclicBarrier(int parties) {
    this(parties, null);
}
```

#### 2.2.3 await

await也有两种方式，一种带超时时间，一种不带超时时间，这个方法没有实际的实现，正真的实现在dowait()方法上，源码如下：

``` java
public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
           BrokenBarrierException,
           TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

#### 2.2.4 dowait

这个方式CyclicBarrier的正真实现，具体实现过程见源码的注释：

``` java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
     
    //获得锁
    lock.lock();
    try {
        //取得当前屏障所属的代
        final Generation g = generation;
         
        //判断当前屏障是否被破坏
        if (g.broken)
            throw new BrokenBarrierException();
             
        //如果当前线程被中断
        if (Thread.interrupted()) {
            //被中断先对当前屏障进行破坏处理
            breakBarrier();
            //再抛出中断异常
            throw new InterruptedException();
        }
 
        //获取省下还没有被拦截的线程数量
        int index = --count;
        if (index == 0) {  // 如果没被拦截的线程数量为0，表示当前线程是最后一个到达屏障的线程
            boolean ranAction = false;
            try {
                // 获取并在当前线程执行后续操作
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                // 标记执行成功
                ranAction = true;
                //重置CyclicBarrier
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction) // 如果barrierCommand没有执行成功
                    breakBarrier();// 对当前屏障进行破坏处理
            }
        }
 
        // loop until tripped, broken, interrupted, or timed out
        // 如果当前线程不是最后一个到达屏障
        // 则进行自旋直到被唤醒，破坏，中断，或者超时
        for (;;) {
            try {
                if (!timed) // 非超时阻塞
                    trip.await();
                else if (nanos > 0L) // 超时阻塞
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {// 阻塞时被中断
                //先判断当前屏障是否被重置并且没有被破坏
                if (g == generation && ! g.broken) {
                    // 破坏处理
                    breakBarrier();
                    // 并抛出异常
                    throw ie;
                } else {
                    // 这种捕获了InterruptException之后调用Thread.currentThread().interrupt()
                    // 是一种通用的方式。其实就是为了保存中断状态
                    // 从而让其他更高层次的代码注意到这个中断。
                    Thread.currentThread().interrupt();
                }
            }
 
            // 如果当前屏障被破坏，当前线程抛BrokenBarrierException
            if (g.broken)
                throw new BrokenBarrierException();
 
            // 如果已经换代，表示当前屏障所有的线程都已经到达屏障
            // 直接返回
            if (g != generation)
                return index;
 
            // 超时，进行屏障破坏处理，并抛TimeoutException
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();// 释放锁
    }
}
 
// 破坏处理方法
private void breakBarrier() {
    // 将破坏表示置为true
    generation.broken = true;
    // 重置count
    count = parties;
    // 唤醒所有被阻塞的线程
    trip.signalAll();
}
 
// 屏障重置方法
private void nextGeneration() {
    // 唤醒所有被阻塞的线程
    trip.signalAll();
    // 重置count
    count = parties;
    // 创建一个新的代
    generation = new Generation();
}
```

#### 2.2.5 reset

重置屏障，先进行屏障破坏处理，再设置下一代generation，源码如下：

``` java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

## 3.CountDowLatch和CyclicBarrier比较

①CountDownLatch:一个线程(或者多个)等待另外N个线程完成某个事情之后才能执行；CyclicBarrier:N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待；
②CountDownLatch:一次性的；CyclicBarrier:可以重复使用；
③CountDownLatch基于AQS；CyclicBarrier基于重入锁和Condition；
④本质上都是依赖于volatile和CAS实现的。