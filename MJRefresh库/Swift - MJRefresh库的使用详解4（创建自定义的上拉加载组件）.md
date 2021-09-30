# Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）

2016-10-28发布：hangge阅读：1985

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1408.html)
- [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
- [当前文章] Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）
- [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
- [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
- [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)

在上文中我介绍了 **MJRefresh** 自带的四个上拉加载实现类：普通自动刷新上拉加载组件（**MJRefreshAutoNormalFooter**、**MJRefreshAutoGifFooter**），自动回弹上拉加载组件（**MJRefreshBackNormalFooter**、**MJRefreshBackGifFooter**）。以及如何修改上面的文字或图标。通常情况下，这四个组件已经可以满足我们大部分需求。

但如果我们需要实现一个个性化的上拉组件，比如在上面添加些其他组件，或者改变原来的布局。那么可以通过继承 **MJRefreshAutoFooter** 或者 **MJRefreshBackFooter** 来实现一个自定义的上拉组件。

**一、继承MJRefreshAutoFooter**

这个实现的是普通自动刷新上拉加载组件，也就是说上拉组件会占用 **tableView** 的一个单元格。
**
**

**1，样例效果图**
（1）这个原来是 **MJRefresh** 提供的一个样例，我这里改成使用 **Swift** 实现。

（2）上拉组件视图中放置3个控件：**UIActivityIndicatorView**、**UILabel**、**UISwitch**。

（3）通常状态下开关是关闭的，到了“正在刷新”状态下开关会自动变成打开状态。
（4）**UIActivityIndicatorView** 活动指示器只有在“正在刷新”状态下会显示出来。

[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102511233952612.png)](https://www.hangge.com/blog/cache/detail_1409.html#)[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102511234555132.png)](https://www.hangge.com/blog/cache/detail_1409.html#)

***\*
\**2，自定义组件实现**

```swift
class MJDIYAutoFooter: MJRefreshAutoFooter
{
    var label:UILabel!
    var s:UISwitch!
    var loading:UIActivityIndicatorView!
     
    //在这里做一些初始化配置（比如添加子控件）
    override func prepare()
    {
        super.prepare()
         
        // 设置控件的高度
        self.mj_h = 50
         
        // 添加label
        self.label =  UILabel()
        self.label.textColor = UIColor(red:1.0, green:0.5, blue:0.0, alpha:1.0)
        self.label.font = UIFont.boldSystemFont(ofSize: 16)
        self.label.textAlignment = .center
        self.addSubview(self.label)
 
        // 打酱油的开关
        self.s =  UISwitch()
        self.addSubview(self.s)
 
        // loading
        self.loading =  UIActivityIndicatorView(activityIndicatorStyle: .gray)
        self.addSubview(self.loading)
    }
     
    //在这里设置子控件的位置和尺寸
    override func placeSubviews()
    {
        super.placeSubviews()
        self.label.frame = self.bounds
        self.s.center = CGPoint(x:self.mj_w - 20, y:self.mj_h - 20)
        self.loading.center = CGPoint(x:30, y:self.mj_h * 0.5)
    }
     
    //监听控件的刷新状态
    override var state: MJRefreshState{
        didSet
        {
            switch (state) {
            case .idle:
                self.label.text = "赶紧上拉吖(开关是打酱油滴)"
                self.loading.stopAnimating()
                self.s.setOn(false, animated:true)
                break
            case .refreshing:
                self.s.setOn(true, animated:true)
                self.label.text = "加载数据中(开关是打酱油滴)"
                self.loading.startAnimating()
                break
            case .noMoreData:
                self.label.text = "木有数据了(开关是打酱油滴)"
                self.s.setOn(false, animated:true)
                self.loading.stopAnimating()
                break
            default:
                break
            }
        }
    }
 
    //监听scrollView的contentOffset改变
    override func scrollViewContentOffsetDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentOffsetDidChange(change)
    }
     
    //监听scrollView的contentSize改变
    override func scrollViewContentSizeDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentSizeDidChange(change)
    }
     
    //监听scrollView的拖拽状态改变
    override func scrollViewPanStateDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewPanStateDidChange(change)
    }
}
```


**3，组件使用样例**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    // 底部加载
    let footer = MJDIYAutoFooter()
     
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110175544865.zip)

**二、继承MJRefreshBackFooter** 

这个实现的是自动回弹上拉加载组件，也就是说上拉组件不会占用 **tableView** 单元格空间。

**1，样例效果图**

（1）这个原来也是 **MJRefresh** 提供的一个样例，我这里改成使用 **Swift** 实现。

（2）下拉组件视图中放置4个控件：最下方一个 **UIImageView**，上方从左到右是 **UISwitch**、**UILabel**、**UIActivityIndicatorView**。

（3）其中文本标签的文字颜色会随着上拉的距离，从红色渐变到蓝色。

（4）“上拉”状态下开关是关闭的，到了“松开加载”、“正在加载”这两个状态下开关会自动变成打开状态。

（5）**UIActivityIndicatorView** 活动指示器只有在“正在加载”状态下会显示出来。

[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/201610251405221235.png)](https://www.hangge.com/blog/cache/detail_1409.html#)[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102514053064742.png)](https://www.hangge.com/blog/cache/detail_1409.html#)

[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102514053629922.png)](https://www.hangge.com/blog/cache/detail_1409.html#)[![原文:Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog_uploads/201610/2016102514054289293.png)](https://www.hangge.com/blog/cache/detail_1409.html#)

**
2，自定义组件实现**

```swift
class MJDIYBackFooter: MJRefreshBackFooter
{
    var label:UILabel!
    var s:UISwitch!
    var logo:UIImageView!
    var loading:UIActivityIndicatorView!
     
    //在这里做一些初始化配置（比如添加子控件）
    override func prepare()
    {
        super.prepare()
         
        // 设置控件的高度
        self.mj_h = 50
         
        // 添加label
        self.label =  UILabel()
        self.label.textColor = UIColor(red:1.0, green:0.5, blue:0.0, alpha:1.0)
        self.label.font = UIFont.boldSystemFont(ofSize: 16)
        self.label.textAlignment = .center
        self.addSubview(self.label)
         
        // 打酱油的开关
        self.s =  UISwitch()
        self.addSubview(self.s)
         
        // logo
        self.logo =  UIImageView(image:UIImage(named:"logo"))
        self.logo.contentMode = .scaleAspectFit
        self.addSubview(self.logo)
         
        // loading
        self.loading =  UIActivityIndicatorView(activityIndicatorStyle: .gray)
        self.addSubview(self.loading)
    }
     
    //在这里设置子控件的位置和尺寸
    override func placeSubviews()
    {
        super.placeSubviews()
         
        self.label.frame = self.bounds
        self.logo.bounds = CGRect(x:0, y:0, width:self.bounds.size.width, height:50)
        self.logo.center = CGPoint(x:self.mj_w * 0.5, y:self.mj_h + self.logo.mj_h * 0.5)
        self.loading.center = CGPoint(x:self.mj_w - 30, y:self.mj_h * 0.5)
    }
     
    //监听控件的刷新状态
    override var state: MJRefreshState{
        didSet
        {
            switch (state) {
            case .idle:
                self.loading.stopAnimating()
                self.s.setOn(false, animated:true)
                self.label.text = "赶紧上拉吖(开关是打酱油滴)"
                break
            case .pulling:
                self.loading.stopAnimating()
                self.s.setOn(true, animated:true)
                self.label.text = "赶紧放开我吧(开关是打酱油滴)"
                break
            case .refreshing:
                self.loading.startAnimating()
                self.s.setOn(true, animated:true)
                self.label.text = "加载数据中(开关是打酱油滴)"
                break
            case .noMoreData:
                self.loading.stopAnimating()
                self.label.text = "木有数据了(开关是打酱油滴)"
                self.s.setOn(false, animated:true)
            default:
                break
            }
        }
    }
     
    //监听拖拽比例（控件被拖出来的比例）
    override var pullingPercent: CGFloat {
        didSet
        {
            // 1.0 0.5 0.0
            // 0.5 0.0 0.5
            let  red =  1.0 - pullingPercent * 0.5
            let green =  0.5 - 0.5 * pullingPercent
            let blue =  0.5 * pullingPercent
            self.label.textColor = UIColor(red:red, green:green, blue:blue, alpha:1.0)
        }
    }
     
    //监听scrollView的contentOffset改变
    override func scrollViewContentOffsetDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentOffsetDidChange(change)
    }
     
    //监听scrollView的contentSize改变
    override func scrollViewContentSizeDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentSizeDidChange(change)
    }
     
    //监听scrollView的拖拽状态改变
    override func scrollViewPanStateDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewPanStateDidChange(change)
    }
}
```


**3，组件使用样例**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    // 底部加载
    let footer = MJDIYBackFooter()
     
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201712/2017122110200448929.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1409.html