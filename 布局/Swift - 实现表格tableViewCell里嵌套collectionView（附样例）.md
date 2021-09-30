# Swift - 实现表格tableViewCell里嵌套collectionView（附样例）

2017-03-29发布：hangge阅读：9495

（本文代码已升级至Swift4）

有时要实现一个复杂的页面布局，单单使用 **UITableView** 实现不了，需要通过 **UITableView** 和 **UICollectionView** 结合实现，即每个单元格 **tableViewCell** 中都嵌套一个 **collectionView**。下面通过样例演示如何实现。

###  1，效果图

（1）表格中每一个单元格对应一个月份的图书列表。

（2）单元格中头部显示月份标题。内部通过 **collectionView** 显示当月所有书籍封面图片，数量不定。整个单元格高度自适应。

（3）这个样例其实只用多 **section** 的 **collectionView** 也能实现（[点击查看](https://www.hangge.com/blog/cache/detail_1592.html)）。本文使用 **UITableView** + **UICollectionView** 演示如何实现同样的功能，

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031211140595015.png)](https://www.hangge.com/blog/cache/detail_1591.html#)

### 2，如何实现单元格高度自适应

（1）我们要对 **collectionView** 设置个高度约束。当在单元格中更新 **collectionView** 的数据时，要获取这个 **collectionView** 的真实的内容高度（**contentSize.height**），然后用 **contentSize.height** 来更新 **collectionView** 的高度约束。这样就实现了单元格内部 **collectionView** 的高度自适应。

（2）而对于单元格 **tableViewCell** 的高度自适应，是通过 **AutoLayout** 特性实现的。利用内容将 **cell** 撑起来。



```swift
//设置estimatedRowHeight属性默认值
self.tableView!.estimatedRowHeight = 44.0
//rowHeight属性设置为UITableViewAutomaticDimension
self.tableView!.rowHeight = UITableViewAutomaticDimension
```

### 3，实现步骤

（1）新建一个自定义的 **collectionView** 单元格类：**MyCollectionViewCell**，同时勾选“**Also create XIB file**”

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031210332017668.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



（2）在 **MyCollectionViewCell.xib** 中添加一个 **ImageView**，并设置好约束。同时在对应的类中作关联

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031210353839898.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



（3）**MyCollectionViewCell.swift** 代码如下：

```swift
import UIKit
 
class MyCollectionViewCell: UICollectionViewCell {
 
    //用于显示封面缩略图
    @IBOutlet weak var imageView: UIImageView!
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
}
```


（4）新建一个自定义的 **tableView** 单元格类：**MyTableViewCell**，同时勾选“**Also create XIB file**”

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031210402271395.png)](https://www.hangge.com/blog/cache/detail_1591.html#)

（5）在 **MyTableViewCell.xib** 中添加一个 **Label** 和一个 **CollectionView**，并设置好约束。同时在对应的类中作关联。

其中 **Label** 设置的是上、下、左、右4个约束：

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031210490866066.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



**CollectionView** 设置的是左、右、下以及高度这个4个约束：

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031210524717157.png)](https://www.hangge.com/blog/cache/detail_1591.html#)

同时调整下 **CollectionView** 的 **Cell Size** 和 **Min Spacing**：

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201801/2018010310192596961.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



（6）**MyTableViewCell.swift** 代码如下：

```swift
import UIKit
 
class MyTableViewCell: UITableViewCell, UICollectionViewDelegate, UICollectionViewDataSource
{
     
    //单元格标题
    @IBOutlet weak var titleLabel: UILabel!
     
    //封面图片集合列表
    @IBOutlet weak var collectionView: UICollectionView!
     
    //collectionView的高度约束
    @IBOutlet weak var collectionViewHeight: NSLayoutConstraint!
     
    //封面数据
    var images:[String] = []
     
    override func awakeFromNib() {
        super.awakeFromNib()
         
        //设置collectionView的代理
        self.collectionView.delegate = self
        self.collectionView.dataSource = self
         
        // 注册CollectionViewCell
        self.collectionView!.register(UINib(nibName:"MyCollectionViewCell", bundle:nil),
                                      forCellWithReuseIdentifier: "myCell")
    }
     
    //加载数据
    func reloadData(title:String, images:[String]) {
        //设置标题
        self.titleLabel.text = title
        //保存图片数据
        self.images = images
         
        //collectionView重新加载数据
        self.collectionView.reloadData()
         
        //更新collectionView的高度约束
        let contentSize = self.collectionView.collectionViewLayout.collectionViewContentSize
        collectionViewHeight.constant = contentSize.height
         
        self.collectionView.collectionViewLayout.invalidateLayout()
    }
     
    //返回collectionView的单元格数量
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return images.count
    }
     
    //返回对应的单元格
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell  = collectionView.dequeueReusableCell(withReuseIdentifier: "myCell",
                                                for: indexPath) as! MyCollectionViewCell
        cell.imageView.image = UIImage(named: images[indexPath.item])
        return cell
    }
     
    //绘制单元格底部横线
    override func draw(_ rect: CGRect) {
        //线宽
        let lineWidth = 1 / UIScreen.main.scale
        //线偏移量
        let lineAdjustOffset = 1 / UIScreen.main.scale / 2
        //线条颜色
        let lineColor = UIColor(red: 0xe0/255, green: 0xe0/255, blue: 0xe0/255, alpha: 1)
         
        //获取绘图上下文
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        //创建一个矩形，它的所有边都内缩固定的偏移量
        let drawingRect = self.bounds.insetBy(dx: lineAdjustOffset, dy: lineAdjustOffset)
         
        //创建并设置路径
        let path = CGMutablePath()
        path.move(to: CGPoint(x: drawingRect.minX, y: drawingRect.maxY))
        path.addLine(to: CGPoint(x: drawingRect.maxX, y: drawingRect.maxY))
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(lineColor.cgColor)
        //设置笔触宽度
        context.setLineWidth(lineWidth)
         
        //绘制路径
        context.strokePath()
    }
     
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
}

```

（7）在 **StoryBoard** 主视图中添加一个 **TableView**，并设置好约束。同时在对应的类中作关联。

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031211000171595.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



（8）**ViewController.swift** 代码如下：

```swift
import UIKit
 
//每月书籍
struct BookPreview {
    var title:String
    var images:[String]
}
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    //所有书籍数据
    let books = [
        BookPreview(title: "五月新书", images: ["0.jpg", "1.jpg","2.jpg", "3.jpg",
                                                    "4.jpg","5.jpg","6.jpg"]),
        BookPreview(title: "六月新书", images: ["7.jpg", "8.jpg", "9.jpg"]),
        BookPreview(title: "七月新书", images: ["10.jpg", "11.jpg", "12.jpg", "13.jpg"])
    ]
     
    //显示内容的tableView
    @IBOutlet weak var tableView: UITableView!
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //设置tableView代理
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
         
        //去除单元格分隔线
        self.tableView!.separatorStyle = .none
         
        //创建一个重用的单元格
        self.tableView!.register(UINib(nibName:"MyTableViewCell", bundle:nil),
                                 forCellReuseIdentifier:"myCell")
         
        //设置estimatedRowHeight属性默认值
        self.tableView!.estimatedRowHeight = 44.0
        //rowHeight属性设置为UITableViewAutomaticDimension
        self.tableView!.rowHeight = UITableViewAutomaticDimension
    }
     
    //在本例中，只有一个分区
    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return 1;
    }
     
    //返回表格行数
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.books.count
    }
     
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "myCell")
            as! MyTableViewCell
         
        //下面这两个语句一定要添加，否则第一屏显示的collection view尺寸，以及里面的单元格位置会不正确
        cell.frame = tableView.bounds
        cell.layoutIfNeeded()
         
        //重新加载单元格数据
        cell.reloadData(title:books[indexPath.row].title,
                        images: books[indexPath.row].images)
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1591.zip](https://www.hangge.com/blog_uploads/201801/201801031035234988.zip)

## 功能修改：每个单元格只显示一行封面图片

上面的样例中，每个单元格内的图片是全部显示出来，有多少显示多少，高度自适应。

我们还可以换种展示方法，每个单元格，即每个 **collectionView** 只显示一行数据，图片如果多的话可以通过左右滑动查看。

### 1，效果图

  [![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031215355432738.png)](https://www.hangge.com/blog/cache/detail_1591.html#)   [![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031215355981489.png)](https://www.hangge.com/blog/cache/detail_1591.html#)

### 2，实现原理

我们只需要把 **collectionView** 的滚动方向改成水平方向即可。

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031215383933454.png)](https://www.hangge.com/blog/cache/detail_1591.html#)



如果不需要显示横向的滚动条，可以去掉“**Shows Horizontal Indicator**”的勾选。

[![原文:Swift - 实现表格tableViewCell里嵌套collectionView（附样例）](https://www.hangge.com/blog_uploads/201703/2017031215401713393.png)](https://www.hangge.com/blog/cache/detail_1591.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1591.html