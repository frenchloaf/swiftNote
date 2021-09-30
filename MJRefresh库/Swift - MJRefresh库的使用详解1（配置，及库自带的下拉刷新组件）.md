# Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）

2016-10-24发布：hangge阅读：9056

> **相关文章系列**（代码均已升级至Swift4）：
>
> - [当前文章] Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）
> - [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
> - [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
> - [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
> - [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
> - [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
> - [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)



“下拉刷新”、“上拉加载更多”在项目中经常会使用到。其中使用自带的 **UIRefreshControl** 就可以很方便地实现下拉刷新功能。我原来也写过几篇相关的文章：

> [Swift - 下拉刷新数据的功能实现（使用UIRefreshControl）](https://www.hangge.com/blog/cache/detail_934.html)
> [Swift - UIRefreshControl下拉时，刷新时分别使用不同的描述文字](https://www.hangge.com/blog/cache/detail_935.html)
> [Swfit - 使用自定义的UIRefreshControl下拉刷新界面](https://www.hangge.com/blog/cache/detail_936.html)

除了使用 **UIRefreshControl**，网上也有许多第三方刷新库可供选择。**MJRefresh** 是其中比较优秀的一个。

## 一、MJRefresh介绍

- **MJRefresh** 是一个使用 **Objective-C** 写的刷新库，使用简单。
- **MJRefresh** 既可以实现下拉刷新，也能实现上拉加载。
- 支持如下控件的刷新：**UIScrollView**、**UITableView**、**UICollectionView**、**UIWebView**。
- **GitHub** 主页地址：https://github.com/CoderMJLee/MJRefresh

**MJRefresh** 类结构图如下：

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102210342315030.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

## 二、MJRefresh的使用

### 1，安装配置

（1）首先将 **MJRefresh** 库下载到本地，将其中的 **MJRefresh** 文件夹添加到项目中来。

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102114574164682.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

（2）由于 **MJRefresh** 是使用 **OC** 写的，所以我们还需要创建一个桥接头文件 **bridge.h**，并在项目中配置。其内容如下：

```
#``import` `"MJRefresh.h"
```



### 2，使用样例

下面给 **tableView** 添加一个下拉刷新功能，每次下拉会随机生成10条数据，并刷新表格。（生成随机数据的时候会等待2秒，模拟网络请求）。具体效果图如下：  

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115080893505.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115081989616.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115082683248.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

（1）对于下拉的响应事件，我们可以通过设置其 **target action** 来关联。样例完整代码如下：

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String]!
    var tableView:UITableView?
     
    // 顶部刷新
    let header = MJRefreshNormalHeader()
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        refreshItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //下拉刷新相关设置
        header.setRefreshingTarget(self, refreshingAction: #selector(ViewController.headerRefresh))
        self.tableView!.mj_header = header
    }
     
    //初始化数据
    func refreshItemData() {
        items = []
        for _ in 0...9 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //顶部下拉刷新
    @objc func headerRefresh(){
        print("下拉刷新.")
        sleep(2)
        //重现生成数据
        refreshItemData()
        //重现加载表格数据
        self.tableView!.reloadData()
        //结束刷新
        self.tableView!.mj_header.endRefreshing()
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122109592385160.zip)

（2）下拉响应方法也可以直接写在闭包（**Block**）中。具体区别见下方代码：

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String]!
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        refreshItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
     
 
        //下拉刷新相关设置,使用闭包Block
        self.tableView!.mj_header = MJRefreshNormalHeader(refreshingBlock: {
            print("下拉刷新.")
            sleep(2)
            //重现生成数据
            self.refreshItemData()
            //重现加载表格数据
            self.tableView!.reloadData()
            //结束刷新
            self.tableView!.mj_header.endRefreshing()
        })
    }
     
    //初始化数据
    func refreshItemData() {
        items = []
        for _ in 0...9 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

## 三、MJRefresh下拉样式的修改

### 1，默认样式

上面的样例使用的就是默认样式。会显示刷新的状态提示文字，刷新时间，左侧还有箭头或环形进度条表示刷新状态。
[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115080893505.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115081989616.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102115082683248.png)](https://www.hangge.com/blog/cache/detail_1406.html#)



### 2，隐藏时间

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116130741735.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116141771480.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116153148554.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

```
//隐藏时间
header.lastUpdatedTimeLabel.isHidden = true
```



### 3，隐藏所有的文字

下面把时间和状态文字都给隐藏掉。

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116224142968.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116224717073.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116225261730.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

```
//隐藏时间
header.lastUpdatedTimeLabel.isHidden = true
//隐藏状态
header.stateLabel.isHidden = true
```



### 4，自定义文字和文字样式  

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116413582386.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102116414197916.png)](https://www.hangge.com/blog/cache/detail_1406.html#)[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/201610211641467921.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

```swift
//修改文字
header.setTitle("下拉下拉下拉", for: .idle)
header.setTitle("松开松开松开", for: .pulling)
header.setTitle("刷新刷新刷新", for: .refreshing)
         
//修改字体
header.stateLabel.font = UIFont.systemFont(ofSize: 15)
header.lastUpdatedTimeLabel.font = UIFont.systemFont(ofSize: 13)
         
//修改文字颜色
header.stateLabel.textColor = UIColor.red
header.lastUpdatedTimeLabel.textColor = UIColor.blue
```



### 5，自定义图标

下拉刷新的图标是可以修改的。不同的状态，我们都可以设置一个图片数组，**MJRefresh** 就会自动播放这几张图片，形成动画。

其中下拉过程中的图片是根据下拉的距离自动改变。而提示松开刷新，以及正在刷新这两个状态下的图片是定时切换播放的。
（注意：如果要设置图标，**header** 就要使用 **MJRefreshGifHeader**,而不是 **MJRefreshNormalHeader**。）

[![原文:Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102320531341415.png)](https://www.hangge.com/blog/cache/detail_1406.html#)

```swift
//下拉过程时的图片集合(根据下拉距离自动改变)
var idleImages = [UIImage]()
for i in 1...10 {
    idleImages.append(UIImage(named:"idle\(i)")!)
}
header.setImages(idleImages, for: .idle) //idle1，idle2，idle3...idle10
 
//下拉到一定距离后，提示松开刷新的图片集合(定时自动改变)
var pullingImages = [UIImage]()
for i in 1...3 {
    pullingImages.append(UIImage(named:"pulling\(i)")!)
}
header.setImages(pullingImages, for: .pulling)
 
//刷新状态下的图片集合(定时自动改变)
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
header.setImages(refreshingImages, for: .refreshing)
```


动画图片切换的时间也是可以修改的：

```swift
//下面表示刷新图片在1秒钟的时间内播放一轮
header.setImages(refreshingImages, duration: 1, for: .refreshing)
```



## 四、通过代码调用下拉刷新

上面的样例中，我们都是通过下拉操作来实现 **MJRefresh** 组件的刷新功能。其实也可以通过调用组件的刷新方法来实现同样功能（这个同样会有下拉、收回等动画效果）

```swift
//手动调用刷新效果
header.beginRefreshing()
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1406.html