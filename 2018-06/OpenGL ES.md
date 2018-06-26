
OpenGL ES 删除了OpenGL API的冗余设计，对同一种操作有多种实现方法的时候，只留下主要方法，多余的技术被删除。

OpenGL ES中只有顶点矩阵存在，立即模式和显示列表被移除。

OpenGL ES被设计成OpenGL的嵌入式子集

OpenGL ES目标是确保最小的图像质量，在有限的屏幕上作图尽可能的保持好的质量。

OpenGL ES 3 完美向后兼容OpenGL ES 2 但是 3.0、2.0 与1.0完全不兼容。
因为 2.0、3.0不支持OpenGL ES 1.x支持的固定功能管道。
* 如果2.0、3.0支持固定功能管线意味着该API将以不以一种方法实现相同的功能，这违反了工作组采用的支持功能确定标准。
可编程管线使应用程序能够用着色器实现固定功能管线，所以没必要兼容。
* ISV的反馈表明，大部分游戏不混合使用可编程管线和固定功能管线。一旦有了可编程管线，就没有理由使用固定功能管线，
因为可编程管线在可渲染特效上有更多的灵活性。
* 如果OpenGL ES2.0/3.0必须同时支持固定功能管线和可编程管线，则驱动程序的内存占用会大很多，对于OpenGL ES所支持的设备，
最小化内存占用是重要的设计原则。所以没必要兼容。

EGL是Khronos（编写OpenGL ES的公司）渲染API（如OpenGL ES）和原生窗口系统之间的接口。但是在实现OpenGL ES时，没有提供EGL的硬性需求。
任何OpenGL ES应用程序都必须在开始渲染之前使用EGL执行如下任务：
* 查询并初始化设备商可用的显示器。
* 创建渲染平面
* 创建渲染上下文

```c++
#include <EGL/egl.h>
#include <GLES3/gl3.h>
// optional
// 扩展列表头文件
#include <GLES2/gl2ext.h> 
```

错误处理

glGetError查询错误代码。一旦查询到错误代码，当前错误代码被复位为GL_NO_ERROR。除了GL_OUT_OF_MEMORY错误之外，生成的错误会被忽略，且不影响OpenGL ES的状态。
如果调用glGetError返回GL_NO_ERROR，则说明从上次调用glGetError以来没有生成任何错误。

基本状态管理

初始化OpenGL ES（EGLContext）时，状态用默认值初始化。除了GL_DITHER被设置为GL_TRUE之外，其他功能的初始化值均被设置为GL_FALSE


