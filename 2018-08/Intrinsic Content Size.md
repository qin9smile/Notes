Intrinsic Content Size ： 固有大小。是UIView的属性。
UILabel、UIImageView、UIButton等子组件都有.
都重写了-(CGSize)intrinsicContentSize： 方法。
并且在需要改变的时候调用 invalidateIntrinsicContentSize方法.

### Intrinsic 冲突
在两个View都使用Intrinsic Content Size 的时候，额外指定了约束导致 width/height 大于Intrinsic Content Size

解决：
* 2个View都显示指定大小，不使用Intrinsic Content Size。
* 1个View使用Intrinsic COntent Size,另一个View 使用 Content Hugging 和 Content Compression Resistance。

Content Hugging Priority: 内容压缩优先级（阻止View返回的实际尺寸比IntrinsicContentSize大）
Content Compression Resistance Priority: 内容抗压缩优先级（阻止View返回的实际尺寸比IntrinsicContentSize小）