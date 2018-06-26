SpriteKit 与SceneKit处于同一级别，且都在Metal之上， SpriteKit用于2D场景， SceneKit用于3D场景。
Metal可以让设备直接使用硬件进行渲染。


Using SpriteKit in ARKit

ARSKView： Bridge between SpriteKit and ARKit

Delegate
* view(view:nodeFor anchor:): 为新添加的Anchor创建一个自定义的SKNode， 如果没有实现该方法，会自动创建一个默认的SKNode，
这个方法返回的Node会被ARKit移动、旋转、缩放来匹配Anchor。所以如果对Node进行transform的话，在设备移动时会被ARKit复写，
所以Anchor的任何子类都不会被修改。ARKit会自动在场景中添加Node,所以不需要开发者刻意添加。
* view(view:didAdd node: for anchor:): 在这个方法中添加子Anchor并添加transform，可以正确执行不会被ARKit复写。
* view(view:willUpdate node: for anchor:), view(view:didUpdate node: for anchor:)
* view(view:didRemove node: for anchor:)

SpriteKit and SceneKit and ARKit

Rendering SpriteKit content in SceneKit
* Set scene on SCNMaterialProperty
* The material works with SceneKit to render 
  - Uses SceneKit's Metal command queue
* SpriteKit texture mapped onto geometry

SKView then works with UIKit or AppKit on Mac OS to get your content on screen. To get your content rendering in SceneKit things are handled a little differently. Instead of setting your scene on the view, you set it on the material property of the geometry on which you want the SpriteKit content to appear.

Allows for full 3D transforms, perspective 
Can mix 2D and 3D content
Works with ARKit, or in general 3D apps

SKRenderer

underhood

SKRenderer let's you determine when SpriteKit performs updating and rendering. It allows you to work directly with Metal and then you can do things like render SpriteKit into an off screen texture to use however you want. Render SpriteKit into an off screen texture to use however you want. 

four stage to using SKRenderer: Initialization, Setting the scene, Updating, Rendering.