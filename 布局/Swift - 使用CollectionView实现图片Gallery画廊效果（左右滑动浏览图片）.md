# Swift - 使用CollectionView实现图片Gallery画廊效果（左右滑动浏览图片）

2017-05-26发布：hangge阅读：4152

本文通过自定义 **UICollectionView** 布局，实现一个用于图片浏览和展示的画廊（**Gallery**）效果。

### 1，效果图

（1）图片从左至右横向排列（只有一行），通过手指拖动可以前后浏览图片。

（2）视图滚动时，每张图片根据其与屏幕中心距离的不同，显示尺寸也会相应地变化。越靠近屏幕中心尺寸就越大，远离屏幕中心的就逐渐变小。

（3）滑动结束后，会有位置自动修正的功能。即将当前最靠近屏幕中点的图片移动到正中央。

（4）点击图片则将该图片删除，点击空白处会在最开始的位置插入一张图片。不管新增还是删除都有动画效果。

（5）点击导航栏上的“**切换**”按钮，可以在普通的流式布局和我们自定义的画廊布局间相互切换。切换时也是有动画效果的。

  [![原文:Swift - 使用CollectionView实现图片Gallery画廊效果（左右滑动浏览图片）](https://www.hangge.com/blog_uploads/201703/2017031615442466525.png)](https://www.hangge.com/blog/cache/detail_1602.html#)  [![原文:Swift - 使用CollectionView实现图片Gallery画廊效果（左右滑动浏览图片）](https://www.hangge.com/blog_uploads/201703/2017031615454969005.png)](https://www.hangge.com/blog/cache/detail_1602.html#)  [![原文:Swift - 使用CollectionView实现图片Gallery画廊效果（左右滑动浏览图片）](https://www.hangge.com/blog_uploads/201703/2017031615444298544.png)](https://www.hangge.com/blog/cache/detail_1602.html#)



### 2，画廊布局类：LinearCollectionViewLayout

```
import UIKit
 
class LinearCollectionViewLayout: UICollectionViewFlowLayout {
     
    //元素宽度
    var itemWidth:CGFloat = 100
    //元素高度
    var itemHeight:CGFloat = 100
     
    //对一些布局的准备操作放在这里
    override func prepare() {
        super.prepare()
        //设置元素大小
        self.itemSize = CGSize(width: itemWidth, height: itemHeight)
        //设置滚动方向
        self.scrollDirection = .horizontal
        //设置间距
        self.minimumLineSpacing = self.collectionView!.bounds.width / 2 -  itemWidth
         
        //设置内边距
        //左右边距为了让第一张图片与最后一张图片出现在最中央
        //上下边距为了让图片横行排列，且只有一行
        let left = (self.collectionView!.bounds.width - itemWidth) / 2
        let top = (self.collectionView!.bounds.height - itemHeight) / 2
        self.sectionInset = UIEdgeInsetsMake(top, left, top, left)
    }
     
    //边界发生变化时是否重新布局（视图滚动的时候也会触发）
    //会重新调用prepareLayout和调用
    //layoutAttributesForElementsInRect方法获得部分cell的布局属性
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
     
    //rect范围下所有单元格位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
        //从父类得到默认的所有元素属性
        let array = super.layoutAttributesForElements(in: rect)
         
        //可见区域（目前显示出来的位于collection view上的矩形区域）
        let visiableRect = CGRect(x: self.collectionView!.contentOffset.x,
                                  y: self.collectionView!.contentOffset.y,
                                  width: self.collectionView!.frame.width,
                                  height: self.collectionView!.frame.height)
         
        //当前屏幕中点，相对于collect view上的x坐标
        let centerX = self.collectionView!.contentOffset.x
            + self.collectionView!.bounds.width / 2
         
        //这个是为了计算缩放比例的
        let maxDeviation = self.collectionView!.bounds.width / 2 + itemWidth / 2
         
        for attributes in array! {
            //与可见区域做碰撞，如果该单元格没显示则直接跳过
            if !visiableRect.intersects(attributes.frame) {continue}
            //显示的单元根据偏移量决定放大倍数（最大放大1.8倍，而离屏幕中央越远的单元格缩放的越小）
            let scale = 1 + (0.8 - abs(centerX - attributes.center.x) / maxDeviation)
            attributes.transform = CGAffineTransform(scaleX: scale, y: scale)
        }
         
        return array
    }
     
    /**
     用来设置collectionView停止滚动那一刻的位置(实现目的是当停止滑动，时刻有一张图片是位于屏幕最中央的)
     proposedContentOffset: 原本collectionView停止滚动那一刻的位置
     velocity:滚动速度
     返回：最终停留的位置
     */
    override func targetContentOffset(forProposedContentOffset
        proposedContentOffset: CGPoint, withScrollingVelocity velocity: CGPoint) -> CGPoint {
        //停止滚动时的可见区域
        let lastRect = CGRect(x: proposedContentOffset.x, y: proposedContentOffset.y,
                              width: self.collectionView!.bounds.width,
                              height: self.collectionView!.bounds.height)
        //当前屏幕中点，相对于collect view上的x坐标
        let centerX = proposedContentOffset.x + self.collectionView!.bounds.width * 0.5;
        //这个可见区域内所有的单元格属性
        let array = self.layoutAttributesForElements(in: lastRect)
         
        //需要移动的距离
        var adjustOffsetX = CGFloat(MAXFLOAT);
        for attri in array! {
            //每个单元格里中点的偏移量
            let deviation = attri.center.x - centerX
            //保存偏移最小的那个
            if abs(deviation) < abs(adjustOffsetX) {
                adjustOffsetX = deviation
            }
        }
        //通过偏移量返回最终停留的位置
        return CGPoint(x: proposedContentOffset.x + adjustOffsetX, y: proposedContentOffset.y)
    }
}
```



### 3，使用样例

（1）自定义单元格类：**MyCollectionViewCell.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 使用CollectionView实现图片Gallery画廊效果（左右滑动浏览图片）](https://www.hangge.com/blog_uploads/201703/2017031414420741968.png)](https://www.hangge.com/blog/cache/detail_1602.html#)

```
import UIKit
 
//自定义的Collection View单元格
class MyCollectionViewCell: UICollectionViewCell {
 
    //用于显示图片
    @IBOutlet weak var imageView: UIImageView!
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
}
```



（2）主视图代码

```
import UIKit
 
class ViewController: UIViewController {
    //普通的flow流式布局
    var flowLayout:UICollectionViewFlowLayout!
    //自定义的线性布局
    var linearLayput:LinearCollectionViewLayout!
     
    var collectionView:UICollectionView!
     
    //重用的单元格的Identifier
    let CellIdentifier = "myCell"
     
    //所有书籍数据
    var images = ["c#.png", "html.png", "java.png", "js.png", "php.png",
                  "react.png", "ruby.png", "swift.png", "xcode.png"]
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化Collection View
        initCollectionView()
         
        //注册tap点击事件
        let tapRecognizer = UITapGestureRecognizer(target: self,
                                    action: #selector(ViewController.handleTap(_:)))
        collectionView.addGestureRecognizer(tapRecognizer)
    }
     
    private func initCollectionView() {
        //初始化flow布局
        flowLayout = UICollectionViewFlowLayout()
        flowLayout.itemSize = CGSize(width: 60, height: 60)
        flowLayout.sectionInset = UIEdgeInsets(top: 74, left: 0, bottom: 0, right: 0)
         
        //初始化自定义布局
        linearLayput = LinearCollectionViewLayout()
         
        //初始化Collection View
        collectionView = UICollectionView(frame: view.bounds,
                                          collectionViewLayout: linearLayput)
         
        //Collection View代理设置
        collectionView.delegate = self
        collectionView.dataSource = self
        collectionView.backgroundColor = .white
         
        //注册重用的单元格
        let cellXIB = UINib.init(nibName: "MyCollectionViewCell", bundle: Bundle.main)
        collectionView.register(cellXIB, forCellWithReuseIdentifier: CellIdentifier)
         
        //将Collection View添加到主视图中
        view.addSubview(collectionView)
    }
     
    //点击手势响应
    func handleTap(_ sender:UITapGestureRecognizer){
        if sender.state == UIGestureRecognizerState.ended{
            let tapPoint = sender.location(in: self.collectionView)
            //点击的是单元格元素
            if let  indexPath = self.collectionView.indexPathForItem(at: tapPoint) {
                //通过performBatchUpdates对collectionView中的元素进行批量的插入，删除，移动等操作
                //同时该方法触发collectionView所对应的layout的对应的动画。
                self.collectionView.performBatchUpdates({ () -> Void in
                    self.collectionView.deleteItems(at: [indexPath])
                    self.images.remove(at: indexPath.row)
                }, completion: nil)
                 
            }
            //点击的是空白位置
            else{
                //新元素插入的位置（开头）
                let index = 0
                images.insert("xcode.png", at: index)
                self.collectionView.insertItems(at: [IndexPath(item: index, section: 0)])
            }
        }
    }
     
    //切换布局样式
    @IBAction func changeLayout(_ sender: Any) {
        self.collectionView.collectionViewLayout.invalidateLayout()
        //交替切换新布局
        let newLayout = collectionView.collectionViewLayout
            .isKind(of: LinearCollectionViewLayout.self) ? flowLayout : linearLayput
        collectionView.setCollectionViewLayout(newLayout!, animated: true)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//Collection View数据源协议相关方法
extension ViewController: UICollectionViewDataSource {
    //获取分区数
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 1
    }
     
    //获取每个分区里单元格数量
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return images.count
    }
     
    //返回每个单元格视图
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取重用的单元格
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier:
            CellIdentifier, for: indexPath) as! MyCollectionViewCell
        //设置内部显示的图片
        cell.imageView.image = UIImage(named: images[indexPath.item])
        return cell
    }
}
 
//Collection View样式布局协议相关方法
extension ViewController: UICollectionViewDelegate {
     
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1602.zip](https://www.hangge.com/blog_uploads/201703/2017031616031763277.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1602.html