# Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）

2016-10-10发布：hangge阅读：2738

我原来写过一篇文章：[Swift - 使用UISearchController实现带搜索栏的表格](https://www.hangge.com/blog/cache/detail_797.html)。介绍如何使用 **UISearchController** 结合 **UITableView** 来实现一个具有搜索功能的表格。效果图如下：

[![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100909474333363.png)](https://www.hangge.com/blog/cache/detail_1386.html#)



当时的做法是将 **searchController** 的 **searchBar** 放到 **tableView** 的 **tableHeaderView** 上。但这样做的话搜索栏会跟着表格一起滚动。如果表格滚动到下方，那么搜索栏就会看不见，要搜索只能再将表格滚回到顶部。

[![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100909535744542.png)](https://www.hangge.com/blog/cache/detail_1386.html#)



如果想让搜索栏的位置固定，我们可以将其放在 **tableView** 的外面而不是放到 **tableView** 的 **tableHeaderView** 上。主要代码如下：

```swift
override func viewDidLoad() {
    super.viewDidLoad()
     
    //配置搜索控制器
    self.countrySearchController = ({
        let controller = UISearchController(searchResultsController: nil)
        controller.searchResultsUpdater = self
        controller.hidesNavigationBarDuringPresentation = true
        controller.dimsBackgroundDuringPresentation = false
        controller.searchBar.searchBarStyle = .minimal
        controller.searchBar.sizeToFit()
        return controller
    })()
     
    //将搜索栏添加到页面上
    let searchBarFrame = CGRect(x: 0, y: 20, width: self.view.frame.width, height: 44)
    countrySearchController.searchBar.frame = searchBarFrame
    self.view.addSubview(countrySearchController.searchBar)
 
    //创建表视图
    let tableViewFrame = CGRect(x: 0, y: 64, width: self.view.frame.width,
                                height: self.view.frame.height - 64)
    self.tableView = UITableView(frame: tableViewFrame, style:.plain)
    self.tableView!.delegate = self
    self.tableView!.dataSource = self
    //创建一个重用的单元格
    self.tableView!.register(UITableViewCell.self,
                                  forCellReuseIdentifier: "MyCell")
    self.view.addSubview(self.tableView!)
}
```


**1，问题描述**
按上面代码调整位置后，通常状态下搜索栏是正常的。它会一直固定在表格上方，而不会随着表格移动。

[![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100910024970036.png)](https://www.hangge.com/blog/cache/detail_1386.html#)



但当我们点击 **searchBar** 进行搜索时，**searchBar** 就会错位，变得只显示一半。如果页面上有导航条（**navigationController**），那整个 **searchBar** 干脆就直接消失了。

​     [![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100910105721710.png)](https://www.hangge.com/blog/cache/detail_1386.html#)      [![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100910110392399.png)](https://www.hangge.com/blog/cache/detail_1386.html#)

**2，问题解决**
不要将 **searchBar** 直接放到视图控制器的 **view** 里，而是先将 **searchBar** 放在一个 **UIView** 中，再将这个 **UIView** 添加到视图控制器的 **view** 里面。

```swift
override func viewDidLoad() {
    super.viewDidLoad()
     
    //配置搜索控制器
    self.countrySearchController = ({
        let controller = UISearchController(searchResultsController: nil)
        controller.searchResultsUpdater = self
        controller.hidesNavigationBarDuringPresentation = true
        controller.dimsBackgroundDuringPresentation = false
        controller.searchBar.searchBarStyle = .minimal
        controller.searchBar.sizeToFit()
        return controller
    })()
     
    //将搜索栏添加到页面上
    let searchBarFrame = CGRect(x: 0, y: 20, width: self.view.frame.width, height: 44)
    let searchBarContainer = UIView(frame: searchBarFrame)
    searchBarContainer.addSubview(countrySearchController.searchBar)
    self.view.addSubview(searchBarContainer)
 
    //创建表视图
    let tableViewFrame = CGRect(x: 0, y: 64, width: self.view.frame.width,
                                height: self.view.frame.height - 64)
    self.tableView = UITableView(frame: tableViewFrame, style:.plain)
    self.tableView!.delegate = self
    self.tableView!.dataSource = self
    //创建一个重用的单元格
    self.tableView!.register(UITableViewCell.self,
                                  forCellReuseIdentifier: "MyCell")
    self.view.addSubview(self.tableView!)
}
```

再次运行程序，可以看到在搜索状态下也是正常的了：

[![原文:Swift - searchBar放在tableView外部，固定位置（点击消失问题解决）](https://www.hangge.com/blog_uploads/201610/2016100910190548162.png)](https://www.hangge.com/blog/cache/detail_1386.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1386.html