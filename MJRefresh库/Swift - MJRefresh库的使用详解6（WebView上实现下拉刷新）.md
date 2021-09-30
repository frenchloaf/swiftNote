# Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）

2016-11-01发布：hangge阅读：1731

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
- [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
- [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
- [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
- [当前文章] Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）
- [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)



在前面文章中介绍了如何使用使用 **MJRefresh**，实现 **tableView**、**collectionView** 的上拉加载，下拉刷新功能。本文继续演示如何在 **UIWebView** 上实现下拉刷新。


**1，样例效果图**

（1）初始化的时候，**webView** 默认加载 [hangge.com](http://www.hangge.com/) 首页。

（2）下拉 **webView** 即可实现当前页面的刷新。

 [![原文:Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102610223083761.png)](https://www.hangge.com/blog/cache/detail_1414.html#) [![原文:Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102610223777386.png)](https://www.hangge.com/blog/cache/detail_1414.html#) [![原文:Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102610224359673.png)](https://www.hangge.com/blog/cache/detail_1414.html#)


**2，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    var webview:UIWebView!
     
    // 顶部刷新
    let header = MJRefreshNormalHeader()
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建webView并初始化
        let frame = CGRect(x:0, y:20, width:UIScreen.main.bounds.width,
                           height:UIScreen.main.bounds.height)
        self.webview = UIWebView(frame: frame)
        self.webview.scalesPageToFit = true
        self.view.addSubview(self.webview)
         
        //加载页面
        let request = URLRequest(url: URL(string: "http://hangge.com")!)
        self.webview.loadRequest(request)
         
        //下拉刷新相关设置
        header.setRefreshingTarget(self, refreshingAction: #selector(ViewController.headerRefresh))
        self.webview.scrollView.mj_header = header
    }
     
    //顶部下拉刷新
    @objc func headerRefresh(){
        print("下拉刷新.")
        sleep(2)
        //刷新页面
        self.webview.reload()
        //结束刷新
        self.webview.scrollView.mj_header.endRefreshing()
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110592952932.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1414.html