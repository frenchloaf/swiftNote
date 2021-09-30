# Swift - 修改UISearchControlle取消按钮的文字（默认为Cancel）

2016-11-28发布：hangge阅读：1447

搜索控制器（**UISearchController**）在开发中常常会用到，通常我们会将其自带的搜索栏放在页面上方或者表头位置。

[![原文:Swift - 修改UISearchControlle取消按钮的文字（默认为Cancel）](https://www.hangge.com/blog_uploads/201611/2016110815412721305.png)](https://www.hangge.com/blog/cache/detail_1429.html#)



点击进入搜索状态时，会发现右侧的取消按钮文字是英文的 **Cancel**，而且不会根据当前系统的语言环境切换。

[![原文:Swift - 修改UISearchControlle取消按钮的文字（默认为Cancel）](https://www.hangge.com/blog_uploads/201611/2016110815434444098.png)](https://www.hangge.com/blog/cache/detail_1429.html#)



有时我们想要将这个按钮修改成其他文字，比如“搜索”。但 **UISearchController** 不提供直接修改这个按钮文字的方法或属性。我原来也写过一篇关于修改 **UISearchBar** 上 **Cancel** 文字的文章：[Swift - 修改搜索条（UISearchBar）中取消按钮的文字、颜色](https://www.hangge.com/blog/cache/detail_1126.html)。但原来那个方法不适用于 **UISearchController**。

### 1，修改Cancel按钮文字标题

我们可以通过 **searchBar** 的 **setValue** 方法来设置默认文字。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    var searchController:UISearchController!
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //配置搜索控制器
        self.searchController = ({
            let controller = UISearchController(searchResultsController: nil)
            //controller.searchResultsUpdater = self
            controller.hidesNavigationBarDuringPresentation = true
            controller.dimsBackgroundDuringPresentation = false
            controller.searchBar.searchBarStyle = .minimal
            controller.searchBar.sizeToFit()
            return controller
        })()
         
        //将搜索栏添加到页面上
        let searchBarFrame = CGRect(x: 0, y: 20, width: self.view.frame.width, height: 44)
        let searchBarContainer = UIView(frame: searchBarFrame)
        searchBarContainer.addSubview(searchController.searchBar)
        self.view.addSubview(searchBarContainer)
         
        //搜索栏取消按钮文字
        searchController.searchBar.setValue("搜索", forKey:"_cancelButtonText")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 2，运行效果

[![原文:Swift - 修改UISearchControlle取消按钮的文字（默认为Cancel）](https://www.hangge.com/blog_uploads/201611/2016110815495149307.png)](https://www.hangge.com/blog/cache/detail_1429.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1429.html