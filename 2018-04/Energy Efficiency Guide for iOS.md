# Energy Efficiency Guide for iOS Apps 笔记

## Energy Essentials

## Reduce and Prioritize Work

当用户没有激活你的app时，系统会将它置于后台运行状态。如果没有需要执行重要的工作比如用户发起的任务或者特别声明了某种后台运行模式，系统可能最终会暂停app。

**Important:** App不应该等待系统来暂停。一旦通知状态发生改变，应该立即开始清理活动。当App完成剩下的任务，应该通知系统后台活动已完成。否则会导致应用在后台持续激活，浪费电量。

### 暂停 & 唤醒
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
![IMAGE](images/WX20180428-142148@2x.png  =1058x250)

### Resolving Runaway Background App Crashes
iOS使用CPU监控后台应用的CPU使用率，假如超出特定限制则会终止App。 执行正常后台活动的大多数应用程序绝不会遇到这种情况。假如超出了限制，崩溃日志会表明这次终止的原因。



### Work Less in the Background


## Minimize and Defer networking

## Use Graphics Animations, and Video Efficiently

## Optimize Location and Motion
## Minimize Peripheral Interaction

## Apple Watch and Energy

## Monitor Energy Use

## Related Resources