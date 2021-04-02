# OOM排查以及常用命令

https://www.cnblogs.com/valjeanshaw/p/13130102.html  

https://blog.csdn.net/qq_16681169/article/details/53296137  

## 常用命令介绍

| 名称     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `jps`    | JVM 进程状态工具。显示系统内的所有 JVM 进程。                |
| `jstat`  | JVM 统计监控工具。监控虚拟机运行时状态信息，它可以显示出 JVM 进程中的类装载、内存、GC、JIT 编译等运行数据。 |
| `jmap`   | JVM 堆内存分析工具。用于打印 JVM 进程对象直方图、类加载统计。并且可以生成堆转储快照（一般称为 heapdump 或 dump 文件）。 |
| `jstack` | JVM 栈查看工具。用于打印 JVM 进程的线程和锁的情况。并且可以生成线程快照（一般称为 threaddump 或 javacore 文件）。 |
| `jhat`   | 用来分析 jmap 生成的 dump 文件。                             |
| `jinfo`  | JVM 信息查看工具。用于实时查看和调整 JVM 进程参数。          |
| `jcmd`   | JVM 命令行调试 工具。用于向 JVM 进程发送调试命令。           |

https://dunwu.github.io/javacore/jvm/jvm-cli-tools.html#_4-2-thread-dump-%E6%96%87%E4%BB%B6  

  

## 什么是OOM

Out Of Memory的简称，就是程序需要的内存大于系统分配的内存空间，程序crash。  

## 导致OOM的原因

1. 分配的少了
2. 应用用得太多，用完没释放

**内存泄露**：申请使用完的内存没有释放，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。  

**内存溢出**：申请的内存超出了JVM能提供的内存大小，此时称之为溢出。  

最常见的OOM情况有以下三种：  

* java.lang.OutOfMemoryError: Java heap space ------>**java堆内存溢出**，此种情况最常见，**一般由于内存泄露或者堆的大小设置不当引起**。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。

* java.lang.OutOfMemoryError: PermGen space 或 java.lang.OutOfMemoryError：MetaSpace ------>**java方法区，（java8 元空间）溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。**此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize=64m -XX:MaxPermSize=256m的形式修改。另外，过多的常量尤其是字符串也会导致方法区溢出。
* java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。**JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出**。可以通过虚拟机参数-Xss来设置栈的大小。

## 排查手段

一般是通过内存映像工具对Dump出来的堆转储快照进行分析，**重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏还是内存溢出。**  

* 如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链。这样就能够找到泄漏的对象是通过怎么样的路径与GC Roots相关联的导致垃圾回收机制无法将其回收。掌握了泄漏对象的类信息和GC Roots引用链的信息，就可以比较准确地定位泄漏代码的位置。
* 如果不存在泄漏，那么就是内存中的对象确实必须存活着，那么此时就需要通过虚拟机的堆参数（ -Xmx和-Xms）来适当调大参数；从代码上检查是否存在某些对象存活时间过长、持有时间过长的情况，尝试减少运行时内存的消耗。

## 实战排查

### 1. 查看服务器情况

利用top命令，实时显示系统中各个进程的资源占用状况。

### 2. 查看Java进程不服务的原因

考虑Java进程突然不可用的原因。这里推荐一个很实用的命令:dmesg。dmesg可以用来查看开机之后的系统日志，其中可以捕捉到一些系统资源与进程的变化信息。`dmesg |grep -E ‘kill|oom|out of memory’ `来搜索内存溢出的信息挺实用。我们这次就是用来这个命令查出来是内存溢出的原因。

### 3. 查看JVM状态

关于OOM出现的情况，一般可以猜想是内存泄露，或者是加载了过多class或者创建了过多对象，给JVM分配的内存不够导致。于是我们用一下常用的JDK自带工具来观察下JVM的状态:  

1. `ps -aux|grep java`当服务重新部署后，可以找出当前Java进程的PID
2. `jstat -gcutil pid interval`用于查看当前GC的状态,它对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。
3. `jmap -histo:live pid` 可用统计存活对象的分布情况，从高到低查看占据内存最多的对象。

### 4. Java dump分析问题

```
public class OomDemo {
    public static void main(String[] args) {
        StringBuilder stringBuilder = new StringBuilder();
        while(true){
            stringBuilder.append(System.currentTimeMillis());
        }
    }
}
```

执行代码时，限制Java堆内存大小，并设置dump的位置  

```
java -Xmx10m -Xms10m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=oom.hprof OomDemo 
```

堆转储快照也可以通过jmap实时生成

```
jmap -dump:format=b,file=$java_pid.hprof     #java_pid为java进程ID
```

运行程序后，出现如下错误：

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to D:\oom.hprof ...
Heap dump file created [3845788 bytes in 0.033 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:700)
	at java.lang.StringBuilder.append(StringBuilder.java:214)
	at OomDemo.main(OomDemo.java:5)
```

用MAT打开堆转储文件，即可定位到OOM发生的代码段，然后具体代码具体分析  

# Java 死锁排查

https://blog.csdn.net/u010648555/article/details/80721815  

## 1. 根据文章中的代码，造死锁

代码造死锁利用了占用了一个Object后，又抢占另一个锁，然后造成了环路等待  

## 2. 利用jps + jstack

jps查询出进程号PID

```
jps -l
```

利用jstack，查出发生死锁的线程  

```
jstack -l PID
```

或者利用JConsole，也可以查询堆栈信息，检测死锁  



# Java CPU 100%占用排查

1. 利用top查询占cpu资源高的进程
2. jps找到当前用户下Java程序PID
3. 利用`pidstat -p < PID > 1 3 -u -t`命令查询CPU占用高的线程TID
4. 将TID转为十六进制
5. `jstack -l pid`查询线程信息
6. 查找 TID对应的线程(输出的线程id为十六进制)，找到对应的代码

# Java服务宕机排查

https://blog.csdn.net/qq_33589510/article/details/104567442

## 可能原因

* 进程闪退
  * 内部崩溃
  * 外部终止
* 线程锁死或无限等待
* 内存溢出

## 进程闪退

### 内部崩溃

JVM 发生内部崩溃，必然会生成"hs_err_pid"开头的文件。  

一种常见情况：无法申请内存，显示`commit_memory`错误

这一般是因为 `Xmx` 设置过大，超过系统可用内存，JVM 申请内存失败。

#### 解决方案

- 减少Xmx值使得所有的综合不超过服务器物理内存
- 调整 Xms=Xmx
- 服务器不要运行其他不必要的东西
- 配置一部分swap空间（虚拟内存）

### 外部终止

如果找不到"hs_err_pid"开头的文件，那么这个进程的闪退必然是被从外部终止的。

#### OOMKILLER


java长期内存占用过高，系统需要内存使用的时候没有内存，Linux的oomkiller机制会干掉最低优先级的内存

检查 /var/logs/message ， /var/logs/dmesg或者对应日期文件，看看有没有类似下面的内容，日志有时间可以判断



## 线程锁死/无限等待

表现是：系统无法访问时，CPU占用非常低

使用jstack输出线程堆栈排查即可。  

或者用jprofiler工具看堆栈，或者其他任何可以拿到堆栈的工具都可以， java的堆栈就是java方法调用的路径，可以定位一些简单的问题  

## 内存溢出

现象：CPU全部占满，内存达到Xmx最大值

### CPU占满缘由

并不是 CPU 不够用，大部分情况来说CPU都是过剩的，而是涉及到JVM的GC 机制。  

JVM 使用GC的方法来回收没有被引用的内存块，**在当前的回收机制中，回收器是并发进行的**，回收的线程个数有一个公式：

> 当CPU核心数：
>
> * 小于等于8：1个核心对应一个GC线程
> * 大于8：gc的线程数= 8 + ((N - 8) * 5/8)
>
> ```
> threads = N <= 8 ? N : (8 + ((N - 8) * 5/8))
> ```

当然也可以通过参数 `-XX:ParallelCMSThreads=20` 来配置 GC 线程数，就不会使用默认的设置，默认情况下不要调整，因为调了也没什么卵用，最多在宕机的时候cpu占用按照你设定的值来。    

当发生内存溢出的时候，或者快要内存溢出的时候，不一定是内存溢出，JVM 发现内存不够了，就会 GC，所有线程开始工作，暂停 JVM 运行，开始回收，如果回收到内存了，jvm可以正确继续执行。  

这也就是为什么有时候配置内存溢出的参数没有自动生成dump的原因，因为他能运行，但是比较慢，所以没有OOM，就不会生成dump。  

**但如果没有回收到什么内存，gc会循环持续执行，这就导致了cpu全部占满的现象，所以说内存溢出的时候，一定伴随cpu占满（按照设置或者公式计算的线程量）**    



# GC调优

https://tech.meituan.com/2017/12/29/jvm-optimize.html  

