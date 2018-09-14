#使用[栈]结构完成一个页面切换控制器 Page Stack





在很多新闻咨询类 App 中, 经常会使用一个滚动视图来展示各种分类信息, 使用 ScrollView 或 CollectionView 添加多个 View, 他们都在一个视图控制器中, 通过滚动达到切换页面的目的, 这种视图的结构和布局大概是这样:

![toutiaoCollectionView](http://7xo0hj.com1.z0.glb.clouddn.com/toutiaoCollectionView.png)



通过在视图控制器添加一层滚动视图扩充它的 contentSize 达到切换内容的目的



这种视图结构在处理单层页面时很方便, 但是如果某些场景需要处理多层视图结构如何处理呢, 例如很多打车软件, 它们都有一个主视图控制器,底层显示着地图,根据业务功能的不同显示不同的视图元素,点击打车或一些按钮会跳转到第二层视图, 但是它们的视图控制器都还是在主视图控制器中



![DiDIStackPage](http://7xo0hj.com1.z0.glb.clouddn.com/DiDIStackPage.png)



这种视图结构很适合使用 栈 来实现, 我们回顾一下系统 UINavigationController 的实现方式, UINavigationController通过 Navigation Stack 来管理 View controller，对View进行push/pop:



# Page Stack的实现

我们参照系统的 UINavigationController 实现一个控制 View 的导航控制器, Page Stack

首先创建一个类, 定义一些基础属性

```swift
public enum StackViewNavigationOperation: Int {
	case none
	case push
	case pop
}

class StackViewNavigation {
    
    var views: [UIView] = []
    
    var isAnimation: Bool = false
	
	var topView: UIView
	
	var rootView: UIView
    
    // 用来临时保存上一个 topView
	var lastTopView: UIView
    
    /// UIViewController 的 View, 所有视图都添加到此视图上
    var superView: UIView
    
    init(viewController:UIViewController, rootView: UIView) {
        self.superView = viewController.view
		self.superViewHeight = viewController.view.bounds.height
		self.rootView = rootView
		self.topView = rootView
		self.lastTopView = rootView
		views.append(rootView)
	}
}
```



考虑到扩展性, 我们可以定义一个视图切换动画协议来处理视图的切换

```swift
class StackViewNavigation: ViewSwitchAnimation {
    ...
}

protocol ViewSwitchAnimation {
	var superView: UIView { get set }
	var superViewHeight: CGFloat { get set }
    /// 记录动画中的 closure, 便于外部监听动画中事件
	var nextViewAnimationAction: ((UIView, Bool) -> Void)? { get set }
	
    /// 设置切换的视图布局, 便于更改视图高度
    var bottomInster: CGFloat { get set }
	var topInster: CGFloat { get set }
}

extension ViewSwitchAnimation {
	/// 视图切换动画
	func animation(operation: StackViewNavigationOperation, topView: UIView, nextView: UIView, animated: Bool,completion: ((Bool) -> Void)?) {
		....
        /// 动画结束 closure
        completion?(finished)
	}
}
```



我们实现视图的一些栈操作, 类似系统的 UINavigationController

```swift
func pushView(_ view: UIView, animated: Bool) -> Bool {
		let success = viewAnimation(operation: .push, nextView: view, animated: animated) { (finished) in
			if finished {
				self.views.append(view)
			}
		}
		return success
}


func popView(animated: Bool) -> UIView? {
		
		guard views.count - 2 >= 0 else {
			print("popView failure, 已经是 rootView")
			return nil
		}
		
		let nextView = views[views.count - 2]
		
		let success = viewAnimation(operation: .pop, nextView: nextView, animated: animated) { (finished) in
			self.views.removeLast()
		}
		
		return success ? topView : nil
}


func popToView(_ view: UIView, animated: Bool) -> [UIView]? {
		
		guard let index = views.index(of: view) else {
			print("PopToView failure, 无法找到当前 view")
			return nil
		}
		
		guard views.last != view else {
			print("PopToView failure, 和当前 topView 是同一个 view")
			return nil
		}
		
		let viewsSlice = views[0...index]
		let popedSlice = views[(index + 1)...]
		
		let success = viewAnimation(operation: .pop, nextView: view, animated: animated) { (finished) in
			if finished {
				self.views = Array(viewsSlice)
			}
		}
		
		return success ? Array(popedSlice) : nil
}


func popToRootView(animated: Bool) -> [UIView]? {
		
		guard rootView != topView else {
			print("popToRootView failure, 和当前 topView 是同一个 view")
			return nil
		}
		
		var popedView = views
		popedView.removeFirst()
		
		let success = viewAnimation(operation: .pop, nextView: rootView, animated: animated) { (finished) in
						if finished {
							self.views = [self.rootView]
						}
					}
		
		return success ? popedView : nil
}

/// 弹出视图到一个新的 stack root view
	///
	/// - Parameters:
	///   - view: view
	///   - animated: 动画
	/// - Returns: 返回上一个 rootView 和 弹出的 views
func popToNewRootView(view: UIView, animated: Bool) -> (UIView?, [UIView]?) {
		
		guard view != topView else {
			print("popToNewRootView failure, 和当前 topView 是同一个 view")
			return (nil, nil)
		}
		
		guard !isAnimation else {
			print("popToNewRootView failure, 前一个动画还未结束")
			return (nil, nil)
		}
		
		rootView = view
		let lastRootView = views.removeFirst()
		views.insert(view, at: 0)
		
		return (lastRootView, popToRootView(animated: animated))
}
```



我们完成了基础的视图栈操作, 包括 Push, Pop,  PopToView, PopToRootView, 它的功能和 UINavigationController 一致, 只是它在一个视图控制器中, 控制显示在视图控制器的视图, 我们可以扩展下它的方法, 比如某层栈视图的横向切换, 例如滴滴业务视图 StackPage1 的左右切换



```swift
/// 使用 ViewSwitchAnimation 协议
public class NavigationRoute: ViewSwitchAnimation {
    
    ...
    
    /// StackViewNavigation 属性
    fileprivate var stackViewNavigation: StackViewNavigation
   
    public init(viewController:UIViewController, rootView: UIView) {
		stackViewNavigation = StackViewNavigation(rootView: rootView)
        /// 协议属性 
		self.superView = viewController.view
		self.superViewHeight = viewController.view.bounds.height
	}
    
    /// stackViewNaviation 的一些方法
    public func pushView(_ view: UIView, animated: Bool) -> Bool {
		return stackViewNavigation.pushView(view, animated: animated)
	}
	
	public func popToRootView(animated: Bool) -> Bool {
		return stackViewNavigation.popToRootView(animated: animated) != nil ? true : false
	}
	
	public func popView(animated: Bool) -> Bool {
		return stackViewNavigation.popView(animated: animated) != nil ? true : false
	}
	
	public func popToView(_ view: UIView, animated: Bool) -> Bool {
		return stackViewNavigation.popToView(view, animated: animated) != nil ? true : false
	}
}
```



通过 NavigationRoute 封装了一层, 我们可以写一些代理方法来记录视图切换的一些事件, 并实现横向切换

```swift
public protocol NavigationRouteDelegate: class {
    /// 当前栈顶视图将要显示
	func navigationRoute(route: NavigationRoute, nextViewDidShow nextView: UIView, topView: UIView)
    /// 当前 top 视图将要隐藏
	func navigationRoute(route: NavigationRoute, topViewWillHidden topView: UIView, nextView: UIView)
    /// 视图切换动画中
	func navigationRoute(route: NavigationRoute, nextViewAnimation nextView: UIView, animated: Bool)
}

/// 横向切换视图, pop: 切换到左边动画,  push: 切换到右边的动画
public func switchTopView(direction: StackViewNavigationOperation, nextView: UIView, animated: Bool) -> Bool {
		
		guard !stackViewNavigation.isAnimation else {
			print("switchTopView failure, 前一个动画还未结束")
			return false
		}
		
		guard stackViewNavigation.topView != nextView else {
			print("switchTopView failure, nextView 和 topView 是同一个 view")
			return false
		}
		
		delegate?.navigationRoute(route: self, topViewWillHidden: stackViewNavigation.topView, nextView: nextView)
		
		stackViewNavigation.isAnimation = true
		
		animation(operation: direction, topView: stackViewNavigation.topView, nextView: nextView, animated: animated) { (finished) in
			
			self.stackViewNavigation.topView = nextView
			
			self.stackViewNavigation.isAnimation = false
			
			self.stackViewNavigation.views.removeLast()
			// 防止更换的 topView 是 rootView
			if self.stackViewNavigation.views.count == 0 {
				self.stackViewNavigation.rootView = nextView
			}
			self.stackViewNavigation.views.append(nextView)
			
			let lastTopView = self.stackViewNavigation.lastTopView
			self.delegate?.navigationRoute(route: self, nextViewDidShow: nextView, topView: lastTopView)
			self.stackViewNavigation.lastTopView = nextView
		}
		
		return true
	}
```



至此, 我们实现了一个视图栈切换功能, switchTopView 方法当然也可以通过传递一个 UIScrollView 或 UICollectionView 来实现, 但是这样的话监听视图切换事件的代理可能就需要外部调用实现, 如果使用内部的 switchTopView 方法, 我们可以把视图切换的所有事件都在内部监听便于外部使用

看看效果:





![DiDIStackPage](http://7xo0hj.com1.z0.glb.clouddn.com/NavigationRoute.gif)



#使用

```swift
/// 设置 RootView, 添加到 ViewController 的 view 中完成布局
let rootView = UIView()
view.addSubview(rootView)
rootView.snp.makeConstraints { (maker) in
	maker.top.equalToSuperview().inset(64)
	maker.left.right.bottom.equalToSuperview()
}		

/// 初始化 NavigationRoute, 并设置 topInster 和 bottomInster
navigationRoute = NavigationRoute(viewController: self, rootView: rootView)
navigationRoute.topInster = 64
navigationRoute.setupDelegate()
navigationRoute.delegate = self

/// 视图切换的一些方法
navigationRoute.pushView(view, animated: animated)

navigationRoute.popView(animated: animated)

navigationRoute.popToRootView(animated: animated)

navigationRoute.switchTopView(direction: .push, nextView: view, animated: animated)


/// 代理方法
func navigationRoute(route: NavigationRoute, nextViewDidShow nextView: UIView, topView: UIView) {
	print("nextViewDidShow")
}
	
func navigationRoute(route: NavigationRoute, topViewWillHidden topView: UIView, nextView: UIView) {
    /// 可设置切换后的视图布局
	nextView.topInster = 100
    naxtView.bottomInster = 50
    print("topViewWillHidden")
}
	
func navigationRoute(route: NavigationRoute, nextViewAnimation nextView: UIView, animated: Bool) {
    /// 切换视图动画中... 可把其他联动动画放在这里
	print("nextViewAnimation")
}
```



这还可以扩展很多方法, 比如上下的切换, 自定义动画, 有兴趣可以自己改造下

项目代码已经放到 github: [https://github.com/ZeroJian/NavigationRoute]







