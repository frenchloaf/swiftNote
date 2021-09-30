# Swift - 使用CollectionView实现图片Stack层叠效果

2017-05-29发布：hangge阅读：3177

本文通过自定义 **UICollectionView** 布局，实现一个用于图片层叠展示（**Stack**）效果。

### 1，效果图

（1）图片从下往上层层堆叠，第一张在最顶层，最后一张在最底部。

（2）每张图片会有一定的旋转角度，体现层叠的效果（最上面的一张不旋转）。

（3）点击图片则将该图片删除，点击空白处会在最上方插入一张图片。不管新增还是删除都有动画效果。

（5）点击导航栏上的“**切换**”按钮，可以在普通的流式布局和我们自定义的层叠布局间相互切换。切换时也是有动画效果的。

  [![原文:Swift - 使用CollectionView实现图片Stack层叠效果](https://www.hangge.com/blog_uploads/201703/2017031709271477356.png)](https://www.hangge.com/blog/cache/detail_1605.html#)   [![原文:Swift - 使用CollectionView实现图片Stack层叠效果](https://www.hangge.com/blog_uploads/201703/2017031709273085843.png)](https://www.hangge.com/blog/cache/detail_1605.html#)



### 2，层叠布局类：StackCollectionViewLayout

```swift
import UIKit
 
class StackCollectionViewLayout: UICollectionViewLayout {
     
    //元素尺寸（如果继承UICollectionViewFlowLayout的话，自动会有这个属性）
    var itemSize = CGSize(width:120, height:120)
     
    //角度（从上到下，每个元素依次使用）
    let angles: [CGFloat]  = [0, 0.2, -0.5, -0.2, 0.5]
     
    //边界发生变化时是否重新布局（视图滚动的时候也会触发）
    //会重新调用prepareLayout和调用
    //layoutAttributesForElementsInRect方法获得部分cell的布局属性
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
     
    //rect范围下所有单元格位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
            var attrArray: [UICollectionViewLayoutAttributes] = []
            let itemCount = self.collectionView!.numberOfItems(inSection: 0)
            for i in 0..<itemCount {
                let attr = self.layoutAttributesForItem(at: IndexPath(item: i, section: 0))!
                attrArray.append(attr)
            }
            return attrArray
    }
     
    //这个方法返回每个单元格的位置、大小、角度
    override func layoutAttributesForItem(at indexPath: IndexPath)
        -> UICollectionViewLayoutAttributes? {
        let attr = UICollectionViewLayoutAttributes(forCellWith: indexPath)
        attr.center = CGPoint(x:self.collectionView!.bounds.width / 2,
                              y:self.collectionView!.bounds.height / 2)
        attr.size = itemSize
        attr.transform = CGAffineTransform(rotationAngle:
            angles[indexPath.item % angles.count])
        //让第一张显示在最上面
        attr.zIndex = self.collectionView!.numberOfItems(inSection: 0) - indexPath.item
        return attr
    }
}
```



### 3，使用样例

（1）自定义单元格类：**MyCollectionViewCell.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 使用CollectionView实现图片Stack层叠效果](https://www.hangge.com/blog_uploads/201703/2017031709312595221.png)](https://www.hangge.com/blog/cache/detail_1605.html#)

```swift
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

```swift
import UIKit
 
class ViewController: UIViewController {
    //普通的flow流式布局
    var flowLayout:UICollectionViewFlowLayout!
    //自定义的层叠布局
    var stackLayput:StackCollectionViewLayout!
     
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
        stackLayput = StackCollectionViewLayout()
         
        //初始化Collection View
        collectionView = UICollectionView(frame: view.bounds,
                                          collectionViewLayout: stackLayput)
         
        //Collection View代理设置
        collectionView.delegate = self
        collectionView.dataSource = self
        collectionView.backgroundColor = .black
         
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
            .isKind(of: StackCollectionViewLayout.self) ? flowLayout : stackLayput
        collectionView.setCollectionViewLayout(newLayout, animated: true)
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1605.zip](https://www.hangge.com/blog_uploads/201703/2017031620350332242.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1605.html