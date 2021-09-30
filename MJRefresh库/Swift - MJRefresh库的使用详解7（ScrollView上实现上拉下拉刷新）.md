# Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）

2016-11-02发布：hangge阅读：2612

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
- [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
- [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
- [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
- [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
- [当前文章] Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）


**MJRefresh** 支持如下控件的刷新：**UITableView**、**UICollectionView**、**UIWebView**、**UIScrollView**。前面三个我在之前的文章已经介绍过了，本文演示 **UIScrollView** 上如何实现下拉刷新、上拉加载更多数据。

**一、下拉刷新**
**1，样例效果**
（1）初始化的时候生成20个随机颜色的色块。
（2）下拉 **scrollView** 即可重新生成数据并刷新。

 [![原文:Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/201610261128275114.png)](https://www.hangge.com/blog/cache/detail_1415.html#) [![原文:Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102611283233632.png)](https://www.hangge.com/blog/cache/detail_1415.html#) [![原文:Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102611283897825.png)](https://www.hangge.com/blog/cache/detail_1415.html#)


**2，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    var scrollView:UIScrollView!
     
    // 顶部刷新
    let header = MJRefreshNormalHeader()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //scrollView初始化
        scrollView = UIScrollView()
        scrollView.frame = self.view.bounds
        self.view.addSubview(scrollView)
         
        //初始化数据
        refreshItemData()
         
        //下拉刷新相关设置
        header.setRefreshingTarget(self, refreshingAction: #selector(ViewController.headerRefresh))
        scrollView.mj_header = header
    }
     
    //初始化数据
    func refreshItemData() {
        //先清空内部元素
        for v in self.scrollView.subviews{
            //如果不是下拉组件，则移除
            if v != header {
                v.removeFromSuperview()
            }
        }
         
        // 增加20条假数据
        for i in 0...19{
            let itemView = UIView(frame: CGRect(x: 0, y: i*60,
                                                width: Int(self.view.bounds.width), height: 60))
            itemView.backgroundColor = UIColor.randomColor
            scrollView.addSubview(itemView)
        }
         
        //设置内容区域尺寸
        scrollView.contentSize = CGSize(width: Int(self.view.bounds.width), height: 20 * 60)
    }
     
    //顶部下拉刷新
    @objc func headerRefresh(){
        print("下拉刷新.")
        sleep(2)
        //重现生成数据
        refreshItemData()
        //结束刷新
        scrollView.mj_header.endRefreshing()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
     
}
 
extension UIColor {
    //返回随机颜色
    open class var randomColor:UIColor{
        get
        {
            let red = CGFloat(arc4random()%256)/255.0
            let green = CGFloat(arc4random()%256)/255.0
            let blue = CGFloat(arc4random()%256)/255.0
            return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
        }
    }
}

```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122111015161948.zip)



**二、上拉加载**
**1，样例效果**
（1）初始化的时候生成10个随机颜色的方块。
（2）上拉 **scrollView** 即可继续新增10个方块并加载进来。

  [![原文:Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102614102913254.png)](https://www.hangge.com/blog/cache/detail_1415.html#)  [![原文:Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102614103529508.png)](https://www.hangge.com/blog/cache/detail_1415.html#)


**2，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    var scrollView:UIScrollView!
     
    //条目数
    var itemCount = 0
     
    // 底部加载
    let footer = MJRefreshAutoNormalFooter()
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //scrollView初始化
        scrollView = UIScrollView()
        scrollView.frame = self.view.bounds
        self.view.addSubview(scrollView)
         
        //初始化数据
        refreshItemData()
         
        //上拉加载相关设置
        footer.setRefreshingTarget(self, refreshingAction: #selector(ViewController.footerLoad))
        scrollView.mj_footer = footer
    }
     
    //初始化数据
    func refreshItemData() {
        // 增加10条假数据
        for _ in 1...10{
            let itemView = UIView(frame: CGRect(x: 0, y: itemCount*60,
                                                width: Int(self.view.bounds.width), height: 60))
            itemView.backgroundColor = UIColor.randomColor
            scrollView.addSubview(itemView)
            itemCount += 1
        }
        //设置内容区域尺寸
        scrollView.contentSize = CGSize(width: Int(self.view.bounds.width), height: itemCount * 60)
    }
     
    //底部上拉加载
    @objc func footerLoad(){
        print("上拉加载.")
        sleep(2)
        //生成并添加数据
        refreshItemData()
        //结束刷新
        scrollView.mj_footer.endRefreshing()
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension UIColor {
    //返回随机颜色
    open class var randomColor:UIColor{
        get
        {
            let red = CGFloat(arc4random()%256)/255.0
            let green = CGFloat(arc4random()%256)/255.0
            let blue = CGFloat(arc4random()%256)/255.0
            return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
        }
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122111025338400.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1415.html