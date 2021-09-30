# Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）

2017-01-22发布：hangge阅读：6401

从 **iOS7** 开始，**ViewController** 便使用全屏布局。同时引入了一个新属性 **edgesForExtendedLayout**，本文来讲讲 **edgesForExtendedLayout** 这个属性。

## 一、edgesForExtendedLayout属性介绍

### 1，默认值

它是一个类型为 **UIExtendedEdge** 的属性，指定边缘要延伸的方向，默认值是 **UIRectEdge.all**，即向四周边缘均延伸。

###  2，修改默认值

如果不想要四个边缘均延伸，有两种方式对其进行修改。

（1）在代码中设置

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //四周均不延伸
        self.edgesForExtendedLayout = []
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


（2）在 **storyboard** 中设置，把相关的勾选给去掉。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915070737364.png)](https://www.hangge.com/blog/cache/detail_1519.html#)

### 3，修改后的注意事项

导航栏默认是半透明的，我们还需要修改 **UIWindow** 的背景色，将其改成白色。

```swift
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions:
        [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
         
        self.window?.backgroundColor = UIColor.white
        return true
    }
 
    func applicationWillResignActive(_ application: UIApplication) {
    }
 
    func applicationDidEnterBackground(_ application: UIApplication) {
    }
 
    func applicationWillEnterForeground(_ application: UIApplication) {
    }
 
    func applicationDidBecomeActive(_ application: UIApplication) {
    }
 
    func applicationWillTerminate(_ application: UIApplication) {
    }
}
```

否则设置不延伸后，透过导航栏就会看到下面的黑色背景。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915124119363.png)](https://www.hangge.com/blog/cache/detail_1519.html#)

## 二、edgesForExtendedLayout属性在开发中的作用

下面通过样例演示这个属性在平时开发中，对我们的布局有什么影响。

###  1，对一般界面元素的影响

默认情况下，即使视图中上有 **navigationBar** 或者 **tabBar**，那么视图仍会延伸覆盖到四周的区域。也就是说默认的布局仍然是从顶部开始。

（1）比如我们在 **(0,0)** 位置处添加一个 **UISwitch** 开关组件，默认情况展示结果如下，**UISwitch** 会被导航栏遮挡。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915165788843.png)](https://www.hangge.com/blog/cache/detail_1519.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let uiswitch = UISwitch()
        uiswitch.frame.origin = CGPoint(x:0, y:0)
        uiswitch.isOn = true
        self.view.addSubview(uiswitch)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



（2）而设置不延伸后，默认布局将出导航栏底部开始。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915201791465.png)](https://www.hangge.com/blog/cache/detail_1519.html#)

###  2，对scrollview的影响

- 这里的 **scrollview** 包括 **UIScrollView**，以及继承自 **UIScrollView** 的 **UITableView**、**UICollectionView**、**UITextView** 等。
- 虽然默认延伸状态下，**scrollview** 也是坐标布局也是从屏幕顶部开始，但由于 **automaticallyAdjustsScrollViewInsets** 属性的存在，**scrollview** 会自动调整内边距从而不会被导航栏给遮挡。（具体参考我前面的文章：[Swift - 当存在导航栏时，scrollview自动下移的问题解决](https://www.hangge.com/blog/cache/detail_1514.html)）

（1）虽然 **scrollview** 不会被遮挡，但由于是自动添加了内边距。**scrollview** 向上滚动的时候其实会延伸到导航栏下面。这里我们把单元格背景色改成黑色，会看的更明显些。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915314914985.png)](https://www.hangge.com/blog/cache/detail_1519.html#)[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915320196469.png)](https://www.hangge.com/blog/cache/detail_1519.html#)

（2）如果改成不延伸的话，不管怎么拖动 **scrollview** 都一直会在导航栏下方区域。

[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/201701091533462522.png)](https://www.hangge.com/blog/cache/detail_1519.html#)[![原文:Swift - edgesForExtendedLayout属性介绍（元素被导航栏遮挡问题）](https://www.hangge.com/blog_uploads/201701/2017010915335332807.png)](https://www.hangge.com/blog/cache/detail_1519.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1519.html