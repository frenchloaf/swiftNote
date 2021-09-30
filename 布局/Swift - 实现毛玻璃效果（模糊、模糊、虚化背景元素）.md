# Swift - 实现毛玻璃效果（模糊、模糊、虚化背景元素）

2016-06-03发布：hangge阅读：10444

苹果从 **iOS 7**起，大量使用了毛玻璃效果。比如上拉的控制中心就使用了毛玻璃效果。但到时候还没有向开发者公开提供毛玻璃效果的API，因此开发者可以自己去自己实现毛玻璃效果第三方类库解决。
后来到了 **iOS 8的**，SDK中直接提供了**UIBlurEffect**类与**UIVisualEffectView**类，配合使用这两个类可以轻松实现毛玻璃效果。

**如图1所示，工作准备**
假设我们在页面视图上放置了一个**ImageView的**。下面演示如何在其上添加毛玻璃效果。

[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216343644575.png)](https://www.hangge.com/blog/cache/detail_1135.html#)

**2，使用UIBlurEffect、UIVisualEffectView实现毛玻璃效果**
（1）创建一个**UIBlurEffect**类的实例，并指定某一种毛玻璃效果样式。
（2）创建**UIVisualEffectView**类的实例，这个可以称为模糊视图。将**前面**创建的**UIBlurEffect**类的实例应用到这个模糊视图上。
（3）将**UIVisualEffectView**类的实例（模糊视图）置于待毛玻璃化的视图之上即可。在其下方的所有视图都会有模糊效果。

[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216351541219.png)](https://www.hangge.com/blog/cache/detail_1135.html#)

```
import` `UIKit` `class` `ViewController``: ``UIViewController` `{` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``    ` `    ``//首先创建一个模糊效果``    ``let` `blurEffect = ``UIBlurEffect``(style: .``Light``)``    ``//接着创建一个承载模糊效果的视图``    ``let` `blurView = ``UIVisualEffectView``(effect: blurEffect)``    ``//设置模糊视图的大小（全屏）``    ``blurView.frame.size = ``CGSize``(width: view.frame.width, height: view.frame.height)``    ``//添加模糊视图到页面view上（模糊视图下方都会有模糊效果）``    ``self``.view.addSubview(blurView)``  ``}` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```

**3，三个模糊效果**
除了上面使用的**Light****样式**。**UIBlurEffect**一共定义了三种效果，这些效果由枚举**UIBlurEffectStyle**来确定，

分别是：**ExtraLight**、**Light**、**Dark**。
我们可以根据背景（**色调**）来确定特效视图与底部视图的混合样式，可以更符合我们的页面风格。
[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/201606021640345824.png)](https://www.hangge.com/blog/cache/detail_1135.html#)[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216404461180.png)](https://www.hangge.com/blog/cache/detail_1135.html#)[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216405043470.png)](https://www.hangge.com/blog/cache/detail_1135.html#)

**如图4所示，添加活力多元**
通过模糊背景，我们可以让前景文字或其它视图元素显得更加突出。 而苹果在其之上又引入了**UIVibrancyEffect**。**UIVibrancyEffect**主要用于放大和调整**UIVisualEffectView**视图下面的内容的颜色，同时让**UIVisualEffectView**的**内容查看**中的内容看起来更加生动。 通常**UIVibrancyEffect**对象是与**UIBlurEffect**一起使用，下面对页面上的文本标签做**活力多元**效果，可以看到文本标签在屏幕上显得更为舒适。

[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216534530822.png)](https://www.hangge.com/blog/cache/detail_1135.html#)





[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216364185006.png)](https://www.hangge.com/blog/cache/detail_1135.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //首先创建一个模糊效果
        let blurEffect = UIBlurEffect(style: .Light)
        //接着创建一个承载模糊效果的视图
        let blurView = UIVisualEffectView(effect: blurEffect)
        //设置模糊视图的大小（全屏）
        blurView.frame.size = CGSize(width: view.frame.width, height: view.frame.height)
         
        //创建并添加vibrancy视图
        let vibrancyView = UIVisualEffectView(effect:
            UIVibrancyEffect(forBlurEffect: blurEffect))
        vibrancyView.frame.size = CGSize(width: view.frame.width, height: view.frame.height)
        blurView.contentView.addSubview(vibrancyView)
         
        //将文本标签添加到vibrancy视图中
        let label=UILabel(frame:CGRectMake(10,20, 300, 100))
        label.text = "hangge.com"
        label.font = UIFont(name: "HelveticaNeue-Bold", size: 40)
        label.textAlignment = .Center
        label.textColor = UIColor.whiteColor()
        vibrancyView.contentView.addSubview(label)
         
        self.view.addSubview(blurView)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


将**blurEffect**的样式**设为.Dark**的效果：

[![原文:Swift - 实现毛玻璃效果（Blur、模糊、虚化背景元素）](https://www.hangge.com/blog_uploads/201606/2016060216362523037.png)](https://www.hangge.com/blog/cache/detail_1135.html#)



```swift
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //首先创建一个模糊效果
        let blurEffect = UIBlurEffect(style: .Dark)
        //接着创建一个承载模糊效果的视图
        let blurView = UIVisualEffectView(effect: blurEffect)
        //设置模糊视图的大小（全屏）
        blurView.frame.size = CGSize(width: view.frame.width, height: view.frame.height)
         
        //创建并添加vibrancy视图
        let vibrancyView = UIVisualEffectView(effect:
            UIVibrancyEffect(forBlurEffect: blurEffect))
        vibrancyView.frame.size = CGSize(width: view.frame.width, height: view.frame.height)
        blurView.contentView.addSubview(vibrancyView)
         
        //将文本标签添加到vibrancy视图中
        let label=UILabel(frame:CGRectMake(10,20, 300, 100))
        label.text = "hangge.com"
        label.font = UIFont(name: "HelveticaNeue-Bold", size: 40)
        label.textAlignment = .Center
        label.textColor = UIColor.whiteColor()
        vibrancyView.contentView.addSubview(label)
         
        self.view.addSubview(blurView)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，源码下载：**![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1135.zip](https://www.hangge.com/blog_uploads/201607/2016072713582237228.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1135.html