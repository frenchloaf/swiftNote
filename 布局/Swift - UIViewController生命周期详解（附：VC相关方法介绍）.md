# Swift - UIViewController生命周期详解（附：VC相关方法介绍）

2016-08-05发布：hangge阅读：9272

**UIViewController**（视图控制器）想必大家都不会陌生，开发中常常会用到。这次讲讲它的生命周期。

**1，视图的生命周期**

说是 **ViewController** 的生命周期，其实指的是它控制的视图（**View**）的生命周期。每当视图的状态发生变化时，视图控制器会自动调用一系列方法来响应变化。

通过这些方法，我们就可以跟踪到视图的整个生命周期。各个方法按执行顺序排列如下：

（1）**init**：初始化程序

（2）**loadView**：视图初始化

这个方法不应该被直接调用，而是由系统自动调用。它会加载或创建一个 **view** 并把它赋值给 **UIViewController** 的 **view** 属性。

同时重写 **loadView** 方法的时候，不要调用父类的方法。

（3）**viewDidLoad**：视图加载完成，但还没在屏幕上显示出来

我们可以重写这个方法，对 **view** 做一些其他的初始化工作。比如可以移除一些视图，修改约束，加载数据等。

（3）**viewWillAppear**：在视图即将显示在屏幕上时调用

我们可以在这个方法里，改变当前屏幕方向或状态栏的风格等。

（4）**viewDidApper**：在视图显示在屏幕上时调用时调用

我们可以在这个方法中，对视图做一些关于展示效果方面的修改。

（5）**viewWillDisappear**：视图即将消失、被覆盖或是隐藏时调用

（6）**viewDidDisappear**：视图已经消失、被覆盖或是隐藏时调用

（7）**viewVillUnload**：当内存过低时，需要释放一些不需要使用的视图时，即将释放时调用

（8）**viewDidUnload**：当内存过低，释放一些不需要的视图时调用。

注意：自 **iOS6** 起，**viewWillUnload** 和 **viewDidUnload** 这两个方法被废除了。当系统发出内存警告的时候，会自动把 **view** 给清除掉，不用我们再特别处理。

同时系统还会调用 **didReceiveMemoryWarning** 方法通知视图控制器，我们可以在这里面进行一些操作，来释放一些额外的资源。（通常来说不用操作，比较最占资源的 **view** 已经被系统给清理了。）

**2，视图状态的转换**

在实际应用中，视图通常不会按照上面列的流程一次执行下来，可能会在可见与不可见的状态间互相转换。比如一开始视图是可见的，接着我们跳转到另一个 **ViewController**，这时原来视图就变成不可见的。后面我们又跳转回来，那么这个视图就又是可见的。

当视图的可见性发生变化时，视图控制器对应的方法也会随之响应。具体可见下图：

[![原文:Swift - UIViewController生命周期详解（附：VC相关方法介绍）](https://www.hangge.com/blog_uploads/201608/2016080410091095970.png)](https://www.hangge.com/blog/cache/detail_1319.html#)

特别要注意的是：**Appearing** 和 **Disappearing** 这两个状态是可以互相转化的。

**3，测试样例说明**

（1）**ViewController** 是首页视图控制器，我们将里面所有的与生命周期有关的函数都打印出来。
（2）同时 **ViewController** 中添加了一个“跳转”按钮，点击后跳转到另一个视图控制器（**AnotherViewController**）。
（3）**AnotherViewController** 里有个“返回”按钮，点击又会回到前一个页面。

[![原文:Swift - UIViewController生命周期详解（附：VC相关方法介绍）](https://www.hangge.com/blog_uploads/201608/2016080420481240527.png)](https://www.hangge.com/blog/cache/detail_1319.html#)



**4，测试代码** 

（1）**ViewController.swift**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //视图初始化
    override func loadView() {
        super.loadView()
        print("loadView")
    }
     
    //视图加载完成
    override func viewDidLoad() {
        super.viewDidLoad()
        print("viewDidLoad")
         
        //创建跳转按钮
        let button:UIButton = UIButton(type: .System)
        button.frame=CGRectMake(10, 50, 100, 30)
        button.setTitle("跳转", forState: .Normal)
        button.addTarget(self,action:#selector(jump),forControlEvents:.TouchUpInside)
        self.view.addSubview(button);
    }
     
    //视图将要出现的时候执行
    override func viewWillAppear(animated: Bool) {
        print("viewWillAppear")
    }
     
    //视图显示完成后执行
    override func viewDidAppear(animated: Bool) {
        print("viewDidAppear")
    }
     
    //视图将要消失的时候执行
    override func viewWillDisappear(animated: Bool) {
        print("viewWillDisappear")
    }
     
    //视图已经消失的时候执行
    override func viewDidDisappear(animated: Bool) {
        print("viewDidDisappear")
    }
     
    //收到内存警告时执行
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
     
    //跳转到另一个视图
    func jump(){
        print("点击按钮，开始跳转！")
        let anotherVC = AnotherViewController()
        presentViewController(anotherVC, animated: true, completion: nil)
    }
}
```

（2）**AnotherViewController.swift**

```swift
import UIKit
 
class AnotherViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //创建返回按钮
        let button:UIButton = UIButton(type: .System)
        button.frame=CGRectMake(10, 150, 100, 30)
        button.setTitle("返回", forState: .Normal)
        button.addTarget(self,action:#selector(back),forControlEvents:.TouchUpInside)
        self.view.addSubview(button);
    }
     
    //返回之前视图
    func back(){
        print("点击按钮，开始返回！")
        self.dismissViewControllerAnimated(true, completion: nil)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，运行测试**
我们从 **ViewController** 跳到 **AnotherViewController**，再从 **AnotherViewController** 跳回 **ViewController**。整个控制台打印出来的流程如下：

[![原文:Swift - UIViewController生命周期详解（附：VC相关方法介绍）](https://www.hangge.com/blog_uploads/201608/2016080420490596692.png)](https://www.hangge.com/blog/cache/detail_1319.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1319.html