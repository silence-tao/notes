# 1.Java垃圾回收机制

## 简介

Java 与 C++ 之间有一堵由内存动态分配和垃圾收集技术所围成的“高墙”，墙外的人想进去，墙里面的人想出来。

Java 内存运行时数据区域中的程序计数器、虚拟机栈、本地方法栈随线程而生灭；栈中的栈帧随着方法的进入和退出而有条不紊地执行着入栈和出栈操作。每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知的（尽管在运行期会由 JIT 编译器进行一些优化），因此这几个区域的内存分配和回收都具备确定性，不需要过多考虑回收的问题，因为方法结束或者线程结束时，内存自然就跟随着回收了。

而 Java 堆不一样，一个接口中的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存也可能不一样，我们只有在程序处于运行期间时才能知道会创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的是这部分内存。

![10022_01](../img/10022_01.png)

垃圾回收（Garbage Collection）是 Java 虚拟机（JVM）垃圾回收器提供的一种用于在空闲时间不定时回收无任何对象引用的对象占据的内存空间的一种机制。垃圾回收回收的是无任何引用的对象占据的内存空间而不是对象本身。换言之，垃圾回收只会负责释放那些对象占有的内存。对象是个抽象的词，包括引用和其占据的内存空间。当对象没有任何引用时其占据的内存空间随即被收回备用，此时对象也就被销毁。但不能说是回收对象，可以理解为一种文字游戏。

## 1.判断对象是否存活

在堆里面存放着 Java 世界中几乎所有的对象实例，垃圾收集器在对堆进行回收前，需要确定这些对象之中哪些还“存活”着，哪些已经“死去”（即没有任何途径使用的对象）。

### 1.1 引用计数器

给对象添加一引用计数器，被引用一次计数器值就加 1；当引用失效时，计数器值就减 1；计数器为 0 时，对象就是不可能再被使用的，简单高效，缺点是无法解决对象之间相互循环引用的问题。

### 1.2 可达性分析算法

通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的。此算法解决了上述循环引用的问题。

![10022_02](../img/10022_02.png)

在 Java 语言中，可作为 GC Roots 的对象包括下面几种：

1. 虚拟机栈（栈帧中的本地变量表）中引用的对象；

2. 方法区中类静态属性引用的对象；

3. 方法区中常量引用的对象；

4. 本地方法栈中 JNI（Native方法）引用的对象。

作为 GC Roots 的节点主要在全局性的引用与执行上下文中。要明确的是，tracing gc 必须以当前存活的对象集为 Roots，因此必须选取确定存活的引用类型对象。GC 管理的区域是 Java 堆，虚拟机栈、方法区和本地方法栈不被 GC 所管理，因此选用这些区域内引用的对象作为 GC Roots，是不会被 GC 所回收的。

其中虚拟机栈和本地方法栈都是线程私有的内存区域，只要线程没有终止，就能确保它们中引用的对象的存活。而方法区中类静态属性引用的对象是显然存活的。常量引用的对象在当前可能存活，因此，也可能是 GC roots 的一部分。

### 1.3 引用类型

在 JDK 1.2 以前，Java 中的引用的定义很传统：如果 reference 类型的数据中存储的数值代表的是另一块内存的起始地址，就称这块内存代表着一个引用。而 JDK 1.2 之后，Java 对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）4 种，这 4 种引用强度依次逐渐减弱。

1. 强引用就是指在程序代码之中普遍存在的，类似 `Object obj = new Object()` 这类的引用，垃圾收集器永远不会回收存活的强引用对象；

2. 软引用就是指还有用但并非必需的对象。在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收；

3. 弱引用也是用来描述非必需对象的，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论内存是否足够，都会回收掉只被弱引用关联的对象；

4. 虚引用是最弱的一种引用关系。无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

![10022_03](../img/10022_03.png)

### 1.4 可达性分析过程

不可达的对象将暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程：

1. 如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 finalize() 方法。

2. 当对象没有覆盖 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”，直接进行第二次标记。

3. 如果这个对象被判定为有必要执行 finalize() 方法，那么这个对象将会放置在一个叫做 F-Queue 的队列之中，并在稍后由一个由虚拟机自动建立的、低优先级的 Finalizer 线程去执行它。

4. 这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，因为如果一个对象在 finalize() 方法中执行缓慢，将很可能会一直阻塞 F-Queue 队列，甚至导致整个内存回收系统崩溃。

使用 finalize() 方法来“拯救”对象的方法是不值得提倡的，它的运行代价高昂，不确定性大，无法保证各个对象的调用顺序。finalize() 能做的工作，使用 try-finally 或者其它方法都更适合、及时，所以可以尽量忘记这个方法的存在。

### 1.5 回收方法区

永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。

1. 回收废弃常量与回收 Java 堆中的对象非常类似，假如一个字符串 "abc" 已经进入了常量池中，但是当前系统没有任何一个 String 对象是叫做 "abc" 的，也没有其他地方引用了这个字面量，如果这时发生内存回收，而且必要的话，这个 "abc" 常量就会被系统清理出常量池。常量池中的其他类（接口）、方法、字段的符号引用也与此类似。
2. 要判断一个类是否是“无用的类”的条件则相对苛刻许多，类需要满足以下 3 个条件才能算“无用的类”：
    - 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例；
    - 加载该类的 ClassLoader 已经被回收；
    - 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

虚拟机可以对满足上述 3 个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样，不使用了就必然会回收。

在大量使用反射、动态代理、CGLib 等 ByteCode 框架、动态生成 JSP 以及 OSGi 这类频繁自定义 ClassLoader 的场景都需要虚拟机具备类卸载的功能，以保证永久代不会溢出。

## 2.垃圾回收算法

垃圾回收算法是垃圾回收的理论基础，这里介绍四种垃圾回收算法：标记-清除算法、复制算法、标记-整理算法及分代收集算法。

### 2.1 标记-清除算法

最基础的收集算法是“标记-清除”（Mark-Sweep）算法，分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

它的主要不足有两个：

1. 效率问题，标记和清除两个过程的效率都不高；

2. 空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

标记—清除算法的执行过程如下图：

![10022_04](../img/10022_04.png)

### 2.2 复制算法

为了解决效率问题，一种称为“复制”（Copying）的收集算法出现了，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

这样使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为了原来的一半。复制算法的执行过程如下图：

![10022_05](../img/10022_05.png)

现在的商业虚拟机都采用这种算法来回收新生代，IBM 研究指出新生代中的对象 98% 是“朝生夕死”的，所以并不需要按照 1:1 的比例来划分内存空间，而是将内存分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 和其中一块 Survivor。当回收时，将 Eden 和 Survivor 中还存活着的对象一次性地复制到另外一块 Survivor 空间上，最后清理掉 Eden 和刚才用过的 Survivor 空间。HotSpot 虚拟机默认 Eden:Survivor = 8:1，也就是每次新生代中可用内存空间为整个新生代容量的 90%（其中一块 Survivor 不可用），只有 10% 的内存会被“浪费”。当然，98% 的对象可回收只是一般场景下的数据，我们没有办法保证每次回收都只有不多于 10% 的对象存活，当 Survivor 空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保（Handle Promotion）。

内存的分配担保就好比我们去银行借款，如果我们信誉很好，在 98% 的情况下都能按时偿还，于是银行可能会默认我们下一次也能按时按量地偿还贷款，只需要有一个担保人能保证如果我不能还款时，可以从他的账户扣钱，那银行就认为没有风险了。内存的分配担保也一样，如果另外一块 Survivor 空间没有足够空间存放上一次新生代收集下来的存活对象时，这些对象将直接通过分配担保机制进入老年代。

### 2.3 标记整理算法

复制算法在对象存活率较高时就要进行较多的复制操作，效率将会变低。更关键的是，如果不想浪费 50% 的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都 100% 存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉末端边界以外的内存，“标记-整理”算法的示意图如下：

![10022_06](../img/10022_06.png)

### 2.4 分代收集算法

当前商业虚拟机的垃圾收集都采用“分代收集”（Generational Collection）算法，根据对象存活周期的不同将内存划分为几块并采用不同的垃圾收集算法。

一般是把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记—清理”或者“标记—整理”算法来进行回收。

## 3.HotSpot的算法实现

### 3.1 枚举根节点

以可达性分析中从 GC Roots 节点找引用链这个操作为例，可作为 GC Roots 的节点主要在全局性的引用（例如常量或类静态属性）与执行上下文（例如栈帧中的本地变量表）中，现在很多应用仅仅方法区就有数百兆，如果要逐个检查这里面的引用，那么必然会消耗很多时间。

另外，可达性分析对执行时间的敏感还体现在 GC 停顿上，因为这项分析工作必须不可以出现分析过程中对象引用关系还在不断变化的情况，否则分析结果准确性就无法得到保证。这点是导致 GC 进行时必须停顿所有 Java 执行线程（Sun 将这件事情称为 "Stop The World"）的其中一个重要原因，即使是在号称（几乎）不会发生停顿的 CMS 收集器中，枚举根节点时也是必须要停顿的。

因此，目前的主流 Java 虚拟机使用的都是准确式 GC（即虚拟机可以知道内存中某个位置的数据具体是什么类型。），所以当执行系统停顿下来后，并不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机应当是有办法直接得知哪些地方存放着对象引用。

在 HotSpot 的实现中，是使用一组称为 OopMap 的数据结构来达到这个目的的，在类加载完成的时候，HotSpot 就把对象内什么偏移量上是什么类型的数据计算出来，在 JIT 编译过程中，也会在特定的位置记录栈和寄存器中哪些位置是引用。这样，GC 在扫描时就可以直接得知这些信息了。

### 3.2 安全点（Safepoint）

在 OopMap 的协助下，HotSpot 可以快速且准确地完成 GC Roots 枚举，但一个很现实的问题随之而来：可能导致引用关系变化，或者说 OopMap 内容变化的指令非常多，如果为每一条指令都生成对应的 OopMap，那将会需要大量的额外空间，这样 GC 的空间成本将会变得很高。

实际上，HotSpot 也的确没有为每条指令都生成 OopMap，前面已经提到，只是在“特定的位置”记录了这些信息，这些位置称为安全点，即程序执行时并非在所有地方都能停顿下来开始 GC ，只有在到达安全点时才能暂停。

Safepoint 的选定既不能太少以致于 GC 过少，也不能过于频繁以致于过分增大运行时的负荷。对于 Safepoint，另一个需要考虑的问题是如何在 GC 发生时让所有线程都“跑”到最近的安全点上再停顿下来。这里有两种方案可供选择：抢先式中断（Preemptive Suspension）和主动式中断（Voluntary Suspension）。

1. 其中抢先式中断不需要线程的执行代码主动去配合，在 GC 发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它“跑”到安全点上。现在几乎没有虚拟机实现采用抢先式中断来暂停线程从而响应 GC 事件。

2. 而主动式中断的思想是当 GC 需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的，另外再加上创建对象需要分配内存的地方。

### 3.3 安全区域（Safe Region）

使用 Safepoint 似乎已经完美地解决了如何进入 GC 的问题，但实际情况却并不一定。Safepoint 机制保证了程序执行时，在不太长的时间内就会遇到可进入 GC 的 Safepoint。但是，程序“不执行”的时候呢？所谓的程序不执行就是没有分配 CPU 时间，典型的例子就是线程处于 Sleep 状态或者 Blocked 状态，这时候线程无法响应 JVM 的中断请求，“走”到安全的地方去中断挂起，JVM 也显然不太可能等待线程重新被分配 CPU 时间。对于这种情况，就需要安全区域（Safe Region）来解决。

安全区域是指在一段代码片段之中，引用关系不会发生变化。在这个区域中的任意地方开始 GC 都是安全的。我们也可以把 Safe Region 看做是被扩展了的 Safepoint。在线程执行到 Safe Region 中的代码时，首先标识自己已经进入了 Safe Region，那样，当在这段时间里 JVM 要发起 GC 时，就不用管标识自己为 Safe Region 状态的线程了。在线程要离开 Safe Region 时，它要检查系统是否已经完成了根节点枚举（或者是整个 GC 过程），如果完成了，那线程就继续执行，否则它就必须等待直到收到可以安全离开 Safe Region 的信号为止。

> 本文摘自《深入理解Java虚拟机》

# 2.Java垃圾收集器

## 简介

如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。这里讨论的收集器基于 JDK 1.7 Update 14 之后的 HotSpot 虚拟机，这个虚拟机包含的所有收集器如下图所示：

![10023_01](../img/10023_01.png)

上图展示了 7 种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用。虚拟机所处的区域，则表示它是属于新生代收集器还是老年代收集器。接下来将逐一介绍这些收集器的特性、基本原理和使用场景，并重点分析 CMS 和 G1 这两款相对复杂的收集器，了解它们的部分运作细节。

## 1.Serial收集器

Serial 收集器是最基本、发展历史最悠久的收集器，曾经是虚拟机新生代收集的唯一选择。这是一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会使用一个 CPU 或一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。"Stop The World" 这个名字也许听起来很酷，但这项工作实际上是由虚拟机在后台自动发起和自动完成的，在用户不可见的情况下把用户正常工作的线程全部停掉，这对很多应用来说都是难以接受的。下图示意了 Serial/Serial Old 收集器的运行过程：

![10023_02](../img/10023_02.png)

实际上到现在为止，它依然是虚拟机运行在 Client 模式下的默认新生代收集器。它也有着优于其他收集器的地方：简单而高效（与其他收集器的单线程比），对于限定单个 CPU 的环境来说，Serial 收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。

在用户的桌面应用场景中，分配给虚拟机管理的内存一般来说不会很大，收集几十兆甚至一两百兆的新生代（仅仅是新生代使用的内存，桌面应用基本上不会再大了），停顿时间完全可以控制在几十毫秒最多一百多毫秒以内，只要不是频繁发生，这点停顿是可以接受的。所以，Serial 收集器对于运行在 Client 模式下的虚拟机来说是一个很好的选择。

## 2.ParNew收集器

ParNew 收集器其实就是 Serial 收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为包括 Serial 收集器可用的所有控制参数（例如：-XX:SurvivorRatio、-XX:PretenureSizeThreshold、-XX:HandlePromotionFailure等）、收集算法、Stop The World、对象分配规则、回收策略等都与 Serial 收集器完全一样，在实现上，这两种收集器也共用了相当多的代码。ParNew 收集器的工作过程如下图所示:

![10023_03](../img/10023_03.png)

ParNew 收集器除了多线程收集之外，其他与 Serial 收集器相比并没有太多创新之处，但它却是许多运行在 Server 模式下的虚拟机中首选的新生代收集器，其中有一个与性能无关但很重要的原因是，除了 Serial 收集器外，目前只有它能与 CMS 收集器（并发收集器，后面有介绍）配合工作。

ParNew 收集器在单 CPU 的环境中不会有比 Serial 收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个 CPU 的环境中都不能百分之百地保证可以超越 Serial 收集器。当然，随着可以使用的 CPU 的数量的增加，它对于 GC 时系统资源的有效利用还是很有好处的。它默认开启的收集线程数与 CPU 的数量相同，在 CPU 非常多（如32个)的环境下，可以使用 -XX:ParallelGCThreads 参数来限制垃圾收集的线程数。

注意，从 ParNew 收集器开始，后面还会接触到几款并发和并行的收集器。这里有必要先解释两个名词：并发和并行。这两个名词都是并发编程中的概念，在谈论垃圾收集器的上下文语境中，它们可以解释如下：

1. 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。

2. 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个 CPU 上。

## 3.Parallel Scavenge收集器

Parallel Scavenge 收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器……看上去和 ParNew 都一样，那它有什么特别之处呢？

Parallel Scavenge 收集器的特点是它的关注点与其他收集器不同，CMS 等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而 Parallel Scavenge 收集器的目标则是达到一个可控制的吞吐量（Throughput）。

所谓吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即 `吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）`，虚拟机总共运行了 100 分钟，其中垃圾收集花掉 1 分钟，那吞吐量就是 99%。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

Parallel Scavenge 收集器提供了两个参数用于精确控制吞吐量，分别是控制最大垃圾收集停顿时间的 -XX:MaxGCPauseMillis 参数以及直接设置吞吐量大小的 -XX:GCTimeRatio 参数。

MaxGCPauseMillis 参数允许的值是一个大于 0 的毫秒数，收集器将尽可能地保证内存回收花费的时间不超过设定值。不过大家不要认为如果把这个参数的值设置得稍小一点就能使得系统的垃圾收集速度变得更快，GC 停顿时间缩短是以牺牲吞吐量和新生代空间来换取的：系统把新生代调小一些，收集 300MB 新生代肯定比收集 500MB 快吧，这也直接导致垃圾收集发生得更频繁一些，原来 10 秒收集一次、每次停顿 100 毫秒，现在变成 5 秒收集一次、每次停顿70毫秒。停顿时间的确在下降，但吞吐量也降下来了。

GCTimeRatio 参数的值应当是一个 0 到 100 的整数，也就是垃圾收集时间占总时间的比率，相当于是吞吐量的倒数。如果把此参数设置为 19，那允许的最大 GC 时间就占总时间的 5%（即 1/（1+19）），默认值为 99，就是允许最大 1%（即 1/（1+99））的垃圾收集时间。

由于与吞吐量关系密切，Parallel Scavenge 收集器也经常称为“吞吐量优先”收集器。除上述两个参数之外，Parallel Scavenge 收集器还有一个参数 -XX:+UseAdaptiveSizePolicy 值得关注。这是一个开关参数，当这个参数打开之后，就不需要手工指定新生代的大小（-Xmn）、Eden 与 Survivor 区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量，这种调节方式称为 GC 自适应的调节策略（GC Ergonomics）。

## 4.Serial Old收集器

Serial Old 是 Serial 收集器的老年代版本，它同样是一个单线程收集器，使用“标记-整理”算法。这个收集器的主要意义也是在于给 Client 模式下的虚拟机使用。如果在 Server 模式下，那么它主要还有两大用途：一种用途是在 JDK 1.5 以及之前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途就是作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。这两点都将在后面的内容中详细讲解。Serial Old 收集器的工作过程如下图所示：

![10023_04](../img/10023_04.png)

## 5.Parallel Old收集器

Parallel Old 是Parallel Scavenge 收集器的老年代版本，使用多线程和“标记-整理”算法。这个收集器是在 JDK 1.6 中才开始提供的，在此之前，新生代的 Parallel Scavenge 收集器一直处于比较尴尬的状态。

原因是，如果新生代选择了 Parallel Scavenge 收集器，老年代除了 Serial Old（PS MarkSweep）收集器外别无选择（Parallel Scavenge 收集器无法与 CMS 收集器配合工作）。

由于老年代 Serial Old 收集器在服务端应用性能上的“拖累”，使用了 Parallel Scavenge 收集器也未必能在整体应用上获得吞吐量最大化的效果，由于单线程的老年代收集中无法充分利用服务器多 CPU 的处理能力，在老年代很大而且硬件比较高级的环境中，这种组合的吞吐量甚至还不一定有 ParNew 加 CMS 的组合“给力”。

直到 Parallel Old 收集器出现后，“吞吐量优先”收集器终于有了比较名副其实的应用组合，在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。Parallel Old 收集器的工作过程如下图所示：

![10023_05](../img/10023_05.png)

## 6.CMS收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。

目前很大一部分的 Java 应用集中在互联网站或者 B/S 系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS 收集器就非常符合这类应用的需求。

从名字（包含"Mark Sweep"）上就可以看出，CMS 收集器是基于“标记—清除”算法实现的，它的运作过程相对于前面几种收集器来说更复杂一些，整个过程分为4个步骤，包括：

1. 初始标记（CMS initial mark）

2. 并发标记（CMS concurrent mark）

3. 重新标记（CMS remark）

4. 并发清除（CMS concurrent sweep）

其中，初始标记、重新标记这两个步骤仍然需要 "Stop The World"。初始标记仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，并发标记阶段就是进行 GC Roots Tracing 的过程，而重新标记阶段则是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。

由于整个过程中耗时最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作，所以，从总体上来说，CMS 收集器的内存回收过程是与用户线程一起并发执行的。CMS 收集器的工作过程如下图所示：

![10023_06](../img/10023_06.png)

CMS 是一款优秀的收集器，它的主要优点在名字上已经体现出来了：并发收集、低停顿，但是 CMS 还远达不到完美的程度，它有以下 3 个明显的缺点：

1. 导致吞吐量降低。CMS 收集器对 CPU 资源非常敏感。其实，面向并发设计的程序都对 CPU 资源比较敏感。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用了一部分线程（或者说 CPU 资源）而导致应用程序变慢，总吞吐量会降低。CMS 默认启动的回收线程数是 `(CPU 数量 + 3) / 4`，也就是当 CPU 在 4 个以上时，并发回收时垃圾收集线程不少于 25% 的 CPU 资源，并且随着 CPU 数量的增加而下降。但是当 CPU 不足 4 个（譬如2个）时，CMS 对用户程序的影响就可能变得很大，如果本来 CPU 负载就比较大，还分出一半的运算能力去执行收集器线程，就可能导致用户程序的执行速度忽然降低了 50%，其实也让人无法接受。

2. CMS 收集器无法处理浮动垃圾（Floating Garbage），可能出现 "Concurrent Mode Failure" 失败而导致另一次 Full GC（新生代和老年代同时回收）的产生。由于 CMS 并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS 无法在当次收集中处理掉它们，只好留待下一次 GC 时再清理掉。这一部分垃圾就称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此 CMS 收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。在 JDK 1.5 的默认设置下，CMS 收集器当老年代使用了 68% 的空间后就会被激活，这是一个偏保守的设置，如果在应用中老年代增长不是太快，可以适当调高参数 -XX:CMSInitiatingOccupancyFraction 的值来提高触发百分比，以便降低内存回收次数从而获取更好的性能，在 JDK 1.6 中，CMS 收集器的启动阈值已经提升至 92% 。要是 CMS 运行期间预留的内存无法满足程序需要，就会出现一次 "Concurrent Mode Failure" 失败，这时虚拟机将启动后备预案：临时启用 Serial Old 收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数 -XX:CMSInitiatingOccupancyFraction 设置得太高很容易导致大量 "Concurrent Mode Failure" 失败，性能反而降低。

3. 产生空间碎片。 CMS 是一款基于“标记—清除”算法实现的收集器，这意味着收集结束时会有大量空间碎片产生。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次 Full GC 。为了解决这个问题，CMS 收集器提供了一个 -XX:+UseCMSCompactAtFullCollection 开关参数（默认就是开启的），用于在 CMS 收集器顶不住要进行 Full GC 时开启内存碎片的合并整理过程，内存整理的过程是无法并发的，空间碎片问题没有了，但停顿时间不得不变长。虚拟机设计者还提供了另外一个参数 -XX:CMSFullGCsBeforeCompaction，这个参数是用于设置执行多少次不压缩的 Full GC 后，跟着来一次带压缩的（默认值为 0，表示每次进入 Full GC 时都进行碎片整理）。

## 7.G1收集器

G1（Garbage-First）收集器是当今收集器技术发展的最前沿成果之一，G1 是一款面向服务端应用的垃圾收集器。HotSpot 开发团队赋予它的使命是（在比较长期的）未来可以替换掉 JDK 1.5 中发布的 CMS 收集器。

### 7.1 G1收集器的特点

与其他 GC 收集器相比，G1 具备如下特点：

1. 并行与并发：G1 能充分利用多 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU核心）来缩短 Stop-The-World 停顿的时间，部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 Java 程序继续执行。

2. 分代收集：与其他收集器一样，分代概念在 G1 中依然得以保留。虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但它能够采用不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次 GC 的旧对象以获取更好的收集效果。

3. 空间整合：与 CMS 的“标记—清理”算法不同，G1 从整体来看是基于“标记—整理”算法实现的收集器，从局部（两个 Region 之间）上来看是基于“复制”算法实现的，但无论如何，这两种算法都意味着 G1 运作期间不会产生内存空间碎片，收集后能提供规整的可用内存。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次 GC 。

4. 可预测的停顿：这是 G1 相对于 CMS 的另一大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒，这几乎已经是实时 Java（RTSJ）的垃圾收集器的特征了。

### 7.2 G1的内存结构 

理解垃圾回收机制，必须先了解 G1 的内存结构，在 G1 之前的其他收集器进行收集的范围都是整个新生代或者老年代，而 G1 不再是这样。使用 G1 收集器时，Java 堆的内存布局就与其他收集器有很大差别，它将整个 Java 堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分 Region（不需要连续）的集合。内存结构如下图：

![10023_07](../img/10023_07.png)

这里有三个关于内存的概念：代，区和内存分段。

G1 把堆内存分为年轻代和老年代。年轻代分为 Eden 和 Survivor 两个区，老年代分为 Old 和 Humongous 两个区。代和区都是逻辑概念。G1 把堆内存分为大小相等的内存分段，默认情况下会把内存分为 2048 个内存分段，可以用 -XX:G1HeapRegionSize 调整内存分段的个数。比如 32G 堆内存，2048 个内存分段每段的大小为 16M。这相当于把内存化整为零。内存分段是物理概念，代表实际的物理内存空间。每个内存分段都可以被标记为 Eden 区，Survivor 区，Old 区，或者 Humongous 区。这样属于不同代，不同区的内存分段就可以不必是连续内存空间了。

新分配的对象会被分配到 Eden 区的内存分段上，每一次年轻代的回收过程都会把 Eden 区存活的对象复制到 Survivor 区的内存分段上，把 Survivor 区继续存活的对象年龄加1，如果 Survivor 区的存活对象年龄达到某个阈值（比如15，可以设置），Survivor 区的对象会被复制到 Old 区。复制过程是把源内存分段中所有存活的对象复制到空的目标内存分段上，复制完成后，源内存分段没有了存活对象，变成了可以使用的空的 Eden 内存分段了；而目标内存分段的对象都是连续存储的，没有碎片，所以复制过程可以达到内存整理的效果，减少碎片。Humongous 区用于保存大对象，如果一个对象占用的空间超过内存分段的一半（比如上面的 8M），则此对象将会被分配在 Humongous 区。如果对象的大小超过一个甚至几个分段的大小，则对象会分配在物理连续的多个 Humongous 分段上。Humongous 对象因为占用内存较大并且连续会被优先回收。

G1 收集器之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个 Java 堆中进行全区域的垃圾收集。G1 在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region（这也就是 Garbage-First 名称的来由），保证了 G1 收集器在有限的时间内可以获取尽可能高的收集效率。

### 7.3 Remembered Set

理解回收过程，需要先了解记忆集合（Remembered Set），以下简称 RS。为了在回收单个内存分段的时候不必对整个堆内存的对象进行扫描（单个内存分段中的对象可能被其他内存分段中的对象引用）引入了 RS 数据结构。RS 使得 G1 可以在年轻代回收的时候不必去扫描老年代的对象，从而提高了性能。每一个内存分段都对应一个 RS，RS 保存了来自其他分段内的对象对于此分段的引用。对于属于年轻代的内存分段（Eden 和 Survivor 区的内存分段）来说，RS 只保存来自老年代的对象的引用。这是因为年轻代回收是针对全部年轻代的对象的，反正所有年轻代内部的对象引用关系都会被扫描，所以 RS 不需要保存来自年轻代内部的引用。对于属于老年代分段的 RS 来说，也只会保存来自老年代的引用，这是因为老年代的回收之前会先进行年轻代的回收，年轻代回收后 Eden 区变空了，G1 会在老年代回收过程中扫描 Survivor 区到老年代的引用。

RS 里的引用信息是怎么样填充和维护的呢？简而言之就是 JVM 会对应用程序的每一个引用赋值语句 `object.field = object` 进行记录和处理，把引用关系更新到 RS 中。但是这个 RS 的更新并不是实时的。G1 维护了一个 Dirty Card Queue。对于应用程序的引用赋值语句 `object.field = object`，JVM 会在之前和之后执行特殊的操作以在 Dirty Card Queue 中入队一个保存了对象引用信息的 card。在年轻代回收的时候，G1 会对 Dirty Card Queue 中所有的 card 进行处理，以更新 RS，保证 RS 实时准确的反映引用关系。那为什么不在引用赋值语句处直接更新 RS 呢？这是为了性能的需要，RS 的处理需要线程同步，开销会很大，使用队列性能会好很多。

在 G1 收集器中，Region 之间的对象引用以及其他收集器中的新生代与老年代之间的对象引用，虚拟机都是使用 Remembered Set 来避免全堆扫描的。G1 中每个 Region 都有一个与之对应的 Remembered Set，虚拟机发现程序在对 Reference 类型的数据进行写操作时，会产生一个 Write Barrier 暂时中断写操作，检查 Reference 引用的对象是否处于不同的 Region 之中（在分代的例子中就是检查是否老年代中的对象引用了新生代中的对象），如果是，便通过 CardTable 把相关引用信息记录到被引用对象所属的 Region 的 Remembered Set 之中。当进行内存回收时，在 GC 根节点的枚举范围中加入 Remembered Set 即可保证不对全堆扫描也不会有遗漏。

### 7.4 G1收集器的工作过程

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：

1. 初始标记（Initial Marking）

2. 并发标记（Concurrent Marking）

3. 最终标记（Final Marking）

4. 筛选回收（Live Data Counting and Evacuation）

G1 的前几个步骤的运作过程和 CMS 有很多相似之处。

初始标记阶段仅仅只是标记一下 GC Roots 能直接关联到的对象，并且修改 TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的 Region 中创建新对象，这阶段需要停顿线程，但耗时很短。

并发标记阶段是从 GC Root 开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行。

而最终标记阶段则是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中，这阶段需要停顿线程，但是可并行执行。

最后在筛选回收阶段首先对各个 Region 的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划，从 Sun 公司透露出来的信息来看，这个阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅提高收集效率。通过下图可以比较清楚地看到 G1 收集器的运作步骤中并发和需要停顿的阶段：

![10023_08](../img/10023_08.png)

### 7.5 G1年轻代回收过程（Young GC）

JVM 启动时，G1 先准备好 Eden 区，程序在运行过程中不断创建对象到 Eden 区，当所有的 Eden 区都满了，G1 会启动一次年轻代垃圾回收过程。年轻代只会回收 Eden 区和 Survivor 区。首先 G1 停止应用程序的执行（Stop-The-World），G1 创建回收集（Collection Set），回收集是指需要被回收的内存分段的集合，年轻代回收过程的回收集包含年轻代 Eden 区和 Survivor 区所有的内存分段。然后开始如下回收过程：

第一阶段，扫描根。

根是指 static 变量指向的对象，正在执行的方法调用链条上的局部变量等。根引用连同 RS 记录的外部引用作为扫描存活对象的入口。

第二阶段，更新 RS。

处理 Dirty Card Queue中的 card，更新 RS。此阶段完成后，RS 可以准确的反映老年代对所在的内存分段中对象的引用。

第三阶段，处理 RS。

识别被老年代对象指向的 Eden 中的对象，这些被指向的 Eden 中的对象被认为是存活的对象。

第四阶段，复制对象。

此阶段，对象树被遍历，Eden 区内存段中存活的对象会被复制到 Survivor 区中空的内存分段，Survivor 区内存段中存活的对象如果年龄未达阈值，年龄会加 1，达到阀值会被会被复制到 Old 区中空的内存分段。

第五阶段，处理引用。

处理 Soft，Weak，Phantom，Final，JNI Weak 等引用。

### 7.6 G1老年代并发标记过程（Concurrent Marking）

当整个堆内存（包括老年代和新生代）被占满一定大小的时候（默认是 45%，可以通过 -XX:InitiatingHeapOccupancyPercent 进行设置），老年代回收过程会被启动。具体检测堆内存使用情况的时机是年轻代回收之后或者 Houmongous 对象分配之后。老年代回收包含标记老年代内的对象是否存活的过程，标记过程是和应用程序并发运行的（不需要 Stop-The-World）。

应用程序会改变指针的指向，并发执行的标记过程怎么能保证标记过程没有问题呢？并发标记过程有一种情形会对存活的对象标记不到。假设有对象 A，B 和 C，一开始的时候 B.c = C，A.c = null。当 A 的对象树先被扫描标记，接下来开始扫描 B 对象树，此时标记线程被应用程序线程抢占后停下来，应用程序把 A.c = C，B.c = null。当标记线程恢复执行的时候 C 对象已经标记不到了，这时候 C 对象实际是存活的，这种情形被称作对象丢失。G1 解决的方法是在对象引用被设置为空的语句（比如 B.c = null）时，把原先指向的对象（C 对象）保存到一个队列，代表它可能是存活的。然后会有一个重新标记（Remark）过程处理这些对象，重新标记过程是 Stop-The-World 的，所以可以保证标记的正确性。上述这种标记方法被称为开始时快照技术（SATB，Snapshot At The Begging）。这种方式会造成某些是垃圾的对象也被当做是存活的，所以 G1 会使得占用的内存被实际需要的内存大。

具体标记过程如下：

1. 先进行一次年轻代回收过程，这个过程是 Stop-The-World 的。老年代的回收基于年轻代的回收（比如需要年轻代回收过程对于根对象的收集，初始的存活对象的标记）；

2. 恢复应用程序线程的执行；

3. 开始老年代对象的标记过程。此过程是与应用程序线程并发执行的。标记过程会记录弱引用情况，还会计算出每个分段的对象存活数据（比如分段内存活对象所占的百分比）；

4. Stop-The-World；

5. 重新标记（Remark）。此阶段重新标记前面提到的 STAB 队列中的对象（例子中的 C 对象），还会处理弱引用；

6. 回收百分之百为垃圾的内存分段。（注意：不是百分之百为垃圾的内存分段并不会被处理，这些内存分段中的垃圾是在混合回收过程（Mixed GC）中被回收的。）由于 Humongous 对象会独占整个内存分段，如果 Humongous 对象变为垃圾，则内存分段百分百为垃圾，所以会在第一时间被回收掉；

7. 恢复应用程序线程的执行。

### 7.7 混合回收过程（Mixed GC）

重新标记过程结束以后，紧跟着就会开始混合回收过程。混合回收的意思是年轻代和老年代会同时被回收。重新标记结束以后，老年代中百分百为垃圾的内存分段被回收了，部分为垃圾的内存分段被计算了出来。默认情况下，这些老年代的内存分段会分 8 次（可以通过 -XX:G1MixedGCCountTarget 设置）被回收。混合回收的回收集（Collection Set）包括八分之一的老年代内存分段，Eden 区内存分段，Survivor 区内存分段。混合回收的算法和年轻代回收的算法完全一样，只是回收集多了老年代的内存分段。具体过程请参考上面的年轻代回收过程。

由于老年代中的内存分段默认分 8 次回收，G1 会优先回收垃圾多的内存分段。垃圾占内存分段比例越高的，越会被先回收。并且有一个阈值会决定内存分段是否被回收，-XX:G1MixedGCLiveThresholdPercent，默认为 65%，意思是垃圾占内存分段比例要达到 65% 才会被回收。如果垃圾占比太低，意味着存活的对象占比高，在复制的时候会花费更多的时间。

混合回收并不一定要进行 8 次。有一个阈值 -XX:G1HeapWastePercent，默认值为 10%，意思是允许整个堆内存中有 10% 的空间被浪费，意味着如果发现可以回收的垃圾占堆内存的比例低于 10%，则不再进行混合回收。因为 GC 会花费很多的时间但是回收到的内存却很少。

### 7.8 Full GC

Full GC 是指上述方式不能正常工作，G1 会停止应用程序的执行（Stop-The-World），使用单线程的内存回收算法进行垃圾回收，性能会非常差，应用程序停顿时间会很长。要避免 Full GC 的发生，一旦发生需要进行调整。什么时候回发生 Full GC 呢？比如堆内存太小，当 G1 在复制存活对象的时候没有空的内存分段可用，则会回退到 Full GC，这种情况可以通过增大内存解决。

### 7.9 其他概念 

1.线程本地分配缓冲区（TLAB: Thread Local Allocation Buffer）

由于堆内存是应用程序共享的，应用程序的多个线程在分配内存的时候需要加锁以进行同步。为了避免加锁，提高性能每一个应用程序的线程会被分配一个 TLAB。TLAB 中的内存来自于 G1 年轻代中的内存分段。当对象不是 Humongous 对象，TLAB 也能装的下的时候，对象会被优先分配于创建此对象的线程的 TLAB 中。这样分配会很快，因为 TLAB 隶属于线程，所以不需要加锁。

2.GC“提升”线程本地分配缓冲区（PLAB: Promotion Thread Local Allocation Buffer）

前面提到过，G1 会在年轻代回收过程中把 Eden 区中的对象复制（“提升”）到 Survivor 区中，Survivor 区中的对象复制到 Old 区中。G1 的回收过程是多线程执行的，为了避免多个线程往同一个内存分段进行复制，那么复制的过程也需要加锁。为了避免加锁，G1 的每个线程都关联了一个 PLAB，这样就不需要进行加锁了。

3.Remembered Set粒度

其实RS的存储分三种粒度，前面提到的 Card 是最小的一种粒度。粒度的存在是因为某些内存分段中的对象可能很热门，被来自非常多的区的对象所引用，为了避免保存太多的数据，会以更大的粒度来保存这些引用，比如最大的粒度是用一个 bitmap 来保存其他内存分段对 RS 所对应的内存分段的引用。每一个内存分段对应一个 bit，如果 bit 为 0 表示该 bit 对应的内存分段中没有引用，为 1 表示有引用。这种方式会减少 RS 的数据，但是会增加扫描和标记时的开销，因为需要扫描所有 bit 为 1 的内存分段中的对象以确定具体是来自哪个对象的引用。 

> 本文摘自《深入理解Java虚拟机》