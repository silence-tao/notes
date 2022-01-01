# Java线程池浅析

为了避免系统频繁地创建和销毁线程，我们可以让创建的线程进行复用，于是我们可以把创建的线程放入一个池子里管理起来，即线程池。在线程池中，总有那么几个活跃线程。当需要使用线程时，可以从池子中随便拿一个空闲线程，当完成工作时，并不急着关闭线程，而是将这个线程退回到池子，方便他人使用。

## 1.JDK对线程池的支持

1.1为了能够更好地控制多线程，JDK提供了一套Executor框架，帮助开发人员有效地进行线程控制，其本质就是一个线程池。
![thread_pool](../../../silentao_blog/source/img/backend/thread_pool.png)

其中ThreadPoolExecutor表示一个线程池，Executor提供了各种类型的线程池，主要有以下工厂方法：

①newFixedThreadPool()方法，该方法返回一个固定线程池数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂时存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务；

②newSingleThreadExecutor()方法：该方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务；

③newCachedThreadPool()方法：该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若没有可复用的线程，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用；

④newSingleThreadScheduledExecutor()方法：该方法返回一个ScheduledExecutorService对象，线程池的大小为1。ScheduleExecutorService接口在ExecutorService接口之上扩展了在给定时间执行某个任务的功能，如在某个固定的延时之后执行，或者周期性执行某个任务；

⑤newScheduledThreadPool()方法：该方法也返回一个ScheduleExecutorService对象，但该线程池可以指定线程数量。

1.2计划任务：ScheduleExecutorService可以根据时间需要对线程进行调度，主要有以下方法：

①schedule()方法：schedule()方法会在给定时间，对任务进行一次调度；

②scheduleAtFixedRate()方法：scheduleAtFixedRate()方法任务调度频率是一定的，它是以上一个任务开始执行时间为起点，之后的period时间，调度下一次任务。也就是说，任务开始于给定的初始延时，后续的任务按照给定的周期进行：后续第一个任务将会在initialDelay+period时执行，后续第二个任务将在initialDelay+2 * period时进行，以此类推。周期如果太短，那么任务就会在上一个任务结束后，立即被调用；

③scheduleWithFixedDelay()方法：scheduleWithFixedDelay()方法则是在上一个任务结束后，再经过delay时间进行任务调度。也就是说，任务开始于初始延时时间，后续任务将会按照给定的延时进行，即上一个任务的结束时间到下一个的开始时间的时间差。

另外一个值得注意的问题是，调度程序实际上并不保证任务会无限期的持续调用。如果任务本身抛出了异常，那么后续的所有执行都会被中断。如果任务遇到异常，那么后续的所有子任务都会停止调度，因此，必须保证异常被及时处理，为周期性任务的稳定调度提供条件。

## 2.线程池ThreadPoolExecutor构造方法参数介绍

①corePoolSize：指定了线程池中的线程数量；

②maximumPoolSize：指定了线程池中的最大线程数量；

③keepAliveTime：当线程池线程数量超过corePoolSize时，多余的空闲线程的存活时间。即，超过corePoolSize的空闲线程，在多长时间内，会被销毁；

④unit：keepAliveTime的单位；

⑤workQueue：任务队列，被提交但尚未被执行的任务；

⑥threadFactory：线程工厂，用于创建线程，一般用默认的即可；

⑦handler：拒绝策略。当任务太多来不及处理，如何拒绝任务。

参数workQueue指被提交但未执行的任务队列，它是一个BlockingQueue接口的对象，用于存放Runable对象。根据队列功能分类，在ThreadPoolExecutor的构造方法中可使用以下几种BlockingQueue：

①直接提交队列（SynchronousQueue）：该功能由SynchronousQueue对象提供，SynchronousQueue是一个特殊的BlockingQueue。SynchronousQueue没有容量，每一个插入操作都要等待一个相应的删除操作，反之，每一个删除操作都要等待对应的插入操作。如果使用SynchronousQueue，提交的任务不会被真实的保存，而总是将新任务提交给线程执行，如果没有空闲的线程，则尝试创建新的进程，如果线程数量已达到最大值，则执行拒绝策略。因此，使用SynchronousQueue队列，通常要设置很大的maximumPoolSize值，否则很容易执行拒绝策略；

②有界任务队列（ArrayBlockingQueue）：有界任务队列可以由ArrayBlockingQueue实现，ArrayBlockingQueue的构造方法必须带一个容量参数，表示该队列的最大容量。当使用有界任务队列时，若有新的任务需要执行，如果线程池的实际线程数小于corePoolSize，则会优先创建新的线程，若大于corePoolSize，则会将新的任务加入等待队列，若等待队列已满，无法加入，则在总线程数不大于maximumPoolSize的前提下，创建新的线程执行任务。若大于maximumPoolSize，则执行拒绝策略。可见有界队列仅当在任务队里装满时，才可能将线程数提升到corePoolSize以上，换言之，除非系统非常繁忙，否则确保核心线程数维持在corePoolSize；

③无界任务队列（LinkedBlockingQueue）：无界任务队列可以通过LinkedBlockingQueue类实现，与有界队列相比，除非系统资源耗尽，否则无界队列不存在任务入队失败的情况。当有新的任务到来，系统的线程数量小于corePoolSize时，线程池会生成新的线程执行任务，但当系统的任务达到corePoolSize，就不会继续增加。若后续仍有新的任务加入，而又没有空闲的线程资源，则任务直接进入任务队列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存；

④优先任务队列（PriorityBlockingQueue）：优先任务队列是带有执行优先级的队列，它通过PriorityBlockingQueue实现，可以控制任务的执行先后顺序，它是一个特殊的无界队列。无论是有界队列ArrayBlockingQueue，还是未指定大小的无界队列LinkedBlockingQueue都是按照先进先出算法处理任务的。而PriorityBlockingQueue则可以根据任务自身的优先级顺序先后执行，在确保系统性能的同时，也能有很好的质量保证（总是确保高优先级的任务先执行）。

## 3.拒绝策略

①AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；

②CallerRunsPolicy策略：只要线程池为关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，认为提交线程的性能极有可能会急剧下降；

③DiscardOledestPolicy策略：该策略将丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务；

④DiscardPolicy策略：该策略默默地丢弃无法处理的任务，不予任何处理。

## 4.优化线程池数量

Ncpu = CPU的数量

Ucpu = 目标CPU的使用率，0  <= Ucpu <= 1

W/C = 等待时间与计算时间的比率

最优的线程池大小等于：

Nthreads = Ncpu * Ucpu * (1 + W/C)