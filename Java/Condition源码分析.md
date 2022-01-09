# JDK源码分析之——Condition

# 简介

`Condition` 是在 java 1.5 中才出现的，它用来替代传统的 `Object` 的 `wait()`、`notify()` 实现线程间的协作，相比使用 `Object` 的 `wait()`、`notify()`，使用 `Condition` 的 `await()`、`signal()` 这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用 `Condition`。

`Condition` 能实现 `synchronized` 和 `wait`、`notify` 搭配的功能，另外比后者更灵活，`Condition` 可以实现多路通知功能，也就是在一个 `Lock` 对象里可以创建多个 `Condition`（即对象监视器）实例，线程对象可以注册在指定的 `Condition` 中，从而可以有选择的进行线程通知，在调度线程上更加灵活。而 `synchronized` 就相当于整个 `Lock` 对象中只有一个单一的 `Condition` 对象，所有的线程都注册在这个对象上。线程开始 `notifyAll` 时，需要通知所有的 WAITING 线程，没有选择权，会有相当大的效率问题。

这里要讲的是 `ReentrantLock` 中的 `Condition`，`Condition` 是一个接口，而 `ReentrantLock` 里使用的 `Condition` 是由 AQS 的内部类 `ConditionObject` 实现的。在 `ConditionObject` 内部维护了一个由 `Node`（AQS 里的另一个类）节点连接而成的等待队列，这个队列是一个单链表，同时 `ConditionObject` 还有指向单链表首尾节点的指针，方便对整个队列进行遍历。

流程图如下：
![10027_1](../../../silentao_blog/source/img/backend/10027/10027_1.png)

由于鄙人不善于作图，所以接下来会有大量的源码注释还有文字说明，如果有说得不清楚的地方，还望评论区指出哈

## 1.ConditionObject的成员变量

`ConditionObject` 的成员变量比较简单，包含两个分别指向队列头尾的指针和两种等待退出的模式，源码如下：

``` java
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;

    /**
     * 指向队列的头指针
     */
    private transient Node firstWaiter;

    /**
     * 指向队列的尾指针
     */
    private transient Node lastWaiter;

    /**
     * 在等待退出时重新标记中断
     */
    private static final int REINTERRUPT =  1;

    /**
     * 在等待退出时抛出中断异常
     */
    private static final int THROW_IE    = -1;
}
```

## 2.await

`await` 方法的作用就是释放当前线程持有的锁，然后将当前线程挂起，直到有其它线程将它唤醒或者被中断。在源码讲解前，先来做个名称解释：

* 同步队列：指的是在 AQS 中等待获取资源的队列，由 AQS 控制；
* 等待队列：指的是调用 `await` 方法之后等待被 `signal` 唤醒的队列，由 AQS 内部类 `ConditionObject` 控制。

`await` 方法具体流程如下：

1. 如果线程被中断，直接抛出中断异常；
2. 创建一个状态为 `CONDITION` 的 `Node` 节点，并将节点放在等待队列的队尾；
3. 完全释放线程占有的全部资源；
4. 循环判断 `Node` 节点是否不在同步队列中，然后将当前线程挂起，否则退出循环，如果线程被中断也会退出循环；
5. 尝试去同步队列中获取所需要的资源，如果暂时拿不到就挂起当前线程，直到当前节点变成同步队列的第二个节点时被唤醒；
6. 如果当前节点在等待队列中还有后继节点，说明当前线程是被中断唤醒的，这个时候要把等待队列中不是 `CONDITION` 状态的节点清理掉；
7. 根据 `interruptMode` 的值来决定是否需要抛出中断异常或者重新标记中断。

源码如下：

``` java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        // 如果线程被中断过，就直接抛出中断异常
        throw new InterruptedException();
    // 创建一个状态为CONDITION的Node节点，并将节点放在等待队列的队尾
    Node node = addConditionWaiter();
    // 完全释放线程占有的全部资源
    int savedState = fullyRelease(node);
    // 标记线程最后退出的模式
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // node节点不在同步队列中（指的是AQS的同步队列）
        // 当前线程被其它线程唤醒时
        // node节点就会被加入到同步队列的队尾，这时自然会退出循环

        // 挂起当前线程
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            // 如果线程被中断，直接退出循环
            break;
    }

    // 尝试去同步队列中获取所需要的资源（这个方法在讲AQS的文章里讲过，这里不再复述）
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        // 如果线程在获取资源时被中断并且不是以抛异常的形式退出
        // 那就将退出模式置为为重新标记中断
        interruptMode = REINTERRUPT;
    
    if (node.nextWaiter != null) // clean up if cancelled
        // 当前节点在等待队列中还有后继节点
        // 说明当前线程是被中断唤醒的
        // 被正常唤醒的节点的后继节点会被置为null且会被放在同步队列的队尾

        // 把等待队列中不是CONDITION状态的节点清理掉    
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        // 表示非正常唤醒（因中断被唤醒）
        // 那就根据interruptMode标记的值来决定是抛出异常还是重新标记中断
        reportInterruptAfterWait(interruptMode);
}
```

### 2.1 addConditionWaiter

`addConditionWaiter` 方法用来创建一个状态为 `CONDITION` 的新 `Node` 节点，并添加等待队列尾部，源码如下：

``` java
private Node addConditionWaiter() {
    // 获取尾节点
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        // 清理状态不是CONDITION节点
        unlinkCancelledWaiters();
        // 再次获取尾节点
        t = lastWaiter;
    }
    // 创建一个状态为CONDITION的新Node节点
    Node node = new Node(Thread.currentThread(), Node.CONDITION);

    // 将新节点添加到等待队列尾部
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

### 2.2 fullyRelease

`fullyRelease` 方法的作用是释放线程持有的所有的资源，源码如下：

``` java
/**
 * 释放线程持有的所有资源（该方法是QAS的方法）
 */
final int fullyRelease(Node node) {
    // 用来标记过程是否是失败
    boolean failed = true;
    try {
        // 获取线程占用的所有资源
        int savedState = getState();
        // 释放资源
        if (release(savedState)) {
            // 成功释放资源
            // 将失败标记置为false
            failed = false;
            // 返回释放的资源
            return savedState;
        } else {
            // 释放资源失败则抛出异常
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            // 资源释放失败就将当前节点标记为CANCELLED（取消状态）
            node.waitStatus = Node.CANCELLED;
    }
}
```

### 2.3 isOnSyncQueue

`isOnSyncQueue` 方法是用来判断 `node` 节点是否在同步队列，源码如下：

``` java
/**
 * node节点是否在同步队列中（该方法是QAS的方法）
 */
final boolean isOnSyncQueue(Node node) {
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        // 如果节点状态为CONDITION
        // 或者node节点的前继节点为空，那么node肯定不在同步队列中

        // 只有两种情况node节点的前继节点会为空
        // 1.node节点不在同步队列里；
        // 2.node节点是同步队列的头结点。
        // 很显然这里不会是同步队列的头结点
        // 因为同步队列头结点的是持有资源的线程对应的节点
        // 如果持有资源的线程调用await方法到这里的node
        // 已经不是同步队列头结点的那个node了
        return false;
    
    // 如果node有后继节点，那么肯定是在同步队列中
    if (node.next != null) // If has successor, it must be on queue
        return true;
    
    // node的前继节点为非空，并不意味着node就在同步队列中
    // 因为CAS操作在将node节点加入到同步队列时可能失败
    // 所以我们必须确保node节点已经在同步队列中
    // 那么就需要从同步队列的尾部遍历
    // 看node节点是否在同步队列中
    return findNodeFromTail(node);
}

/**
 * 从同步队列尾部遍历看node节点是否在同步队列中（该方法是QAS的私有方法）
 */
private boolean findNodeFromTail(Node node) {
    // 获取同步队列的尾部节点
    Node t = tail;
    for (;;) {
        if (t == node)
            // 节点在同步队列中，返回true
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

### 2.4 checkInterruptWhileWaiting

`checkInterruptWhileWaiting` 方法主要是校验 `node` 节点对应的线程是否被中断唤醒，如果是在被 `signal` 唤醒之前被中断就返回 `THROW_IE`（表示在退出时要抛出中断异常），如果是在被 `signal` 唤醒之后被中断则返回 `REINTERRUPT`（表示在退出时要重新标记中断），如果是被 `signal` 唤醒没有被中断则返回 0（表示线程整个挂起的过程中都没有被中断），源码如下：

``` java
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
}

/**
 * 判断中断发生的时机（该方法是QAS的私有方法）
 * 返回true：表示在被signal唤醒之前被中断
 * 返回false：表示在被signal唤醒之后被中断
 */
final boolean transferAfterCancelledWait(Node node) {
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        // 通过CAS将node节点的状态修改为0
        // 成功表示此时node节点还在等待队列中
        // 间接表明此时的node节点是被中断唤醒的

        // 将node节点加入到同步队列尾部
        enq(node);
        return true;
    }
    
    // 到了这里说明中断是在被signal唤醒之后发生的
    // 此时我们需要保证node节点已经在同步队列中
    // 才能继续往下执行
    while (!isOnSyncQueue(node))
        // 如果node不在同步队列中，那么调用线程让步
        // 直到node节点已经在同步队列中为止
        Thread.yield();
    return false;
}
```

### 2.5 unlinkCancelledWaiters

`unlinkCancelledWaiters` 方法的作用是删除等待队列中不是 `CONDITION` 状态的节点，从头开始挨个遍历每个节点，删除状态不是 `CONDITION` 状态的节点，也就是单链表的节点删除操作，源码如下：

``` java
/**
 * 从等待队列中删除状态不是CONDITION状态的节点
 */
private void unlinkCancelledWaiters() {
    // 获取头节点，从头节点开始遍历
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            // 如果当前节点状态不是CONDITION就从链表中删除
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            
            // next为空表示当前节点是最后一个节点
            // 将尾节点指针指向next
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

### 2.6 reportInterruptAfterWait

`reportInterruptAfterWait` 方法控制线程退出时是抛出中断异常还是重新标记中断，源码如下

``` java
/**
 * interruptMode == THROW_IE：抛出中断异常
 * interruptMode == REINTERRUPT：重新标记中断
 */
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        // 抛出中断异常
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        // 重新标记中断
        selfInterrupt();
}
```

### 2.7 小结

`await` 方法的源码分析在这里就告一段落啦，其中还有不响应中断的 `awaitUninterruptibly` 方法，带超时的 `awaitNanos` 和 `await` 方法以及指定超时截止时间的 `awaitUntil` 方法，逻辑大致和上面的 `await` 方法如出一辙，这里就不一一叙述啦，感兴趣的同学可以自行研究。`await` 方法会将线程占有的所有资源都释放掉，然后将线程挂起直到其它线程将它唤醒或者被中断。期间涉及到了 `node` 节点从等待队列到同步队列的转移，然后线程在同步队列中重新获取资源（即重新获得锁）后继续执行的整个过程。虽然过程不是很复杂，但是同时调用了内部类 `ConditionObject` 和 AQS 的方法，还需仔细品味等待队列和同步队列之间的关系，才能明白它们的巧妙之处。加油鸭！

## 3.signal

`signal` 方法主要是将等待队列中等待最久的那个没有取消的 `node` 节点转移到同步队列中的尾部，使该 `node` 节点有在同步队列中等待获取资源（获得锁）的资格，得以继续执行 `await` 方法后面的逻辑。具体过程如下：

1. 判断当前线程是否为持有锁的线程，如果不是则抛出 `IllegalMonitorStateException` 异常；
2. 获取等待队列中头结点，然后将头结点指针指向头结点的下一个节点，再尝试将当前结点转移到同步队列的尾部；
3. 如果当前节点在同步队列的前继节点已经取消或者修改其前继节点状态为 `SIGNAL` 失败时，换醒当前节点；
4. 如果上述操作失败就重复2、3过程，直到成功将一个节点转移到同步队列，或者遍历完等待队列就结束整个过程。

源码如下：

``` java
/**
 * 尝试唤醒等待队列中等待了最近的线程
 */
public final void signal() {
    if (!isHeldExclusively())
        // 如果当前线程不是持有锁的线程
        // 抛出IllegalMonitorStateException异常
        throw new IllegalMonitorStateException();
    
    // 获取等待队列的头结点
    Node first = firstWaiter;
    if (first != null)
        // 头结点不为空，开始尝试唤醒头结点
        doSignal(first);
}
```

### 3.1 doSignal

`doSignal` 主要是从等待队列头部开始遍历，然后将节点从等待队列中移除，并尝试将节点转移到同步队列中，直到有一个成功或者遍历完整个等待队列为止，源码如下：

``` java
/**
 * 遍历等待队列并将相应节点从等待队列中移除
 * 然后尝试将节点转移到同步队列中
 */
private void doSignal(Node first) {
    do {
        // 将头部指针指向first.nextWaiter
        // 这里是要将first节点删除掉
        if ( (firstWaiter = first.nextWaiter) == null)
            // first.nextWaiter为空表示整个等待队列已经空了
            // 将等待队列尾部指针也置为空
            lastWaiter = null;
        
        // first.nextWaiter置为空
        // 彻底将first断开
        first.nextWaiter = null;

        // 尝试将first节点转移到同步队列中，有一次成功就退出循环
        // 或者遍历完整个等待队列时再退出
    } while (!transferForSignal(first) &&
                (first = firstWaiter) != null);
}

/**
 * 将node节点转移到同步队列的尾部（该方法是QAS的方法）
 * 返回true表示成功转移
 * 返回false表示node可能已经取消或者已经转移过了
 */
final boolean transferForSignal(Node node) {
    // 尝试将node节点状态修改为0
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        // 修改失败可能node节点已经取消了或者已经转移到同步队列里了
        return false;

    // 将node节点转移到同步队列尾部并返回它的前继节点
    Node p = enq(node);
    // 获取前继节点的状态
    int ws = p.waitStatus;
    // 这一步应该是为了保证node节点的前继节点为SIGNAL状态
    // 好在前继执行完成时可以顺利唤醒node节点
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        // 如果前继节点状态为取消
        // 或者修改前继节点状态为SIGNAL失败
        // 就唤醒node节点对应的线程
        LockSupport.unpark(node.thread);
    
    // node节点转移成功，返回true
    return true;
}
```

### 3.2 小结

`signal` 方法主要是将在等待队列中等待了最久的且没有取消的一个节点转移到同步队列中，以便节点可以在同步队列中有再次获取资源的资格，可以继续完成后面的事情。而 `signalAll` 方法和 `signal` 方法唯一不同的是，`signalAll` 方法是要将所有等待队列中未取消的节点转移到同步队列中，以便它们可以依次获取资源继续完成后面的事情，因为逻辑上差不多，这里也不再叙述，有兴趣的同学可以自己看看。

## 4.总结

本篇讲述的是 `ReentrantLock` 中用到的条件变量 `Condition`，是由 AQS 的内部类 `ConditionObject` 实现的，`ConditionObject` 维护着一个以 AQS 内部类 `Node` 为节点的等待队列，这是一个单链表，等待队列里的都是调用了 `await` 方法后等待被唤醒的节点。`ConditionObject` 主要用两个方法，`await` 和 `signal` 方法。其中 `await` 方法主要是创建一个保存当前线程的 `Node` 节点并放在等待队列中，然后将线程持有的资源完全释放然后将线程挂起，直到线程被唤醒或被中断时，会将节点转移同步队列中，以便线程可以在同步队列中有再次获取资源的资格；而 `signal` 方法主要是将节点从等待队列转移到同步队列中，以便线程可以在同步队列中有再次获取资源的资格。`await` 和 `signal` 需要配合使用，才能维护共享资源的同步。

源码虽然枯燥无味，但是它包含着精髓，同学们需要细细品味，加油鸭！