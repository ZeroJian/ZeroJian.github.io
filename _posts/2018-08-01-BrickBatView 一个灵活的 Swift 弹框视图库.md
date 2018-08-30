##BrickBatView 一个灵活的 Swift 弹框视图库



BrickBatView 可通过组件化的形式配置视图的每行元素, 支持扩展, 它类似砌墙, 每个砖块都可以自定义, 通过叠加砖块展示一面墙(视图), 通过链式调用的方式配置弹框的各个元素

![1111](http://7xo0hj.com1.z0.glb.clouddn.com/BrickBatView1.png)



通过 BrickBatView 可灵活添加各种视图元素

![Example](http://7xo0hj.com1.z0.glb.clouddn.com/BrickBatView2.png)



##使用

简单调用方式:

```swift
BrickBatView(inView: view)?
      .setup()
      .addTitleItem(title: "Title", infoicon: nil)
      .addMessageItem(text: "message")
      .addButtonItem(title: ["cancel", "done"], style: .fill)
      .show()
```



配置各种属性和自定义

```
BrickBatView(inView: view)?
      .handle(action: { (index) in
          print("sender index: \(index)")
      }, tapHidden: true)

      .identifier("BrickView_SETUP")

      .lifeCyle(showFinishedAction: { (show) in
          print("isShowFinished")
      }, hiddenAction: {
          print("isHidden")
      })
      .offset(10)
      .position(.bottom, edgeInster: 20)
```



通过自定义视图组件配置成 Item

```	swift
let item = BrickBarItem()
brickBatView
      .addBrickItem(item)
...

let imageView = UIimageView()
brickBatView
      .addGesture([imageView])
      
extension BrickBatView {
	func addExtensionTextField() -> Self {
      let textField = UITextField()
      textField.bounds.size.height = 50
      textField.placeholder = "BrickView addTextField Extension"
      textField.borderStyle = .roundedRect
      return addContentView(textField)
    }
}

brickBatView
      .addExtensionTextField()
...
```

扩展组件元素可通过传入子定义 View 和遵循 BrickBat 协议两种方式扩展

```swift
  /// 添加自定义 view
    ///
    /// - Parameters:
    ///   - view: 自定义 view
    ///   - controls: 事件响应对象(例如:可把自定义 view 里面增加的 button 传入进来, 由 BrickView,响应点击事件)
    /// - Returns: self
    public func addContentView(_ view: UIView, controls: [UIControl]? = default) -> Self

    /// 添加 BrickBat 协议类型 Item
    ///
    /// - Parameter item: 遵循 BrickBat 协议 Item
    /// - Returns: self
    public func addBrickItem<B>(_ item: B) -> Self where B : BrickBat
```



自定义视图组件可传入 UIControl , 点击事件由 BrickBatView处理, 点击 index 通过增加的 control 自增长

```swift
let buttonView = ButtonView()
brickBat
	.addContentView(buttonView, controls: buttonView.button)
	.handle(action: { (index) in
          print("sender index: \(index)")
      }, tapHidden: true)

```



##小结

源码和Demo请点[这里](https://github.com/ZeroJian/BrickBatView)

更多的细节欢迎运行demo 或者查看源代码 有任何问题欢迎提出来大家一起讨论研究 :)