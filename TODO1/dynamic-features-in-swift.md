> * 原文地址：[Dynamic Features in Swift](https://www.raywenderlich.com/5743-dynamic-features-in-swift)
> * 原文作者：[Mike Finney](https://www.raywenderlich.com/u/finneycanhelp)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/dynamic-features-in-swift.md](https://github.com/xitu/gold-miner/blob/master/TODO1/dynamic-features-in-swift.md)
> * 译者：[iWeslie](https://github.com/iWeslie)
> * 校对者：

# Swift 中的动态特性

> 在本教程中，你将学习如何使用 Swift 中的动态特性编写干净的代码，编写清晰的代码并快速解决无法预料的问题。

作为一个忙碌的 Swift 开发者，你有特定于你的世界的需求，还要适用于所有人。 你希望编写整洁的代码，一目了然地了解代码中的内容并快速解决无法预料的问题。

本教程将 Swift 的动态性和灵活性结合在一起来满足那些需求。 通过使用最新的 Swift 技术，你将学习如何自定义输出到控制台，挂钩第三方对象状态更改，并使用一些甜蜜的语法糖来编写更清晰的代码。

具体来说，你将学习以下内容：

*   `Mirror`
*   `CustomDebugStringConvertible`
*   使用 keypath 进行键值监听（KVO）
*   动态查找成员
*   相关技术

最重要的是，你将度过一段美好的时光！

本教程需要 Swift 4.2 或更高版本。你必须下载最新的 [Xcode 10](https://developer.apple.com/download/) 或安装最新的 [Swift 4.2](https://swift.org/download/#snapshots)。

此外，你必须了解基本的 Swift 类型。Swift 入门教程（[原文链接](https://www.raywenderlich.com/119881/enums-structs-and-classes-in-swift)）中的[枚举](https://www.cnswift.org/enumerations)，[类和结构体](https://www.cnswift.org/classes-and-structures)是一个很好的起点。 虽然不是严格要求，但你也可以查看在 Swift 中实现[自定义下标](https://www.cnswift.org/subscripts)（[原文链接](https://www.raywenderlich.com/123102/implementing-custom-subscripts-swift)）。

## 入门

在执行任何操作之前，**请首先[下载材料](https://koenig-media.raywenderlich.com/uploads/2018/08/DynamicFeaturesInSwift.zip)**（入门项目和最终项目）。

你将很高兴地知道，所需专注学习 Swift 动态特性所需的所有代码已经为你编写好了！ 就像和友好的导盲犬一起散步一样，本教程将指导你完成入门项目代码中的所有内容。

![](https://koenig-media.raywenderlich.com/uploads/2018/06/smiling_dog_small.jpg)

​			快乐的狗狗

在名为 **DynamicFeaturesInSwift-Starter** 的入门项目代码目录中，你将看到三个 Playground 页面：**DogMirror**，**DogCatcher** 和 **KennelsKeyPath**。Playground 在macOS上运行。本教程与平台无关，仅侧重于 Swift 语言。

## 使用 Mirror 的反射机制与调试输出

无论你是断点调试追踪问题还是只探索正在运行的代码，控制台中的信息是否整洁都会产生比较大的影响。 Swift 提供了许多自定义控制台输出和捕获关键事件的方法。 对于自定义输出，它没有 Mirror 深入。 Swift 提供比最强大的雪橇犬还要强大的力量，能把你从冰冷的雪地拉出来！

![](https://koenig-media.raywenderlich.com/uploads/2018/06/siberian_husky_small.jpg)

​			西伯利亚雪橇犬

在了解有关 `Mirror` 的更多信息之前，你首先要为一个类型编写一些自定义的控制台输出。 这将有助于你更清楚地了解目前正在发生的事情。

### CustomDebugStringConvertible

用 Xcode 打开 **DynamicFeaturesInSwift.playground** 并前往 **DogMirror** 页面。

为了纪念那些迷路的可爱的小狗，它们被捕手抓住然后与它们的主人团聚，这个页面有 Dog 类和 DogCatcherNet 类。 首先我们看一下 DogCatcherNet 类。

由于丢失的小狗必须被捕获并与其主人团聚，所以我们必须支持捕狗者。你在以下项目中编写的代码将帮助捕狗者评估网络的质量。

在 Playground 里，看看以下内容：

```swift
enum CustomerReviewStars { case one, two, three, four, five }
```

```swift
class DogCatcherNet {
  let customerReviewStars: CustomerReviewStars
  let weightInPounds: Double
  // ☆ Add Optional called dog of type Dog here

  init(stars: CustomerReviewStars, weight: Double) {
    customerReviewStars = stars
    weightInPounds = weight
  }
}

```

```swift
let net = DogCatcherNet(stars: .two, weight: 2.6)
debugPrint("Printing a net: \(net)")
debugPrint("Printing a date: \(Date())")
print()

```

`DogCatcherNet` 有两个属性：`customerReviewStars` 和 `weightInPounds`。客户评论的星星数量反映了客户对净产品的感受。 以磅为单位的重量告诉狗捕捉者他们将经历拖拽网的负担。

运行 Playground。你应该看到的内容前两行与下面类似：

```
"Printing a net: __lldb_expr_13.DogCatcherNet"
"Printing a date: 2018-06-19 22:11:29 +0000"
```

正如你所见，控制台中的调试输出会打印与网络和日期相关的内容。保佑它吧！代码的输出看起来像是由机器宠物制作的。这只宠物已经尽力了，但它需要我们人类的帮助。正如您所看到的，它打印出了诸如 **_lldb_expr** 之类的额外信息。打印出的日期可以提供更有用的功能，但是这是否足以帮助你追踪一直困扰着你的问题还尚不清楚。

为了增加成功的机会，你需要用到 **CustomDebugStringConvertible** 的魔力来基础自定义制台输出。在 Playground 上，在 **DogCatcherNet **里的 **☆ Add Conformance to CustomDebugStringConvertible** 下面添加以下代码：

```swift
extension DogCatcherNet: CustomDebugStringConvertible {
  var debugDescription: String {
    return "DogCatcherNet(Review Stars: \(customerReviewStars),"
      + " Weight: \(weightInPounds))"
  }
}

```

For something small like `DogCatcherNet`, a type can conform to `CustomDebugStringConvertible` and provide its own debug description using the `debugDescription` property.

Run the playground. Except for a date value difference, the first two lines should include:

对于像 `DogCatcherNet` 这样的小东西，一个类可以遵循 `CustomDebugStringConvertible` 并使用 `debugDescription` 属性来提供自己的调试信息。

运行 Playground。除日期值会有差异外，前两行应包括：

```
"Printing a net: DogCatcherNet(Review Stars: two, Weight: 2.6)"
"Printing a date: 2018-06-19 22:10:31 +0000"
```

For a larger type with many properties, this approach comes with the cost of explicit boilerplate to type. That’s not a problem for one with dogged determination. If short on time, there are other options such as `dump`.

### Dump

How to avoid needing to add boilerplate code manually? One solution is to use `dump`. `dump` is a generic function that prints out all the names and values of a type’s properties.

The playground already contains calls that dump out the net and `Date`. The code looks like this:

```
dump(net)
print()

dump(Date())
print()
```

Run the playground. The console output looks something like:

```
▿ DogCatcherNet(Review Stars: two, Weight: 2.6) #0
  - customerReviewStars: __lldb_expr_3.CustomerReviewStars.two
  - weightInPounds: 2.6

▿ 2018-06-26 17:35:46 +0000
  - timeIntervalSinceReferenceDate: 551727346.52924
```

Due to the work you’ve done so far with `CustomDebugStringConvertible`, the `DogCatcherNet` looks better than it otherwise would. The output contains:

```
DogCatcherNet(Review Stars: two, Weight: 2.6)
```

`dump` also spits out each property automatically. Great! It’s time to make those properties more readable by using Swift’s `Mirror`.

### Swift Mirror

![](https://koenig-media.raywenderlich.com/uploads/2018/06/mirror_dog_small.jpg)

Mirror, mirror, on the wall. Who’s the fairest dog of them all?

`Mirror` lets you display values of any type instance through the playground or the debugger at runtime. In short, `Mirror`‘s power is introspection. Introspection is a subset of [reflection](https://developer.apple.com/documentation/swift/swift_standard_library/debugging_and_reflection).

### Creating a Mirror-Powered Dog Log

It’s time to create a Mirror-powered dog log. To help with debugging, it’s ideal to display the values of the net to the console through a log function with custom output complete with emoticons. The log function should be able to handle any item you pass it.

### Creating a Mirror

It’s time to create a log function that uses a mirror. To start, add the following code right under _☆ Create log function here_:

```
func log(itemToMirror: Any) {
  let mirror = Mirror(reflecting: itemToMirror)
  debugPrint("Type: 🐶 \(type(of: itemToMirror)) 🐶 ")
}
```

This creates the mirror for the passed-in item. A mirror lets you iterate over the parts of an instance.

Add the following code to the end of the `log(itemToMirror:)`:

```
for case let (label?, value) in mirror.children {
  debugPrint("⭐ \(label): \(value) ⭐")
}
```

This accesses the `children` property of the mirror, gets each label-value pair, then prints them out to the console. The label-value pair is type-aliased as `Mirror.Child`. For a `DogCatcherNet` instance, the code iterates over the properties of a net object.

To clarify, a child of the instance being inspected has nothing to do with a superclass or subclass hierarchy. The children accessible through a mirror are just the parts of the instance being inspected.

Now, it’s time to call your new log method. Add the following code right under _☆ Log out the net and a Date object here_:

```
log(itemToMirror: net)
log(itemToMirror: Date())
```

Run the playground. You’ll see at the bottom of the console output some doggone great output:

```
"Type: 🐶 DogCatcherNet 🐶 "
"⭐ customerReviewStars: two ⭐"
"⭐ weightInPounds: 2.6 ⭐"
"Type: 🐶 Date 🐶 "
"⭐ timeIntervalSinceReferenceDate: 551150080.774974 ⭐"
```

This shows all the properties’ names and values. The names appear as they do in your code. For example, `customerReviewStars` is literally how the property name is spelled in code.

### CustomReflectable

What if you wanted more of a dog and pony show in which the property names are displayed more clearly as well? What if you didn’t want some of the properties displayed? What if you wanted items displayed that are not technically part of the type? You’d use `CustomReflectable`.

`CustomReflectable` provides the hook with which you can specify what parts of a type instance are shown by using a custom `Mirror`. To conform to `CustomReflectable`, a type must define the `customMirror` property.

After speaking with several dog catcher programmers, you’ve discovered that spitting out the `weightInPounds` of the net has not helped with debugging. However, the `customerReviewStars` information is extremely helpful and they’d like the label for `customerReviewStars` to appear as “Customer Review Stars.” Now, it’s time to make `DogCatcherNet` conform to `CustomReflectable`.

Add the following code right under _☆ Add Conformance to CustomReflectable for DogCatcherNet here_:

```
extension DogCatcherNet: CustomReflectable {
  public var customMirror: Mirror {
    return Mirror(DogCatcherNet.self,
                  children: ["Customer Review Stars": customerReviewStars,
                            ],
                  displayStyle: .class, ancestorRepresentation: .generated)
  }
}
```

Run the playground and see the following output:

```
"Type: 🐶 DogCatcherNet 🐶 "
"⭐ Customer Review Stars: two ⭐"
```

_Where’s the Dog?_  
The whole point of the net is to handle having a dog. When the net is populated with a dog, there must be a way to pull out information about the dog in the net. Specifically, you need the dog’s name and age.

The playground page already has a `Dog` class. It’s time to connect `Dog` with `DogCatcherNet`. In the spot labeled as _☆ Add Optional called dog of type Dog here_, add the following property to `DogCatcherNet`:

```
var dog: Dog?
```

With the dog property added to the `DogCatcherNet`, it’s time to add the dog to the `customMirror` for the `DogCatcherNet`. Add the following dictionary entries right after the line `children: ["Customer Review Stars": customerReviewStars,`:

```
"dog": dog ?? "",
"Dog name": dog?.name ?? "No name"
```

This will output the dog using its default debug description and dog’s name, respectively labeled “dog” and “Dog name.”

Time to gently put a dog into the net. Right under _☆ Uncomment assigning the dog_, uncomment that line so the cute little dog is put into the net:

```
net.dog = Dog() // ☆ Uncomment out assigning the dog
```

Run the playground and see the following:

```
"Type: 🐶 DogCatcherNet 🐶 "
"⭐ Customer Review Stars: two ⭐"
"⭐ dog: __lldb_expr_23.Dog ⭐"
"⭐ Dog name: Abby ⭐"
```

_Mirror Convenience_  
It’s pretty nice to be able to see everything. However, there are those times when you just want to pluck out a part from a mirror. To do that, you use [`descendant(_:_:)`](https://developer.apple.com/documentation/swift/mirror/1540759-descendant). Add the following code to the end of the playground page to create a mirror and use `descendant(_:_:)` to pluck out the name and age:

```
let netMirror = Mirror(reflecting: net)

print ("The dog in the net is \(netMirror.descendant("dog", "name") ?? "nonexistent")")
print ("The age of the dog is \(netMirror.descendant("dog", "age") ?? "nonexistent")")
```

Run the playground and see at the bottom of the console output:

```
The dog in the net is Bernie
The age of the dog is 2
```

That’s doggone dynamic introspection there. It can be quite useful for debugging your own types! Having deeply explored `Mirror`, you’re done with _DogMirror.xcplaygroundpage_.

### Wrapping Up Mirror and Debug Output

There are many ways to track, like a bloodhound, what’s going on in a program. `CustomDebugStringConvertible`, `dump` and `Mirror` let you see more clearly what you are hunting for. Swift’s introspection power is highly useful — especially as you start building bigger and more complex applications!

## Key Paths

On the subject of tracking what’s going on in a program, Swift has something wonderful called key paths. For capturing an event such as when a value has changed in a third-party library object, look `to KeyPath`‘s `observe` for help.

In Swift, key paths are strongly typed paths whose types are checked at compile time. In Objective-C, they were only strings. The tutorial [What’s New in Swift 4?](https://www.raywenderlich.com/163857/whats-new-swift-4) does a great job covering the concepts in the Key-Value Coding section.

There are several different types of `KeyPath`. Commonly discussed types include [KeyPath](https://developer.apple.com/documentation/swift/keypath), [WritableKeyPath](https://developer.apple.com/documentation/swift/writablekeypath) and [ReferenceWritableKeyPath](https://developer.apple.com/documentation/swift/referencewritablekeypath). Here’s a summary of the different ones:

*   `KeyPath`: Specifies a root type on a specific value type.
*   `WritableKeyPath`: A KeyPath whose value you can also write to. It doesn’t work with classes.
*   `ReferenceWritableKeyPath`: A WritableKeyPath used for classes since classes are reference types.

A practical example of using a key path is in observing or capturing when a value changes on an object.

When you encounter a bug involving a third-party object, there is immense power in knowing when the state of that object changes. Beyond debugging, sometimes it just makes sense to hook up your custom code to respond when a value changes in a third-party object such as Apple’s UIImageView object. In [Design Patterns on iOS using Swift – Part 2/2](https://www.raywenderlich.com/160653/design-patterns-ios-using-swift-part-22), you can learn more about the observer pattern in the section titled _Key-Value Observing (KVO)_.

However, there’s a use case here related to the kennels that fits right into our doggy world. Without this KVO power, how would dog catchers easily know when the kennels are available to put more dogs inside? Although many dog catchers would just love to take home each and every lost dog they find, it’s just not practical.

So dog catchers who just want to help dogs find their way home need to know when the kennels are available to place dogs into. The first step to making this possible is creating a key path. Open the _KennelsKeyPath_ page and, right after _☆ Add KeyPath here_, add this:

```
let keyPath = \Kennels.available
```

This is how you create a `KeyPath`. You use a backslash on the type, followed by a chain of dot-separated properties — in this case, one property deep. To use the `KeyPath` to observe changes to the `available` property, add the following code after _☆ Add observe method call here_:

```
kennels.observe(keyPath) { kennels, change in
  if (kennels.available) {
    print("kennels are available")
  }
}
```

Click run and see the following message output to the console:

```
Kennels are available.
```

This approach can also be great for figuring out when a value has changed. Imagine being able to debug state changes to third-party objects! Nailing down when an item of interest changes can really keep you from barking up the wrong tree.

You’re done with the _KennelsKeyPath_ page!

## Understanding Dynamic Member Lookup

If you’ve been keeping up with Swift 4.2 changes, you may have heard about _Dynamic Member Lookup_. If not, you’ll go beyond just learning the concept here.

In this part of the tutorial, you’ll see the power of _Dynamic Member Lookup_ in Swift by going over an example of how to create a real JSON DSL (Domain Specification Language) that allows the caller to use dot notation to access values from a JSON dictionary.

_Dynamic Member Lookup_ empowers the coder to use dot syntax for properties that don’t exist at compile time as opposed to messier ways. In short, you’re coding on faith that the members will exist at runtime and getting nice-to-read code in the process.

As mentioned in the [proposal for this feature](https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md) and [associated conversations in the Swift community](https://forums.swift.org/t/se-0195-introduce-user-defined-dynamic-member-lookup-types/8658/10), this power offers great support for interoperability with other languages such as Python, database implementors and creating boilerplate-free wrappers around “stringly-typed” APIs such as CoreImage.

### Introducing @dynamicMemberLookup

Open the _DogCatcher_ page and review the code. In the playground, `Dog` represents the way a dog is running with `Direction`.

With the `dynamicMemberLookup` power, `directionOfMovement` and `moving` can be accessed even though those properties don’t explicitly exist. It’s time to make `Dog` dynamic.

### Adding dynamicMemberLookup to the Dog

The way to activate this dynamic power is through the use of the type attribute `@dynamicMemberLookup`.

Add the following code under _☆ Add subscript method that returns a Direction here_:

```
subscript(dynamicMember member: String) -> Direction {
  if member == "moving" || member == "directionOfMovement" {
    // Here's where you would call the motion detection library
    // that's in another programming language such as Python
    return randomDirection()
  }
  return .motionless
}
```

Now add `dynamicMemberLookup` to `Dog` by uncommenting the line that’s marked _☆ Uncomment this line_ above `Dog`.

You can now access a property named `directionOfMovement` or `moving`. Give it a try by adding the following on the line after _☆ Use the dynamicMemberLookup feature for dynamicDog here_:

```
let directionOfMove: Dog.Direction = dynamicDog.directionOfMovement
print("Dog's direction of movement is \(directionOfMove).")

let movingDirection: Dog.Direction = dynamicDog.moving
print("Dog is moving \(movingDirection).")
```

Run the playground. With the values sometimes being _left_ and sometimes being _right_, the first two lines you should see are similar to:

```
Dog's direction of movement is left.
Dog is moving left.
```

### Overloading subscript(dynamicMember:)

Swift supports [overloading subscript declarations](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Declarations.html#//apple_ref/doc/uid/TP40014097-CH34-ID379) with different return types. Try this out by adding a `subscript` that returns an `Int` right under _☆ Add subscript method that returns an Int here_:

```
subscript(dynamicMember member: String) -> Int {
  if member == "speed" {
    // Here's where you would call the motion detection library
    // that's in another programming language such as Python.
    return 12
  }
  return 0
}
```

Now, you can access a property named `speed`. Speed to victory by adding the following under `movingDirection` that you added earlier:

```
let speed: Int = dynamicDog.speed
print("Dog's speed is \(speed).")
```

Run the playground. The output should contain:

```
Dog's speed is 12.
```

Pretty nice, huh? That’s a powerful feature that keeps the code looking nice even if you need to access other programming languages such as Python. As hinted at earlier, there’s a catch…

![](https://koenig-media.raywenderlich.com/uploads/2018/06/dog_ears_perk_up2_small.jpg)

“A catch?” I’m all ears.

### Compiler and Code Completion Gone to the Dogs

In exchange for this dynamic runtime feature, you don’t get the benefits of compile-time checking of properties that depend on the `subscript(dynamicMember:)` functionality. Also, Xcode’s code completion feature can’t help you out either. However, the good news is that professional iOS developers read more code than they write.

The syntactic sugar that _Dynamic Member Lookup_ gives you is nothing to just throw away. It’s a nice feature that makes certain specific use cases of Swift and language interoperability bearable and enjoyable to view.

### Friendly Dog Catcher

The original proposal for _Dynamic Member Lookup_ addressed language interoperability, particularly with Python. However, that’s not the only circumstance where it’s useful.

To demonstrate a pure Swift use case, you’re going to work on the `JSONDogCatcher` code found in _DogCatcher.xcplaygroundpage_. It’s a simple struct with a few properties designed to handle `String`, `Int` and JSON dictionary. With a struct like this, you can create a `JSONDogCatcher` and ultimately go foraging for specific `String` or `Int` values.

_Traditional Subscript Method_  
A traditional way of drilling down into a JSON dictionary with a struct like this is to use a `subscript` method. The playground already contains a traditional `subscript` implementation. Accessing the `String` or `Int` values using the `subscript` method typically looks like the following and is also in the playground:

```
let json: [String: Any] = ["name": "Rover", "speed": 12,
                          "owner": ["name": "Ms. Simpson", "age": 36]]

let catcher = JSONDogCatcher.init(dictionary: json)

let messyName: String = catcher["owner"]?["name"]?.value() ?? ""
print("Owner's name extracted in a less readable way is \(messyName).")
```

Although you have to look past the brackets, quotes and question marks, this works.

Run the playground. You can now see the following:

```
Owner's name extracted in a less readable way is Ms. Simpson.
```

Although it works fine, it would be easier on the eyes to just use dot syntax. With _Dynamic Member Lookup_, you can drill down a multi-level JSON data structure.

_Adding dynamicMemberLookup to the Dog Catcher_  
Like `Dog`, it’s time to add the `dynamicMemberLookup` attribute to the `JSONDogCatcher` struct.

Add the following code right under _☆ Add subscript(dynamicMember:) method that returns a JSONDogCatcher here_:

```
subscript(dynamicMember member: String) -> JSONDogCatcher? {
  return self[member]
}
```

The `subscript(dynamicMember:)` method calls the already existing `subscript` method but takes away the boilerplate code of using brackets and `String` keys. Now, uncomment the line which has _☆ Uncomment this line_ above `JSONDogCatcher`:

```
@dynamicMemberLookup
struct JSONDogCatcher {
```

With that in place, you can use dot notation to get the dog’s speed and owner’s name. Try it out by adding the following right under _☆ Use dot notation to get the owner’s name and speed through the catcher_:

```
let ownerName: String = catcher.owner?.name?.value() ?? ""
print("Owner's name is \(ownerName).")

let dogSpeed: Int = catcher.speed?.value() ?? 0
print("Dog's speed is \(dogSpeed).")
```

Run the playground. See the speed and the owner’s name in the console:

```
Owner's name is Ms. Simpson.
Dog's speed is 12.
```

Now that you have the owner’s name, the dog catcher can contact the owner and let her know her dog has been found!

What a happy ending! The dog and its owner are together again and the code looks cleaner. Through the power of dynamic Swift, this dynamic dog can go back to chasing bunnies in the backyard.

![](https://koenig-media.raywenderlich.com/uploads/2018/06/bunny_small.jpg)

Simpson’s dog loves chasing but not catching

## Where to Go From Here?

You can download the completed version of the project using the _Download Materials_ button at the top or bottom of this tutorial.

In this tutorial, you harnessed the dynamic power that Swift offers in version 4.2. You learned about Swift’s introspective reflection powers such as `Mirror`, customizing console output, hooking into _Key-Value Observing with KeyPaths_ and _Dynamic Member Lookup_.

With that dynamic smorgasbord, you can clearly see helpful information, have more readable code and hook into some powerful runtime capabilities for your app or general purpose framework and library.

Sniffing around [Apple’s documentation about Mirror](https://developer.apple.com/documentation/swift/mirror) and related items is a worthy endeavor. For more about _Key-Value Observing_, have a look at [Design Patterns on iOS using Swift](https://www.raywenderlich.com/160651/design-patterns-ios-using-swift-part-12). Be sure to check out the [What’s New in Swift 4.2?](https://www.raywenderlich.com/194066/whats-new-in-swift-4-2) to see more of what’s in Swift 4.2.

In regards to the _Dynamic Member Lookup_ Swift 4.2 feature, it doesn’t hurt to look at the Swift proposal [SE-0195: “Introduce User-defined ‘Dynamic Member Lookup’ Types”](https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md) that introduces the `dynamicMemberLookup` type attribute and potential use cases. On a related note, a Swift proposal to keep an eye on is the close cousin of _Dynamic Member Lookup_ called [SE-216: “Introduce User-defined Dynamically ‘callable’ Types](https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md) that introduces the `dynamicCallable` type attribute.

Have more to say or ask about the dynamic power of Swift? Join in on the forum discussion below!

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
