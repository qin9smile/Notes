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

## Minimize and Defer networking

## Use Graphics Animations, and Video Efficiently

## Optimize Location and Motion
## Minimize Peripheral Interaction

## Apple Watch and Energy

## Monitor Energy Use

## Related Resources
