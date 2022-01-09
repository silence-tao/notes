# JDK源码分析之——AQS

在 Java 高并发编程中，我们经常会用到 `ReentrantLock`、`CountDownLatch`、`Semaphore` 等同步类，它们的功能可能不尽相同，但是它们都有一个共同点，就是实现依赖了 `AQS（AbstractQueuedSynchronizer）`。这是一个抽象的队列同步器，它定义了一套多线程访问共享资源的同步器框架。今天我们通过源码分析来看看它的真面目。

## 1.框架
![CLH_Queue](../img/CLH_Queue.png)

它维护了一个 `volatile int state`（代表共享资源）和一个 FIFO 线程同步队列（多线程争用资源被阻塞时会进入此队列）。state的访问方式有三种:

1. getState()
2. setState()
3. compareAndSetState()

AQS 定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如 `ReentrantLock`）和 Share（共享，多个线程可同时执行，如 `Semaphore/CountDownLatch`）。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程同步队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

1. `isHeldExclusively()`：该线程是否正在独占资源。只有用到 `Condition` 才需要去实现它；
2. `tryAcquire(int)`：独占方式。尝试获取资源，成功则返回 true，失败则返回 false；
3. `tryRelease(int)`：独占方式。尝试释放资源，成功则返回 true，失败则返回 false；
4. `tryAcquireShared(int)`：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源；
5. `tryReleaseShared(int)`：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回 true，否则返回 false。

以 `ReentrantLock` 为例，`state` 初始化为 0，表示未锁定状态。A 线程 `lock()` 时，会调用 `tryAcquire()` 独占该锁并将 `state + 1`。此后，其他线程再 `tryAcquire()` 时就会失败，直到 A 线程 `unlock()` 到 `state = 0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（`state` 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 `state` 是能回到零态的。

再以 `CountDownLatch` 以例，任务分为 N 个子线程去执行，`state` 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 `countDown()` 一次，`state` 会 CAS 减 1。等到所有子线程都执行完后(即 `state = 0`)，会 `unpark()` 主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 `tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared` 中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如 `ReentrantReadWriteLock`。

## 2.源码分析：独占模式

本节依照 `acquire-release`、`acquireShared-releaseShared` 的次序来讲解 AQS 的源码。

### 2.1 acquire(int)

此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入同步队列，直到获取到资源为止，且整个过程忽略中断的影响。这也正是 `lock()` 的语义，当然不仅仅只限于 `lock()`。获取到资源后，线程就可以去执行其临界区代码了。下面是 `acquire()` 的源码：

``` java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

方法流程如下：

1. `tryAcquire()` 尝试直接去获取资源，如果成功则直接返回；
2. `addWaiter()` 将该线程加入同步队列的尾部，并标记为独占模式；
3. `acquireQueued()` 使线程在同步队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回 true，否则返回 false；
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断 `selfInterrupt()`，将中断补上。

#### 2.1.1 tryAcquire(int)

此方法尝试去获取独占资源。如果获取成功，则直接返回 true，否则直接返回 false。这也正是 `tryLock()` 的语义，还是那句话，当然不仅仅只限于 `tryLock()`。如下是 `tryAcquire()` 的源码：

``` java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

AQS这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过 `state` 的 get/set/CAS）至于能不能重入，能不能加塞，那就看具体的自定义同步器怎么去设计了。当然，自定义同步器在进行资源访问时要考虑线程安全的影响。

这里之所以没有定义成 abstract，是因为独占模式下只用实现 `tryAcquire-tryRelease`，而共享模式下只用实现 `tryAcquireShared-tryReleaseShared`。如果都定义成 abstract，那么每个模式也要去实现另一模式下的接口。

#### 2.1.2 addWaiter(Node)

此方法用于将当前线程加入到同步队列的队尾，并返回当前线程所在的结点。还是上源码吧：

``` java
private Node addWaiter(Node mode) {
    // 以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 尝试快速的方式将新节点放入队尾
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 当队列为空则通过此方法入队
    enq(node);
    return node;
}
```

`Node` 结点是对每一个访问同步代码的线程的封装，其包含了需要同步的线程本身以及线程的状态，如是否被阻塞，是否等待唤醒，是否已经被取消等。变量 `waitStatus` 则表示当前被封装成 `Node` 结点的等待状态，共有 4 种取值 `CANCELLED`、`SIGNAL`、`CONDITION`、`PROPAGATE`。

1. `CANCELLED`：值为 1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该 `Node` 的结点，其结点的 `waitStatus` 为 `CANCELLED`，即结束状态，进入该状态后的结点将不会再变化；
2. `SIGNAL`：值为 -1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，只要前继结点释放锁，就会通知标识为 `SIGNAL` 状态的后继结点的线程执行；
3. `CONDITION`：值为 -2，与 `Condition` 相关，该标识的结点处于同步队列中，结点的线程等待在 `Condition` 上，当其他线程调用了 `Condition` 的 `signal()` 方法后，`CONDITION` 状态的结点将从同步队列转移到同步队列中，等待获取同步锁；
4. `PROPAGATE`：值为 -3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态；
5. 0 状态：值为 0，代表初始化状态。

AQS 在判断状态时，通过用 `waitStatus > 0` 表示取消状态，而 `waitStatus < 0` 表示有效状态。

##### 2.1.2.1 enq(Node)

此方法用于当同步队列为空时，创建头结点，并将node加入队尾。源码如下：

``` java
private Node enq(final Node node) {
    // CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
            if (compareAndSetHead(new Node()))
                tail = head;
        } else { //将node节点放入队尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

#### 2.1.3 acquireQueued(Node, int)

通过 `tryAcquire()` 和 `addWaiter()`，该线程获取资源失败，已经被放入同步队列尾部了。接下来应该进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了。源码如下：

``` java
// 在同步队列中排队拿号（中间没其它事干可以休息），直到拿到号后再返回
final boolean acquireQueued(final Node node, int arg) {
    // 标记是否获取资源失败
    boolean failed = true;
    try {
        // 标记等待过程中是否被中断过
        boolean interrupted = false;
        for (;;) {
            // 获取当前节点的前驱节点
            final Node p = node.predecessor();
            // 如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源
            // 可能是老大释放完资源唤醒自己的，当然也可能被interrupt了
            if (p == head && tryAcquire(arg)) {
                // 拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                setHead(node);
                p.next = null; // help GC
                failed = false;
                //返回等待过程中是否被中断过
                return interrupted;
            }
            // 因为暂时无法获取到资源，所以先进入waiting状态，直到有资源释放时被unpark()唤醒
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // 如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

##### 2.1.3.1 shouldParkAfterFailedAcquire(Node, Node)

此方法主要用于检查状态，看看自己是否真的可以进入 waiting 状态。源码如下：

``` java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 拿到前驱节点的状态
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        // 如果已经设置为了SIGNAL，那就可以直接进入waiting状态了
        return true;
    if (ws > 0) {
        // 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边
        // 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被GC回收
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

##### 2.1.3.2 parkAndCheckInterrupt()

此方法就是让线程真正进入等待状态，源码如下：

``` java
private final boolean parkAndCheckInterrupt() {
    // 调用park()使线程进入waiting状态
    LockSupport.park(this);
    // 如果被唤醒，查看自己是不是被中断的，注意：此方法会清除中断状态
    return Thread.interrupted();
}
```

`park()` 会让当前线程进入 waiting 状态。在此状态下，有两种途径可以唤醒该线程：

1. 被unpark()；
2. 被interrupt()。

需要注意的是，`Thread.interrupted()` 会清除当前线程的中断标记位。 

##### 2.1.3.3 小结

看了 `shouldParkAfterFailedAcquire()` 和 `parkAndCheckInterrupt()`，现在让我们再回到 `acquireQueued()`，总结下该函数的具体流程：

1. 结点进入队尾后，检查状态，找到安全休息点；
2. 调用 `park()` 进入 waiting 状态，等待 `unpark()` 或 `interrupt()` 唤醒自己；
3. 被唤醒后，看自己是不是有资格能拿到号。如果拿到，head 指向当前结点，并返回从入队到拿到号的整个过程中是否被中断过；如果没拿到，继续流程 1。

#### 2.1.4 acquire方法小结

`acquireQueued()` 分析完之后，我们接下来再回到 `acquire()`，源码如下：

``` java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

再来总结下它的流程吧：

1. 调用自定义同步器的 `tryAcquire()` 尝试直接去获取资源，如果成功则直接返回；
2. 没成功，则 `addWaiter()` 将该线程加入同步队列的尾部，并标记为独占模式；
3. `acquireQueued()` 使线程在同步队列中休息，有机会时（轮到自己，会被 `unpark()`）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回 true，否则返回 false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断 `selfInterrupt()`，将中断补上。

流程图：
![AQS_process](../../../silentao_blog/source/img/backend/AQS_process.png)

至此，`acquire()` 的流程终于算是告一段落了。这也就是 `ReentrantLock.lock()` 的流程，不信你去看其 `lock()` 源码吧，整个函数就是一条 `acquire(1)`！！！


### 2.2 release(int)

此方法是独占模式下线程释放共享资源的顶层入口，它会释放指定量的资源，如果彻底释放了（即 `state=0`），它会唤醒同步队列里的其他线程来获取资源。这也正是 `unlock()` 的语义，当然不仅仅只限于 `unlock()`。下面是 `release()` 的源码：

``` java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒同步队列里的下一个线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

它调用 `tryRelease()` 来释放资源。有一点需要注意的是，它是根据 `tryRelease()` 的返回值来判断该线程是否已经完成释放掉资源了，所以自定义同步器在设计 `tryRelease()` 的时候要明确这一点。

#### 2.2.1 tryRelease(int)

此方法尝试去释放指定量的资源。下面是 `tryRelease()` 的源码：

``` java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

跟 `tryAcquire()` 一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，`tryRelease()` 都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(`state -= arg`)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，`release()` 是根据 `tryRelease()` 的返回值来判断该线程是否已经完成释放掉资源了！所以自义定同步器在实现时，如果已经彻底释放资源( `state = 0`)，要返回 true，否则返回 false。

#### 2.2.2 unparkSuccessor(Node)

此方法用于唤醒同步队列中下一个线程。下面是源码：

``` java
private void unparkSuccessor(Node node) {
    // node一般为当前线程所在的结点
    int ws = node.waitStatus;
    if (ws < 0)// 置零当前线程所在的结点状态，允许失败
        compareAndSetWaitStatus(node, ws, 0);
 
    // 找到下一个需要唤醒的结点s
    Node s = node.next;
    // 如果这个节点不存在，或已经取消等待
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从队列尾部开始遍历，获取从头部开始算第一个可用的节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread); // 唤醒
}
```

这个方法并不复杂。一句话概括：用 `unpark()` 唤醒同步队列中最前边的那个未放弃线程，这里我们也用 s 来表示吧。此时，再和 `acquireQueued()` 联系起来，s 被唤醒后，进入 `if (p == head && tryAcquire(arg))` 的判断（即使 `p != head` 也没关系，它会再进入 `shouldParkAfterFailedAcquire()` 寻找一个安全点。这里既然 s 已经是同步队列中最前边的那个未放弃线程了，那么通过 `shouldParkAfterFailedAcquire()` 的调整，s 也必然会跑到 `head` 的 `next` 结点，下一次自旋 `p == head` 就成立啦），然后 s 把自己设置成 `head` 标杆结点，表示自己已经获取到资源了，`acquire()` 也返回了，就可以执行线程自己的任务了。

#### 2.2.3小结

`release()` 是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即 `state = 0`）,它会唤醒同步队列里的其他线程来获取资源。

## 3.源码分析：共享模式

### 3.1 acquireShared(int)

此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入同步队列，直到获取到资源为止，整个过程忽略中断。下面是 `acquireShared()` 的源码：

``` java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

这里 `tryAcquireShared()` 依然需要自定义同步器去实现。但是 AQS 已经把其返回值的语义定义好了：

- 负值代表获取失败；
- 0 代表获取成功，但没有剩余资源；
- 正数表示获取成功，还有剩余资源，其他线程还可以去获取。

所以这里 `acquireShared()` 的流程就是：

1. `tryAcquireShared()` 尝试获取资源，成功则直接返回；
2. 失败则通过 `doAcquireShared()` 进入同步队列，直到获取到资源为止才返回。

#### 3.1.1 doAcquireShared(int)

此方法用于将当前线程加入同步队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。下面是doAcquireShared()的源码：

``` java
private void doAcquireShared(int arg) {
    // 现将当前线程加入队列尾部
    final Node node = addWaiter(Node.SHARED);
    // 是否成功标志
    boolean failed = true;
    try {
        // 等待过程中是否被中断过的标志
        boolean interrupted = false;
        for (;;) {
            // 前驱
            final Node p = node.predecessor();
            // 如果到head的下一个，因为head是拿到资源的线程
            // 此时node被唤醒，很可能是head用完资源来唤醒自己的
            if (p == head) {
                // 尝试获取资源
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 将head指向自己，还有剩余资源可以再唤醒之后的线程
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    // 如果等待过程中被打断过，此时将中断补上
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
             
            // 判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

这个方法和 `acquireQueued()` 很相似，在流程上并没有太大区别。只不过这里将补中断的 `selfInterrupt()` 放到 `doAcquireShared()` 里了，而独占模式是放到 `acquireQueued()` 之外，其实都一样。

跟独占模式比，还有一点需要注意的是，这里只有线程是 `head.next`（“老二”）时，才会去尝试获取资源，有剩余的话还会唤醒之后的队友。那么问题就来了，假如老大用完后释放了 5 个资源，而老二需要 6 个，老三需要 1 个，老四需要 2 个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！老二会继续 `park()` 等待其他线程释放资源，也更不会去唤醒老三和老四了。独占模式，同一时刻只有一个线程去执行，这样做未尝不可；但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，而把后面量小的老三和老四也都卡住了。当然，这并不是问题，只是 AQS 保证严格按照入队顺序唤醒罢了（保证公平，但降低了并发）。

##### 3.1.1.1 setHeadAndPropagate(Node, int)

``` java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    // head指向自己
    setHead(node);
    // 如果还有剩余量，继续唤醒下一个邻居线程
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

此方法在 `setHead()` 的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式。

#### 3.1.2小结

至此，`acquireShared()` 也要告一段落了。让我们再梳理一下它的流程：

1. `tryAcquireShared()` 尝试获取资源，成功则直接返回；
2. 失败则通过 `doAcquireShared()` 进入同步队列 `park()`，直到被 `unpark()/interrupt()` 并成功获取到资源才返回。整个等待过程也是忽略中断的，其实跟 `acquire()` 的流程大同小异，只不过多了个自己拿到资源后，还会去唤醒后继队友的操作（这才是共享嘛）。

### 3.2 releaseShared()

上一小节已经把 `acquireShared()` 说完了，这一小节就来讲讲它的反操作 `releaseShared()` 吧。此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒同步队列里的其他线程来获取资源。下面是 `releaseShared()` 的源码：

``` java
public final boolean releaseShared(int arg) {
    // 尝试释放资源
    if (tryReleaseShared(arg)) {
        // 唤醒后继结点
        doReleaseShared();
        return true;
    }
    return false;
}
```

此方法的流程也比较简单，一句话：释放掉资源后，唤醒后继。跟独占模式下的 `release()` 相似，但有一点稍微需要注意：独占模式下的 `tryRelease()` 在完全释放掉资源（`state = 0`）后，才会返回 true 去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的 `releaseShared()` 则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如，资源总量是 13，A（5）和 B（7）分别获取到资源并发运行，C（4）来时只剩 1 个资源就需要等待。A 在运行过程中释放掉 2 个资源量，然后 `tryReleaseShared(2)` 返回 true 唤醒 C，C 一看只有 3 个仍不够继续等待；随后 B 又释放 2 个，`tryReleaseShared(2)` 返回 true 唤醒 C，C 一看有 5 个够自己用了，然后 C 就可以跟 A 和 B 一起运行。而 `ReentrantReadWriteLock` 读锁的 `tryReleaseShared()` 只有在完全释放掉资源（`state = 0`）才返回 true，所以自定义同步器可以根据需要决定 `tryReleaseShared()` 的返回值。

#### 3.2.1 doReleaseShared()

此方法主要用于唤醒后继。下面是它的源码：

``` java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                // 唤醒后继节点
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // head未发生改变
        if (h == head)                   // loop if head changed
            break;
    }
}
```


## 4.总结

本节我们详解了独占和共享两种模式下获取-释放资源(`acquire-release`、`acquireShared-releaseShared`)的源码，相信大家都有一定认识了。值得注意的是，`acquire()` 和 `acquireShared()` 两种方法下，线程在同步队列中都是忽略中断的。AQS 也支持响应中断的，`acquireInterruptibly()/acquireSharedInterruptibly()` 即是，这里相应的源码跟 `acquire()` 和 `acquireShared()` 差不多，这里就不再详解了。

