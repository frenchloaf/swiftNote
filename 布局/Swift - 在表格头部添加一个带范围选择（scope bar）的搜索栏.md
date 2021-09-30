# Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏

2016-07-27发布：hangge阅读：2457

（本文代码已升级至Swift3）

原来我写过一篇文章，讲的是通过使用 **UISearchController** 配合 **UITableView**，可以实现一个带有搜索功能的列表。地址：[Swift - 使用UISearchController实现带搜索栏的表格](https://www.hangge.com/blog/cache/detail_797.html)
这次做个功能改进，就是在搜索条（**searchBar**）上增加一个范围选择（**scopeBar**），用户可以根据对应的分类更精确地筛选查询结果。

**1，效果图**

（1）通常状态下，表格头部只有一个搜索条。

[![原文:Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏](https://www.hangge.com/blog_uploads/201607/2016072110273612157.png)](https://www.hangge.com/blog/cache/detail_1293.html#)

（2）点击搜索条进入查询状态，同时顶部导航栏消失。输入条件后下方出现匹配的电影，默认查询全部。

[![原文:Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏](https://www.hangge.com/blog_uploads/201607/2016072110280051633.png)](https://www.hangge.com/blog/cache/detail_1293.html#)

（3）可以点击 **scopeBar** 上具体的分类来过滤结果，比如只显示“动画”类别下的查询结果。

[![原文:Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏](https://www.hangge.com/blog_uploads/201607/201607211028263474.png)](https://www.hangge.com/blog/cache/detail_1293.html#)

**2，样例代码**

（1）**Movie.swift**（电影类）

```swift
import Foundation
 
struct Movie{
    //电影名
    let name : String
    //类别
    let category : String
}
```

（2）**ViewController.swift**（主视图代码）

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //展示列表
    var tableView: UITableView!
     
    //搜索控制器
    var searchController = UISearchController(searchResultsController: nil)
     
    //电影集合
    var movies = [Movie]()
     
    //搜索过滤后的结果集
    var filteredMovies:[Movie] = [Movie](){
        didSet  {self.tableView.reloadData()}
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化音乐集合
        movies = [
            Movie(name:"杀破狼", category:"动作"),
            Movie(name:"蝙蝠侠战超人", category:"动作"),
            Movie(name:"我的少女时代", category:"爱情"),
            Movie(name:"超能陆战队", category:"动画"),
            Movie(name:"西游记之大圣归来", category:"动画")]
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame,
                                     style:UITableViewStyle.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
         
        self.view.addSubview(self.tableView!)
         
        //配置搜索控制器
        searchController.searchResultsUpdater = self
        searchController.searchBar.delegate = self
        definesPresentationContext = true
        searchController.dimsBackgroundDuringPresentation = false
        //搜索状态下隐藏顶部导航栏
        searchController.hidesNavigationBarDuringPresentation = true
         
        //配置分段条
        searchController.searchBar.scopeButtonTitles = ["全部", "动作", "爱情", "动画"]
        tableView.tableHeaderView = searchController.searchBar
    }
     
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(true)
        self.tableView.reloadData()
    }
     
    //根据条件过滤数据
    func filterContentForSearchText(_ searchText: String, scope: String = "全部") {
        filteredMovies = movies.filter({( movie : Movie) -> Bool in
            let categoryMatch = (scope == "全部") || (movie.category == scope)
            return categoryMatch && movie.name.lowercased().contains(
                searchText.lowercased())
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension ViewController: UITableViewDataSource {
    //返回表格行数
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
    {
        if (self.searchController.isActive) {
            return self.filteredMovies.count
        } else {
            return self.movies.count
        }
    }
     
    //单元格内容设置
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
             
            let cell = UITableViewCell(style: UITableViewCellStyle.subtitle,
                                       reuseIdentifier: "myCell")
             
            if (self.searchController.isActive) {
                cell.textLabel?.text = self.filteredMovies[indexPath.row].name
                cell.detailTextLabel?.text = self.filteredMovies[indexPath.row].category
                return cell
            } else {
                cell.textLabel?.text = self.movies[indexPath.row].name
                cell.detailTextLabel?.text = self.movies[indexPath.row].category
                return cell
            }
    }
}
 
extension ViewController: UITableViewDelegate{
    //单元格选中事件
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
 
extension ViewController: UISearchBarDelegate {
    //范围分段条的选中项改变事件
    func searchBar(_ searchBar: UISearchBar, selectedScopeButtonIndexDidChange selectedScope: Int) {
        filterContentForSearchText(searchBar.text!,
                                   scope: searchBar.scopeButtonTitles![selectedScope])
    }
}
 
extension ViewController: UISearchResultsUpdating{
    //搜索栏文字改变后事件
    func updateSearchResults(for searchController: UISearchController) {
        let searchBar = searchController.searchBar
        let scope = searchBar.scopeButtonTitles![searchBar.selectedScopeButtonIndex]
        filterContentForSearchText(searchController.searchBar.text!, scope: scope)
    }
}
```

**源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1293.zip](https://www.hangge.com/blog_uploads/201701/2017012016345347657.zip)**

## 功能改进

上面样例中，当我们点击搜索栏，如果不输入文字那么搜索结果都是空的。

[![原文:Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏](https://www.hangge.com/blog_uploads/201701/2017012017005564597.png)](https://www.hangge.com/blog/cache/detail_1293.html#)

如果我们需要在搜索框输入为空的情况下，将当前分类条目都显示出来，只需修改 **filterContentForSearchText** 这个方法即可。效果图如下：

[![原文:Swift - 在表格头部添加一个带范围选择（scope bar）的搜索栏](https://www.hangge.com/blog_uploads/201701/2017012017010340418.png)](https://www.hangge.com/blog/cache/detail_1293.html#)

```swift
//根据条件过滤数据
func filterContentForSearchText(_ searchText: String, scope: String = "全部") {
    filteredMovies = movies.filter({( movie : Movie) -> Bool in
        let categoryMatch = (scope == "全部") || (movie.category == scope)
        let textMarch = searchText == "" || movie.name.lowercased().contains(
            searchText.lowercased())
        return categoryMatch && textMarch
    })
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1293.html