> * 原文地址：[Intermediate Design Patterns in Swift](https://www.raywenderlich.com/2102-intermediate-design-patterns-in-swift)
> * 原文作者：[raywenderlich.com](https://www.raywenderlich.com)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/intermediate-design-patterns-in-swift.md](https://github.com/xitu/gold-miner/blob/master/TODO1/intermediate-design-patterns-in-swift.md)
> * 译者：[iWeslie](https://github.com/iWeslie)
> * 校对者：

# iOS 设计模式进阶

设计模式对于代码的维护和提高可读性非常有用，通过本教程你讲学习 Swift 中的其他设计模式。

**更新说明**：本教程已由译者针对 iOS 12，Xcode 10 和 Swift 4.2 进行了更新

**新手教程**：没了解过设计模式？来看看设计模式的 [入门教程](https://github.com/xitu/gold-miner/blob/master/TODO1/design-patterns-on-ios-using-swift-part-1-2.md) 来阅读之前的基础知识吧。

在本教程中，您将学习如何使用 Swift 中的设计模式来重构一个名为 **Tap the Larger Shape** 的游戏。

了解设计模式对于编写可维护且无错误的应用程序至关重要，了解何时采用何种设计模式是一项只能通过实践学习的技能。这本教程再好不过了！

但究竟什么是设计模式呢？这是一个针对常见问题的正式文档型解决方案。例如，考虑一下遍历一个集合，您在此处使用 **迭代器** 设计模式：

```swift
var collection = ...

// for 循环使用迭代器设计模式
for item in collection {
    print("Item is: \(item)")
}
```

**迭代器** 设计模式的价值在于它抽象出了访问集合中每一项的实际底层机制。无论 `集合` 是数组，字典还是其他类型，您的代码都可以用相同的方式访问它们中的每一项。

不仅如此，设计模式也是开发者文化的一部分，因此维护或扩展代码的另一个开发人员可能会理解迭代器设计模式，它们是用于推理出软件架构的语言。

在 iOS 编程中有很多设计模式频繁出现，例如 **MVC** 出现在几乎每个应用程序中，**代理** 是一个强大的，通常未被充分利用的模式，比如说你曾用过的 tableView，本教程讨论了一些鲜为人知但非常有用的设计模式。

如果您不熟悉设计模式的概念，此篇文章可能不适合现在的你，不妨先看一下 [使用 Swift 的 iOS 设计模式](https://github.com/xitu/gold-miner/blob/master/TODO1/design-patterns-on-ios-using-swift-part-1-2.md) 来开始吧。

## 入门

Tap the Larger Shape 是一个有趣但简单的游戏，你会看到一对相似的形状，你需要点击两者中较大的一个。如果你点击较大的形状，你会得到一分，反之你会失去一分。

你看起来好像你涂鸦出了一些随机的方块、圆圈和三角形涂鸦，不过孩子们会买单的！:\]

下载 [入门项目](https://github.com/iWeslie/SwiftDesignPatterns) 并在 Xcode 中打开。

> **注意**：您需要使用 Xcode 10 和 Swift 4.2 及以上版本从而获得最大的兼容性和稳定性。

此入门项目包含完整游戏，您将在本教程中对改项目进行重构并利用一些设计模式来使您的游戏更易于维护并且更加有趣。

使用 iPhone 8 模拟器，编译并运行项目，随意点击几个图形来了解这个游戏的规则。您会看到如下图所示的内容：

[![Tap the larger shape and gain points.](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot2-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot2.png)

点击较大的图形就能得分。

[![Tap the smaller shape and lose points.](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot3-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot3.png)

点击较小的图形则会扣分。

## 理解这款游戏

在深入了解设计模式的细节之前，先看一下目前编写的游戏。打开 **Shape.swift** 看一看并找到以下代码，您无需进行任何更改，只需要看看就行：

```swift
import UIKit

class Shape {

}

class SquareShape: Shape {
	var sideLength: CGFloat!
}
```

`Shape` 类是游戏中可点击图形的基本模型。具体的一个子类 `SquareShape` 表示一个正方形：一个具有四条等长边的多边形。

接下来打开 **ShapeView.swift** 并查看 `ShapeView` 的代码：

```swift
import UIKit

class ShapeView: UIView {
	var shape: Shape!

	// 1
	var showFill: Bool = true {
		didSet {
			setNeedsDisplay()
		}
	}
	var fillColor: UIColor = UIColor.orange {
		didSet {
			setNeedsDisplay()
		}
	}

	// 2
	var showOutline: Bool = true {
		didSet {
			setNeedsDisplay()
		}
	}
	var outlineColor: UIColor = UIColor.gray {
		didSet {
			setNeedsDisplay()
		}
	}

	// 3
	var tapHandler: ((ShapeView) -> ())?

	override init(frame: CGRect) {
		super.init(frame: frame)

		// 4
		let tapRecognizer = UITapGestureRecognizer(target: self, action: #selector(handleTap))
		addGestureRecognizer(tapRecognizer)
	}

	required init(coder aDecoder: NSCoder) {
		fatalError("init(coder:) has not been implemented")
	}

	@objc func handleTap() {
		// 5
		tapHandler?(self)
	}

	let halfLineWidth: CGFloat = 3.0
}
```

`ShapeView` 是呈现通用 `Shape` 模型的 view。以下是其中代码的逐行解析：

1. 指明应用程序是否使用，并使用哪种颜色来填充图形，这是图形内部的颜色。

2. 指明应用程序是否使用，并使用哪种颜色来给图形描边，这是图形边框的颜色。

3. 一个处理点击事件的闭包（例如更新得分）。如果您不熟悉 Swift 闭包，可以在 [Swift 闭包](https://www.cnswift.org/closures) 中查看它们，但请记住它们与 Objective-C 里的 block 类似。

4. 设置一个 tap gesture recognizer，当玩家点击 view 时调用 `handleTap`。

5. 当检测到点击手势时调用 `tapHandler`。

现在向下滚动并且查看 `SquareShapeView`：

```swift
class SquareShapeView: ShapeView {
	override func draw(_ rect: CGRect) {
		super.draw(rect)

        // 1
		if showFill {
			fillColor.setFill()
			let fillPath = UIBezierPath(rect: bounds)
			fillPath.fill()
		}

        // 2
		if showOutline {
			outlineColor.setStroke()

            // 3
			let outlinePath = UIBezierPath(rect: CGRect(x: halfLineWidth, y: halfLineWidth, width: bounds.size.width - 2 * halfLineWidth, height: bounds.size.height - 2 * halfLineWidth))
			outlinePath.lineWidth = 2.0 * halfLineWidth
			outlinePath.stroke()
		}
	}
}
```

以下是 `SquareShapeView` 如何进行绘制的：

1. 如果配置为显示填充，则使用填充颜色填充 view。

2. 如果配置为显示轮廓，则使用轮廓颜色给 view 描边。

3. 由于 iOS 是以 position 为中心绘制线条的，因此我们在描边路径时需要将从 view 的 bounds 里减去 `halfLineWidth`。

很棒！现在您已经了解了这个游戏里的图形是如绘制的，打开 **GameViewController.swift** 并查看其中的逻辑：

```swift
import UIKit

class GameViewController: UIViewController {

	override func viewDidLoad() {
		super.viewDidLoad()
        // 1
		beginNextTurn()
	}

	override var prefersStatusBarHidden: Bool {
		return true
	}

	private func beginNextTurn() {
        // 2
		let shape1 = SquareShape()
		shape1.sideLength = Utils.randomBetweenLower(lower: 0.3, andUpper: 0.8)
		let shape2 = SquareShape()
		shape2.sideLength = Utils.randomBetweenLower(lower: 0.3, andUpper: 0.8)

        // 3
		let availSize = gameView.sizeAvailableForShapes()

        // 4
		let shapeView1: ShapeView = SquareShapeView(frame: CGRect(x: 0, y: 0, width: availSize.width * shape1.sideLength, height: availSize.height * shape1.sideLength))
		shapeView1.shape = shape1
		let shapeView2: ShapeView = SquareShapeView(frame: CGRect(x: 0, y: 0, width: availSize.width * shape2.sideLength, height: availSize.height * shape2.sideLength))
		shapeView2.shape = shape2

        // 5
		let shapeViews = (shapeView1, shapeView2)

        // 6
		shapeViews.0.tapHandler = { tappedView in
			self.gameView.score += shape1.sideLength >= shape2.sideLength ? 1 : -1
			self.beginNextTurn()
		}
		shapeViews.1.tapHandler = { tappedView in
			self.gameView.score += shape2.sideLength >= shape1.sideLength ? 1 : -1
			self.beginNextTurn()
		}

        // 7
		gameView.addShapeViews(newShapeViews: shapeViews)
	}

	private var gameView: GameView { return view as! GameView }
}
```

以下是游戏逻辑的工作原理：

1. 当 `GameView` 加载后开始新的一局。

2. 在 `[0.3, 0.8]` 区间内取边长绘制正方形，绘制的图形也可以在任何屏幕尺寸下缩放。

3. 由 `GameView` 确定哪种尺寸的图形适合当前屏幕。

4. 为每个形状创建一个 `SquareShapeView`，并通过将图形的 `sideLength` 比例乘以当前屏幕的相应 `availSize` 来调整形状的大小。

5. 将形状存储在元组中以便于操作。

6. 在每个 shape view 上设置点击事件并根据玩家是否点击较大的 view 来计算分数。

7. 将形状添加到 `GameView` 以便布局显示。

以上就是游戏的完整逻辑。是不是很简单？:\]

## 为什么要使用设计模式？

你可能想问自己：“嗯，所以当我有一个工作游戏时，为什么我需要设计模式呢？”那么如果你想支持除了正方形以外的形状又要怎么办呢？

您 **本可以** 在 `beginNextTurn` 中添加代码来创建第二个形状，但是当您添加第三种、第四种甚至第五种形状时，代码将变得难以管理。

如果你希望玩家能够选择别人的形状又要怎么办呢？

如果你把所有代码放在 `GameViewController` 中，你最终会得到难以管理的包含硬编码依赖的耦合度很高的代码。

以下是您的问题的答案：设计模式有助于将您的代码解耦成分离地很开的单位。

在进行下一步之前，我坦白，我已经偷偷地进入了一个设计模式。

[![ragecomic1](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic1-e1415029446968-480x268.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic1-e1415029446968.png)

现在，关于设计模式，以下的每个部分都描述了不同的设计模式。我们开始吧！

## 抽象工厂模式

`GameViewController` 与 `SquareShapeView` 紧密耦合，这将不能为以后使用不同的视图来表示正方形或引入第二个形状留出余地。

您的第一个任务是使用 **抽象工厂** 设计模式给您的`GameViewController` 进行简化和解耦。您将要在代码中使用此模式，该代码建立用于构造一组相关对象的API，例如您将暂时使用的 shape view，而无需对特定类进行硬编码。

新建一个 Swift 文件，命名为 **ShapeViewFactory.swift** 并保存，然后添加以下代码：

```swift
import UIKit

// 1
protocol ShapeViewFactory {
    // 2
    var size: CGSize { get set }
    // 3
    func makeShapeViewsForShapes(shapes: (Shape, Shape)) -> (ShapeView, ShapeView)
}
```

以下是您新的工厂的工作原理：

1. 将 `ShapeViewFactory` 定义为 Swift 协议，它没有理由成为一个类或结构体，因为它只描述了一个接口而本身并没有功能。

2. 每个工厂应当有一个定义了创建形状的边界的尺寸，这对使用工厂生成的 view 布局代码至关重要。

3. 定义生成形状视图的方法。这是工厂的“肉”，它需要两个 Shape 对象的元组，并返回两个 ShapeView 对象的元组。这基本上是从其原材料 -- 模型中制造 view。

在 **ShapeViewFactory.swift** 的最后添加以下代码：

```swift
class SquareShapeViewFactory: ShapeViewFactory {
    var size: CGSize

    // 1
    init(size: CGSize) {
        self.size = size
    }

    func makeShapeViewsForShapes(shapes: (Shape, Shape)) -> (ShapeView, ShapeView) {
        // 2
        let squareShape1 = shapes.0 as! SquareShape
        let shapeView1 = SquareShapeView(frame: CGRect(
            x: 0,
            y: 0,
            width: squareShape1.sideLength * size.width,
            height: squareShape1.sideLength * size.height))
        shapeView1.shape = squareShape1

        // 3
        let squareShape2 = shapes.1 as! SquareShape
        let shapeView2 = SquareShapeView(frame: CGRect(
            x: 0,
            y: 0,
            width: squareShape2.sideLength * size.width,
            height: squareShape2.sideLength * size.height))
        shapeView2.shape = squareShape2

        // 4
        return (shapeView1, shapeView2)
    }
}
```

您的 `SquareShapeViewFactory` 建造了 `SquareShapeView` 实例，如下所示：

1. 使用一致的最大尺寸来初始化工厂。

2. 从第一个传递的形状构造第一个 shape view。

3. 从第二个传递的形状构造第二个 shape view。

4. 返回包含两个刚创建的 shape view 的元组。

最后，是时候使用 `SquareShapeViewFactory` 了。打开 **GameViewController.swift**，并全部替换为以下内容：

```swift
import UIKit

class GameViewController: UIViewController {

	override func viewDidLoad() {
		super.viewDidLoad()
        // 1 ***** 附加
        shapeViewFactory = SquareShapeViewFactory(size: gameView.sizeAvailableForShapes())

		beginNextTurn()
	}

	override var prefersStatusBarHidden: Bool {
		return true
	}

	private func beginNextTurn() {
		let shape1 = SquareShape()
		shape1.sideLength = Utils.randomBetweenLower(lower: 0.3, andUpper: 0.8)
		let shape2 = SquareShape()
		shape2.sideLength = Utils.randomBetweenLower(lower: 0.3, andUpper: 0.8)

        // 2 ***** 附加
        let shapeViews = shapeViewFactory.makeShapeViewsForShapes(shapes: (shape1, shape2))

        shapeViews.0.tapHandler = {
            tappedView in
            self.gameView.score += shape1.sideLength >= shape2.sideLength ? 1 : -1
            self.beginNextTurn()
        }
        shapeViews.1.tapHandler = {
            tappedView in
            self.gameView.score += shape2.sideLength >= shape1.sideLength ? 1 : -1
            self.beginNextTurn()
        }

        gameView.addShapeViews(newShapeViews: shapeViews)
	}

	private var gameView: GameView { return view as! GameView }

    // 3 ***** 附加
    private var shapeViewFactory: ShapeViewFactory!
}
```

这里有三行新代码：

1. 初始化并存储一个 `SquareShapeViewFactory`。

2. 使用此新工厂创建你的 shape view。

3. 将新的 shape view 工厂存储为实例属性。

主要的好处在于第二部分，其中您用一行替换了六行代码。更好的是，您将复杂的 shape view 的创建代码移出了 `GameViewController` 从而使类更小也更容易理解。

将 view 创建代码移出 controller 是很有帮助的，因为 `GameViewController` 充当 Controller 在 Model 和 View 之间进行协调。

编译并运行，然后您应该看到类似以下内容：

[![Screenshot4](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot4-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot4.png)

您游戏的视觉效果没有任何改变，但您确实简化了代码。

如果你用 `SomeOtherShapeView` 替换 `SquareShapeView`，那么 `SquareShapeViewFactory` 的好处就会大放异彩。具体来说，您不需要更改 `GameViewController`，您可以将所有更改分离到 `SquareShapeViewFactory`。

既然您已经简化了 shape view 的创建，那么您也同时可以简化 shape 的创建。像之前那样创建一个新的 Swift 文件，命名为 **ShapeFactory.swift**，并把以下代码粘贴进去：

```swift
import UIKit

// 1
protocol ShapeFactory {
    func createShapes() -> (Shape, Shape)
}

class SquareShapeFactory: ShapeFactory {
    // 2
    var minProportion: CGFloat
    var maxProportion: CGFloat

    init(minProportion: CGFloat, maxProportion: CGFloat) {
        self.minProportion = minProportion
        self.maxProportion = maxProportion
    }

    func createShapes() -> (Shape, Shape) {
        // 3
        let shape1 = SquareShape()
        shape1.sideLength = Utils.randomBetweenLower(lower: minProportion, andUpper: maxProportion)

        // 4
        let shape2 = SquareShape()
        shape2.sideLength = Utils.randomBetweenLower(lower: minProportion, andUpper: maxProportion)

        // 5
        return (shape1, shape2)
    }
}
```

你的新 `ShapeFactory` 生产 shape 的具体步骤如下：

1. 再一次地，就像你对 `ShapeViewFactory` 所做的那样，将 `ShapeFactory` 声明为一个协议来获得最大的灵活性。

2. 您希望您的 shape 工厂生成具有单位尺寸的形状，例如，在 `[0, 1]` 的范围内，因此您要存储这个范围。

3. 创建具有随机尺寸的第一个方形。

4. 创建具有随机尺寸的第二个方形。

5. 将这对方形形状作为元组返回。

现在打开 **GameViewController.swift** 并在底部大括号结束之前的插入以下代码：

```swift
private var shapeFactory: ShapeFactory!
```

然后在 `viewDidLoad` 的底部 `beginNextTurn` 的调用之上插入以下代码：

```swift
shapeFactory = SquareShapeFactory(minProportion: 0.3, maxProportion: 0.8)
```

最后把 `beginNextTurn` 替换为以下代码:

```swift
private func beginNextTurn() {
    // 1
    let shapes = shapeFactory.createShapes()

    let shapeViews = shapeViewFactory.makeShapeViewsForShapes(shapes: shapes)

    shapeViews.0.tapHandler = { tappedView in
        // 2
        let square1 = shapes.0 as! SquareShape, square2 = shapes.1 as! SquareShape
        // 3
        self.gameView.score += square1.sideLength >= square2.sideLength ? 1 : -1
        self.beginNextTurn()
    }
    shapeViews.1.tapHandler = { tappedView in
        let square1 = shapes.0 as! SquareShape, square2 = shapes.1 as! SquareShape
        self.gameView.score += square2.sideLength >= square1.sideLength ? 1 : -1
        self.beginNextTurn()
    }

    gameView.addShapeViews(newShapeViews: shapeViews)
}
```

以下是上面代码的解析：

1. 使用新的 shape 工厂创建一个形状元组。

2. 从元组中提取形状。

3. 这样你就可以在这里比较它们了。

Once again, using the **Abstract Factory** design pattern simplified your code by moving shape generation out of `GameViewController`.再一次使用 **抽象工厂** 设计模式，通过将创建形状的部分移出 `GameViewController` 来简化代码。

## 雇工模式

现在你甚至可以添加第二个形状，例如圆圈。您对正方形的唯一硬性依赖是下面 `beginNextTurn` 中的得分计算：

```swift
shapeViews.1.tapHandler = { tappedView in
    // 1
    let square1 = shapes.0 as! SquareShape, square2 = shapes.1 as! SquareShape

    // 2
    self.gameView.score += square2.sideLength >= square1.sideLength ? 1 : -1
    self.beginNextTurn()
}
```

在这里您把形状转换为 `SquareShape` 以便您可以访问它们的 `sideLength`，圆没有 `sideLength`，而是“直径”。

解决方案是使用 **雇工** 设计模式，它通过一个通用接口为一组类（如形状类）提供分数计算等方法。在您现在的情况下，分数计算是雇工，形状类作为服务对象，并且 `area` 属性扮演公共接口的角色。

打开 **Shape.swift** 并在 `Shape` 类的底部添加以下代码：

```swift
var area: CGFloat { return 0 }
```

然后在 `SquareShape` 类的底部添加以下代码:

```swift
override var area: CGFloat { return sideLength * sideLength }
```

现在您可以根据其面积来判断哪个形状更大。

打开 **GameViewController.swift** 并把 `beginNextTurn` 替换成以下内容：

```swift
private func beginNextTurn() {
    let shapes = shapeFactory.createShapes()

    let shapeViews = shapeViewFactory.makeShapeViewsForShapes(shapes: shapes)

    shapeViews.0.tapHandler = {
        tappedView in
        // 1
        self.gameView.score += shapes.0.area >= shapes.1.area ? 1 : -1
        self.beginNextTurn()
    }
    shapeViews.1.tapHandler = {
        tappedView in
        // 2
        self.gameView.score += shapes.1.area >= shapes.0.area ? 1 : -1
        self.beginNextTurn()
    }

    gameView.addShapeViews(newShapeViews: shapeViews)
}
```

1. 根据形状区域确定较大的形状。

2. 还是根据形状区域确定较大的形状。

编译并运行，您应该看到类似下面的内容，虽然游戏看起来相同，但代码现在更灵活了。

[![Screenshot6](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot6-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot6.png)

恭喜，您已经从游戏逻辑中完全解除了对正方形的依赖关系，如果您要创建和使用一些圆形的工厂，您的游戏将变得更加完善。

[![ragecomic2](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic2.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic2.png)

## 利用抽象工厂实现游戏的多功能性

“不要做一个正块！”在现实生活中可能是一种侮辱，你的游戏感觉它被装在一个形状中，它渴望更流畅的线条和更多的符合空气动力学的形状。

你需要引入一些流畅的“善良的圆”，现在打开 **Shape.swift** 并在文件底部添加以下代码：

```swift
class CircleShape: Shape {
    var diameter: CGFloat!
    override var area: CGFloat { return CGFloat.pi * diameter * diameter / 4.0 }
}
```

你的圆只需要知道它可以计算自身面积的“直径”就可以支持 **雇工** 模式。

接下来通过添加 `CircleShapeFactory` 来构建 `CircleShape` 对象。打开 **ShapeFactory.swift** 并在文件底部添加以下代码：

```swift
class CircleShapeFactory: ShapeFactory {
	var minProportion: CGFloat
	var maxProportion: CGFloat

	init(minProportion: CGFloat, maxProportion: CGFloat) {
		self.minProportion = minProportion
		self.maxProportion = maxProportion
	}

	func createShapes() -> (Shape, Shape) {
		// 1
		let shape1 = CircleShape()
		shape1.diameter = Utils.randomBetweenLower(lower: minProportion, andUpper: maxProportion)

		// 2
		let shape2 = CircleShape()
		shape2.diameter = Utils.randomBetweenLower(lower: minProportion, andUpper: maxProportion)

		return (shape1, shape2)
	}
}
```

这段代码遵循一个熟悉的模式：**第1部分** 和 **第2部分** 创建了一个 `CircleShape` 并为其指定一个随机的 `diameter`。

你需要解决另一个问题，这样做可能会防止一个混乱的几何图形的革命。看吧，你现在拥有的是 “没有代表性的几何图形”，你知道当形状不足时，形状会变得多么干净哈！

取悦你的玩家很容易，你需要的只是用 `CircleShapeView` 在屏幕上 **代表** 你的新 `CircleShape` 对象。:\]

打开 `ShapeView.swift` 并在文件底部添加以下内容：

```swift
class CircleShapeView: ShapeView {
	override init(frame: CGRect) {
		super.init(frame: frame)
		// 1
		self.isOpaque = false
		// 2
		self.contentMode = UIView.ContentMode.redraw
	}

	required init(coder aDecoder: NSCoder) {
		fatalError("init(coder:) has not been implemented")
	}

	override func draw(_ rect: CGRect) {
		super.draw(rect)

		if showFill {
			fillColor.setFill()
			// 3
			let fillPath = UIBezierPath(ovalIn: self.bounds)
			fillPath.fill()
		}

		if showOutline {
			outlineColor.setStroke()
			// 4
			let outlinePath = UIBezierPath(ovalIn: CGRect(
				x: halfLineWidth,
				y: halfLineWidth,
				width: self.bounds.size.width - 2 * halfLineWidth,
				height: self.bounds.size.height - 2 * halfLineWidth))
			outlinePath.lineWidth = 2.0 * halfLineWidth
			outlinePath.stroke()
		}
	}
}
```

对上述内容的解释依次为以下几个部分：

1. 由于圆无法填充其 view 的 bounds，因此您需要告诉 **UIKit** 该 view 是透明的，这意味着能透过它看到背后的东西。如果你没有意识到这点，那么这个圆将会有一个丑陋的黑色背景。

2. 由于视图是透明的，因此应在 bounds 更改时进行重绘。

3. 画一个用 `fillColor` 填充的圆圈。稍后，您将创建 `CircleShapeViewFactory`，它会确保 `CircleView` 具有相等的宽度和高度，因此画出来的形状将是圆形而不是椭圆形。

4. 给圆用 lineWidth 进行描边。

现在您将在 `CircleShapeViewFactory` 中创建` CircleShapeView` 对象。

打开 **ShapeViewFactory.swift** 并在文件的底部添加以下代码：

```swift
class CircleShapeViewFactory: ShapeViewFactory {
	var size: CGSize

	init(size: CGSize) {
		self.size = size
	}

	func makeShapeViewsForShapes(shapes: (Shape, Shape)) -> (ShapeView, ShapeView) {
		let circleShape1 = shapes.0 as! CircleShape
		// 1
		let shapeView1 = CircleShapeView(frame: CGRect(
			x: 0,
			y: 0,
			width: circleShape1.diameter * size.width,
			height: circleShape1.diameter * size.height))
		shapeView1.shape = circleShape1

		let circleShape2 = shapes.1 as! CircleShape
		// 2
		let shapeView2 = CircleShapeView(frame: CGRect(
			x: 0,
			y: 0,
			width: circleShape2.diameter * size.width,
			height: circleShape2.diameter * size.height))
		shapeView2.shape = circleShape2

		return (shapeView1, shapeView2)
	}
}
```

这是将创建圆而不是正方形的工厂。**第1部分** 和 **第2部分** 使用传入的形状创建 `CircleShapeView` 实例。请注意你的代码是如何确保圆圈具有相同的宽度和高度，因此它们呈现为完美的圆形而不是椭圆形。

最后，打开 **GameViewController.swift** 并替换 `viewDidLoad` 中对应的两行，用以下内容分配形状和视图工厂：

```swift
shapeViewFactory = CircleShapeViewFactory(size: gameView.sizeAvailableForShapes())
shapeFactory = CircleShapeFactory(minProportion: 0.3, maxProportion: 0.8)
```

现在编译并运行项目，你应该看到类似下面的截图。

[![Screenshot7](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot7-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot7.png)
瞧，你造出了圆形！

请注意你是如何在 `GameViewController` 中添加新形状而不会对游戏逻辑产生太大影响的，抽象工厂和雇工设计模式使之成为可能。

## 建造者模式模式

现在是时候来看看第三种设计模式了：**建造者**。

假设您想要改变 `ShapeView` 实例的外观 - 例如它们是否应显示，以及用什么颜色来填充和描边。 **建造者** 设计模式使这种对象的配置变得更加容易和灵活。

解决此配置问题的一种方法是添加各种构造函数，可以使用诸如 `CircleShapeView.redFilledCircleWithBlueOutline()` 之类的类便利初始化方法，也可以添加具有各种参数和默认值的初始化方法。

然而不幸的是，它不是一种可扩展的技术，因为您需要为每种组合编写新方法或初始化程序。

建造者非常优雅地解决了这个问题，因为它创建了一个具有单一用途的类来配置已经初始化的对象。如果您将让建造者来构建红色的圆，然后再构建蓝色的圆，则无需更改 `CircleShapeView` 就可达到目的。

创建一个新文件 **ShapeViewBuilder.swift** 并添加以下代码：

```swift
import UIKit

class ShapeViewBuilder {
	// 1
	var showFill  = true
	var fillColor = UIColor.orange

	// 2
	var showOutline  = true
	var outlineColor = UIColor.gray

	// 3
	init(shapeViewFactory: ShapeViewFactory) {
		self.shapeViewFactory = shapeViewFactory
	}

	// 4
	func buildShapeViewsForShapes(shapes: (Shape, Shape)) -> (ShapeView, ShapeView) {
		let shapeViews = shapeViewFactory.makeShapeViewsForShapes(shapes: shapes)
		configureShapeView(shapeView: shapeViews.0)
		configureShapeView(shapeView: shapeViews.1)
		return shapeViews
	}

	// 5
	private func configureShapeView(shapeView: ShapeView) {
		shapeView.showFill  = showFill
		shapeView.fillColor = fillColor
		shapeView.showOutline  = showOutline
		shapeView.outlineColor = outlineColor
	}

	private var shapeViewFactory: ShapeViewFactory
}
```

以下是您的新的 `ShapeViewBuilder` 的工作原理：

1. 存储配置 `ShapeView` 的填充属性。

2. 存储配置 `ShapeView` 的描边属性。

3. 初始化建造者来保存 `ShapeViewFactory` 从而构造 view。这意味着建造者并不需要知道它是来建造 `SquareShapeView` 还是 `CircleShapeView` 抑或是其他形状的 view。

4. 这是公共 API，当有一对 `Shape` 时，它会创建并初始化一对 `ShapeView`。

5. 根据建造者的存储了的配置来对 `ShapeView` 进行配置。

现在来部署你新的 `ShapeViewBuilder`，打开 **GameViewController.swift**，在大括号结束之前将以下代码添加到类的底部：

```swift
private var shapeViewBuilder: ShapeViewBuilder!
```

现在在 `viewDidLoad` 里 `beginNextTurn` 调用的上方添加以下代码来填充新属性：

```swift
shapeViewBuilder = ShapeViewBuilder(shapeViewFactory: shapeViewFactory)
shapeViewBuilder.fillColor = UIColor.brown
shapeViewBuilder.outlineColor = UIColor.orange
```

最后用以下代码替换 `beginNextTurn` 中创建 `shapeViews` 的那一行：

```swift
let shapeViews = shapeViewBuilder.buildShapeViewsForShapes(shapes: shapes)
```

编译并运行，你将看到以下内容：

[![Screenshot8](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot8-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot8.png)

说实话我也觉得填充颜色很丑，但是先别吐槽，毕竟我们目前关注点不在于它是有多么好看。

现在来强化建造者的力量。还是在 `GameViewController.swift` 里，将 `viewDidLoad` 对应的两行更改为使用方形工厂：

```swift
shapeViewFactory = SquareShapeViewFactory(size: gameView.sizeAvailableForShapes())
shapeFactory = SquareShapeFactory(minProportion: 0.3, maxProportion: 0.8)
```

编译并运行，你将看到以下内容：

[![Screenshot9](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot9-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot9.png)

注意建造者模式是如何使新的颜色方案来应用到正方形和圆形上的。没有它的话你需要在 `CircleShapeViewFactory` 和 `SquareShapeViewFactory` 中来单独设置颜色。

此外，更改为另一种配色方案将涉及大量代码的修改。通过将 `ShapeView` 颜色配置限制为单个 `ShapeViewBuilder`，您还可以将颜色更改隔离到单个类。

## Design Pattern: Dependency Injection

Every time you tap a shape, you’re taking a turn in your game, and each turn can be a match or not a match.

Wouldn’t it be helpful if your game could track all the turns, stats and award point bonuses for hot streaks?

Create a new file called **Turn.swift**, and replace its contents with the following code:

```swift

class Turn {
  // 1
  let shapes: [Shape]
  var matched: Bool?

  init(shapes: [Shape]) {
    self.shapes = shapes
  }

  // 2
  func turnCompletedWithTappedShape(tappedShape: Shape) {
    var maxArea = shapes.reduce(0) { $0 > $1.area ? $0 : $1.area }
    matched = tappedShape.area >= maxArea
  }
}
```

Your new `Turn` class does the following:

1.  Store the shapes that the player saw during the turn, and also whether the turn was a match or not.

2.  Records the completion of a turn after a player taps a shape.

To control the sequence of turns your players play, create a new file named **TurnController.swift**, and replace its contents with the following code:

```swift

class TurnController {
  // 1
  var currentTurn: Turn?
  var pastTurns: [Turn] = [Turn]()

  // 2
  init(shapeFactory: ShapeFactory, shapeViewBuilder: ShapeViewBuilder) {
    self.shapeFactory = shapeFactory
    self.shapeViewBuilder = shapeViewBuilder
  }

  // 3
  func beginNewTurn() -> (ShapeView, ShapeView) {
    let shapes = shapeFactory.createShapes()
    let shapeViews = shapeViewBuilder.buildShapeViewsForShapes(shapes)
    currentTurn = Turn(shapes: [shapeViews.0.shape, shapeViews.1.shape])
    return shapeViews
  }

  // 4
  func endTurnWithTappedShape(tappedShape: Shape) -> Int {
    currentTurn!.turnCompletedWithTappedShape(tappedShape)
    pastTurns.append(currentTurn!)

    var scoreIncrement = currentTurn!.matched! ? 1 : -1

    return scoreIncrement
  }

  private let shapeFactory: ShapeFactory
  private var shapeViewBuilder: ShapeViewBuilder
}
```

Your `TurnController` works as follows:

1.  Stores both the current turn and past turns.

2.  Accepts a `ShapeFactory` and `ShapeViewBuilder`.

3.  Uses this factory and builder to create shapes and views for each new turn and records the current turn.

4.  Records the end of a turn after the player taps a shape, and returns the computed score based on whether the turn was a match or not.

Now open **GameViewController.swift**, and add the following code at the bottom, just above the closing curly brace:

```swift
private var turnController: TurnController!
```

Scroll up to `viewDidLoad`, and just before the line invoking `beginNewTurn`, insert the following code:

```swift
turnController = TurnController(shapeFactory: shapeFactory, shapeViewBuilder: shapeViewBuilder)
```

Replace `beginNextTurn` with the following:

```swift
private func beginNextTurn() {
  // 1
  let shapeViews = turnController.beginNewTurn()

  shapeViews.0.tapHandler = {
    tappedView in
    // 2
    self.gameView.score += self.turnController.endTurnWithTappedShape(tappedView.shape)
    self.beginNextTurn()
  }

  // 3
  shapeViews.1.tapHandler = shapeViews.0.tapHandler

  gameView.addShapeViews(shapeViews)
}
```

Your new code works as follows:

1.  Asks the `TurnController` to begin a new turn and return a tuple of `ShapeView` to use for the turn.

2.  Informs the turn controller that the turn is over when the player taps a `ShapeView`, and then it increments the score. Notice how `TurnController` abstracts score calculation away, further simplifying `GameViewController`.

3.  Since you removed explicit references to specific shapes, the second shape view can share the same `tapHandler` closure as the first shape view.

An example of the **Dependency Injection** design pattern is that it passes in its dependencies to the `TurnController` initializer. The initializer parameters essentially inject the shape and shape view factory dependencies.

Since `TurnController` makes no assumptions about which type of factories to use, you’re free to swap in different factories.

Not only does this make your game more flexible, but it makes automated testing easier since it allows you to pass in special `TestShapeFactory` and `TestShapeViewFactory` classes if you desire. These could be special stubs or mocks that would make testing easier, more reliable or faster.

Build and run and check that it looks like this:

[![Screenshot10](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot10-180x320.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot10.png)

There are no visual differences, but `TurnController` has opened up your code so it can use more sophisticated turn strategies: calculating scores based on streaks of turns, alternating shape type between turns, or even adjusting the difficulty of play based on the player’s performance.

## Design Pattern: Strategy

I’m happy because I’m eating a piece of pie while writing this tutorial. Perhaps that’s why it was imperative to add circles to the game. :\]

You should be happy because you’ve done a great job using design patterns to refactor your game code so that it’s easy to expand and maintain.

[![ragecomic3](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic3.png)](https://koenig-media.raywenderlich.com/uploads/2014/10/ragecomic3.png)

Speaking of pie, err, Pi, how do you get those circles back in your game? Right now your `GameViewController` can use **either** circles or squares, but only one or the other. It doesn’t have to be all restrictive like that.

Next, you’ll use the **Strategy** design pattern to manage which shapes your game produces.

The **Strategy** design pattern allows you to design algorithm behaviors based on what your program determines at runtime. In this case, the algorithm will choose which shapes to present to the player.

You can design many different algorithms: one that picks shapes randomly, one that picks shapes to challenge the player or help him be more successful, and so on. **Strategy** works by defining a family of algorithms through abstract declarations of the behavior that each strategy must implement. This makes the algorithms within the family interchangeable.

If you guessed that you’re going to implement the Strategy as a Swift `protocol`, you guessed correctly!

Create a new file named **TurnStrategy.swift**, and replace its contents with the following code:

```swift

// 1
protocol TurnStrategy {
  func makeShapeViewsForNextTurnGivenPastTurns(pastTurns: [Turn]) -> (ShapeView, ShapeView)
}

// 2
class BasicTurnStrategy: TurnStrategy {
  let shapeFactory: ShapeFactory
  let shapeViewBuilder: ShapeViewBuilder

  init(shapeFactory: ShapeFactory, shapeViewBuilder: ShapeViewBuilder) {
    self.shapeFactory = shapeFactory
    self.shapeViewBuilder = shapeViewBuilder
  }

  func makeShapeViewsForNextTurnGivenPastTurns(pastTurns: [Turn]) -> (ShapeView, ShapeView) {
    return shapeViewBuilder.buildShapeViewsForShapes(shapeFactory.createShapes())
  }
}

class RandomTurnStrategy: TurnStrategy {
  // 3
  let firstStrategy: TurnStrategy
  let secondStrategy: TurnStrategy

  init(firstStrategy: TurnStrategy, secondStrategy: TurnStrategy) {
    self.firstStrategy = firstStrategy
    self.secondStrategy = secondStrategy
  }

  // 4
  func makeShapeViewsForNextTurnGivenPastTurns(pastTurns: [Turn]) -> (ShapeView, ShapeView) {
    if Utils.randomBetweenLower(0.0, andUpper: 100.0) < 50.0 {
      return firstStrategy.makeShapeViewsForNextTurnGivenPastTurns(pastTurns)
    } else {
      return secondStrategy.makeShapeViewsForNextTurnGivenPastTurns(pastTurns)
    }
  }
}
```

Here's what your new `TurnStrategy` does line-by-line:

1.  Declare the behavior of the algorithm. This is defined in a protocol, with one method. The method takes an array of the past turns in the game, and returns the shape views to display for the next turn.

2.  Implement a basic strategy that uses a `ShapeFactory` and `ShapeViewBuilder`. This strategy implements the existing behavior, where the shape views just come from the single factory and builder as before. Notice how you're using **Dependency Injection** again here, and that means this strategy doesn't care which factory or builder it's using.

3.  Implement a random strategy which randomly uses one of two other strategies. You've used composition here so that `RandomTurnStrategy` can behave like two potentially different strategies. However, since it's a `Strategy`, that composition is hidden from whatever code uses `RandomTurnStrategy`.

4.  This is the meat of the random strategy. It randomly selects either the first or second strategy with a 50 percent chance.

Now you need to use your strategies. Open **TurnController.swift**, and replace its contents with the following:

```swift

class TurnController {
  var currentTurn: Turn?
  var pastTurns: [Turn] = [Turn]()

  // 1
  init(turnStrategy: TurnStrategy) {
    self.turnStrategy = turnStrategy
  }

  func beginNewTurn() -> (ShapeView, ShapeView) {
    // 2
    let shapeViews = turnStrategy.makeShapeViewsForNextTurnGivenPastTurns(pastTurns)
    currentTurn = Turn(shapes: [shapeViews.0.shape, shapeViews.1.shape])
    return shapeViews
  }

  func endTurnWithTappedShape(tappedShape: Shape) -> Int {
    currentTurn!.turnCompletedWithTappedShape(tappedShape)
    pastTurns.append(currentTurn!)

    var scoreIncrement = currentTurn!.matched! ? 1 : -1

    return scoreIncrement
  }

  private let turnStrategy: TurnStrategy
}
```

Here's what's happening, section by section:

1.  Accepts a passed strategy and stores it on the `TurnController` instance.

2.  Uses the strategy to generate the `ShapeView` objects so the player can begin a new turn.

> **Note:** This will cause a syntax error in **GameViewController.swift**. Don't worry, it's only temporary. You're going to fix the error in the very next step.

Your last step to use the **Strategy** design pattern is to adapt your `GameViewController` to use your `TurnStrategy`.

Open **GameViewController.swift** and replace its contents with the following:

```swift
import UIKit

class GameViewController: UIViewController {

  override func viewDidLoad() {
    super.viewDidLoad()

    // 1
    let squareShapeViewFactory = SquareShapeViewFactory(size: gameView.sizeAvailableForShapes())
    let squareShapeFactory = SquareShapeFactory(minProportion: 0.3, maxProportion: 0.8)
    let squareShapeViewBuilder = shapeViewBuilderForFactory(squareShapeViewFactory)
    let squareTurnStrategy = BasicTurnStrategy(shapeFactory: squareShapeFactory, shapeViewBuilder: squareShapeViewBuilder)

    // 2
    let circleShapeViewFactory = CircleShapeViewFactory(size: gameView.sizeAvailableForShapes())
    let circleShapeFactory = CircleShapeFactory(minProportion: 0.3, maxProportion: 0.8)
    let circleShapeViewBuilder = shapeViewBuilderForFactory(circleShapeViewFactory)
    let circleTurnStrategy = BasicTurnStrategy(shapeFactory: circleShapeFactory, shapeViewBuilder: circleShapeViewBuilder)

    // 3
    let randomTurnStrategy = RandomTurnStrategy(firstStrategy: squareTurnStrategy, secondStrategy: circleTurnStrategy)

    // 4
    turnController = TurnController(turnStrategy: randomTurnStrategy)

    beginNextTurn()
  }

  override func prefersStatusBarHidden() -> Bool {
    return true
  }

  private func shapeViewBuilderForFactory(shapeViewFactory: ShapeViewFactory) -> ShapeViewBuilder {
    let shapeViewBuilder = ShapeViewBuilder(shapeViewFactory: shapeViewFactory)
    shapeViewBuilder.fillColor = UIColor.brownColor()
    shapeViewBuilder.outlineColor = UIColor.orangeColor()
    return shapeViewBuilder
  }

  private func beginNextTurn() {
    let shapeViews = turnController.beginNewTurn()

    shapeViews.0.tapHandler = {
      tappedView in
      self.gameView.score += self.turnController.endTurnWithTappedShape(tappedView.shape)
      self.beginNextTurn()
    }
    shapeViews.1.tapHandler = shapeViews.0.tapHandler

    gameView.addShapeViews(shapeViews)
  }

  private var gameView: GameView { return view as! GameView }
  private var turnController: TurnController!
}
```

Your revised `GameViewController` uses `TurnStrategy` as follows:

1.  Create a strategy to create squares.

2.  Create a strategy to create circles.

3.  Create a strategy to randomly select either your square or circle strategy.

4.  Create your turn controller to use the random strategy.

Build and run, then go ahead and play five or six turns. You should see something similar to the following screenshots.

[![Screenshot111213**Animatedv2](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot111213**Animatedv2.gif)](https://koenig-media.raywenderlich.com/uploads/2014/10/Screenshot111213**Animatedv2.gif)

Notice how your game randomly alternates between square shapes and circle shapes. At this point, you could easily add a third shape like triangle or parallelogram and your `GameViewController` could use it simply by switching up the strategy.

## Design Patterns: Chain of Responsibility, Command and 迭代器

Think about the example at the beginning of this tutorial:

```swift
var collection = ...

// The for loop condition uses the 迭代器 design pattern
for item in collection {
  print("Item is: \(item)")
}
```

What is it that makes the `for item in collection` loop work? The answer is Swift's `SequenceType`.

By using the **迭代器** pattern in a `for ... in` loop, you can iterate over any type that conforms to the `SequenceType` protocol.

The built-in collection types `Array` and `Dictionary` already conform to `SequenceType`, so you generally don't need to think about `SequenceType` unless you code your own collections. Still, it's nice to know. :\]

Another design pattern that you'll often see used in conjunction with **迭代器** is the **Command** design pattern, which captures the notion of invoking a specific behavior on a target when asked.

For this tutorial, you'll use **Command** to determine if a `Turn` was a match, and compute your game's score from that.

Create a new file named **Scorer.swift**, and replace its contents with the following code:

```swift

// 1
protocol Scorer {
  func computeScoreIncrement<S: SequenceType where Turn == S.Generator.Element>(pastTurnsReversed: S) -> Int
}

// 2
class MatchScorer: Scorer {
  func computeScoreIncrement<S : SequenceType where Turn == S.Generator.Element>(pastTurnsReversed: S) -> Int {
    var scoreIncrement: Int?
    // 3
    for turn in pastTurnsReversed {
      if scoreIncrement == nil {
      	// 4
        scoreIncrement = turn.matched! ? 1 : -1
        break
      }
    }

    return scoreIncrement ?? 0
  }
}
```

Taking each section in turn:

1.  Define your **Command** type, and declare its behavior to accept a collection of past turns that you can iterate over using the **迭代器** design pattern.

2.  Declare a concrete implementation of `Scorer` that will score turns based on whether they matched or not.

3.  Use the **迭代器** design pattern to iterate over past turns.

4.  Compute the score as +1 for a matched turn and -1 for a non-matched turn.

Now open **TurnController.swift** and add the following line near the end, just before the closing brace:

```swift
private let scorer: Scorer
```

Then add the following line to the end of the initializer `init(turnStrategy:)`:

```swift
self.scorer = MatchScorer()
```

Finally, replace the line in `endTurnWithTappedShape` that declares and sets `scoreIncrement` with the following:

```swift
var scoreIncrement = scorer.computeScoreIncrement(pastTurns.reverse())=
```

Take note of how how you reverse `pastTurns` before passing it to the scorer because the scorer expects turns in reverse order (newest first), whereas `pastTurns` stores oldest-first (In other words, it appends newer turns to the end of the array).

Build and run your code. Did you notice something strange? I bet your scoring didn't change for some reason.

You need to make your scoring change by using the **Chain of Responsibility** design pattern.

The **Chain of Responsibility** design pattern captures the notion of dispatching multiple commands across a set of data. For this exercise, you'll dispatch different `Scorer` commands to compute your player's score in multiple additive ways.

For example, not only will you award +1 or -1 for matches or mismatches, but you'll also award bonus points for streaks of consecutive matches. **Chain of Responsibility** allows you add a second `Scorer` implementation in a manner that doesn't interrupt your existing scorer.

Open **Scorer.swift** and add the following line to the top of `MatchScorer`

```swift
var nextScorer: Scorer? = nil
```

Then add the following line to the end of the `Scorer` protocol:

```swift
var nextScorer: Scorer? { get set }
```

Now both `MatchScorer` and any other `Scorer` implementations declare that they implement the **Chain of Responsibility** pattern through their `nextScorer` property.

Replace the `return` statement in `computeScoreIncrement` with the following:

```swift
return (scoreIncrement ?? 0) + (nextScorer?.computeScoreIncrement(pastTurnsReversed) ?? 0)
```

Now you can add another `Scorer` to the chain after `MatchScorer`, and its score gets automatically added to the score computed by `MatchScorer`.

> **Note:** The `??` operator is Swift's **nil coalescing operator**. It unwraps an optional to its value if non-nil, else returns the other value if the optional is nil. Effectively, `a ?? b` is the same as `a != nil ? a! : b`. It's a nice shorthand and I encourage you to use it in your code.

To demonstrate this, open **Scorer.swift** and add the following code to the end of the file:

```swift
class StreakScorer: Scorer {
  var nextScorer: Scorer? = nil

  func computeScoreIncrement<S : SequenceType where Turn == S.Generator.Element>(pastTurnsReversed: S) -> Int {
    // 1
    var streakLength = 0
    for turn in pastTurnsReversed {
      if turn.matched! {
        // 2
        ++streakLength
      } else {
        // 3
        break
      }
    }

    // 4
    let streakBonus = streakLength >= 5 ? 10 : 0
    return streakBonus + (nextScorer?.computeScoreIncrement(pastTurnsReversed) ?? 0)
  }
}
```

Your nifty new `StreakScorer` works as follows:

1.  Track streak length as the number of consecutive turns with successful matches.

2.  If a turn is a match, the streak continues.

3.  If a turn is not a match, the streak is broken.

4.  Compute the streak bonus: 10 points for a streak of five or more consecutive matches!

To complete the **Chain of Responsibility** open **TurnController.swift** and add the following line to the end of the initializer `init(turnStrategy:)`:

```swift
self.scorer.nextScorer = StreakScorer()
```

Excellent, you're using **Chain of Responsibility**.

Build and run. After five successful matches in the first five turns you should see something like the following screenshot.

[![ScreenshotStreakAnimated](https://koenig-media.raywenderlich.com/uploads/2014/10/ScreenshotStreakAnimated.gif)](https://koenig-media.raywenderlich.com/uploads/2014/10/ScreenshotStreakAnimated.gif)

Notice how the score hits 15 after only 5 turns since 15 = 5 points for successful 5 matches + 10 points streak bonus.

## Where To Go From Here?

Here is the [final completed project](https://koenig-media.raywenderlich.com/uploads/2014/12/DesignPatternsInSwift**FinalCompleted.zip) for this tutorial.

You've taken a fun game, **Tap the Larger Shape** and used design patterns to add even more shapes and enhance their styling. You've also used design patterns to add more elaborate scoring.

Most remarkably, even though the final project has many more features, its code is actually simpler and more maintainable than what you started with.

Why not use these design patterns to extend your game even further? Some ideas follow.

**Add more shapes like triangle, parallelogram, star, etc**
Hint: Think back to how you added circles, and follow a similar sequence of steps to add new shapes. If you come up with some really cool shapes, please post screenshots of them in the comments at the bottom of this tutorial!

**Add an animation whenever the score changes**
Hint: Use the `didSet` property observer on `GameView.score`.

**Add controls so that players can choose which types of shapes the game uses**
Hint: Add three `UIButton` or a `UISegmentedControl` with three choices (Square, Circle, Mixed) in `GameView`, which should forward any target actions from the controls on to an **Observer** (Swift closure). `GameViewController` can use these closures to adjust which `TurnStrategy` it uses.

**Persist shape settings to preferences that you can restore**
Hint: Store the player's choice of shape type in `NSUserDefaults`. Try to use the **Facade** design pattern ([Facade details](http://en.wikipedia.org/wiki/Facade**pattern)) to hide your choice of persistence mechanism for this preference from the rest of your code.

**Allow players to select the color scheme for the game**
Hint: Use `NSUserDefaults` to persist the player's choice. Create a `ShapeViewBuilder` that can accept the persisted choice and adjust the app's UI accordingly. Could you use `NSNotificationCenter` to inform all interested views that the color scheme changed so that they can immediately update themselves?

**Have your game play a happy sound when a match occurs and a sad sound when a match fails**
Hint: Extend the **Observer** pattern used between `GameView` and `GameViewController`.

**Use Dependency Injection to pass in the Scorer to TurnController**
Hint: Remove the hard-coded dependency on `MatchScorer` and `StreakScorer` from the initializer.

Thank you for working through this tutorial! Please join the discussion below and share your questions, ideas and cool ways you kicked the game up a few notches.

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。


---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
