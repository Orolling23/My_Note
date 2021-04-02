# GoLang
https://github.com/geektutu/interview-questions  

## Goroutine
**Go为了提供更容易使用的并发方法，使用了goroutine和channel**。goroutine来自协程的概念，让一组可复用的函数运行在一组线程之上，即使有协程阻塞，该线程的其他协程也可以被runtime调度，转移到其他可运行的线程上。最关键的是，程序员看不到这些底层的细节，这就降低了编程的难度，提供了更容易的并发。  

Go中，协程被称为goroutine，它非常轻量，**一个goroutine只占几KB**，并且这几KB就足够goroutine运行完，这就能在有限的内存空间内支持大量goroutine，支持了更多的并发。虽然一个goroutine的栈只占几KB，但实际是可伸缩的，如果需要更多内容，runtime会自动为goroutine分配。  

## GPM
GMP是Go的并发模型方案，其中M是machine，P是Processer，G是goroutine。  

![GPM](./Pics/GPM.jpg)  

* M代表了系统的线程，是对内核级线程的封装；
* G是协程，为用户及的轻量级线程，每个Goroutine对象中的sched保存着其上下文信息；G并非执行体，G必须要绑定到P才能执行。
* P是G和M的调度器，用来调度G和M之间的关联关系，其数量可通过GOMAXPROCS来设置，默认为核心数。对于G来说，P相当于CPU核心，G只有绑定到P才能被执行。对M来说，P提供了相关的执行环境，如内存分配状态，任务队列等。**P的数量决定了最大可并行的G的数量（前提是CPU核心数>=P的数量）**   
* 全局队列：存放等待运行的G
* P的本地队列：存放等待运行的G，存的数量有限，不超过256个。新建G时，G优先加入的P的本地队列，如果队列满了则会把本地队列中一半的G移动到全局队列。
* P列表：所有P都在程序启动时创建，并保存在数组中，最多有GOMAXPROCS个
* Shed：Go调度器，它维护有存储 M 和 G 的队列以及调度器的一些状态信息等。

### go func() 调度流程
1. 通过go func() 创建一个goroutine
2. 有两个存储G的队列，一个是P局部的，一个是全局的。新创建的G会先保存在P的本地队列，如果P的满了就存在全局的队列中。
3. G只能运行在M中，M必须持有一个P。M:P=1:1。M会从P的本地队列弹出一个可执行状态的G来执行，如果P的本地队列为空，会从其他的MP组合**窃取一个可执行的G来执行**  
4. 一个M调度G的执行过程是一个循环机制
5. 当M执行某个G时发生了syscall或者其余的阻塞操作，M会被阻塞。如果当前有一些G在执行，runtime会把这个线程M从P中摘除，然后创建一个新的操作系统线程（如果有空闲线程则复用）来服务这个P
6. 当M系统调用结束后，这个G会尝试获取一个空闲的P，并放入这个P的本地队列，如果获取不到，那么这个M变成休眠状态，加入到空闲线程中，然后这个G会被放入全局队列中。

这其中涉及到几个优化的设计策略：
* 复用线程：避免频繁创建、销毁线程，而是对线程复用
* work stealing：当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。
* hand off：当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执行。
* 利用并行：GOMAXPROCS限制并发的程度
* 全局G队列：全局G队列的作用被弱化了。当M执行任务窃取，从别的P队列偷不到G时，则从全局G队列获取G

## Channel
channel是Golang在语言层面提供的goroutine间的通信方式

### 数据结构
`src/runtime/chan.go:hchan`定义了channel的数据结构，从数据结构可以看出channel由队列、类型信息、goroutine等待队列组成，下面分别说明其原理。  

```
type hchan struct {
	qcount   uint           // 当前队列中剩余元素个数
	dataqsiz uint           // 环形队列长度，即可以存放的元素个数
	buf      unsafe.Pointer // 环形队列指针
	elemsize uint16         // 每个元素的大小
	closed   uint32	        // 标识关闭状态
	elemtype *_type         // 元素类型
	sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
	recvx    uint           // 队列下标，指示元素从队列的该位置读出
	recvq    waitq          // 等待读消息的goroutine队列
	sendq    waitq          // 等待写消息的goroutine队列
	lock mutex              // 互斥锁，chan不允许并发读写
}
```

### 环形队列
chan内部实现了一个**环形队列作为其缓冲区**，队列的长度是创建chan时指定的。  

### 等待队列
从channel读数据，如果channel缓冲区已空或者没有缓冲区，当前goroutine会被阻塞。  

从channel写数据，如果channel缓冲区已满或者没有缓冲区，当前goroutine会被阻塞。  

被阻塞的goroutine会被挂在channel的**等待队列**中：  

* 因读阻塞的goroutine会被向channel写入数据的goroutine唤醒；
* 因写阻塞的goroutine会被从channel读数据的goroutine唤醒；

### 类型信息
一个channel只能传递一种类型的值，类型信息存储在hchan数据结构中。

### 锁
一个channel同时仅允许被一个goroutine读写，不需要另外加锁

## Go中协程与线程的区别
1. 线程一般有固定的栈大小，goroutine为了避免资源浪费，采用动态扩张收缩的策略，初始量为2k，最大可以扩张到1G 
2. goroutine只在用户态由协程调度器完成调度，调度只需要修改三个寄存器，不需要陷入内核态，开销比线程小很多。



## CGO是怎么实现的

