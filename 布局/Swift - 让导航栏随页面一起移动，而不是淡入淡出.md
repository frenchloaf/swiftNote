# Swift - 让导航栏随页面一起移动，而不是淡入淡出

2016-04-28发布：hangge阅读：4653

发现市面上有很多APP在页面切换的时候，顶部导航栏是随着所在的页面一起移动的，使用手势滑动返回可以看得更明显些。比如下图是网易新闻的页面截图：

  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522195168755.png)](https://www.hangge.com/blog/cache/detail_1117.html#)  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522201024553.png)](https://www.hangge.com/blog/cache/detail_1117.html#)  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522200386078.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

我们应用中如果使用导航控制器（**navigationController**），其默认自带的导航栏在页面切换的时候是固定不动的，而上面的元素（标题文字，返回按钮等）会随着页面移动而有淡入淡出的效果：

  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522245080565.png)](https://www.hangge.com/blog/cache/detail_1117.html#)  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522245739309.png)](https://www.hangge.com/blog/cache/detail_1117.html#)  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522250488148.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

下面实现让导航栏随视图移动的效果。

**1，实现原理**

我们将 **navigationController** 自带的导航栏隐藏，然后每个页面分别使用自定义的导航栏（**navigationBar**）

**2，实现步骤**

（1）选中 **Navigation Controller**，将“**Show Navigation Bar**”取消勾选，这样自带的导航栏就不会显示了。

[![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522290414535.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

（2）分别给首页和详情页添加 **Navigation Bar**，并设置颜色。注意：把“**Translucent**”取消勾选，防止导航栏移动的时候添加渐变透明度效果。

[![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522291498312.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

（3）如果觉得顶部状态栏使用黑色或白色背景都不好看的话，可以在上面添加个20个点高度的 **UIView**，然后设置需要的颜色，这里用和导航栏一样的绿色。

[![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522292331995.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

（4）二级页面（详情页）导航栏左侧再加个按钮用于返回

[![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522293187311.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

（5）详情页对应类（**DetailViewController.swift**）中添加返回按钮相关代码。同时由于隐藏了默认导航条，所以滑动返回失效，还要在代码中加个手势实现滑动返回。

```swift
import UIKit
 
class DetailViewController: UIViewController, UIGestureRecognizerDelegate {
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //启用滑动返回（swipe back）
        self.navigationController?.interactivePopGestureRecognizer!.delegate = self
    }
 
    //返回按钮点击响应
    @IBAction func backToPrevious(sender: AnyObject) {
         self.navigationController?.popViewControllerAnimated(true)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**3，运行效果**

  [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522321438367.png)](https://www.hangge.com/blog/cache/detail_1117.html#)   [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/2016042522324635844.png)](https://www.hangge.com/blog/cache/detail_1117.html#)   [![原文:Swift - 让导航栏随页面一起移动，而不是淡入淡出](https://www.hangge.com/blog_uploads/201604/201604252233007689.png)](https://www.hangge.com/blog/cache/detail_1117.html#)

**源码下载：**![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1117.zip](https://www.hangge.com/blog_uploads/201604/2016042522352785145.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1117.html