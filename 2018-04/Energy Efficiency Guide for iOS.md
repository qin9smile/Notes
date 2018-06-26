# Energy Efficiency Guide for iOS Apps 笔记

## Energy Essentials
### Energy Efficiency and User Experience
* 电量生命周期
* 运行速度
* 响应速度
* 冷却措施

iOS 采用以下技术来决定如何使用资源和尽可能搞笑的运行程序。
* 集成硬件软件：硬件和软件的集成工作优化用户体验，延长电量。
* 智能应用管理：系统管理着app的生命周期，当用户结束了app的操作，app进入后台模式，活动受到限制，应用可能会被暂停。如果在后台长时间运行或CPU使用率过高，可能会被系统终止运行。
* 延迟网络操作：系统通过设定的延迟信息来推迟运行，
* 任务优先级：
* 开发者工具：可以在开发过程中标识定位问题

即使是很小的性能问题，也会显著影响app的电量、性能和响应速度。
作为一个开发者，有责任确保app尽可能高效的运行。使用推荐的API能让系统决定如何最好的管理App和协调资源。无论何时尽可能批量操作网络请求，阻止不必要的用户界面更新。密集型高电量操作应该在用户控制之下。

### Fundamental Concepts
* **CPU：** CPU 是电量的主要消耗者。基本app中的任何操作都用到了CPU。只在必要时通过批处理，调度和优先级进行工作。
* **Device wake：** iOS设备以来休眠来节省电量。无论什么情况，唤醒设备都是高消耗，因为屏幕和其他资源必须使用电量。特别是当app在后台操作的时候应该尽可能的闲置，除非必要情况避免使用推送通知或其他活动唤醒设备。
* **Networking Operations：** 当执行网络请求时，风潮移动网络、WiFi高消耗电量。通过批量、减少操作，压缩数据、适当处理错误来降低电量的消耗。
* **Graphics, animating and video：**每次更新界面UI的时候，都会消耗电量来更新像素，特别是动画和视频特别消耗。不必要的和不预期的也会消耗电量。App应该要禁止那些并没有向用户展示的更新。遵循Human Interface Guidelines 和 Graphics and Animation 指南可以进行优化。
* **Location：**更高的精度、更长的位置请求会增加电量消耗。App应该尽可能的减少高精度和持续激活位置信息。当不再需要位置信息的时候应该停止位置请求。
* **Motion：**加速度计（accelerometer）、陀螺仪（gyroscope）、磁力计（magnetometer）和其他运动数据浪费电量，仅在需要时使用。
* **Bluetooth：**蓝牙会高消耗iOS设备和蓝牙设备的电量。尽可能的批量和缓冲蓝牙活动，并且减少数据轮询。

#### Energy and Power
Power是时间点的功耗， Energy是时间段的功耗，存储在电池中，是有限的。

#### CPU Usage and Power Draw
![Image](%08images/Energy-CPU_Power_Draw.jpg)
#### Fixed Cost and Dynamic Cost
iOS非常擅长在不使用时将设备置于低功耗状态。即使在微秒级别，例如击键之间，系统也能够关闭未被使用的资源。

闲置时，耗电量很小，能源影响很小。当任务正在发生时，正在使用系统资源并且这些资源需要能源。然而，当设备没有做任何事情时，零星的任务会导致设备进入中间状态 - 既不处于空闲状态，也不处于活动状态。在这些中间状态期间，设备可能没有足够的时间在下一个任务执行之前达到绝对空闲状态。发生这种情况时，能量会浪费，用户的电池耗尽更快。

任务您的应用程序执行动态成本 - 您的应用程序通过执行实际工作需要多少能量。他们也有固定的成本 - 通过使系统和各种资源达到多少能量，以使您的应用程序能够工作，并在工作完成后退出。当发生大量零星工作时，动态成本和固定成本也会很高，因为资源可能永远不会有机会在零星任务之间实现真正的闲置。这种情况导致大量的能量被用于相对较少量的实际工作。

**IMPORTANT：**网络在iOS中的固定成本很高。无论何时联网，蜂窝无线电和Wi-Fi必须加电。考虑到额外的工作，即使工作完成后，这些资源仍会长时间运行并消耗能量。

![Image](%08images/Energy-Fixed-Dynamic-Cost.jpg)

#### Trading Dynamic Cost for Fixed Cost
您的应用可以通过批处理任务并减少执行频率来避免零星的工作。例如，不是在同一个线程上执行一系列连续的任务，而是在多个线程中同时分配这些相同的任务，如图2-3所示。每次访问CPU时，都必须启动内存，高速缓存，总线等。通过配料活动，组件可以启动一次并在较短的时间内使用。 这种策略带来了更大的前期动态成本 - 在给定的时间内完成了更多的工作，需要更多的动力。作为交换，您可以大幅降低固定成本，从而随着时间的推移节省大量能源。你的应用程序可以获得更多的功能，但它的效率更高，时间更短。这样可以让CPU恢复到闲置状态，并让其他组件更快地关闭电源。 在开发应用程序时，应全面考虑其行为，尽可能降低固定成本。

![Image](%08images/Energy-Using_Multithreading.jpg)
## Reduce and Prioritize Work

### Work Less in the Background
当用户没有激活你的app时，系统会将它置于后台运行状态。如果没有需要执行重要的工作比如用户发起的任务或者特别声明了某种后台运行模式，系统可能最终会暂停app。

**Important:** App不应该等待系统来暂停。一旦通知状态发生改变，应该立即开始清理活动。当App完成剩下的任务，应该通知系统后台活动已完成。否则会导致应用在后台持续激活，浪费电量。

#### 暂停 & 唤醒
在`UIApplicationDelegate`中有以下代理方法用来接收当app从激活状态到闲置状态或从前台转到后台运行状态的改变。

```swift
/*
 * 当app进入未激活状态时调用，比如当有电话、短信接入时，或者用户切换到其他app时，你的app切换到后台运行状态。实现此代理方* 法可以暂停激活，保存需要的数据，和为唤醒做准备。
 */
optional func applicationWillResignActive(_ application: UIApplication) {
    // 暂停操作、动画和UI更新
}

/*
 * 当App进入后台运行状态时立即调用。立即停止任何操作、动画、和UI更新
 */
optional func applicationDidEnterBackground(_ application: UIApplicaton) {
    // 暂停操作、动画和UI更新
    /*
     * iOS只允许此方法运行几秒钟时间。假如App需要更多的时间来完成执行用户触发的必要的操作。应该请* 求更多的后台执行时间，调用此方法可以获得额外的时间，异步或开启第二个线程来执行剩余的任务
     */
    /// 请求额外的时间
    var backgroundTaskIdentifier: UIBackgroundTaskIdentifier = 1000
    backgroundTaskIdentifier = application.beginBackgroundTaskWithExpirationHandler() {
        /// 执行完成后的completion handler
    }

    /// 初始化后台任务...

    /* 如果不执行此方法，那么等到请求的时间结束后才会调用completion handler, 之后在暂停应用 */
    application.endBackgroundTask(backgroundTaskIdentifier)
}

/*
 * 当app从后台状态将要切换到前台状态时激活App，开始恢复操作、加载数据、初始化UI。
 */
optional func applicationWillEnterForeground(_ application: UIApplication) {
    // Prepare to resume operations, animations, and UI updates
}

/*
 * 当App从后台切换到前台以后立即调用，完整恢复任何被中断的操作。
 */
optional func applicationDidBecomeActive(_ application: UIApplication) {
    // Resume operations, animations, and UI updates
}
```
**Important：** App不应该依赖状态的改变来保存数据。实现这些代理方法只是为了防止数据的丢失，以及重新唤醒App时的数据恢复。

#### Resolving Runaway Background App Crashes
iOS使用CPU监控后台应用的CPU使用率，假如超出特定限制则会终止App。 执行正常后台活动的大多数应用程序绝不会遇到这种情况。假如超出了限制，崩溃日志会表明这次终止的原因。

![Image](%08images/Energy-CrashLog.png)

### Prioritize Work with Quality of Service Classes
App和操作竞争CPU、内存、网络等资源。为了保持界面响应和效率，系统需要任务优先级来智能的决定什么时候执行。

直接影响用户使用的操作优先级非常高比如UI的更新，是非常重要并且优先级高于其他在后台发生的操作。这种高优先级的操作通常会使用更多的资源，因为可能需要大量、及时访问系统资源。

#### About Quality of Service Classes

Quality of Service（Qos）允许对由`NSOperation, NSOperationQueue, NSThread, DispatchQueues, PThreads(POSIX threads)`对象执行的操作进行分类。通过结合Qos，可以标明操作的重要性，系统会优先考虑并作出相应的安排。

因为较高优先级的工作比较低优先级的工作更快速且资源更多地执行，所以通常需要比低优先级工作更多的能量。准确地指定适当的QoS类别以确保您的应用具有响应能力和节能效果。
#### Choosing a Quality of Service Class

![Image](%08images/Energy-Primary_QoS_classes.png)

#### Special Quality of Service Classes
除了以上主要的QoS类之外还有两个不常用的特殊的QoS类

![Image](%08images/Energy-Special_QoS_classes.png)

#### Specify a QoS for Operations and Queues

默认的QoS是`Background`

##### Quality of Service Inference and Promotion
![Image](%08images/Energy-QoS_inference_and_promotion_rules.png)

![Image](%08images/Energy-inference_and_promotion_rules.png)

##### Adjust the QoS of Running Operation
* 改变Operation的`qualityOfService`属性。这样也会改变当前执行Operation所在的Thread的QoS
* 添加一个新的有着更高级别QoS的Operation到当前执行的Operation Queue中。使执行Queue的QoS匹配Operation的QoS
* 使用`addDependency:`为正在运行中的Operation添加一个更高级别的Operation作为依赖
* 使用`waitUntilFinished`或`waitUnitilAllOperationsAreFinished`。促进执行中的Operation的QoS来匹配调用方的QoS。

#### Specify a QoS for Dispatch Queues and Blocks

##### Dispatch Queues
```swift
/// 创建Queue并指定QoS
let qosQueue = DispatchQueue(label: "label", qos: .utility, attributes: .concurrent, autoreleaseFrequency: .inherit, target: nil)

/*
 * 每个QoS类都有一个全局并发队列。检索对应QoS类型的全局队列
 */
let globalQueue = DispatchQueue.global(qos: .utility)

public struct DispatchQoS : Equatable {

    ...

    public enum QoSClass {

        /// background
        @available(OSX 10.10, iOS 8.0, *)
        case background

        /// priority low
        @available(OSX 10.10, iOS 8.0, *)
        case utility

        /// priority default
        @available(OSX 10.10, iOS 8.0, *)
        case `default`

        /// priority high
        @available(OSX 10.10, iOS 8.0, *)
        case userInitiated

        /// priority high
        @available(OSX 10.10, iOS 8.0, *)
        case userInteractive

        case unspecified

        @available(OSX 10.10, iOS 8.0, *)
        public init?(rawValue: qos_class_t)

        @available(OSX 10.10, iOS 8.0, *)
        public var rawValue: qos_class_t { get }
    }

    public init(qosClass: DispatchQoS.QoSClass, relativePriority: Int)
}
```
##### Priority Inversions
高优先级的操作依赖于低优先级的操作时，会发生优先级反转`Priority Inversions`。可能会发生阻塞、旋转和轮询。

在同步情况下，系统将尝试通过在倒置期间提高较低优先级工作的QoS来自动解决优先级倒置。这将会发生在如下情况中：
* 当`dispatch_sync()`和`dispatch_wait()`在串行队列中调用时。
* 当`pthread_mutex_lock()`被调用时，互斥由具有低级别的QoS线程保存。持有锁的线程被提升为调用方。但是多个锁之间不会发生QoS提升。

#### Specify a Qos for Threads
NSThread拥有一个NSQualityOfService类型的qualityOfService属性。该类不会基于其执行上下文来推断QoS，因此只能在线程启动之前更改此属性的值。随时读取线程的qualityOfService可提供其当前值。


##### The Main Thread and the Current Thread
主线程已经自动集成了QoS。在App中，基于`user-interactive`类型。在XPC服务中，基于`default`类型。

```swift
/// 主线程
let mainQosClass = qos_class_main()

/// 当前线程
let currentQosClass = qos_class_self()
```

##### pthreads

```swift
var thread = pthread_t()
var qosAttribute = pthread_attr_t()
pthread_attr_init(&qosAttribute)
pthread_attr_set_qos_class_np(&qosAttribute, QOS_CLASS_UTILITY, 0)
pthread_create(&thread, &qosAttribute, f, nil)

/// 改变pthread的QoS
pthread_set_qos_class_self_np(QOS_CLASS_BACKGROUND, 0)
```

#### About CloudKit and Quality of Service
一些CloudKit类是默认实现自定义Qos行为。
* **CKOperation** 是`NSOperation`的子类。虽然`NSOperation`有默认的Qos级别（Background），但是`CKOperation`的默认级别是`ServiceUtility`。在这个级别，当App未被使用时，网络请求被视为可以自由选择。在低功耗模式下，操作会被暂停。
* **CKContainer** 是`NSObject`的子类。默认是`NSQualityOfServiceUserInitiated`。
* **CKDatabase** 是`NSObject`的子类。默认是`NSQualityOfServiceUserInitiated`。

### Minimize Timer Use
通过使用高效的API代替Timers可以降低能源消耗。比如说`URLSession`提供在后台执行URL会话，并在结束后发出通知。如果说必须要使用Timer，请高效使用它们。

#### The High Cost of Timers
`Timer`允许延迟执行或响应一个操作。`Timer`计时器会一直等到某个时间间隔过去，然后触发，执行特定的操作，比如说向其目标对象发送消息。从空闲状态唤醒系统会产生能源成本。如果定时器导致系统唤醒，则会增加能源成本。

大部分时候App并不需要使用`Timer`。假如在App里使用了`Timer`，考虑到是否真的需要他们。比如说，一些App应该使用`Timer`来响应时间代替轮询状态的改变。其他App应该使用信号量（semaphores）或锁来达到最高效的性能来代替使用`Timer`作为异步工具。一些`Timer`并没有在合适的延时后执行，导致在不需要的时候还在执行。

#### Get Event Notifications Without Using Timers
某些App使用`Timer`来监控文件内容的改变、网络可用性和其他状态的改变。`Timer`阻止CPU进入或保持空闲状态，这增加了能量使用并消耗电池功率。


```swift
let myFile = @"/Path/To/File"
let fileDescriptor = open(myFile.fileSystemRepresentation, O_EVTONLY)
let myQueue = dispatch_get_main_queue()
let dispatchFlags = DISPATCH_VNODE_DELETE | DISPATCH_VNODE_WRITE
let mySource = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE, fileDescriptor, dispatchFlags, myQueue)
dispatch_source_set_event_handler(mySource) {
    self.checkForFile()
}
dispatch_resume(mySource)
```

![Image](%08images/Energy-Obtaining_event_notifications_for_system_services.png)

#### Use GCD Tools for Synchronization Instead of Timers
`Grand Central Dispatch(GCD)`提供的调度队列、调度信号量和其他异步特性比`Timer`的性能更高。

```swift
/// Not Recommended
var workIsDone = false
 
/* thread one */
func doWork() {
    /* wait for network ... */
    workIsDone = true
}
 
/* thread two: completion handler */  /*** Not Recommended ***/
func waitForWorkToFinish() {
    while (!workIsDone) {
        usleep(100000) /* 100 ms */  /*** Not Recommended ***/
    }
    WorkController.workDidFinish()
}

/// Recommended
let myQueue = dispatch_queue_create("com.myapp.myq", DISPATCH_QUEUE_SERIAL)
let block = dispatch_block_create(0) {
    /* wait for network ... */
}
 
/* thread one */
func beginWork() {
    dispatch_async(myQueue, block)
}
 
/* thread two */
func waitForWorkToFinish() {
    dispatch_block_wait(block, DISPATCH_TIME_FOREVER)
    WorkController.workDidFinish()
}



/// 在一个线程上执行长时间运行的操作，以及长时间运行操作完成后另一个线程上的额外工作。例如，可以使用此技术来阻止应用程序主线程上的阻塞工作。
dispatch_async(thread2_queue) {
    /* do long work */
    dispatch_async(thread1_queue) {
        /* continue with next work */
    }
}
```

#### If You Must Use a Timer, Employ It Efficiently
游戏和其他图像密集型App经常基于`Timer`来初始化屏幕或更新动画。许多编程接口将进程延迟至指定时间段。任何传递相对的、绝对的时间期限的方法都可能是Timer API：
* 高级Timer API包括`Dispatch Timer Source`,`CFRunLoopTimerCreate`和其他`CFRunLoopTimer`方法, `Timer`类,和`performSelector:withObject:afterDelay:`方法
* 低级Timer API包括`sleep, usleep, nsnosleep, pthread_cond_timedwait, select, poll, kevent, dispatch_after,dispatch_semaphore_wait`.

如果你决定你的App依赖于`Timer`，遵从以下几点：
* Use timers economically by specifying suitable timeouts.
* Invalidate repeating timers when they’re no longer needed.
* Set tolerance for when timers should fire.

##### Specify Suitable Timeouts
```swift
/// Not Recommended
repeat {
    let timeout = dispatch_time(DISPATCH_TIME_NOW, 500 * Double(NSEC_PER_SEC))
    let semaphoreReturnValue = dispatch_semaphore_wait(mySemaphore, timeout)
    if (havePendingWork) {
        self.doPendingWork()
    }
} while true

/// Recommended
repeat {
    let timeout = DISPATCH_TIME_FOREVER
    let semaphoreReturnValue = dispatch_semaphore_wait(mySemaphore, timeout)
    if (havePendingWork) {
        self.doPendingWork()
    }
} while true
```

##### Invalidate Repeating Timers You No Longer Need
```swift
var myTimer = NSTimer.initWithFireDate(date, interval: 1.0, target: self, selector:"timerFired:", userInfo:nil repeats: true)
 
/* Do work until the timer is no longer needed */
 
myTimer.invalidate() /* Recommended */
```
##### Specify a Tolerance for Batching Timers Systemwide
为定时器启动时的准确度指定容差。系统将使用这种灵活性将定时器的执行时间缩短一段时间（在其容差范围内），以便可以同时执行多个定时器。使用这种方法会显着增加处理器闲置时间，而用户检测到系统响应速度没有变化。
为定时器设置容差后，会在触发时间和触发时间期间触发（it may fire anytime between its scheduled fire date and the scheduled fire date）。 Timer不会在触发时间之前触发。
对于重复定时器，下一个触发时间总是会跟初始时间进行计算，来保证正确的执行时间。
一般将容差设置为重复计时器时间的10%。
```swift
/// Set a tolerance for NSTimer timers
myTimer.tolerance(0.3)
RunLoop.current.addTimer(myTimer, forMode: .default)

/// Set a tolerance for Dispatch Source Timers
let myDispatchSourceTimer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, myQueue)
dispatch_source_set_timer(myDispatchSourceTimer, DISPATCH_TIME_NOW, 1 * Double(NSEC_PER_SEC), Double(NSEC_PER_SEC) / 10)
dispatch_source_set_event_handler(myDispatchSourceTimer) {
    self.timerFired()
}
dispatch_resume(myDispatchSourceTimer)

/// Set a tolerance for CFRunLoop timers
myRunLoopTimer = CFRunLoopTimerCreate(kCFAllocatorDefault, fireDate, 2.0, 0, 0, &timerFired,  NULL)
CFRunLoopTimerSetTolerance(myRunLoopTimer, 0.2)
CFRunLoopAddTimer(CFRunLoopGetCurrent(), myRunLoopTimer, kCFRunLoopDefaultMode)
```

### Minimize I/O
每次您的应用程序执行与I / O相关的任务（如写入文件数据）时，都会使系统脱离空闲状态。通过少写数据，集中写入，明智地使用缓存，调度网络事务以及最大限度地减少总体I / O，可以提高应用的能效和性能。

#### Optimize File Access
* **最小化数据写入。** 只有在内容发生变化时才写入文件，并尽可能将更改汇总为单个写入。如果只有几个字节发生了变化，请避免写出整个文件。如果您经常更改大型文件的小部分，请考虑使用数据库来存储数据。
* **避免过于频繁访问内存。** 如果您的应用程序保存了状态信息，请仅在状态信息更改时才这样做。尽可能批量更改，以避免频繁编写小的更改。
* **尽可能地按顺序读取和写入数据。** 在文件中跳转需要花费额外的时间来寻找新的位置。
* **从文件中读、写大量数据时， 要注意一次性读取太多数据可能会导致其他问题。**例如，读取32 MB文件的全部内容可能会在操作完成之前触发对这些内容的分页。
* **读写大量指定的数据，考虑使用`dispatch_io`,提供一个基于GCD的异步线程用来处理文件I/O。** 使用`dispatch_io`可以让您高度指定数据需求，因此系统可以优化您的访问。
* **如果您的数据由随机访问的结构化内容组成，请将其存储在数据库中并使用它进行访问。** 如果您操作的数据量可能增长到超过几兆字节，则使用数据库尤其重要。
* **了解系统如何缓存文件数据并知道如何优化这些缓存的使用。** [File System Programming Guide](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010672-CH1-SW1)

### React to Low Power Mode on Iphones(iOS 9+)
使用节能模式系统会执行如下操作：
* 降低CPU、GPU性能
* 暂停后台活动，包括网络操作
* 降低屏幕亮度
* 减少自动锁定设备的超时时间
* 禁用邮件提取
* 禁用 Motion效果
* 禁用动画壁纸
当电量足够时，省电模式会自动禁用。
当低功耗模式处于活动状态时，您的应用程序应采取额外措施来帮助系统节能。例如，您的应用可以减少动画的使用，降低帧速率，停止位置更新，禁用同步和备份等。

```swift
/// Register for Power State Notifications
NSNotificationCenter.defaultCenter().addObserver(
    self,
    selector: “yourMethodName:”,
    name: NSProcessInfoPowerStateDidChangeNotification,
    object: nil
)

/// Determine the Power State
/// If the device’s power state is unknown or if the device doesn’t support Low Power Mode, then this property always has a value of NO.
if NSProcessInfo.processInfo().lowPowerModeEnabled {
    // Low Power Mode is enabled. Start reducing activity to conserve energy.
} else {
    // Low Power Mode is not enabled.
}
```


## Minimize and Defer Networking

### Energy and Networking

#### Device Networking Overhead
每当您的应用执行网络操作时，都会涉及大量的间接成本。网络硬件（例如蜂窝和Wi-Fi无线电）在默认情况下会关闭以节省电量。必须启动这些资源才能执行活动。然后，他们在整个活动期间保持活动，并在预计会有更多工作的情况下继续工作一段时间。零星的网络交易导致高昂的开销并且可能会很快耗尽设备的电池。
![Image](%08images/Energy-overhead_due_to_recurring_network_activity.png)

#### Networking Variable Effect on Energy
* 蜂窝移动网络比Wifi更叫消耗能量
* 信号状况不佳或波动，可能会导致一些必须要重试的慢或有问题的会话。
* 弱网带宽意味着无线电必须保持长时间的连接才能执行会话。
* 由于信号条件和吞吐量可能会有所不同，因此即使地理位置和移动提供商的选择也会影响能耗。

应用程序使用率最终决定了网络流量的发生。应用程序使用网络的效率越高，设备的电池寿命就越长。

### Minimize Networking
#### Reduce Data Sizes
##### Reduce Media Quality and Size
假如App有上传、下载或媒体流内容，降低质量和减小大小，可减少发送和接收的数据量。
##### 压缩数据大小

#### 避免冗余传输（同样的数据不应该下载第二遍）
##### 缓存数据
缓存不常更新的数据，只在数据变化的时候更新，或者用户请求时更新
`URLCache`和`URLSession`可以用来实现为URL请求的数据缓存数据至内存或硬盘
##### 可暂停和可恢复的传输
网络状态差或信号弱时有发生。需要为恢复中断的传输做准备以免相同的数据多次下载。`NSURLSession`可以不需要缓存实现暂停和恢复的功能

#### 错误处理
不要在网络不可用湿执行网络操作

##### 判断信号情况
假如网络操作失败，使用`SCNetworkReachability`API来查看主机是否可见。假如是信号问题，提醒用户或者延迟执行。
```swift
// Checking the availability of a host


import SystemConfiguration
...
// Create a reachability object for the desired host
let hostName = "someHostName"
let reachability = SCNetworkReachabilityCreateWithName(nil, (hostName as NSString).UTF8String).takeRetainedValue()
 
// Create a place in memory for reachability flags
var flags: SCNetworkReachabilityFlags = 0
 
// Check the reachability of the host
SCNetworkReachabilityGetFlags(reachability, &flags)
 
// Check to see if the reachable flag is set
if ((flags & kSCNetworkReachabilityFlagsReachable) == 0) {
    // The target host is not reachable
    // Alert the user or defer the activity
}
```

不要永远等待永远不会到来的服务器响应。让用户取消长时间运行或停止的网络操作，并设置合适的超时时间，以便您的应用程序不会不必要地保持连接打开。
如果交易失败，请在网络可用时重试。当网络再次可用时，使用SCNetworkReachability API来确定或通知。

### Defer Networking






### Voice Over IP(VoIP) Best Pratices






## Use Graphics Animations, and Video Efficiently
### Avoid Extraneous Graphics and Animations
* 减少app使用的View的数量
* 减少使用透明度。如果您需要使用不透明度，请避免将其用于频繁更改的内容。否则，能源成本会放大，因为无论何时更改内容，都必须更新背景视图和半透明视图
* 当您的应用或其内容不可见时消除绘画，例如当您的应用内容被其他视图，剪辑或屏幕外遮挡时。
* 尽可能为动画使用较低的帧速率。例如，在游戏播放期间高帧速率可能是有意义的，但较低的帧速率可能足够用于菜单屏幕。只有在用户体验需要时才使用高帧速率。
* 执行动画时使用一致的帧频。例如，如果您的应用每秒显示60帧，请在整个动画生命周期中保持该帧速率。
* 避免在屏幕上一次使用多个帧率。例如，游戏中没有角色以每秒60帧的速度移动，而天空中的云以每秒30帧的速度移动。两者都使用相同的帧速率，即使这意味着提高其中一个帧速率。

## Optimize Location and Motion









## Optimize Notifications 
### Notification Best Practices
`Notifications`允许App不在前台运行时给用户发送通知。iOS支持本地和远程通知。
本地通知是由App安排的用语同一设备上的。
远程通知是将通知发送到Apple推送通知服务，Apple服务会将这些通知发送给用户的设备。
#### Use Local Notifications Whenever Possible
如果您的应用需要基于时间的通知而不依赖外部数据，则应使用本地通知为网络硬件留出余地。作为一个好处，即使你的应用程序没有运行，本地通知也会发生。尽管本地通知仍会唤醒闲置设备，因此您应该始终避免不必要地发送通知。
#### Prioritize Remote Notification Delivery
服务器向Apple通知服务提供的远程通知包括各种元素，包括有效负载数据，到期日期，优先级等。远程通知支持两个级别的推送优先级。一个立即发送通知。另一个延迟通知的交付，直到节能时间。除非通知确实要求立即交付，否则请使用延期交付方式。


## Minimize Peripheral Interaction
### Bluetooth Best Practices
`Core Bluetooth`提供了与支持蓝牙低功耗无线技术的设备进行通信的类别。在开发与蓝牙设备交互的应用程序时，请记住蓝牙与其他形式的无线通信（例如Wi-Fi）共享设备的无线电，以通过无线传输信号。此外，与蓝牙设备进行交互不仅仅是在iOS设备上使用能量。它也使用蓝牙设备上的能量。如果你让你的iOS应用节能，蓝牙设备也将受益。
通常，尽可能减少对无线电的使用，以减少对其他资源和设备电池的影响。这可以通过缓冲数据而不是流式传输来完成，并通过批量处理来完成。
#### Scan for Devices Only When Needed
```swift
/// Scanning for a Bluetooth device and then stopping
func beginScanningForDevice() {
    // Create a Core Bluetooth Central Manager object
    self.myCentralManager = CBCentralManager(delegate: self, queue: nil, options: nil)
    
    // Scan for peripherals
    self.myCentralManager.scanForPeripheralsWithServices(nil, options: nil)
}
 
func centralManager(central: CBCentralManager, didDiscoverPeripheral peripheral: CBPeripheral, advertisementData: [NSObject: AnyObject]!, RSSI: NSNumber!) {
    // Connect to the newly discovered device
    
    // Stop scanning for devices
    self.myCentralManager.stopScan()
}
```
#### Minimize Processing of Duplicate Device Discoveries
`Remote peripheral devices may send out multiple advertising packets per second to announce their presence to listening apps. By default, these packets are combined into a single event and delivered to your app once per peripheral. You should avoid changing this behavior—don’t specify the CBCentralManagerScanOptionAllowDuplicatesKey constant as a scan option when calling the scanForPeripheralsWithServices:options: method. Doing so results in excess events that can drain battery life.`

#### Only Discover Services and Characteristics You Need
```swift
// Look for services matching a specific set of UUIDs
peripheral.discoverServices([firstServiceUUID, secondServiceUUID])

// Look for characterstics matching a specific set of UUIDs for a given service
peripheral.discoverCharacteristics([firstCharacteristicUUID, secondCharacteristicUUID], forService: interestingService)
```

#### Request Notifications Rather than Polling for Characteristic Value Changes
```swift
/// Subscribing and responding to characteristic value change notifications
func subscribeToCharacteristic() {
    // Subscribe to a characteristic value
    self.peripheral.setNotifyValue(true, forCharacteristic: interestingCharacteristic)
}
 
func peripheral(peripheral: CBPeripheral, didUpdateNotificationStateForCharacteristic characteristic: CBCharacteristic, error: NSError! {
    // Process the characteristic value update
}
```
#### Disconnect from a Device When You No Longer Need 
```swift
// Unsubscribe from a characteristic value
self.peripheral.notifyValue(false, forCharacteristic: interestingCharacteristic)
 
// Disconnect from the device
self.myCentralManager.cancelPeripheralConnection(peripheral)
```




## Monitor Energy Use
