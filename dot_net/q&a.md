.Net(c#) Q&A
====

## 如何理解.net内存分配模型

CLR管理内存的区域，主要有三块：

- 线程的堆栈

		主要由操作系统管理，不受GC的控制。存储效率高，但容量有限。
- GC堆

		小于85K的引用对象实例分配在GC heap。
- LOH

		大于85K的引用对象实例分配在LOH上，不会进行GC压缩，只有在进行full GC时才会对空间进行释放。


值类型分配在栈上，引用类型分配在堆上。

值类型变量，包括函数参数、局部变量，都分配在stack上，引用类型的对象分配在heap中，在stack中保存对heap对象的引用指针。

GC只负责heap对象的释放和内存空间管理。

### LOH

85K字节以下的对象称为小对象，分配在Gen 0 heap，超过85K的对象称为大对象，分配在LOH（Large Object Heap）中。LOH只在2代GC时进行回收，采用`Mark-Sweep`算法，LOH中的内存分配是不连续的。比较容易产生内存碎片。

> 在.net framework 4.5中，对LOH的管理方式做了一些改进：1，显著改进了运行时对空闲列表的管理方式，能更有效的利用碎片。2，当处于Server GC模式时，运行时会在每个堆之间平衡LOH的分配。

### 什么时候触发垃圾回收

Gen0和Gen1的垃圾回收主要由阀值控制，初始时Gen0的heap大小与CPU缓存大小相关，运行时CLR根据内存请求动态调整Gen 0 heap大小，Gen0和Gen1总大小保持在16M左右；Gen2 heap和LOH都在full gc时进行回收，主要由两类事件触发：
	
- 进入Gen2 heap和LOH的对象很多，超过了一定的比例。
	
		可分别为Gen 2和LOH设定这个值。RegisterForFullGCNotification的参数可设置。

- 操作系统内存紧张时，CLR会接收OS内存紧张通知消息，触发full GC。

### GC方式

- Workstation GC with Concurrent GC off

		用于单CPU机器实现高吞吐量，采用一系列策略观察内存分配以及每次GC的状况，动态调整GC策略。但进行GC时会冻结所有线程。

- Workstation GC with Concurrent GC on

		用于响应时间非常重要的交互式程序。这种方式利用CPU对full GC进行并行处理。

- Server GC

		用于多CPU的服务器，.net为每个CPU创建一组heap和一个GC线程，每个CPU可以独立的为相应的heap执行GC操作，而其他CPU则正常执行处理。最佳应用场景是多线程之间内存结构基本相同，执行的工作相同或类似。

单CPU机器上只能使用Workstation GC，默认为with concurrent GC on方式。对于ASP.NET应用程序应当尽量保证一个CPU仅对应一个GC线程，防止同一CPU上的多个GC线程性能冲突。如果使用了Web Garden，则应当使用with concurrent GC off。

### Finalization

具有`finalize method`的对象在垃圾回收时，.net先调用`finalize method`然后再进行回收。`finalize method`被设计为非托管资源的释放，需要专门交给一个独立的线程`finalizer thread`异步进行处理。

## 动态代理、静态代理、透明代理的含义及区别

## 描述asp.net mvc页面生命周期

## webform/mvc下的身份验证一般如何实现

## LINQ里面Join和GroupJoin方法的区别




