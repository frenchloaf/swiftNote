# Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）

2016-10-31发布：hangge阅读：3160

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
- [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
- [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
- [当前文章] Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）
- [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
- [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)



前面几篇文章都是介绍如何使用 **MJRefresh**，实现 **tableView** 的上拉加载，下拉刷新的。本文演示在 **CollectionView** 上实现同样的功能，实现的方法也是一样的。（对于修改或者自定义下拉、上拉组件的样式，可以参考我前面的文章，这里就不再说明了。）


**一、下拉刷新**
**1，样例效果**

（1）初始化的时候生成50个随机颜色的方块。

（2）下拉 **collectionView** 即可重新生成数据并刷新。

  [![原文:Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102609383020629.png)](https://www.hangge.com/blog/cache/detail_1410.html#)  [![原文:Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102609384218456.png)](https://www.hangge.com/blog/cache/detail_1410.html#)  [![原文:Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102609384895650.png)](https://www.hangge.com/blog/cache/detail_1410.html#)


**2，样例代码**

```swift
import UIKit
 
class ViewController: UICollectionViewController {
     
    // 顶部刷新
    let header = MJRefreshNormalHeader()
     
    var colors:[UIColor] = []
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
       //随机生成一些初始化数据
        refreshItemData()
         
        //注册CollectionViewCell
        self.collectionView?.register(UICollectionViewCell.self,
                                      forCellWithReuseIdentifier: "DesignViewCell")
        self.collectionView?.backgroundColor = UIColor.white
         
        //下拉刷新相关设置
        header.setRefreshingTarget(self, refreshingAction: #selector(ViewController.headerRefresh))
        self.collectionView!.mj_header = header
    }
     
    //初始化数据
    func refreshItemData() {
        colors = []
        // 增加50条假数据
        for _ in 1...50{
            colors.append(UIColor.randomColor)
        }
    }
     
    //顶部下拉刷新
    @objc func headerRefresh(){
        print("下拉刷新.")
        sleep(2)
        //重现生成数据
        refreshItemData()
        //重现加载表格数据
        self.collectionView!.reloadData()
        //结束刷新
        self.collectionView!.mj_header.endRefreshing()
    }
     
    // CollectionView行数
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return colors.count;
    }
     
    // 获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                                 cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = self.collectionView!.dequeueReusableCell(withReuseIdentifier: identify,
                                                            for: indexPath)
        cell.backgroundColor = colors[indexPath.row]
         
        return cell
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110514260041.zip)



**二、上拉加载**
**1，样例效果**

（1）初始化的时候生成50个随机颜色的方块。

（2）上拉 **collectionView** 便继续新增50个方块并加载进来。

  [![原文:Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/201610260954287951.png)](https://www.hangge.com/blog/cache/detail_1410.html#)  [![原文:Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog_uploads/201610/2016102609543376208.png)](https://www.hangge.com/blog/cache/detail_1410.html#)


**2，样例代码**

```swift
import UIKit
 
class ViewController: UICollectionViewController {
     
    // 底部加载
    let footer = MJRefreshAutoNormalFooter()
     
    var colors:[UIColor] = []
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
       //随机生成一些初始化数据
        loadItemData()
         
        //注册CollectionViewCell
        self.collectionView?.register(UICollectionViewCell.self,
                                      forCellWithReuseIdentifier: "DesignViewCell")
        self.collectionView?.backgroundColor = UIColor.white
         
        //上拉加载相关设置
        footer.setRefreshingTarget(self, refreshingAction: #selector(ViewController.footerLoad))
        self.collectionView!.mj_footer = footer
    }
     
    //初始化数据
    func loadItemData() {
        // 增加50条假数据
        for _ in 1...50{
            colors.append(UIColor.randomColor)
        }
    }
 
    //底部上拉加载
    @objc func footerLoad(){
        print("上拉加载.")
        sleep(2)
        //生成并添加数据
        loadItemData()
        //重现加载表格数据
        self.collectionView!.reloadData()
        //结束刷新
        self.collectionView!.mj_footer.endRefreshing()
    }
     
    // CollectionView行数
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return colors.count;
    }
     
    // 获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                                 cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = self.collectionView!.dequeueReusableCell(withReuseIdentifier: identify,
                                                            for: indexPath)
        cell.backgroundColor = colors[indexPath.row]
         
        return cell
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110525643874.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1410.html