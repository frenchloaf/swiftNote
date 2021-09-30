# Swift - 实现UICollectionView中元素的环形布局

2017-05-10发布：hangge阅读：2957

本文通过继承 **UICollectionViewLayout** 来自定义一个 **collection view** 的布局类，从而实现其内部元素的圆形排列（环形排列）

### 1，效果图

（1）初始化时，**collectionview** 内的图标以 **3** 点钟位置为起点，顺时针方向按圆形轨迹围成一个圈（每个图标间距相等）

（2）点击页面空白处会在随机位置插入一个图标，点击对应图标则会将其删除。

（3）不管添加还是删除图标元素，都会有动画效果。

（4）点击导航栏的“**切换**”按钮，则可在普通的流式布局和我们自定义的环形布局间互相切换。

  [![原文:Swift - 实现UICollectionView中元素的环形布局](https://www.hangge.com/blog_uploads/201703/2017031510384430850.png)](https://www.hangge.com/blog/cache/detail_1601.html#)  [![原文:Swift - 实现UICollectionView中元素的环形布局](https://www.hangge.com/blog_uploads/201703/2017031510384964220.png)](https://www.hangge.com/blog/cache/detail_1601.html#)  [![原文:Swift - 实现UICollectionView中元素的环形布局](https://www.hangge.com/blog_uploads/201703/2017031510385448926.png)](https://www.hangge.com/blog/cache/detail_1601.html#)



### 2，环状布局类：LoopCollectionViewLayout

```swift
import UIKit
 
//环形布局样式
class LoopCollectionViewLayout: UICollectionViewLayout {
     
    //元素尺寸
    var itemSize:CGFloat = 60.0
     
    //元素个数
    fileprivate var _itemCount:Int?
    //整个collection view视图尺寸
    fileprivate var _viewSize:CGSize?
    //整个collection view视图中心点位置
    fileprivate var _center:CGPoint?
    //圆环路径的半径
    fileprivate var _radius:CGFloat?
 
    fileprivate var insertIndexPaths = [IndexPath]()
     
    //设定一些必要的layout的结构和初始需要的参数
    override func prepare() {
        super.prepare()
        _viewSize = self.collectionView?.frame.size
        _itemCount = self.collectionView?.numberOfItems(inSection: 0)
        _center = CGPoint(x: _viewSize!.width / 2.0, y: _viewSize!.height / 2.0)
        _radius = (min(_viewSize!.width, _viewSize!.height) - itemSize) / 2
    }
     
    //所有单元格位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
        var attributesArray = [UICollectionViewLayoutAttributes]()
        if let count = self._itemCount {
            for i in 0 ..< count{
                //这里利用layoutAttributesForItem方法来获取attributes
                let indexPath = IndexPath(item: i, section: 0)
                let attributes =  self.layoutAttributesForItem(at: indexPath)
                attributesArray.append(attributes!)
            }
        }
        return attributesArray
    }
     
    //这个方法返回每个单元格的位置和大小
    override func layoutAttributesForItem(at indexPath: IndexPath)
        -> UICollectionViewLayoutAttributes? {
        let attrs = UICollectionViewLayoutAttributes(forCellWith: indexPath)
        attrs.size = CGSize(width: itemSize, height: itemSize)
        let x = Double(_center!.x) + Double(_radius!)
            * cos(Double(2 * indexPath.item) * M_PI/Double(_itemCount!))
        let y = Double(_center!.y) + Double(_radius!)
            * sin(Double(2 * indexPath.item) * M_PI/Double(_itemCount!))
        attrs.center = CGPoint(x: CGFloat(x), y: CGFloat(y))
        return attrs
    }
     
 
    //插入或者删除 item的时候，collection view将会通知布局对象它将调整布局
    override func prepare(forCollectionViewUpdates
        updateItems: [UICollectionViewUpdateItem]) {
        super.prepare(forCollectionViewUpdates: updateItems)
        self.insertIndexPaths = [IndexPath]()
        //这里我们将新插入IndexPath存储起来，后面设置动画效果要用
        for update in updateItems{
            if update.updateAction == UICollectionUpdateAction.insert{
                self.insertIndexPaths.append(update.indexPathAfterUpdate!)
            }
        }
    }
     
    //删除或添加item时该方法会调用,返回开始的布局信息（在prepareforCollectionViewUpdates后调用）
    override func initialLayoutAttributesForAppearingItem(at itemIndexPath: IndexPath)
        -> UICollectionViewLayoutAttributes? {
        var attributes = super.initialLayoutAttributesForAppearingItem(at: itemIndexPath)
        //如果时插入元素，插入时设置最开始的一些属性（插入过程中自动会有动画效果）
        if self.insertIndexPaths.contains(itemIndexPath) {
             
            if let _ = attributes{
                attributes = self.layoutAttributesForItem(at: itemIndexPath)
            }
             
            //动画开始时该元素是全透明的，位置在视图中央
            attributes!.alpha = 0.0
            attributes!.center =  _center!
        }
        return attributes
    }
}
```



### 3，使用样例

（1）自定义单元格类：**MyCollectionViewCell.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 实现UICollectionView中元素的环形布局](https://www.hangge.com/blog_uploads/201703/2017031414420741968.png)](https://www.hangge.com/blog/cache/detail_1601.html#)

```swift
import UIKit
 
//自定义的Collection View单元格
class MyCollectionViewCell: UICollectionViewCell {
 
    //用于显示图标图片
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
    //自定义的环形布局
    var loopLayput:LoopCollectionViewLayout!
     
    var collectionView:UICollectionView!
     
    //重用的单元格的Identifier
    let CellIdentifier = "myCell"
     
    //所有书籍数据
    var images = ["c#.png", "html.png", "java.png", "js.png", "php.png",
                  "react.png", "ruby.png", "swift.png"]
     
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
        loopLayput = LoopCollectionViewLayout()
         
        //初始化Collection View
        collectionView = UICollectionView(frame: view.bounds,
                                          collectionViewLayout: loopLayput)
         
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
                print("------")
                self.collectionView.performBatchUpdates({ () -> Void in
                    self.collectionView.deleteItems(at: [indexPath])
                    self.images.remove(at: indexPath.row)
                }, completion: nil)
                 
            }
                //点击的是空白位置
            else{
                //新元素插入的位置（随机）
                let index = Int(arc4random_uniform(UInt32(images.count)))
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
            .isKind(of: LoopCollectionViewLayout.self) ? flowLayout : loopLayput
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
        //设置遮罩
        cell.imageView.layer.masksToBounds = true
        //设置圆角半径(宽度的一半)，显示成圆形。
        cell.imageView.layer.cornerRadius = cell.bounds.width/2
        return cell
    }
     
}
 
//Collection View样式布局协议相关方法
extension ViewController: UICollectionViewDelegate {
     
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1601.zip](https://www.hangge.com/blog_uploads/201703/2017031510510249488.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1601.html