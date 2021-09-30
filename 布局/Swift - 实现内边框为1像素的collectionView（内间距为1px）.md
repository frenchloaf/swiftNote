# Swift - 实现内边框为1像素的collectionView（内间距为1px）

2017-01-11发布：hangge阅读：2384

我原来写过一篇文章介绍如何让 **CollectionView** 单元格实现固定间距：[Swift - 设置网格UICollectionView的单元格间距](https://www.hangge.com/blog/cache/detail_762.html)。文章提到一种方法是让单元格宽度动态变化，使得间距固定。另一种方法是让单元格宽度、间距都固定，两侧填充内边距。

本文综合上面的两种方法实现一个极细边框的 **collectionView**（即间距为**1**像素、或者说内边框为**1**像素）。



### 1，效果图

（1）不管屏幕尺寸、分辨率如何改变，单元格间距都是固定 **1** 像素。

[![原文:Swift - 实现内边框为1像素的collectionView（内间距为1px）](https://www.hangge.com/blog_uploads/201611/2016111715074037738.png)](https://www.hangge.com/blog/cache/detail_1445.html#)


（2）同时每行单元格数量可以设置。单元格大小会自适应，并且单元格间距仍然固定是**1**像素。

[![原文:Swift - 实现内边框为1像素的collectionView（内间距为1px）](https://www.hangge.com/blog_uploads/201611/2016111715083097680.png)](https://www.hangge.com/blog/cache/detail_1445.html#)



### 2，实现原理

（1）由于系统设备的 **DPI** 不同，**iOS** 在不同设备的缩放因子都不同。也就是说非 **Retina** 屏幕、**Retina** 屏幕（又分 **plus** 与非 **plus** 设备）下**1**个点（**Point**）对应的像素（**Pixel**）个数都是不同的。所以首先我们要根据缩放因子计算出 **1** 个像素对应的点的尺寸是多少，作为单元格间距。

（2）由于 **iOS** 系统的反锯齿（**antialiasing**）技术。我们不能简单的通过“**(屏幕宽度-所有间距)/每行单元格数量**”来设置单元格的尺寸，而是计算后再进行规整（多余的空间就作为**CollectionView** 两侧的内边距）。不这样做的话就不能保证每个单元格的每个像素都正好填充在一个显示单元中，从而有可能造成间隔线失真，或有的地方间隔小些，有些地方间隔大些。

### 3，样例代码

```swift
import UIKit
 
class ViewController: UICollectionViewController {
     
    //间距（1像素）
    let cellSpace = 1 / UIScreen.main.scale
     
    //列数
    let columnsNum = 7
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.collectionView?.backgroundColor = UIColor.white
        let layout = self.collectionViewLayout as! UICollectionViewFlowLayout
         
        //水平间隔
        layout.minimumInteritemSpacing = cellSpace
        //垂直行间距
        layout.minimumLineSpacing = cellSpace
         
        //整个view的宽度
        let collectionViewWidth = self.collectionView!.bounds.width
        //整个view横向除去间距后，剩余的像素个数
        let pxWidth = collectionViewWidth * UIScreen.main.scale - CGFloat(columnsNum - 1)
         
        //单元格宽度（像素）
        let itemWidthPx = CGFloat(Int(pxWidth / CGFloat(columnsNum)))
        //单元格宽度（点）
        let itemWidth = itemWidthPx / UIScreen.main.scale
         
        //设置单元格宽度和高度
        layout.itemSize = CGSize(width:itemWidth, height:itemWidth)
         
        //剩余像素（作为左右内边距）
        let remainderPx = pxWidth - itemWidthPx * CGFloat(columnsNum)
         
        //左内边距
        let paddingLeftPx = CGFloat(Int(remainderPx/2))
        let paddingLeft = paddingLeftPx / UIScreen.main.scale
         
        //右内边距
        let paddingRightPx = remainderPx - paddingLeftPx
        //右内边距做-0.001特殊处理，否则在plus设备下可能摆不下
        let paddingRight = paddingRightPx / UIScreen.main.scale - 0.001
         
        //设置内边距
        layout.sectionInset = UIEdgeInsets(top: 0, left: paddingLeft,
                                           bottom: 0, right: paddingRight)
    }
     
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return 50
    }
     
    override func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = (self.collectionView?.dequeueReusableCell(
            withReuseIdentifier: identify, for: indexPath))! as UICollectionViewCell
        let label = cell.contentView.viewWithTag(1) as! UILabel
        label.text = "\(indexPath.row)"
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_762.zip](https://www.hangge.com/blog_uploads/201611/2016111716231020876.zip)



## 功能改进1：左右不留内边距

如果嫌 **collectionView** 左右可能会出现的白边太丑的话，我们可以不设置 **collectionView** 的内边距，改成将多余的空间平摊到每个单元格上。比如还剩下 **4** 个像素的空间，那么我们就给让每行的前四个单元格宽度 **+1** 像素，其余的不变。这样刚好占满一行，而且由于尺寸只相差 **1** 像素，用户是看不出来的。

[![原文:Swift - 实现内边框为1像素的collectionView（内间距为1px）](https://www.hangge.com/blog_uploads/201611/201611171708109048.png)](https://www.hangge.com/blog/cache/detail_1445.html#)



这里我们通过 **UICollectionViewDelegateFlowLayout** 代理中的 **sizeForItemAt** 方法来设置调整每个单元格的尺寸，完整代码如下：

```swift
import UIKit
 
class ViewController: UICollectionViewController, UICollectionViewDelegateFlowLayout {
     
    //间距（1像素）
    let cellSpace = 1 / UIScreen.main.scale
     
    //列数
    let columnsNum = 7
     
    //单元格宽度（动态计算）
    var itemWidth:CGFloat!
     
    //剩余像素
    var remainderPx = 0
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.collectionView?.backgroundColor = UIColor.white
        let layout = self.collectionViewLayout as! UICollectionViewFlowLayout
         
        //水平间隔
        layout.minimumInteritemSpacing = cellSpace
        //垂直行间距
        layout.minimumLineSpacing = cellSpace
         
        //整个view的宽度
        let collectionViewWidth = self.collectionView!.bounds.width
        //整个view横向除去间距后，剩余的像素个数
        let pxWidth = collectionViewWidth * UIScreen.main.scale - CGFloat(columnsNum - 1)
         
        //单元格宽度（像素）
        let itemWidthPx = CGFloat(Int(pxWidth / CGFloat(columnsNum)))
        //单元格宽度（点）
        itemWidth = itemWidthPx / UIScreen.main.scale
         
        //剩余像素（作为左右内边距）
        remainderPx = Int(pxWidth - itemWidthPx * CGFloat(columnsNum))
 
    }
     
    //返回单元格数量
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return 50
    }
     
    //获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                             cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = (self.collectionView?.dequeueReusableCell(
            withReuseIdentifier: identify, for: indexPath))! as UICollectionViewCell
        let label = cell.contentView.viewWithTag(1) as! UILabel
        label.text = "\(indexPath.row)"
         
        return cell
    }
     
    //获取每个单元格的尺寸
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        //列索引
        let columnIndex = indexPath.row % columnsNum
        //cell宽度
        var width:CGFloat!
         
        //每行前面几个单元格宽度都加1个像素
        if columnIndex < remainderPx {
            width = itemWidth + cellSpace
             
            //前面最后一个做-0.001特殊处理，否则在plus设备下可能摆不下
            if columnIndex == (remainderPx-1) {
                width = itemWidth + cellSpace - 0.001
            }
        }
        //每行剩余单元格使用默认宽度
        else{
            width = itemWidth
        }
         
        return CGSize(width: width, height: itemWidth)
    }
     
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

## 功能改进2：在单元格内部绘制分隔线

上面的样例是将 **collectionView** 内部单元格间距设为 **1px**，这样 **collectionView** 的背景色透过间隙露出来就显示成 **1** 像素的内边框了。

我们还可以换种方式来实现，即将单元格间距设为 **0**，但是在单元格内部绘制边框线。而且每个单元格只绘制下方和右侧的线条（每行最后一个单元格不画右侧竖线），这样所有单元格拼接起来就成为一个 **1px** 细边框的 **GridView** 了。

[![原文:Swift - 实现内边框为1像素的collectionView（内间距为1px）](https://www.hangge.com/blog_uploads/201611/2016111721173999097.png)](https://www.hangge.com/blog/cache/detail_1445.html#)



（1）**MyCollectionViewCell.swift**（自定义单元格）

```swift
import UIKit
 
class MyCollectionViewCell: UICollectionViewCell {
    @IBOutlet weak var label: UILabel!
     
    //是否是最后一个单元格
    var isLastCell = false
     
    open override func draw(_ rect: CGRect) {
        super.draw(rect)
         
        //线宽
        let lineWidth = 1 / UIScreen.main.scale
         
        //线偏移量
        let lineOffset = 1 / UIScreen.main.scale / 2
         
        //获取绘图上下文
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        //创建一个矩形，它的下方右方内缩固定的偏移量
        let drawingRect = CGRect(x: self.bounds.minX, y: self.bounds.minY,
                                 width: self.bounds.width - (isLastCell ? 0:lineOffset),
                                 height: self.bounds.height - lineOffset)
         
        //创建并设置路径(单元格下方和右侧的线条)
        let path = CGMutablePath()
        path.move(to: CGPoint(x: drawingRect.minX, y: drawingRect.maxY))
        path.addLine(to: CGPoint(x: drawingRect.maxX, y: drawingRect.maxY))
        //最后一个单元格不画右侧的线
        if !isLastCell {
            path.addLine(to: CGPoint(x: drawingRect.maxX, y: drawingRect.minY))
        }
         
        //添加路径到图形上下文
        context.addPath(path)
         
        //设置笔触颜色
        context.setStrokeColor(UIColor.lightGray.cgColor)
        //设置笔触宽度
        context.setLineWidth(lineWidth)
         
        //绘制路径
        context.strokePath()
    }
}
```


（2）**ViewController.swift**（主页代码）

```swift
import UIKit
 
class ViewController: UICollectionViewController, UICollectionViewDelegateFlowLayout {
     
    //列数
    let columnsNum = 7
     
    //单元格宽度（动态计算）
    var itemWidth:CGFloat!
     
    //剩余像素
    var remainderPx = 0
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let layout = self.collectionViewLayout as! UICollectionViewFlowLayout
         
        //水平间隔设为0
        layout.minimumInteritemSpacing = 0
        //垂直行间距设为0
        layout.minimumLineSpacing = 0
         
        //整个view的宽度（像素）
        let pxWidth = self.collectionView!.bounds.width * UIScreen.main.scale
         
        //单元格宽度（像素）
        let itemWidthPx = CGFloat(Int(pxWidth / CGFloat(columnsNum)))
        //单元格宽度（点）
        itemWidth = itemWidthPx / UIScreen.main.scale
         
        //剩余像素（作为左右内边距）
        remainderPx = Int(pxWidth - itemWidthPx * CGFloat(columnsNum))
 
    }
     
    //返回单元格数量
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return 28
    }
     
    //获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "myCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = self.collectionView?.dequeueReusableCell(
            withReuseIdentifier: identify, for: indexPath) as! MyCollectionViewCell
        cell.label.text = "\(indexPath.row)"
        //判断是否是一行中的最后一个单元格
        if (indexPath.row + 1) % columnsNum == 0 {
            cell.isLastCell = true
        }else{
            cell.isLastCell = false
        }
        return cell
    }
     
    //获取每个单元格的尺寸
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        //列索引
        let columnIndex = indexPath.row % columnsNum
        //cell宽度
        var width:CGFloat!
         
        //每行前面几个单元格宽度都加1个像素
        if columnIndex < remainderPx {
            width = itemWidth + 1 / UIScreen.main.scale
             
            //前面最后一个做-0.001特殊处理，否则在plus设备下可能摆不下
            if columnIndex == (remainderPx-1) {
                width = itemWidth + 1 / UIScreen.main.scale - 0.001
            }
        }
        //每行剩余单元格使用默认宽度
        else{
            width = itemWidth
        }
         
        return CGSize(width: width, height: itemWidth)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_762.zip](https://www.hangge.com/blog_uploads/201611/2016111721270915480.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1445.html