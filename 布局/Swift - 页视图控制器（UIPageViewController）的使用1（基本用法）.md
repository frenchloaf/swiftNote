# Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）

2016-07-19发布：hangge阅读：8179

我们可以将多个水平关系的 **ViewController** 用 **UIPageViewController** 来管理。通过 **pageViewController**，用户可以方便地在不同的页面内容之间导航切换。这个在App的新手教学、引导页上比较常见。下面通过样例演示 **pageViewController** 的用法。

**1，样例说明**

（1）**pageViewController** 下有三个 **VC** 页面，通过滑动屏幕则可以进行页面切换，切换时还有翻书效果。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715370281710.png)](https://www.hangge.com/blog/cache/detail_1282.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715370921858.png)](https://www.hangge.com/blog/cache/detail_1282.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715371468927.png)](https://www.hangge.com/blog/cache/detail_1282.html#)

（2）页面切换除了使用翻页方式，还有滚动方式。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715370281710.png)](https://www.hangge.com/blog/cache/detail_1282.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715381974253.png)](https://www.hangge.com/blog/cache/detail_1282.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071715371468927.png)](https://www.hangge.com/blog/cache/detail_1282.html#)

**2，实现步骤**

（1）首先在 **storyboard** 中拖入一个 **UIPageViewController**，并将其设为初始控制器“**Is Initial View Controller**”

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071713540555911.png)](https://www.hangge.com/blog/cache/detail_1282.html#)



（2）自定义一个继承自 **UIPageViewController** 的子类（我这里叫 **PageViewController**），并指定给前面创建 **UIPageViewController**。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071713572968685.png)](https://www.hangge.com/blog/cache/detail_1282.html#)



（3）在 **storyboard** 中继续添加3个 **UIViewController**，将它们的 **Storyboard ID** 分别命名为“**firstVC**”、“**secondVC**”和“**thirdVC**”

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071714001297795.png)](https://www.hangge.com/blog/cache/detail_1282.html#)

（4）**PageViewController.swift** 的完整代码如下。

```swift
import UIKit
 
class PageViewController: UIPageViewController, UIPageViewControllerDataSource {
     
    //所有页面的视图控制器
    private(set) lazy var allViewControllers: [UIViewController] = {
        return [self.getViewController("firstVC"),
                self.getViewController("secondVC"),
                self.getViewController("thirdVC")]
    }()
     
    //页面加载完毕
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //数据源设置
        dataSource = self
         
        //设置首页
        if let firstViewController = allViewControllers.first {
            setViewControllers([firstViewController],
                               direction: .Forward,
                               animated: true,
                               completion: nil)
        }
    }
     
    //根据indentifier获取Storyboard里的视图控制器
    private func getViewController(indentifier: String) -> UIViewController {
        return UIStoryboard(name: "Main", bundle: nil) .
            instantiateViewControllerWithIdentifier("\(indentifier)")
    }
     
    //获取前一个页面
    func pageViewController(pageViewController: UIPageViewController,
                            viewControllerBeforeViewController
                            viewController: UIViewController) -> UIViewController? {
        guard let viewControllerIndex = allViewControllers.indexOf(viewController) else {
            return nil
        }
         
        let previousIndex = viewControllerIndex - 1
         
        guard previousIndex >= 0 else {
            return nil
        }
         
        guard allViewControllers.count > previousIndex else {
            return nil
        }
         
        return allViewControllers[previousIndex]
    }
     
    //获取后一个页面
    func pageViewController(pageViewController: UIPageViewController,
                            viewControllerAfterViewController
                            viewController: UIViewController) -> UIViewController? {
        guard let viewControllerIndex = allViewControllers.indexOf(viewController) else {
            return nil
        }
         
        let nextIndex = viewControllerIndex + 1
        let orderedViewControllersCount = allViewControllers.count
         
        guard orderedViewControllersCount != nextIndex else {
            return nil
        }
         
        guard orderedViewControllersCount > nextIndex else {
            return nil
        }
         
        return allViewControllers[nextIndex]
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1282.zip](https://www.hangge.com/blog_uploads/201607/2016071717420993837.zip)
**
**

**3，改变页面切换方式**
上面样例页面切换时，使用默认的翻页方式。我们可以将 **pageViewController** 的 **Transition Style** 设置为 **Scroll**，改成滚动的方式。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog_uploads/201607/2016071717473269316.png)](https://www.hangge.com/blog/cache/detail_1282.html#)



**4，让页面循环滚动**

修改下面两个方法，可以实现页面循环切换。比如：从第一页一直翻到最后一页，还可以继续滑动，便又从第一页开始。向前滑动也是一样的效果，翻到第一页后再翻又会到最后一页。

```swift
//获取前一个页面
func pageViewController(pageViewController: UIPageViewController,
                        viewControllerBeforeViewController
                        viewController: UIViewController) -> UIViewController? {
    guard let viewControllerIndex = allViewControllers.indexOf(viewController) else {
        return nil
    }
     
    let previousIndex = viewControllerIndex - 1
     
    //如果当前是首页，向右滑显示最后一个页面
    guard previousIndex >= 0 else {
        return allViewControllers.last
    }
     
    guard allViewControllers.count > previousIndex else {
        return nil
    }
     
    return allViewControllers[previousIndex]
}
 
//获取后一个页面
func pageViewController(pageViewController: UIPageViewController,
                        viewControllerAfterViewController
                        viewController: UIViewController) -> UIViewController? {
    guard let viewControllerIndex = allViewControllers.indexOf(viewController) else {
        return nil
    }
     
    let nextIndex = viewControllerIndex + 1
    let orderedViewControllersCount = allViewControllers.count
     
    //如果当前是最后一个页面，向左滑显示首页
    guard orderedViewControllersCount != nextIndex else {
        return allViewControllers.first
    }
     
    guard orderedViewControllersCount > nextIndex else {
        return nil
    }
     
    return allViewControllers[nextIndex]
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1282.html