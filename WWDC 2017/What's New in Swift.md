# Swift 4 

So, we're also introducing Swift 3.2. Most important thing about Swift 3.2 is it's not a separate compiler or a different toolchain.
It's a compilation mode of the Swift 4 compiler that emulates Swift 3 behavior.
And so if some syntax or semantics change from Swift 3 to 4, it will provide the Swift 3 behavior. Moreover, it understands the changes that have been made in the new SDK. And so, if an API projects differently in Swift 4 than it did in Swift 3, it will actually roll back those changes to the Swift 3 view.
The end result here is that when you open up your Swift 3 project in Xcode 9 and build it with Swift 3.2, pretty much everything should just build and work the way it did before.
And this makes fantastic path to adopting the new features of Swift 4, because most of them are available also in Swift 3.2 as well as all of the great new APIs and frameworks in this years' SDKs. Now, when you're ready to migrate to Swift 4, and then, in previous years we've always provided a migrator to take your code from Swift 3 and move it to Swift 4. Now, unlike in previous years, this migration effort isn't stop the world, get nothing else done until the entire stack has been moved forward.
The reason is Swift 3.2 and Swift 4 can co-exist in the same application.