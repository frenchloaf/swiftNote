# Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）

2016-10-25发布：hangge阅读：2598

**相关文章系列**（代码均已升级至Swift4）：

- [Swift - MJRefresh库的使用详解1（配置，及库自带的下拉刷新组件）](https://www.hangge.com/blog/cache/detail_1406.html)
- [当前文章] Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）
- [Swift - MJRefresh库的使用详解3（库自带的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1407.html)
- [Swift - MJRefresh库的使用详解4（创建自定义的上拉加载组件）](https://www.hangge.com/blog/cache/detail_1409.html)
- [Swift - MJRefresh库的使用详解5（CollectionView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1410.html)
- [Swift - MJRefresh库的使用详解6（WebView上实现下拉刷新）](https://www.hangge.com/blog/cache/detail_1414.html)
- [Swift - MJRefresh库的使用详解7（ScrollView上实现上拉下拉刷新）](https://www.hangge.com/blog/cache/detail_1415.html)

在上文中我介绍了 **MJRefresh** 自带的两个下拉刷新实现类：**MJRefreshNormalHeader** 和 **MJRefreshGifHeader**。以及如何修改上面的文字或图标。通常情况下，两个组件已经可以满足我们大部分需求。
如果我们需要实现一个个性化的下拉组件，比如在上面添加些其他组件，或者改变原来的布局。那么可以通过继承 **MJRefreshHeader** 来实现一个自定义的下拉组件。

**1，样例效果图**

（1）这个原来是 **MJRefresh** 提供的一个样例，我这里改成使用 **Swift** 实现。

（2）下拉组件视图中放置4个控件：上方一个 **UIImageView**，下方从左到右是 **UISwitch**、**UILabel**、**UIActivityIndicatorView**。

（3）其中文本标签的文字颜色会随着下拉的距离，从红色渐变到蓝色。

（4）“下拉”状态下开关是关闭的，到了“松开刷新”、“正在刷新”这两个状态下开关自动变成打开。
（5）**UIActivityIndicatorView** 活动指示器只有在“正在刷新”状态下会显示出来。

   [![原文:Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102509123599907.png)](https://www.hangge.com/blog/cache/detail_1408.html#)    [![原文:Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102509124240781.png)](https://www.hangge.com/blog/cache/detail_1408.html#)

   [![原文:Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/2016102509125140225.png)](https://www.hangge.com/blog/cache/detail_1408.html#)   [![原文:Swift - MJRefresh库的使用详解2（创建自定义的下拉刷新组件）](https://www.hangge.com/blog_uploads/201610/201610250912563710.png)](https://www.hangge.com/blog/cache/detail_1408.html#)

**2，自定义组件实现**

```swift
class MJDIYHeader: MJRefreshHeader {
 
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
         
        //self.logo.backgroundColor = UIColor.red
        self.logo.bounds = CGRect(x:0, y:0, width:self.bounds.size.width, height:50)
        self.logo.center = CGPoint(x:self.mj_w * 0.5, y:-30)
         
        self.loading.center = CGPoint(x:self.mj_w - 30, y:self.mj_h * 0.5)
    }
     
    //监听控件的刷新状态
    override var state: MJRefreshState{
        didSet
        {
            switch (state) {
            case .idle:
                self.loading.stopAnimating()
                self.s.setOn(false, animated: true)
                self.label.text = "赶紧下拉吖(开关是打酱油滴)"
                break
            case .pulling:
                self.loading.stopAnimating()
                self.s.setOn(true, animated: true)
                self.label.text = "赶紧放开我吧(开关是打酱油滴)"
                break
            case .refreshing:
                self.s.setOn(true, animated: true)
                self.label.text = "加载数据中(开关是打酱油滴)"
                self.loading.startAnimating()
                break
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

原文出自：www.hangge.com  转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1408.html
```


**3，组件使用样例**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String]!
    var tableView:UITableView?
     
    // 顶部刷新
    let header = MJDIYHeader()
     
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

原文出自：www.hangge.com  转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1408.html
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1406.zip](https://www.hangge.com/blog_uploads/201610/2016102509173430409.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1408.html