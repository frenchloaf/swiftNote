# Swift - 表格tableView上拉加载新数据的功能实现（附样例）

2016-08-12发布：hangge阅读：6463

（本文代码已升级至Swift4）

对于表格（**tableView**）来说，下拉刷新数据、上拉加载数据应该是两个最常用的数据更新操作了。对于前者，我原来写过一篇相关的文章：[Swift - 下拉刷新数据的功能实现（使用UIRefreshControl）](https://www.hangge.com/blog/cache/detail_934.html)。本次我来讲讲后者的实现。

说是上拉加载数据，其实就是当我们将表格内容滚动到最后一行时，系统就会自动获取新的内容并添加到列表尾部（具体效果可以参考百度贴吧的**App**）。下面我们通过一个小样例来演示上拉加载的实现。

**1，样例效果图**
（1）当初次进入程序时，先加载前**20**条数据。
（2）当 **tableView** 滚动条移到底部时，会看到表格底部有正在加载的提示，并继续加载接下来的**20**条数据。
（3）数据加载完毕后，会把新数据添加到表格底部。
（4）如果表格再次滚到底部，则循环上面的操作，继续加载新数据。

[![原文:Swift - 表格tableView上拉加载新数据的功能实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016081110583310405.png)](https://www.hangge.com/blog/cache/detail_1254.html#)[![原文:Swift - 表格tableView上拉加载新数据的功能实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016081110584083522.png)](https://www.hangge.com/blog/cache/detail_1254.html#)[![原文:Swift - 表格tableView上拉加载新数据的功能实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016081110584661715.png)](https://www.hangge.com/blog/cache/detail_1254.html#)

**2，实现原理**
（1）上拉加载的关键点在于判断何时开始加载新数据。这里我们在最后一条数据项显示出来的时候就自动开始加载。也就是当用户上拉表格，等到看到最后一条数据的时候就加载。
（2）同时我们在表格的 **tableFooterView** 中添加一个提示视图，用来告诉用户数据正在加载中。
（3）每次收到数据后，就将新数据并到本地的数据集合中，再调用 **tableView的reloadData()** 方法重新刷新下表格数据。
（4）这里要注意每次发起请求的时候要做个保护，保证在原来的请求没有返回时就不会发起新的请求。防止用户在最后一页来回滚动造成多次请求。

（5）实际的项目这些数据是通过网络请求的。这里就直接本地生成，并使用计时器加个延时，可以更好地看到效果。

**3，完整代码**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    //表格视图
    @IBOutlet weak var tableView: UITableView!
     
    //数据集合
    var dataArray = [String]()
     
    //表格底部用来提示数据加载的视图
    var loadMoreView:UIView?
     
    //计数器（用来做延时模拟网络加载效果）
    var timer: Timer!
     
    //用了记录当前是否允许加载新数据（正则加载的时候会将其设为false，放置重复加载）
    var loadMoreEnable = true
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.tableView.delegate = self
        self.tableView.dataSource = self
         
        //上拉刷新
        self.setupInfiniteScrollingView()
        self.tableView.tableFooterView = self.loadMoreView
         
        //首次加载数据
        loadMore()
    }
     
    //上拉刷新视图
    private func setupInfiniteScrollingView() {
        self.loadMoreView = UIView(frame: CGRect(x:0, y:self.tableView.contentSize.height,
                                                 width:self.tableView.bounds.size.width, height:60))
        self.loadMoreView!.autoresizingMask = UIViewAutoresizing.flexibleWidth
        self.loadMoreView!.backgroundColor = UIColor.orange
         
        //添加中间的环形进度条
        let activityViewIndicator = UIActivityIndicatorView(activityIndicatorStyle: .white)
        activityViewIndicator.color = UIColor.darkGray
        let indicatorX = self.view.frame.width/2-activityViewIndicator.frame.width/2
        let indicatorY = self.loadMoreView!.frame.size.height/2-activityViewIndicator.frame.height/2
        activityViewIndicator.frame = CGRect(x:indicatorX, y:indicatorY,
                                             width:activityViewIndicator.frame.width,
                                             height:activityViewIndicator.frame.height)
        activityViewIndicator.startAnimating()
        self.loadMoreView!.addSubview(activityViewIndicator)
    }
     
    // 返回记录数
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataArray.count;
    }
     
    //单元格数据设置
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //设置单元格数据
            let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
            cell.textLabel?.text = self.dataArray[indexPath.row]
             
            //当下拉到底部，执行loadMore()
            if (loadMoreEnable && indexPath.row == self.dataArray.count-1) {
                loadMore()
            }
             
            return cell
    }
     
    //加载更多数据
    func loadMore(){
        print("加载新数据！")
        loadMoreEnable = false
        timer = Timer.scheduledTimer(timeInterval: 2.0, target: self,
                selector: #selector(ViewController.timeOut), userInfo: nil, repeats: true)
    }
     
    //计时器时间到
    @objc func timeOut() {
        let start = self.dataArray.count + 1
        //随机添加10条新数据（时间是当前时间）
        for i in start..<start+20 {
            self.dataArray.append("新闻标题\(i)")
        }
        self.tableView.reloadData()
        loadMoreEnable = true
         
        timer.invalidate()
        timer = nil
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin0822/include/edit/sysimage/icon16/zip.gif)[hangge_1254.zip](https://www.hangge.com/blog_uploads/201809/2018092111161311562.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1254.html