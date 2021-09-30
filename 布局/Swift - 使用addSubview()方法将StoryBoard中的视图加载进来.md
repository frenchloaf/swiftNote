# Swift - 使用addSubview()方法将StoryBoard中的视图加载进来

2016-05-11发布：hangge阅读：4415

（本文代码已升级至Swift3）

使用 **Storyboard** 我们可以很方便地搭建好各种复杂的页面，同时通过 **segue** 连接可以轻松实现页面的跳转。

**但除了segue，我们还可以使用纯代码的方式实现Storyboard界面的跳转。**

比如：使用 **presentViewController()** 方法将当前页面视图切换成新视图

```swift
let myNavigaiton = UIStoryboard(name: "Main", bundle: nil)
    .instantiateViewController(withIdentifier: "myNavigaiton") as! UINavigationController
self.present(myNavigaiton, animated: false, completion: nil)
```



如果有视图导航控制器的话，还可以使用 **self.navigationControler.pushViewController** 和 **popViewController** 来实现前进到下一个视图或回到上一个视图。

```swift
let viewController = UIStoryboard(name: "Second", bundle: nil)
    .instantiateViewController(withIdentifier: "SecondView") as UIViewController
self.navigationController?.pushViewController(viewController, animated: true)
```


**本文介绍第三种方式：使用addChildViewController()将StoryBoard中的ViewController加载进来**
比如下面的storyboard，左边一个是初始页面，点击上面的“加载”按钮，希望能把右边的logo页的视图加载进来。

[![原文:Swift - 使用addSubview()方法将StoryBoard中的视图加载进来](https://www.hangge.com/blog_uploads/201605/2016050520252210728.png)](https://www.hangge.com/blog/cache/detail_1167.html#)


我们可以通过 **self.view.addSubview()** 方法将另一个VC视图加载进来：

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func btnClick(sender: AnyObject) {
        //从StoryBoard中获取视图控制器
        let logoView  = UIStoryboard(name: "Main", bundle: nil)
            .instantiateViewController(withIdentifier: "logoView")
        //添加获取到的视图控制器的视图
        self.view.addSubview(logoView.view)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

但运行后发现会发现个问题，图片错位了，也就是说logo页上定义的约束条件失效了。这是由于我们没将logo页对应的视图控制器添加进来。

[![原文:Swift - 使用addSubview()方法将StoryBoard中的视图加载进来](https://www.hangge.com/blog_uploads/201605/2016050519320080516.png)](https://www.hangge.com/blog/cache/detail_1167.html#)


所以除了用 **addSubview()** 方法把视图添加进来，还要用 **addChildViewController()** 方法将视图对应的视图控制器给加载进来。
**下面是最终的代码：**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func btnClick(sender: AnyObject) {
        //从StoryBoard中获取视图控制器
        let logoView  = UIStoryboard(name: "Main", bundle: nil)
            .instantiateViewController(withIdentifier: "logoView")
        //添加获取到的视图控制器的视图
        self.view.addSubview(logoView.view)
        //添加子视图控制器
        addChildViewController(logoView)
        logoView.didMove(toParentViewController: self)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

运行后可以看到，页面显示正常了。

[![原文:Swift - 使用addSubview()方法将StoryBoard中的视图加载进来](https://www.hangge.com/blog_uploads/201605/2016050519324069951.png)](https://www.hangge.com/blog/cache/detail_1167.html#)

**源码下载：**![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1167.zip](https://www.hangge.com/blog_uploads/201703/2017030614032085571.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1167.html