# Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）

2016-07-21发布：hangge阅读：3289

在之前的文章：[Swift - 页视图控制器（UIPageViewController）的使用1（基本用法）](https://www.hangge.com/blog/cache/detail_1282.html)。介绍了如何使用 **UIPageViewController** 来加载管理多个视图控制器，以及页面切换效果的设置。

本文中我们对之前的样例做个功能改进。就是在页面下方添加页控件（**UIPageControl**）,这样用户可以根据分页小圆点知道当前处在哪个位置。

**1，样例说明**
用户可以根据小白点知道总页数，以及当前页的位置。翻页时圆点的状态也会随之改变。
[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071816343623653.png)](https://www.hangge.com/blog/cache/detail_1283.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071816344135067.png)](https://www.hangge.com/blog/cache/detail_1283.html#)[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071816344872540.png)](https://www.hangge.com/blog/cache/detail_1283.html#)

**2，实现步骤**

（1）要添加 **pageControl** 我们就不能将 **UIPageViewController** 作为初始化视图控制器了。而是仍然使用 **UIViewController** 作为初始化视图控制器。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071816035047941.png)](https://www.hangge.com/blog/cache/detail_1283.html#)


（2）然后往 **viewController** 中加入一个 **ContainerView**，连接 **ContainerView** 和 **pageViewController**（使用 **embed**）

[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071815471345192.png)](https://www.hangge.com/blog/cache/detail_1283.html#)


（3）再在 **UIViewController** 中添加一个**UIPageControl**。

[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071816023226952.png)](https://www.hangge.com/blog/cache/detail_1283.html#)


（4）最后 **StoryBoard** 中所有控制器结构如下（**UIPageViewController** 中加载的3个子视图控制器都还是和前面样例一样 ）

[![原文:Swift - 页视图控制器（UIPageViewController）的使用2（添加分页圆点）](https://www.hangge.com/blog_uploads/201607/2016071815475832315.png)](https://www.hangge.com/blog/cache/detail_1283.html#)



**3，页面代码**
（1）**PageViewController.swift**
注意：我这里自定义了个协议（**PageViewControllerDelegate**），这样当 **pageViewController** 里的视图页面切换时，**viewController** 那边能够触发对应方法，从而改变页控件的状态。

```swift
import UIKit
 
class PageViewController: UIPageViewController, UIPageViewControllerDataSource,
        UIPageViewControllerDelegate {
     
    //页面改变的委托
    weak var pageDelegate: PageViewControllerDelegate?
     
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
 
        //自身委托设置
        delegate = self
         
        //设置首页
        if let firstViewController = allViewControllers.first {
            setViewControllers([firstViewController],
                               direction: .Forward,
                               animated: true,
                               completion: nil)
        }
         
        //页面数量改变，通知委托对象
        pageDelegate?.pageViewController(self, didUpdatePageCount: allViewControllers.count)
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
     
     
    //页面切换完毕
    func pageViewController(pageViewController: UIPageViewController,
                            didFinishAnimating finished: Bool,
                                               previousViewControllers: [UIViewController],
                                               transitionCompleted completed: Bool) {
        if let firstViewController = viewControllers?.first,
            let index = allViewControllers.indexOf(firstViewController) {
            //当前页改变，通知委托对象
            pageDelegate?.pageViewController(self, didUpdatePageIndex: index)
        }
    }
}
 
//自定义视图控制器代理协议
protocol PageViewControllerDelegate: class {
     
    //当页面数量改变时调用
    func pageViewController(pageViewController: PageViewController,
                                    didUpdatePageCount count: Int)
     
    //当前页索引改变时调用
    func pageViewController(pageViewController: PageViewController,
                                    didUpdatePageIndex index: Int)
}
```


（2）**ViewController.swift**

```swift
import UIKit
 
class ViewController: UIViewController, PageViewControllerDelegate {
 
    //页控件
    @IBOutlet weak var pageControl: UIPageControl!
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //场景切换
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if let pageViewController = segue.destinationViewController as? PageViewController {
            //设置委托（当页面数量、索引改变时当前视图控制器能触发页控件的改变）
            pageViewController.pageDelegate = self
        }
    }
     
    //当页面数量改变时调用
    func pageViewController(pageViewController: PageViewController,
                                    didUpdatePageCount count: Int) {
        pageControl.numberOfPages = count
    }
     
    //当前页索引改变时调用
    func pageViewController(pageViewController: PageViewController,
                                    didUpdatePageIndex index: Int) {
        pageControl.currentPage = index
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载：**![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1283.zip](https://www.hangge.com/blog_uploads/201607/2016071816450988508.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1283.html