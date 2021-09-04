# Java零碎笔记

## 1.Java 中各种概念

### 1.1 变量

通常说的变量，就是将对象的状态存储到字段中，如下列代码所示的就是变量：

``` java
int num = 1;
```

### 1.2 静态变量

类成员，属于类不属于某一个实例对象，一般有关键字 `static` 修饰，如下列代码所示的就是静态变量：

``` java
static int num = 1;
```

### 1.3 常量

不可改变的量，一般由关键字 `final` 修饰，如下列代码所示的就是常量：

``` java
final int num = 1;
```

### 1.4 静态常量

由关键字 `static` 和 `final` 同时修饰的变量，属于类的不可变量，如下列代码所示的就是静态常量：

```java
static final int num = 1;
```

### 1.5 局部变量

有局部作用域的变量，这样的变量只能由声明它的方法或块中访问。如下面的例子所示，变量 num 只在 get 方法内部起作用，为局部变量：

``` java
public void get() {
    int num = 1;
}
```

### 1.6 字面量

字面量可以理解为实际值，如下列代码所示：

``` java
int num = 1;
String str = "Hello";
```

其中上面代码中的 1 和 Hello 就是字面量。

### 1.7 符号引用

符号引用就是一个字符串，只要我们在代码中引用了一个非字面量的东西，不管它是变量还是常量，它都只是由一个字符串定义的符号，在编译的时候，无法确定其内存地址。这个字符串存在常量池里，类加载的时候第一次加载到这个符号时，就会将这个符号引用（字符串）解析成直接引用（指针），如下列代码所示：

``` java
String str = "Hello";
System.out.println(str);
```

其中 `System.out.println(str);` 代码中的 str 在编译的时候就会编译为符号引用，在类加载时就解析成直接引用。

### 1.8 引用

引用是对象的别名，其实值就是功能受限但是安全性更高的指针。

### 1.9 句柄

句柄是指针的指针，句柄实际上是一个数据，是一个Long (整长型)的数据。句柄是一个标识符，是拿来标识对象或者项目的。

## 2.Java 线程状态

[Java线程的生命周期](https://juejin.cn/post/6844903558433734669)

[Java线程的六种状态以及切换](https://segmentfault.com/a/1190000038392244)

## 3.常用命令

### 3.1 jmap

**3.1 -heap**

查看堆内存初始化配置信息以及堆内存使用情况，垃圾收集器类型。

``` shell
$ jmap -heap <pid>
```

举例：

``` shell
$ jmap -heap 10936
Attaching to process ID 10936, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.111-b14

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   # 最大堆内存
   MaxHeapSize              = 3068133376 (2926.0MB)
   # 单个线程栈的初始内存
   NewSize                  = 63963136 (61.0MB)
   # 单个线程栈的最大内存
   MaxNewSize               = 1022361600 (975.0MB)
   OldSize                  = 128974848 (123.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 96468992 (92.0MB)
   used     = 64589336 (61.597190856933594MB)
   free     = 31879656 (30.402809143066406MB)
   66.9534683227539% used
From Space:
   capacity = 7864320 (7.5MB)
   used     = 5820880 (5.5512237548828125MB)
   free     = 2043440 (1.9487762451171875MB)
   74.01631673177083% used
To Space:
   capacity = 8388608 (8.0MB)
   used     = 0 (0.0MB)
   free     = 8388608 (8.0MB)
   0.0% used
PS Old Generation
   capacity = 79691776 (76.0MB)
   used     = 7781416 (7.420936584472656MB)
   free     = 71910360 (68.57906341552734MB)
   9.76439024272718% used

12820 interned Strings occupying 1111304 bytes.
```

**3.2 -histo**

统计各个类的实例数目以及占用内存，并按照内存使用量从多至少的顺序排列。

``` shell
$ jmap -histo <pid>
```

举例（live只统计堆中的存活对象）：

``` shell
$ jmap -histo:live 10936
 num     #instances         #bytes  class name
----------------------------------------------
   1:         30083        2808864  [C
   2:         29770         714480  java.lang.String
   3:          6383         705440  java.lang.Class
   4:         18722         599104  java.util.concurrent.ConcurrentHashMap$Node
   5:          9350         546880  [Ljava.lang.Object;
   6:          1838         368728  [B
   7:          2946         336968  [I
   8:          3094         248992  [Ljava.util.WeakHashMap$Entry;
   9:          5644         225760  java.util.LinkedHashMap$Entry
  10:           114         212704  [Ljava.util.concurrent.ConcurrentHashMap$Node;
```

**3.3 -dump**

``` shell
$ jmap -dump <pid>
```

把堆内存的使用情况dump到文件中，live只保存堆中的存活对象

```shell
$ jmap -dump[:live][,format=b][,file=dump.bin] <pid>
```

> Heap dump是Java进程在特定时间点的一个内存快照，快照包含在dump时间点java堆中的对象和类的信息。 通常在dump时, 会触发一个完整的GC, 故而Heap Dump文件中只包含哪些未被回收的对象的信息。

举例：

``` shell
$ C:\Users\Silence>jmap -dump:live,format=b,file=dump.bin 10936
Dumping heap to C:\Users\Silence\dump.bin ...
Heap dump file created
```

### 3.2  jstat

jstat可用来打印目标Java进程的性能数据，以-gc为前缀的子命令，它们将打印垃圾回收相关的数据。

### 3.3 jstack

jstack可以用来打印目标Java进程中各个线程的栈轨迹，以及这些线程所持有的锁。

## 4.Java启动参数

[Java启动参数](https://www.cnblogs.com/haycheng/p/12781261.html)

### 4.1 打印GC日志

当堆内存空间溢出时输出堆的内存快照：

```shell
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/java/logs/
```

输出 GC 详细日志并指定日志路径：

```diff
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 以基准时间的形式输出GC的时间戳（JVM 启动时间为起点的相对时间）
-XX:+PrintGCDateStamps 以日期的形式输出GC的时间戳（2013-05-04 T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:/data/java/logs/usercenter/gc.log 日志文件的输出路径
```

