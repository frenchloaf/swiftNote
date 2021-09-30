# Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）

2016-08-29发布：hangge阅读：5250

（本文代码已升级至Swift3）

相关文章：
[Swift - tableView单元格高度自适应1（默认单元格，只有1个Label标签）](https://www.hangge.com/blog/cache/detail_1341.html)
[Swift - tableView单元格高度自适应2（自定义单元格，有2个Label标签）](https://www.hangge.com/blog/cache/detail_1342.html)

在之前的两篇文章中，列表单元格中使用的都是文本标签（**Label**）。本文演示图文混合的单元格（即一个单元格中除了 **Label** 还有 **UIImageView**）如何实现高度自适应。

**1，样例效果图**

（1）单元格上面是文字，下面是图片。

（2）文字标签（**UILabel**）行数是自增长的。

（3）图片显示（**UIImageVIew**）宽度占满一行，高度随原图比例自适应高度。

（4）整个单元格高度自适应，能完整地显示全部的内容。

   [![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082810251163785.png)](https://www.hangge.com/blog/cache/detail_1343.html#)   [![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082810252372205.png)](https://www.hangge.com/blog/cache/detail_1343.html#)

***\*
\**2，实现步骤**
（1）我们先创建一个自定义的单元格类（**ImageTableViewCell.swift**）,同时创建其对应的 **XIB** 文件（**ImageTableViewCell.xib**）。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/201608281030353438.png)](https://www.hangge.com/blog/cache/detail_1343.html#)

（2）打开 **ImageTableViewCell.xib**，在里面添加一个 **Label**（用来显示标题文字），和一个 **ImageView**（用来显示内容图片）。并将它们在对应类中做关联引用。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082810334897185.png)](https://www.hangge.com/blog/cache/detail_1343.html#)



（3）为了让 **Label** 标签能自动增长，将其 **Lines** 属性设置为 **0**。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082118184418302.png)](https://www.hangge.com/blog/cache/detail_1343.html#)

（4）对 **Label** 设置上下左右 **4** 个约束。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082118231915320.png)](https://www.hangge.com/blog/cache/detail_1343.html#)

（5）对 **ImageView** 设置左右下 **3** 个约束。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082118245698336.png)](https://www.hangge.com/blog/cache/detail_1343.html#)

（6）**ImageTableViewCell.swift** 代码
上面约束设置后，**imageView** 的宽度就固定下来了。为了让其高度自适应，我们需要在代码中通过获取其内部显示图片的宽高比，来动态地设置 **imageView** 的宽高比约束。保证 **imageView** 与原始图片的比例是一样的。

```swift
import UIKit
 
class ImageTableViewCell: UITableViewCell {
     
    //标题文本标签
    @IBOutlet weak var titleLabel: UILabel!
     
    //内容图片
    @IBOutlet weak var contentImageView: UIImageView!
     
    //内容图片的宽高比约束
    internal var aspectConstraint : NSLayoutConstraint? {
        didSet {
            if oldValue != nil {
                contentImageView.removeConstraint(oldValue!)
            }
            if aspectConstraint != nil {
                contentImageView.addConstraint(aspectConstraint!)
            }
        }
    }
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
     
    override func prepareForReuse() {
        super.prepareForReuse()
        //清除内容图片的宽高比约束
        aspectConstraint = nil
    }
     
     
    //加载内容图片（并设置高度约束）
    func loadImage(name: String) {
        if let image = UIImage(named: name) {
            //计算原始图片的宽高比
            let aspect = image.size.width / image.size.height
            //设置imageView宽高比约束
            aspectConstraint = NSLayoutConstraint(item: contentImageView,
                                            attribute: .width, relatedBy: .equal,
                                            toItem: contentImageView, attribute: .height,
                                            multiplier: aspect, constant: 0.0)
            //加载图片
            contentImageView.image = image
        }else{
            //去除imageView里的图片和宽高比约束
            aspectConstraint = nil
            contentImageView.image = nil
        }
    }
}
```


（7）**ViewController.swit**
在首页中我们创建一个 **tabbleView** 来测试上面我们自定义的单元格。这里没什么特别的，同前文差不多。

```swift
import UIKit
 
class ViewController: UIViewController , UITableViewDelegate, UITableViewDataSource {
     
    var catalog = [[String]]()
    var tableView:UITableView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化列表数据
        catalog.append(["第一节：Swift 环境搭建", "img1.jpg"])
        catalog.append(["第二节：Swift 基本语法（类型定义、循环遍历、判断、继承）", "img2.jpg"])
        catalog.append(["第三节：Swift 数据类型", "img3.jpg"])
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style: .plain)
        self.tableView.delegate = self
        self.tableView.dataSource = self
         
        //创建一个重用的单元格
        self.tableView!.register(UINib(nibName:"ImageTableViewCell", bundle:nil),
                                    forCellReuseIdentifier:"myCell")
         
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
            //同一形式的单元格重复使用
        let cell = tableView.dequeueReusableCell(withIdentifier: "myCell",
                                                 for: indexPath) as! ImageTableViewCell
            //获取对应的条目内容
            let entry = catalog[indexPath.row]
            //单元格标题和内容设置
            cell.titleLabel.text = entry[0]
            cell.loadImage(name: entry[1])
             
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1343.zip](https://www.hangge.com/blog_uploads/201611/2016112111090352071.zip)



**3，从网络获取图片**
上面的样例我们是直接加载本地图片的。在实际项目中，我们通常会通过 **url** 地址加载网络上的图片并显示。

[![原文:Swift - tableView单元格高度自适应3（图片宽度固定，高度自适应）](https://www.hangge.com/blog_uploads/201608/2016082811160552122.png)](https://www.hangge.com/blog/cache/detail_1343.html#)



只需要将 **ImageTableViewCell** 类做如下修改可以满足需求，同样支持图片的高度自适应。

```swift
import UIKit
 
class ImageTableViewCell: UITableViewCell {
     
    //标题文本标签
    @IBOutlet weak var titleLabel: UILabel!
     
    //内容图片
    @IBOutlet weak var contentImageView: UIImageView!
     
    //内容图片的宽高比约束
    internal var aspectConstraint : NSLayoutConstraint? {
        didSet {
            if oldValue != nil {
                contentImageView.removeConstraint(oldValue!)
            }
            if aspectConstraint != nil {
                contentImageView.addConstraint(aspectConstraint!)
            }
        }
    }
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
     
    override func prepareForReuse() {
        super.prepareForReuse()
        //清除内容图片的宽高比约束
        aspectConstraint = nil
    }
     
    //加载内容图片（并设置高度约束）
    func loadImage(urlString: String) {
        //定义NSURL对象
        let url = URL(string: urlString)
        let data = try? Data(contentsOf: url!)
        //从网络获取数据流,再通过数据流初始化图片
        if let imageData = data, let image = UIImage(data: imageData) {
            //计算原始图片的宽高比
            let aspect = image.size.width / image.size.height
            //设置imageView宽高比约束
            aspectConstraint = NSLayoutConstraint(item: contentImageView,
                                                  attribute: .width, relatedBy: .equal,
                                                  toItem: contentImageView, attribute: .height,
                                                  multiplier: aspect, constant: 0.0)
            //加载图片
            contentImageView.image = image
        }else{
            //去除imageView里的图片和宽高比约束
            aspectConstraint = nil
            contentImageView.image = nil
        }
    }
}
```

使用时，由原来的图片名称改成图片 **url** 地址即可。

```swift
//初始化列表数据
catalog.append(["iPhone 6的故障率达26% 稳超安卓", "http://www.hangge.com/img1.png"])
catalog.append(["不用导航 这款无人机能够“自制”地图飞行", "http://www.hangge.com/img2.png"])
catalog.append(["无人汽车如何应对道德困境？谷歌表示不知道", "http://www.hangge.com/img33.png"])
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1343.zip](https://www.hangge.com/blog_uploads/201611/2016112111224888468.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1343.html