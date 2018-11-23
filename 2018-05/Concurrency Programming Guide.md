# 并发编程指南

* 并行：多个线程同时被多个CPU同时执行，同时执行。
* 并发：多个线程被一个CPU轮流执行，顺序执行。

## 并发应用编程
在计算机早期时代，一个计算机的每秒最大执行数由CPU的计算速度决定。但是随着技术的发展和处理器的设计变得更加紧凑，热和其他物理限制开始限制处理器的最大处理数。芯片制造厂商开始寻找其他的方式来增加芯片的总体性能。解决方案是增加每个芯片的处理器内核数。通过增加内核数，一个芯片能够在不增加CPU速度或改变芯片的大小或热消耗的情况下每秒执行更多的指令。唯一的问题是如何利用这些额外的内核。

为了利用多核，一个计算机需要软件能够同时做多个事情。像OS X或iOS的现代多处任务操作系统，能够在给定时间内运行一百多个或更多的程序，所以安排每个程序到不同的芯片是可能的。然而其中大部分应用是系统守护程序或后台应用且实际处理时间非常短。相反，真正需要的是单个应用程序更有效地利用额外内核的方法。

App使用多核的传统方式是创建多个线程。然而，随着内核数的增加，线程解决方案存在一些问题。最大的问题是线程代码不能很好的扩展到任意数量的内核。你不能创建跟内核一样多的线程还希望程序良好执行。你需要知道的是这些数量的内核能够被高效的使用，这对一个应用来说自己计算内核数是一个挑战。就是那你得到了正确的内核数量，让这些线程高效执行、或防止线程之间互相影响也是挑战。

总结了这些问题，需要有一个方法来让App利用可变数量的计算机内核。单个应用程序执行的工作量也需要能够动态扩展以适应不断变化的系统条件。解决方案必须足够简单，以免增加利用这些内核所需的工作量。

### The Move Away from Threads
虽然线程已经被使用了很多年兵器而到现在还有着他们的用处，但是并没有解决以可扩展的方式来执行多任务。使用线程，直接将创建一个可扩展的解决方案的负担丢给了开发者。你需要决定创建多少线程并且随着系统条件的改变而动态改变数量。另外一个问题是你的应用程序承担了大部分的开销来处理创建和维护任何线程的使用。

取代基于线程，OS X和iOS使用异步设计方法来解决并发问题。异步方法在操作系统中存在了很多年，常用来于初始化那些可能需要花费长时间的任务，比如说从硬盘中读取数据。当被调用时，异步方法在开始任务执行之前会做一些工作并在任务实际执行完成之前就返回。典型的例子是，在后台在设置的线程中按时开始执行任务，然后当任务结束时，向用户发送一个通知（通常是通过一个回调）。在过去，如果一个异步函数不存在你想要做的事情，你将不得不编写自己的异步函数并创建自己的线程。但是现在，OS X和iOS提供的技术允许您异步执行任何任务，而无需自己管理线程。

Grand Central Dispatch(GCD)处理线程管理，可以将应用程序代码降到系统级别。你需要做的是定义你想要执行的任务并且将任务添加到适合的DispatchQueue。 GCD会创建需要的线程并安排你的任务在那些线程中执行。因为线程管理现在是系统的一部分，GCD提供全部的方法来管理、执行，提供比传统线程更好的效率。

Operation Queue是非常类似Dispatch Queue的Objective-C对象。

#### Dispatch Queues
Dispatch Queue基于C语言。无论是顺序、并发执行，总是`first-in,first-out`。顺序Dispatch Queue总是一个时间执行一个任务，直到任务结束才开始下一个任务。相对应的，并发Dispatch Queue执行尽可能多的任务，而不需要等待已经开始的任务执行结束。
* 提供简单便捷的接口
* 提供自动化的全部线程池管理
* 提供协调配合的速度（the speed of tuned assembly）
* 更多的内存效率，因为线程栈和队列不会再应用内存中残留
* 在加载之前不会占用内核
* 异步分发任务给队列不会导致死锁
* they scale gracefully under contention
* 串行调度队列为锁和其他同步原语提供了更高效的替代方案。

你提交给Dispatch Queue的任务必须是包裹在一个方法或者一个块对象里的。块对象是C语言的特性，在OS Xv10.6+和iOS4.0+之后比较类似于方法指针的概念。通常不是在自己的词法范围内定义块，而是在另一个函数或方法中定义块，以便它们可以从该函数或方法访问其他变量。块也可以移出它们原来的范围并复制到堆上，这是当提交到调度队列时会发生的事情。所有这些语义使得可以用相对较少的代码实现非常动态的任务。

Dispatch Queue 是GCD技术的一部分也是C Runtime的一部分。Block的详情见[Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)

#### Dispatch Sources
`Dispatch Sources`是基于C语言的用于异步处理特定类型的系统事件。`Dispatch Sources`是GCD技术的一部分, 一个Dispatch Source信息含有一部分的系统事件，并在事件发生时，传递一个特定的Block或方法给Dispatch Queue。你能够用Dispatch Source 来监控以下系统事件：
* Timers
* Signal Handlers
* Descriptor-related events
* Process-related events
* Mach port events
* Custom events that you trigger

#### Operation Queues
Operation Queue 相当于Cocoa并发调度队列，由OperationQueue类实现。