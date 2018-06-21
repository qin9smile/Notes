UIImage and UIImageView Custom drawing with UIKit Advanced CPU and GPU techniques

touch briefly on integrating

focusing primarily on our use of two scarce resources on the device; memory and CPU.

tend to things as separate quantities, they have their own tracks in the debug navigator, they have their own instruments in the instruments application. But really, they're intricately, use more CPU.

Your app and other application running.. more CPU usage, has further detrimental effects on bettery life and performance

Buffer: Contiguous region of memory

buffer each element describe color and transparencty of a single pixel

Image Buffer
* in memory representation of an imagve
* each element describes color and transparencty of a single pixel
* buffer size is proportional to image size

iphone 60 HZ
iPad with ProMotion with 120Hz

if nothing change in application, the display hardware will get the same data back out of the frame buffer.

Data Buffer
store contents of an image file in memory  metadata describing dimensions of image  Image itself encoded as JPEG\PNG\or other usually compressed form 
Bytes do not directly describe pixels
sequence of byte


Decoding Concerns

CPU-intensive process
retained for repeat rendering
persistent large memory allocation
proportional to original image size, not view size



Consequences of  Excessive Memory Usage
* Increased fragmentation
* Poor locality of reference (???)
* System starts compressiong memory
* Process termination

Fragmentation

UIImageView ask UIImage to render, UIImage will hang onto that image buffer, so that it only does that work once. 
Application for every imag decodeed, could have a persistent and large memory allocation hanging out. And this allocation, is proportional to the size of the input image. 
The large allocation in your application's address space could force other related content apart form content that it wants to reference. This is called fragmentation.

if application accumulating a lot of memory usage, the operation system will transparently compressing the content of physical memory.
the CPU will involved in this operation so in addition to any CPU usage in your own application.

DownSampling


Load Image Source -> Decode -> UIImage -> Render -> Thumbnail
Load Image Source -> Thumbnail -> Decode -> UIImage -> Render  : wind up with a lower total memory usage

```swift
/// Downsampling large images for display at smaller size
func downsample(imageAt imageURL: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage {
  /// just create an object that represents the infomation stored in the file url, don't go ahead and decode immediately.
  let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
  let imageSource = CGImageSourceCreateWithURL(imageURL as CFURL, imageSourceOptions)
  
  let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
  let downsampleOptions =
    [kCGImageSourceCreateThumbnailFromImageAlways: true,
     kCGImageSourceShouldCacheImmediately: true,
     kCGImageSourceCreateThumbnailWithTransform: true,
     kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels] as CFDictionary
  let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource!, 0, downsampleOptions)!
  return UIImage(cgImage: downsampledImage)
}

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! MyCollectionViewCell
  cell.layoutIfNeeded() // Ensure imageView is its final size.

  let iamgeViewSize = cell.imageView.bounds.size
  let scale = collectionView.traitCollection.displayScale
  cell.imageView.image = downsample(imageAt: imageURLs[indexPath.item], to: imageViewSize, scale: scale)
  return cell
}

```

fluid motion as the frame buffer is updated and the display hardware is able to get the new frame on time. So much so, we don't get around to re-rendering the frame buffer.
but the display hardware is operating on a fixed interval. so from user's perspective, the application is stutted. we're done decoding these images, we're able to provide those cells back to UICollectionView. And animation continues on. Just saw a visual hitch, there.


subtle detrimental effect on bettery life

Solution:
* Prefetching
* Background decoding/downsampling


```swift
/// this will cause thread explosion
func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
  for indexPath in indexPaths {
    DispathQueue.global(qos: .userInitiated).async {
      let downsampledImage = downsample(images[indexPath.row])
      DispatchQueue.main.async {
        let downsampledImage = downsample(images[indexPath.row])
        self.update(at: indexPath, with: downsampledImage)
      }
    }
  }
}


/// update
let serialQueue = DispatchQueue(label: "Decode queue")
func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
  for indexPath in indexPaths {
    serialQueue.async {
      let downsampledImage = downsample(images[indexPath.row])
      DispatchQueue.main.async {
        let downsampledImage = downsample(images[indexPath.row])
        self.update(at: indexPath, with: downsampledImage)
      }
    }
  }
}
```

Thread explosion
* More images to decode than available CPUs
* GCD continues creating threads as new work is enqueued
* Each thread gets less time to actually decode images

GCD会捕获所有我们要求他做的工作，在同时执行中，会在各个线程之间来回切换，而这种切换是开销很大的。 2017 Modernizing GCD Usage

an individual image might not start making progress until later than before.

Image Sources
Image assets in asset catelog
Files in application/framework bundle
Files in Documents and Caches directories
Data downloaded from network


Prefer Image Assets
optimized name- and trait-based lookup
Smarter buffer caching (runtime)
Per-device thinning
Vector artwork


Image assets is optimized by name based and trait-based
has some unrelated to runtime performance that are exclusive to image assets including features like per-device thinning means
your application download image resources that are relevant to the device that it's going to run on and vector artwork

Image Assets
Vector -> Bitmap

Vector Artwork Optimizations
* Xcode rasterizes artwork for relevant scale factors while compiling
* Prerasterized artwork used when image is drawn at natural size
* If artwork has fixed sizes, use multiple image assets instead of relying on vector rasterization

Custom Drawing in Runtime

```swift
override func draw(_ rect: CGRect) {
  let roundRectPath = UIBezierPath(roundedRect: self.bounds, cornerRadius: 4.0)
  UIColor.yellow.set()
  roundRectPath.fill()

  let image = UIImage(named: "LivePhotosIcon")
  image.draw(at" CGPoint(x: 2.0, y: 2.0))

  let text = NSAttributedString(string: "LIVE", attributes: ...)
  text.draw(at: CGPoint(x: 20.0, y: 2.0))
}
```

Compare UIView Subclass and UIImageView
ImageView creates the asks the image to create the decoded image buffer. 
And then hands that decoded image over to CALayer to use as the content of its layer.

For custom View overrode draw, it's similar, but slightly different. 
The layers responsible for creating an image buffer to hold the contents of our draw method,
and then our view, excuses draw function and populates the contents of that image buffer. Coped the frame buffer as needed by the device.

Backing Store Memory Costs
Proportional to pixel size of view
Element size grows to accommodate color range used by drawing (color range -> iOS 12)
Setting CALayer.contentsFormat opts out
Update layerWillDraw(_:) implementations

Reducing Backing Store Use
Refactor large views into subview hierarchies
Reduce or eliminate overrides of draw(_:)
Eliminate duplicate copies of image data
Use optimized view properties and subclasses

don't use pattern color in backgroundColor property, because use pattern color will create backing store
use cornerRaduis to clip border if use UIView.maskView or CALayer.maskLayer will create extra allocation to store that mask

Reducing Backing Store Use - Eliminating duplicate image data
set UIImage.withRenderingMode(_:) as templete and set tintColor to any solid color

this operation will 
UIImage, as it's rendering your image to the frame buffer, will apply that solid color during that copy operation.
Rather than having to hold on to a separate copy of your image with your solid color applied to.



 memory use less 75% when displaying monochrome text than when displaying color text or emojis

artwork offscreen stored in an image buffer in memory, in UIKit is UIGraphicsImageRender ,a olderis UIGraphicsBeginContext this will render image with wide color range, so don't use it

Advanced Image Effects
 CoreImage 
 Consider Core Image for realtime effects Executes on GPU, freeing up CPU UIImage

 Use CVPixelBUffer to move data to frameworks like Metal, Vision, and Accelerate
 Use the best initializer -- don't unwind work that's already been done
 Guard against moving work between GPU and CPU
 Ensure buffers are correct format for Accelerate
