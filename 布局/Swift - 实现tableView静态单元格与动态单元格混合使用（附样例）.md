# Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）

2016-10-20发布：hangge阅读：5638

使用表格（**UITableView**）时，我们可以选择其采用静态单元格（**Static Cell**）还是动态单元格（**Dynamic Cell**）。前者使用方便，只需要在 **storyboard** 中设置好各个单元格即可。后者使用灵活，可以根据绑定的数据动态创建对应的单元格并显示。

但这二者并不是互斥的，我们可以在同一个 **tableView** 混合使用它们。比如部分单元格使用动态的，部分单元格使用静态的。下面通过一个样例作为演示。

**1，样例效果图**
（1）下面是一 **wifi** 选择页面。单元格一共分为三个 **section**。
（2）第1个、第3个 **section** 里的单元格都是固定的，所以我们可以使用 **Static Cell**。
（3）中间 **section** 里的单元格是根据 **wifi** 列表数据来动态显示的，所以使用 **Dynamic Cell**。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/2016100914310449500.png)](https://www.hangge.com/blog/cache/detail_1387.html#)

**2，实现步骤**

（1）在 **Main.storyboard** 中添加一个 **table view controller**。作为演示，这里勾选“**Is Initial View Controller**”将其作为初始页面。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/2016100910504429088.png)](https://www.hangge.com/blog/cache/detail_1387.html#)

（2）将 **table view** 的 **Content** 改成 **Static Cells**，**Sections** 改成 **3**。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/201610091056559374.png)](https://www.hangge.com/blog/cache/detail_1387.html#)

（3）由于第1个、第3个的分组里的 **cell** 都是不变的。所以我们这里直接配置好它们内部的组件。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/2016100914185597351.png)](https://www.hangge.com/blog/cache/detail_1387.html#)



（4）而第2个分组里的单元格是动态的，这里就将其 **Rows** 设为1。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/2016100914012637974.png)](https://www.hangge.com/blog/cache/detail_1387.html#)

（5）创建一个自定义类 **TableViewController**（继承自 **UITableViewController**）。将场景中的 **TableViewController** 与新建这个新建类进行绑定。

[![原文:Swift - 实现tableView静态单元格与动态单元格混合使用（附样例）](https://www.hangge.com/blog_uploads/201610/2016100914222893206.png)](https://www.hangge.com/blog/cache/detail_1387.html#)



（6）**TableViewController.swift** 代码如下：

```swift
import UIKit
 
class TableViewController: UITableViewController {
     
    //wifi数据集合
    let wifiArray = ["hangge-wifi-1","hangge-wifi-2","hangge-wifi-3"]
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "wifiCell")
        //去除尾部多余的空行
        self.tableView.tableFooterView = UIView(frame:CGRect.zero)
        //设置分区头部字体颜色和大小
        UILabel.appearance(whenContainedInInstancesOf: [UITableViewHeaderFooterView.self])
            .textColor = UIColor.gray
        UILabel.appearance(whenContainedInInstancesOf: [UITableViewHeaderFooterView.self])
            .font = UIFont.systemFont(ofSize: 13.0, weight: UIFontWeightMedium)
    }
     
    //返回分区数
    override func numberOfSections(in tableView: UITableView) -> Int {
        return 3
    }
 
    //返回每个分区中单元格的数量
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int)
        -> Int {
        if section == 1 {
            return wifiArray.count
        }else{
            return 1
        }
    }
 
    //设置cell
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //只有第二个分区是动态的，其它默认
        if indexPath.section == 1 {
            //用重用的方式获取标识为wifiCell的cell
            let cell = tableView.dequeueReusableCell(withIdentifier: "wifiCell",
                                                     for: indexPath)
            cell.textLabel?.text = wifiArray[indexPath.row]
            return cell
        }else{
            return super.tableView(tableView, cellForRowAt: indexPath)
        }
    }
     
    //因为第二个分区单元格动态添加，会引起cell高度的变化，所以要重新设置
    override func tableView(_ tableView: UITableView,
                            heightForRowAt indexPath: IndexPath) -> CGFloat {
        if indexPath.section == 1{
            return 44
        }else{
            return super.tableView(tableView, heightForRowAt: indexPath)
        }
    }
     
    //当覆盖了静态的cell数据源方法时需要提供一个代理方法。
    //因为数据源对新加进来的cell一无所知，所以要使用这个代理方法
    override func tableView(_ tableView: UITableView,
                            indentationLevelForRowAt indexPath: IndexPath) -> Int {
        if indexPath.section == 1{
            //当执行到日期选择器所在的indexPath就创建一个indexPath然后强插
            let newIndexPath = IndexPath(row: 0, section: indexPath.section)
            return super.tableView(tableView, indentationLevelForRowAt: newIndexPath)
        }else{
            return super.tableView(tableView, indentationLevelForRowAt: indexPath)
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1387.zip](https://www.hangge.com/blog_uploads/201610/2016100914324621327.zip)

关于动态添加、移除单元格，还可以参考我的另一篇文章：[Swift - 动态添加删除TableView的单元格（以及内部元件）](https://www.hangge.com/blog/cache/detail_727.html)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1387.html