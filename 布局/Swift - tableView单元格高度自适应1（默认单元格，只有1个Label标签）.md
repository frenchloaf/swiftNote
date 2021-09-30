# Swift - tableView单元格高度自适应1（默认单元格，只有1个Label标签）

2016-08-24发布：hangge阅读：2606

（本文代码已升级至Swfit3）

**1，单元格高度自适应**
有时我们使用表格（**UITableView**）显示列表数据。不希望每个单元格的高度是固定的，最好能根据单元格中的内容来自适应高度。在过去，我们如果想在表格视图中展示可变高度的动态内容，需要手动计算行高，比较麻烦。

自 **iOS8** 起，**UITableView** 加入了一项新功能：**Self Sizing Cells**。设置后表格会自动计算单元格的尺寸并渲染，大大节省了开发时间。

**2，使用Self Sizing Cells的条件**

（1）单元格 **Cell** 需要使用 **Auto Layout** 约束。

（2）需要指定 **tableView** 的 **estimatedRowHeight** 属性默认值。

（2）将 **tableView** 的 **rowHeight** 属性设置为 **UITableViewAutomaticDimension**。

**3，样例效果图**

下面先通过一个最简单的样例演示如何使用 **Self Sizing Cells**。单元格就是最原始的 **UITableViewCell**，即里面只有一个文本标签（**textLabel**）。不过单元格会随着内部文字自适应高度，使文本标签能将内容完全显示出来。

[![原文:Swift - tableView单元格高度自适应1（默认单元格，只有1个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082110554424852.png)](https://www.hangge.com/blog/cache/detail_1341.html#)

**4，样例代码**

这里注意我们要将单元格 **label** 的 **numberOfLines** 属性设为**0**（默认是**1**）。这样就可以允许标签自动增长。

```swift
import UIKit
 
class ViewController: UIViewController , UITableViewDelegate, UITableViewDataSource {
     
    var catalog = [[String]]()
    var tableView:UITableView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化列表数据
        catalog.append(["第一节：Swift 环境搭建",
                        "由于Swift开发环境需要在OS X系统中运行，下面就一起来学习一下。"])
        catalog.append(["第二节：Swift 基本语法",
                        "本节介绍Swift中一些常用的关键字。以及引入、注释等相关操作。"])
        catalog.append(["第三节: Swift 数据类型",
                        "Swift 提供了非常丰富的数据类型。"])
        catalog.append(["第四节: Swift 变量",
                        "Swift 每个变量都指定了特定的类型，该类型决定了变量占用内存的大小。"])
        catalog.append(["第五节: Swift 可选(Optionals)类型",
                        "Swift 的可选（Optional）类型，用于处理值缺失的情况。"])
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView.delegate = self
        self.tableView.dataSource = self
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "SwiftCell")
         
        //设置estimatedRowHeight属性默认值
        self.tableView.estimatedRowHeight = 44.0;
        //rowHeight属性设置为UITableViewAutomaticDimension
        self.tableView.rowHeight = UITableViewAutomaticDimension;
         
        self.view.addSubview(self.tableView!)
    }
     
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.catalog.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "SwiftCell"
        //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath) as UITableViewCell
        //获取对应的条目内容
        let entry = catalog[indexPath.row]
        //允许标签自动增长
        cell.textLabel?.numberOfLines = 0
        cell.textLabel?.text = "\(entry[0]): \(entry[1])"
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，功能改进**
前面样例中，每节标题和内容简介都放一起不太美观。下面通过 **textLabel** 的 **attributedText** 属性来设置具有样式属性的字符串。让标题文字加粗并有颜色，而内容简介另起一行开始显示。

[![原文:Swift - tableView单元格高度自适应1（默认单元格，只有1个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082111090157044.png)](https://www.hangge.com/blog/cache/detail_1341.html#)

```swift
import UIKit
 
class ViewController: UIViewController , UITableViewDelegate, UITableViewDataSource {
     
    var catalog = [[String]]()
    var tableView:UITableView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化列表数据
        catalog.append(["第一节：Swift 环境搭建",
                        "由于Swift开发环境需要在OS X系统中运行，下面就一起来学习一下。"])
        catalog.append(["第二节：Swift 基本语法",
                        "本节介绍Swift中一些常用的关键字。以及引入、注释等相关操作。"])
        catalog.append(["第三节: Swift 数据类型",
                        "Swift 提供了非常丰富的数据类型。"])
        catalog.append(["第四节: Swift 变量",
                        "Swift 每个变量都指定了特定的类型，该类型决定了变量占用内存的大小。"])
        catalog.append(["第五节: Swift 可选(Optionals)类型",
                        "Swift 的可选（Optional）类型，用于处理值缺失的情况。"])
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView.delegate = self
        self.tableView.dataSource = self
        //创建一个重用的单元格
        self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "SwiftCell")
         
        //设置estimatedRowHeight属性默认值
        self.tableView.estimatedRowHeight = 44.0;
        //rowHeight属性设置为UITableViewAutomaticDimension
        self.tableView.rowHeight = UITableViewAutomaticDimension;
         
        self.view.addSubview(self.tableView!)
    }
     
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.catalog.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "SwiftCell"
        //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath) as UITableViewCell
        //获取对应的条目内容
        let entry = catalog[indexPath.row]
        //允许标签自动增长
        cell.textLabel?.numberOfLines = 0
        //使用属性文本
        cell.textLabel?.attributedText = getAttributedString(title: entry[0],
                                                             subtitle: entry[1])
        return cell
    }
     
    //获取条目属性文本
    func getAttributedString(title: String, subtitle: String) -> NSAttributedString {
        //标题字体样式
        let titleFont = UIFont.preferredFont(forTextStyle: UIFontTextStyle.headline)
        let titleColor = UIColor(red: 45/255, green: 153/255, blue: 0/255, alpha: 1)
        let titleAttributes = [NSFontAttributeName: titleFont,
                               NSForegroundColorAttributeName: titleColor]
        //简介字体样式
        let subtitleFont = UIFont.preferredFont(forTextStyle: UIFontTextStyle.subheadline)
        let subtitleAttributes = [NSFontAttributeName: subtitleFont]
        //拼接并获取最终文本
        let titleString = NSMutableAttributedString(string: "\(title)\n",
            attributes: titleAttributes)
        let subtitleString = NSAttributedString(string: subtitle,
                                                attributes: subtitleAttributes)
        titleString.append(subtitleString)
        return titleString
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1341.html