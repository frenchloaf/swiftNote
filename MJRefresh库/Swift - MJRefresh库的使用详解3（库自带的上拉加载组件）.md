# Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）

2016-10-27发布：hangge阅读：6023

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
- [当前文章] Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）
- [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
- [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
- [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
- [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)

前面介绍了使用 **MJRefresh** 来实现下拉刷新功能。本文接着演示上拉加载更多数据的实现。

**一、上拉加载组件的使用**

**1，样例效果**

（1）初始化的时候自动生成20条数据用于表格默认显示。

（2）当点击最底部的“点击或上拉加载更多”，或者在列表底部继续上拉就会添加20条新数据进来（随机生成，生成数据的时候会等待2秒，模拟网络请求） 。

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102413512183287.png)](https://www.hangge.com/blog/cache/detail_1407.html#)[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102413515959679.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

**2，样例代码**

（1）对于上拉加载的响应事件，我们可以通过设置其 **target action** 来关联。样例完整代码如下：

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    // 底部加载
    let footer = MJRefreshAutoNormalFooter()
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        loadItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //上刷新相关设置
        footer.setRefreshingTarget(self, refreshingAction: #selector(ViewController.footerLoad))
        //是否自动加载（默认为true，即表格滑到底部就自动加载）
        footer.isAutomaticallyRefresh = false
        self.tableView!.mj_footer = footer
    }
     
    //初始化数据
    func loadItemData() {
        for _ in 0...20 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //底部上拉加载
    @objc func footerLoad(){
        print("上拉加载.")
        sleep(2)
        //生成并添加数据
        loadItemData()
        //重现加载表格数据
        self.tableView!.reloadData()
        //结束刷新
        self.tableView!.mj_footer.endRefreshing()
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

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110092860419.zip)

（2）上拉响应方法也可以直接写在闭包（**Block**）中。具体区别见下方代码：

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        loadItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //上拉加载相关设置,使用闭包Block
        self.tableView!.mj_footer = MJRefreshAutoNormalFooter(refreshingBlock: {
            print("上拉加载.")
            sleep(2)
            //生成并添加数据
            self.loadItemData()
            //重现加载表格数据
            self.tableView!.reloadData()
            //结束刷新
            self.tableView!.mj_footer.endRefreshing()
        })
    }
     
    //初始化数据
    func loadItemData() {
        for _ in 0...20 {
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


**二、MJRefresh上拉样式的修改**

**1，默认样式**

上面的样例使用的就是默认样式。会显示提示文字，刷新时左侧还有环形进度条。

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102414171066386.png)](https://www.hangge.com/blog/cache/detail_1407.html#)[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102414172624532.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

**2，自定义刷新图标**

同下拉组件里的刷新图标一样。上拉加载里的刷新图标我们也可以修改。通过设置一个图片数组，**MJRefresh** 就会自动播放这几张图片，形成动画。

（注意：如果要设置图标，**footer** 就要使用 **MJRefreshAutoGifFooter**，而不是 **MJRefreshNormalFooter**。）

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102414440048328.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
// 底部加载
let footer = MJRefreshAutoGifFooter()
 
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
footer.setImages(refreshingImages, for: .refreshing)
```


动画图片切换的时间也是可以修改的：

```swift
//下面表示刷新图片在1秒钟的时间内播放一轮
footer.setImages(refreshingImages, duration: 1, for: .refreshing)
```


**3，刷新状态下隐藏文字，只显示图标**

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102414522733829.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
//刷新时不显示文字（其它情况下还是有提示文字的）
footer.isRefreshingTitleHidden = true
```


**4，将状态修改成“全部加载完毕”**
如果请求后发现所有的数据都已加载完毕。可以调用组件 **endRefreshingWithNoMoreData()** 方法，表示没有更多的数据可以加载了。其相关的提示文字也会改变。

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102414590973845.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
self.tableView!.mj_footer.endRefreshingWithNoMoreData()
```

设置为“全部加载”状态后，怎么上拉都不会触发加载事件。如果想恢复上拉加载功能，可以使用 **resetNoMoreData()** 方法进行重置。

```swift
self.tableView!.mj_footer.resetNoMoreData()
```


**5，自定义文字和文字样式**

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102509551470791.png)](https://www.hangge.com/blog/cache/detail_1407.html#)[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102509544558923.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
//修改文字
footer.setTitle("上拉上拉上拉", for: .idle)
footer.setTitle("加载加载加载", for: .refreshing)
footer.setTitle("没有没有更多数据了", for: .noMoreData)
 
//修改字体
footer.stateLabel.font = UIFont.systemFont(ofSize: 15)
 
//修改文字颜色
footer.stateLabel.textColor = UIColor.red
```


**6，隐藏上拉加载组件**
当然隐藏后上拉加载的功能也失效了。

```swift
self.tableView!.mj_footer.isHidden = true
```


**7，使用自动回弹的上拉加载组件**
前面介绍的都是普通上拉组件，即默认会占用表格最后一行的空间。而使用 **MJRefreshBackNormalFooter**，单元格空间不会被占用。只有在最后一行位置上拉时，才回显示出上拉加载组件。具体效果类似于下拉刷新组件。

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102510315983519.png)](https://www.hangge.com/blog/cache/detail_1407.html#)[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102510320996716.png)](https://www.hangge.com/blog/cache/detail_1407.html#)[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102510321767718.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
//自动回弹的上拉加载组件
let footer = MJRefreshBackNormalFooter()
```


**8，自定义自动回弹的上拉加载组件的刷新图标**
同样地。对于自动回弹上拉组件不同状态下的图片数组也是可以修改设置的。如果要设置图标，我们可以改用 **MJRefreshBackGifFooter** 即可。

[![原文:Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102510444016148.png)](https://www.hangge.com/blog/cache/detail_1407.html#)

```swift
let footer = MJRefreshBackGifFooter()
 
//上拉过程时的图片集合(根据下拉距离自动改变)
var idleImages = [UIImage]()
for i in 1...10 {
    idleImages.append(UIImage(named:"idle\(i)")!)
}
footer.setImages(idleImages, for: .idle) //idle1，idle2，idle3...idle10
 
//上拉到一定距离后，提示松开刷新的图片集合(定时自动改变)
var pullingImages = [UIImage]()
for i in 1...3 {
    pullingImages.append(UIImage(named:"pulling\(i)")!)
}
footer.setImages(pullingImages, for: .pulling)
 
//刷新状态下的图片集合(定时自动改变)
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
footer.setImages(refreshingImages, for: .refreshing)
```


**9、通过代码调用自动回弹的上拉组件的加载行为**
前面的样例中，我们都是通过上拉操作来实现 **MJRefreshBackNormalFooter** 组件的上拉加载功能。其实也可以通过调用组件的刷新方法来实现同样功能（这个同样会有上拉、收回等动画效果）

```
//手动调用刷新效果``footer.beginRefreshing()
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1407.html