在使用`Optional`的时候必须要先解包（unwrap）

在`Dictionary`中，使用key返回的都是`Optional`类型的。
```swift
let imagePaths = ["star": "/glyphs/star.png",
                  "portrait": "/images/content/portrait.jpg",
                  "spacer": "/images/shared/spacer.gif"]
imagePaths["star"] // Optional<String>

guard let starPath = imagePaths["star"] else {
  print("the star image is null");
  return;
}

print("\(starPath)");
```

强制解包`!` 如果`Optional`类型value为空，则会导致运行时错误。

```swift
enum Optional<Wrapped> {
  case none
  case some(Wrapped)

  public init (_ some: Wrapped) {
    self = .some(some)
  }
}
```