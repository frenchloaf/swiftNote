# Swift - 当存在导航栏时，scrollview自动下移的问题解决

2017-01-20发布：hangge阅读：3715

### 1，automaticallyAdjustsScrollViewInsets属性介绍 

- **automaticallyAdjustsScrollViewInsets** 是 **iOS7** 后新增的属性，其默认值是 **true**。
- 当其为 **true** 时。控制器会根据所在界面的 **statusbar**、**navigationbar**、**tabbar** 的高度，自动调整 **scrollview** 的 **inset**，防止其被导航栏等遮挡。
- 这里的 **scrollview** 包括 **UIScrollView**，以及继承自 **UIScrollView** 的 **UITableView**、**UICollectionView**、**UITextView** 等。
- 要注意的是，如果有多个 **scrollview**，控制器只会自动调整其下第一个 **scrollview** 的 **inset** 属性。

比如我们在视图中添加一个 **tableview**，虽然我们将 **tableview** 的 **frame** 设置成控制器 **view** 的 **frame**。但表格也不会被导航栏给遮挡。

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var tableView:UITableView?
     
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
    }
     
    //******
```

[![原文:Swift - 当存在导航栏时，scrollview自动下移的问题解决](https://www.hangge.com/blog_uploads/201701/2017010818235349243.png)](https://www.hangge.com/blog/cache/detail_1514.html#)



### 2，automaticallyAdjustsScrollViewInsets为true造成的问题

虽然自动调整 **inset** 很方便，但有时也会造成一些问题。

（1）比如我们添加一个 **textview**，由于自动调整，**textview** 上面会空出一大块区域。

[![原文:Swift - 当存在导航栏时，scrollview自动下移的问题解决](https://www.hangge.com/blog_uploads/201701/2017010818570073166.png)](https://www.hangge.com/blog/cache/detail_1514.html#)



（2）或者我们布局的时候就已经把导航栏考虑在内，即起始位置从 **(0,64)** 开始。但由于内边距的自动调成，会造成内容偏移。

[![原文:Swift - 当存在导航栏时，scrollview自动下移的问题解决](https://www.hangge.com/blog_uploads/201701/2017010819390832765.png)](https://www.hangge.com/blog/cache/detail_1514.html#)



### 3，解决办法

将 **scrollview** 自动调整 **inset** 功能关闭即可，有两种方法修改。
（1）在代码中设置

```swift
self.automaticallyAdjustsScrollViewInsets = false
```


（2）在 **storyboard** 中设置 

[![原文:Swift - 当存在导航栏时，scrollview自动下移的问题解决](https://www.hangge.com/blog_uploads/201701/2017010818061784716.png)](https://www.hangge.com/blog/cache/detail_1514.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1514.html