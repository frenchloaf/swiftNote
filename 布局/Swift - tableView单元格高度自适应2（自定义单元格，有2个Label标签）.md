# Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）

2016-08-26发布：hangge阅读：2637

（本文代码已升级至Swift3）

在上文中：[Swift - tableView单元格高度自适应1（默认单元格，只有1个Label标签）](https://www.hangge.com/blog/cache/detail_1341.html)，我简单演示了通过 **Self Sizing Cells** 这个新特性实现单元格自适应功能。当时使用的是最原始的 **UITableViewCell**，在实际开发中我们单元格会更复杂些，比如我们在 **StoryBoard** 中给单元格添加两个文本标签（**Label**），让标题和内容分开。 下面演示如何让自定义单元格也实现高度自适应功能。

**1，效果图**

单元格内有两个文本标签，分别显示标题和内容简介。同时两个标签都是自增长的。

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/201608211810478308.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



**2，StoryBoard设置**
（1）将单元格 **cell** 的 **identifier** 设置成“**myCell**”

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082118132272172.png)](https://www.hangge.com/blog/cache/detail_1342.html#)

（2）从对象库中拖入2个 **Label** 控件到 **cell** 中，分别用于显示标题和内容。

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/201608211815466737.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



（3）并在 **Attributes** 面板中将两个 **Label** 的 **Tag** 值分别设置为 **1** 和 **2**，供代码中获取标签。 

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082118163651527.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



（4）最后为了让两个 **Label** 标签能自动增长，将其 **Lines** 属性设置为 **0**。

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082118184418302.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



**3，约束设置**

除了单元格 **Cell** 需要使用 **Auto Layout** 约束，单元格内的标签也要设置正确的约束，否则无法实现 **Self Sizing Cells**。

（1）第一个 **Label**（用于显示标题）设置上下左右 **4** 个约束。

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082118231915320.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



（2）第二个 **Label**（用于显示内容简介）设置左右下 **3** 个约束。

[![原文:Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog_uploads/201608/2016082118245698336.png)](https://www.hangge.com/blog/cache/detail_1342.html#)



**4，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController , UITableViewDelegate, UITableViewDataSource {
     
    var catalog = [[String]]()
     
    @IBOutlet weak var tableView: UITableView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化列表数据
        catalog.append(["第一节：Swift 环境搭建",
                        "由于Swift开发环境需要在OS X系统中运行，下面就一起来学习一下。"])
        catalog.append(["第二节：Swift 基本语法。这个标题很长很长很长。",
                        "本节介绍Swift中一些常用的关键字。以及引入、注释等相关操作。"])
        catalog.append(["第三节: Swift 数据类型",
                        "Swift 提供了非常丰富的数据类型，比如：Int、UInt、浮点数、布尔值、字符串等。"])
        catalog.append(["第四节: Swift 变量",
                        "Swift 每个变量都指定了特定的类型，该类型决定了变量占用内存的大小。"])
        catalog.append(["第五节: Swift 可选(Optionals)类型",
                        "Swift 的可选（Optional）类型，用于处理值缺失的情况。"])
         
        //创建表视图
        self.tableView.delegate = self
        self.tableView.dataSource = self
         
        //设置estimatedRowHeight属性默认值
        self.tableView.estimatedRowHeight = 44.0;
        //rowHeight属性设置为UITableViewAutomaticDimension
        self.tableView.rowHeight = UITableViewAutomaticDimension;
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
            let cell = tableView.dequeueReusableCell(withIdentifier: "myCell",
                                                     for: indexPath) as UITableViewCell
        //获取对应的条目内容
        let entry = catalog[indexPath.row]
        let titleLabel = cell.viewWithTag(1) as! UILabel
        let subtitleLabel = cell.viewWithTag(2) as! UILabel
        titleLabel.text = entry[0]
        subtitleLabel.text = entry[1]
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1342.zip](https://www.hangge.com/blog_uploads/201611/2016112110574539938.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1342.html