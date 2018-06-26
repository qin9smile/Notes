Augmented Reality 增强现实：创建错觉，虚拟对象被放在物理世界中 

iOS 9+ （>=A9)

1. World Tracking: 获得设备在物理环境中的相对位置

    Visual inertial odometry, 使用Camera images和Motion Data获得设备精确位置和方向

2. Scene Understanding: 场景理解 决定设备周围的环境特性火属性的能力，提供类似平面探测的功能
    （平面探测、环境光估测算、尺度测算、iPhoneX前置摄像头的人脸识别等）
    Plane detection ：平面探测是决定物理环境中的表面或平面
    Hit-testing：通过现实世界拓扑获取一个交叉点
    Light estimation：用于渲染虚拟几何体实现正确打光，以匹配物理世界的实际环境

    平面探测：使用单目SLAM技术（平面、和边界）
    尺度测算：单目SLAM
    环境光：曝光程度
    iPhoneX人脸识别：提取脸部51个特征值。

3. Rendering：渲染

使用SceneKit或SpriteKit，提供了通用的ARViews，能够为你实现大部分的渲染。
如果用的是其他Rendering，在XCode中提供了一个Metal 模板，能够帮助你将ARKit整合到你的Renderer中。
Unity和UnReal会支持ARKit的所有功能

Processing => ARKit

Rendering => SceneKit / SpriteKit / Metal


          ARKit
-------------------------
        Capturing
AVFoundation | CoreMotion

ARKit is a session-based API.
ARSession created a AVCaptureSession and a CMMotionManager
ARSession 获取CurrentFrame（实时快照）包括关于会话的全部状态

ARSessionConfiguration （基类）是不提供任何场景理解的功能，碰撞测试也不能用

提供自由追踪的3个角度

ARWorldTrackingSessionConfiguration
提供自由追踪的6个角度，设备的方向和设备的相对位置、获取场景信息，特征点及现实世界中的物理位置

session.run(configuration) 可以反复多次调用，以切换不同的配置，而且在不同的配置之间，不会丢帧

/// Reset Tracking 会重置进行中的所有追踪 重新从（x: 0, y: 0, z: 0）
session.run(configuration, options: .resetTracking)


ARFrame 
* Captured Image ：渲染背景
* Tracking information ： 追踪设备的方向或位置
* Scene Information：空间信息，特征点及现实世界中的物理位置

空间中的物理位置，ARKit信息通过ARFrame或ARAnchor展示

ARAnchor
现实世界的位置和方向
当在探测平面的时候ARAnchor会自动加载

Tracking 探测真实物理位置
Tracking追踪的全部位置信息都是基于开始追踪的起始位置而言的
提供与物理真实世界的比例尺

每个ARFrame 包含一个ARCamera 代表虚拟相机 呈现设备的方向和位置，提供一个Transform

ARCamera

Transform 是一个矩阵 SIMD浮点4 * 4
Tracking State 提供可以如何使用Transform
Camera Intrinsics 相机内参 每一帧都有相机内参，提供诸如焦距、主点等，用3于寻找投影矩阵，渲染虚拟几何体


Tracking 还需要足够的视觉复杂度 比如说光线，移动物体

ARCamera提供追踪状态：不可用、正常、受限
not avaliable 相机的Transform 还没有被填充，单位矩阵没有准备好

Scene Understanding
3D场景拓扑 光照

通过 Plane Detection
计算3D坐标 从屏幕坐标发送射线到探测平面上。找到3D坐标

平面探测基于重力
ARKit通过多帧的聚合信息实现

ARPlaneAnchor 呈现现实世界位置和方向

HitTest
向现实世界发送激活交叉射线来寻找交叉点 使用3D特征点 交叉射线获得交叉点返回按距离排序的数组。

寻找平面，如果没有寻找到平面的话 会根据所有3D特征点的坐标 拟定一个平面并返回

Light Estimation
通过曝光信息决定相对亮度 光照良好的情况下是1000lumen（默认值）
将值付给SEN光，作为外界强度属性
默认是启动的，

SceneKit 
SpritKit

SCNView -> ARSCNView -> ARSession

使用光估计 会自动在场景中放置一个SCN光探针

Draw captured image
Updates a SCNCamera
Updates scene lighting
Maps SCNNodes to ARAnchors

SpriteKit
Contains ARSession
Draws CapturedImage
Maps SKNodes to ARAnchors
Sprites are billboarded to physical locations

但是SpriteKit是2D引擎 所以不能更新移动中的相机


Metal
1. 在后台绘制相机创建一个纹理，在后台绘制相机。
2. 使用ARCamera更新虚拟Camera 设置视图矩阵、投影矩阵
3. 更新光照信息


Important: If your app requires ARKit for its core functionality, use the arkit key in the UIRequiredDeviceCapabilities section of your app’s Info.plist file to make your app available only on devices that support ARKit. If AR is a secondary feature of your app, use the isSupported property to determine whether to offer AR-based features.

> **Important:** If your app requires ARKit for its core functionality, use the `arkit` key in the [`UIRequiredDeviceCapabilities`][7] section of your app's `Info.plist` file to make your app available only on devices that support ARKit. If AR is a secondary feature of your app, use the [`isSupported`][8] property to determine whether to offer AR-based features.

世界追踪的工作原理
为了在真实和虚拟空间之间创建对应关系，ARKit 使用一种叫做视觉惯性测距（visual-inertial odometry）的技术。这个过程将来自 iOS 设备运动传感器的信息和设备相机可见场景的计算机视觉分析相结合。ARKit 识别场景图像中的显著特征，从视频的每一帧中跟踪这些特征位置的差异，并将该信息与运动感测数据进行比较。结果是设备的位置和运动的高精度模型。

ARKit 2


