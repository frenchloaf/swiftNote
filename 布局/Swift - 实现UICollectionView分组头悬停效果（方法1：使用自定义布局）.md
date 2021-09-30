# Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）

2017-05-03发布：hangge阅读：5871

我们知道表格 **UITableView** 可以设置多个 **section**（分区、分组）,而且如果 **tableView** 是使用 **plain** 样式的话，分组头还会有有 **sticky** 效果（粘性效果、悬停效果）。

而在 **iOS9** 之前，**UICollectionView** 虽然也可以设置多个 **section**，但其 **section header** 并没有悬停效果，而是跟随单元格一同上下移动。

  [![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201703/2017031414350479486.png)](https://www.hangge.com/blog/cache/detail_1599.html#)   [![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201703/2017031414351364232.png)](https://www.hangge.com/blog/cache/detail_1599.html#)

下面演示如何通过自定义布局类，来实现 **collectionView** 的分组头悬停效果。

### 1，效果图

可以看到随着视图上下滚动，当前分组的分组头会一直停留在固定位置（对应分组可视区域的顶端）。

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201703/2017031414383443714.png)](https://www.hangge.com/blog/cache/detail_1599.html#)

### 2，实现代码

（1）自定义单元格类：**MyCollectionViewCell.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201703/2017031414420741968.png)](https://www.hangge.com/blog/cache/detail_1599.html#)

```swift
import UIKit
 
//自定义的Collection View单元格
class MyCollectionViewCell: UICollectionViewCell {
 
    //用于显示书籍封面图片
    @IBOutlet weak var imageView: UIImageView!
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
}
```


（2）自定义分组头：**MySectionHeader.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201703/2017031414481359494.png)](https://www.hangge.com/blog/cache/detail_1599.html#)

```swift
import UIKit
 
//自定义的Collection View分组头
class MySectionHeader: UICollectionReusableView {
 
    //用于显示分组标题
    @IBOutlet weak var titleLabel: UILabel!
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
}
```



（3）自定义布局类：**StickyHeadersFlowLayout.swift**

```swift
import UIKit
 
//自定义的具有粘性分组头的Collection View布局类
class StickyHeadersFlowLayout: UICollectionViewFlowLayout {
     
    //边界发生变化时是否重新布局（视图滚动的时候也会调用）
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
     
    //所有元素的位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
        //从父类得到默认的所有元素属性
        guard let layoutAttributes = super.layoutAttributesForElements(in: rect)
            else { return nil }
         
        //用于存储元素新的布局属性,最后会返回这个
        var newLayoutAttributes = [UICollectionViewLayoutAttributes]()
        //存储每个layout attributes对应的是哪个section
        let sectionsToAdd = NSMutableIndexSet()
         
        //循环老的元素布局属性
        for layoutAttributesSet in layoutAttributes {
            //如果元素师cell
            if layoutAttributesSet.representedElementCategory == .cell {
                //将布局添加到newLayoutAttributes中
                newLayoutAttributes.append(layoutAttributesSet)
            } else if layoutAttributesSet.representedElementCategory == .supplementaryView {
                //将对应的section储存到sectionsToAdd中
                sectionsToAdd.add(layoutAttributesSet.indexPath.section)
            }
        }
         
        //遍历sectionsToAdd，补充视图使用正确的布局属性
        for section in sectionsToAdd {
            let indexPath = IndexPath(item: 0, section: section)
             
            //添加头部布局属性
            if let headerAttributes = self.layoutAttributesForSupplementaryView(ofKind:
                UICollectionElementKindSectionHeader, at: indexPath) {
                newLayoutAttributes.append(headerAttributes)
            }
             
            //添加尾部布局属性
            if let footerAttributes = self.layoutAttributesForSupplementaryView(ofKind:
                UICollectionElementKindSectionFooter, at: indexPath) {
                newLayoutAttributes.append(footerAttributes)
            }
        }
         
        return newLayoutAttributes
    }
     
    //补充视图的布局属性(这里处理实现粘性分组头,让分组头始终处于分组可视区域的顶部)
    override func layoutAttributesForSupplementaryView(ofKind elementKind: String,
                    at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        //先从父类获取补充视图的布局属性
        guard let layoutAttributes = super.layoutAttributesForSupplementaryView(ofKind:
            elementKind, at: indexPath) else { return nil }
         
        //如果不是头部视图则直接返回
        if elementKind != UICollectionElementKindSectionHeader {
            return layoutAttributes
        }
         
        //根据section索引，获取对应的边界范围
        guard let boundaries = boundaries(forSection: indexPath.section)
            else { return layoutAttributes }
        guard let collectionView = collectionView else { return layoutAttributes }
         
        //保存视图内入垂直方向的偏移量
        let contentOffsetY = collectionView.contentOffset.y
        //补充视图的frame
        var frameForSupplementaryView = layoutAttributes.frame
         
        //计算分组头垂直方向的最大最小值
        let minimum = boundaries.minimum - frameForSupplementaryView.height
        let maximum = boundaries.maximum - frameForSupplementaryView.height
         
        //如果内容区域的垂直偏移量小于分组头最小的位置，则将分组头置于其最小位置
        if contentOffsetY < minimum {
            frameForSupplementaryView.origin.y = minimum
        }
        //如果内容区域的垂直偏移量大于分组头最小的位置，则将分组头置于其最大位置
        else if contentOffsetY > maximum {
            frameForSupplementaryView.origin.y = maximum
        }
        //如果都不满足，则说明内容区域的垂直便宜量落在分组头的边界范围内。
        //将分组头设置为内容偏移量，从而让分组头固定在集合视图的顶部
        else {
            frameForSupplementaryView.origin.y = contentOffsetY
        }
         
        //更新布局属性并返回
        layoutAttributes.frame = frameForSupplementaryView
        return layoutAttributes
    }
     
    //根据section索引，获取对应的边界范围（返回一个元组）
    func boundaries(forSection section: Int) -> (minimum: CGFloat, maximum: CGFloat)? {
        //保存返回结果
        var result = (minimum: CGFloat(0.0), maximum: CGFloat(0.0))
         
        //如果collectionView属性为nil，则直接fanhui
        guard let collectionView = collectionView else { return result }
         
        //获取该分区中的项目数
        let numberOfItems = collectionView.numberOfItems(inSection: section)
         
        //如果项目数位0，则直接返回
        guard numberOfItems > 0 else { return result }
         
        //从流布局属性中获取第一个、以及最后一个项的布局属性
        let first = IndexPath(item: 0, section: section)
        let last = IndexPath(item: (numberOfItems - 1), section: section)
        if let firstItem = layoutAttributesForItem(at: first),
            let lastItem = layoutAttributesForItem(at: last) {
            //分别获区边界的最小值和最大值
            result.minimum = firstItem.frame.minY
            result.maximum = lastItem.frame.maxY
             
            //将分区都的高度考虑进去，并调整
            result.minimum -= headerReferenceSize.height
            result.maximum -= headerReferenceSize.height
             
            //将分区的内边距考虑进去，并调整
            result.minimum -= sectionInset.top
            result.maximum += (sectionInset.top + sectionInset.bottom)
        }
         
        //返回最终的边界值
        return result
    }
}
```



（4）使用样例：**ViewController.swift**

```swift
import UIKit
 
//每月书籍
struct BookPreview {
    var title:String
    var images:[String]
}
 
class ViewController: UIViewController {
     
    //重用的单元格和分区头的Identifier
    let CellIdentifier = "myCell"
    let HeaderIdentifier = "myHeader"
     
    //所有书籍数据
    let books = [
        BookPreview(title: "五月新书", images: ["0.jpg", "1.jpg","2.jpg", "3.jpg",
                                                    "4.jpg","5.jpg","6.jpg"]),
        BookPreview(title: "六月新书", images: ["7.jpg", "8.jpg", "9.jpg"]),
        BookPreview(title: "七月新书", images: ["10.jpg", "11.jpg", "12.jpg", "13.jpg"])
    ]
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //去除存在导航栏时内内边距自动调整功能，防止对自定义的Collection View分区头停留功能造成影响
        self.automaticallyAdjustsScrollViewInsets = false
         
        //初始化Collection View
        initCollectionView()
    }
 
    private func initCollectionView() {
        //初始化自定义的flow布局
        let layout = StickyHeadersFlowLayout()
         
        //Collection View的位置尺寸
        let frame = CGRect(x: 0, y: 64, width: view.bounds.width,
                           height: view.bounds.height - 64)
         
        //初始化Collection View
        let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
         
        //Collection View代理设置
        collectionView.delegate = self
        collectionView.dataSource = self
        collectionView.backgroundColor = .white
         
        //注册重用的单元格
        let cellXIB = UINib.init(nibName: "MyCollectionViewCell", bundle: Bundle.main)
        collectionView.register(cellXIB, forCellWithReuseIdentifier: CellIdentifier)
         
        //注册重用的分组头
        let headerXIB = UINib.init(nibName: "MySectionHeader", bundle: Bundle.main)
        collectionView.register(headerXIB, forSupplementaryViewOfKind:
            UICollectionElementKindSectionHeader, withReuseIdentifier: HeaderIdentifier)
         
        //将Collection View添加到主视图中
        view.addSubview(collectionView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//Collection View数据源协议相关方法
extension ViewController: UICollectionViewDataSource {
    //获取分区数
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return books.count
    }
     
    //获取每个分区里单元格数量
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return books[section].images.count
    }
     
    //返回每个单元格视图
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取重用的单元格
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier:
            CellIdentifier, for: indexPath) as! MyCollectionViewCell
        //设置内部显示的图片
        cell.imageView.image = UIImage(named: books[indexPath.section].images[indexPath.item])
        return cell
    }
     
    //分区的header
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind
        kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        // 获取重用的分组头
        if let header = collectionView.dequeueReusableSupplementaryView(ofKind:
            UICollectionElementKindSectionHeader, withReuseIdentifier: HeaderIdentifier,
                                                  for: indexPath) as? MySectionHeader {
            //设置分组标题
            header.titleLabel.text = books[indexPath.section].title
            return header
        }
        fatalError("获取重用视图失败!")
    }
     
}
 
//Collection View样式布局协议相关方法
extension ViewController: UICollectionViewDelegate, UICollectionViewDelegateFlowLayout {
    //返回分组头大小
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        referenceSizeForHeaderInSection section: Int) -> CGSize {
        return CGSize(width: collectionView.bounds.width, height: 45.0)
    }
     
    //返回单元格大小
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        let itemWidth = (collectionView.bounds.width - 5)/3
        let itemHeight = itemWidth / 3 * 4
        return CGSize(width: itemWidth, height: itemHeight)
    }
     
    //每个分组的内边距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        insetForSectionAt section: Int) -> UIEdgeInsets {
        return UIEdgeInsets.zero
    }
     
    //单元格的行间距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 2.0
    }
     
    //单元格横向的最小间距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
        return 0.0
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1599.zip](https://www.hangge.com/blog_uploads/201705/2017051817055191946.zip)

## 补充：给CollectionView添加个页眉

### 1，效果图

（1）**collectionView** 除了每个分组都有对应的分组头外，还有个整体的 **header view**。

（2）单元格上移时，整体的页眉也会上移，而分组头有悬停效果。

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201705/201705161523167964.png)](https://www.hangge.com/blog/cache/detail_1599.html#)[![原文:Swift - 实现UICollectionView分组头悬停效果（方法1：使用自定义布局）](https://www.hangge.com/blog_uploads/201705/2017051615224818756.png)](https://www.hangge.com/blog/cache/detail_1599.html#)

### 2，ViewController.swift（高亮部分为修改的地方）

```swift
import UIKit
 
//每月书籍
struct BookPreview {
    var title:String
    var images:[String]
}
 
class ViewController: UIViewController {
     
    //重用的单元格和分区头的Identifier
    let CellIdentifier = "myCell"
    let HeaderIdentifier = "myHeader"
     
    //所有书籍数据
    let books = [
        BookPreview(title: "五月新书", images: ["0.jpg", "1.jpg","2.jpg", "3.jpg",
                                                    "4.jpg","5.jpg","6.jpg"]),
        BookPreview(title: "六月新书", images: ["7.jpg", "8.jpg", "9.jpg"]),
        BookPreview(title: "七月新书", images: ["10.jpg", "11.jpg", "12.jpg", "13.jpg"])
    ]
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //去除存在导航栏时内内边距自动调整功能，防止对自定义的Collection View分区头停留功能造成影响
        self.automaticallyAdjustsScrollViewInsets = false
         
        //初始化Collection View
        initCollectionView()
    }
 
    private func initCollectionView() {
        //初始化自定义的flow布局
        let layout = StickyHeadersFlowLayout()
         
        //Collection View的位置尺寸
        let frame = CGRect(x: 0, y: 64, width: view.bounds.width,
                           height: view.bounds.height - 64)
         
        //初始化Collection View
        let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
         
        //Collection View代理设置
        collectionView.delegate = self
        collectionView.dataSource = self
        collectionView.backgroundColor = .white
         
        //注册重用的单元格
        let cellXIB = UINib.init(nibName: "MyCollectionViewCell", bundle: Bundle.main)
        collectionView.register(cellXIB, forCellWithReuseIdentifier: CellIdentifier)
         
        //注册重用的分组头
        let headerXIB = UINib.init(nibName: "MySectionHeader", bundle: Bundle.main)
        collectionView.register(headerXIB, forSupplementaryViewOfKind:
            UICollectionElementKindSectionHeader, withReuseIdentifier: HeaderIdentifier)
         
        //页眉高度
        let headerViewH:CGFloat = 60
        //创建页眉
        let headerView:UIView = UIView(frame:
            CGRect(x:0, y:-headerViewH, width:frame.size.width, height:60))
        let headerlabel:UILabel = UILabel(frame: headerView.bounds)
        headerlabel.textColor = UIColor.white
        headerlabel.backgroundColor = UIColor.clear
        headerlabel.font = UIFont.systemFont(ofSize: 16)
        headerlabel.text = "CollectionView 页眉"
        headerView.addSubview(headerlabel)
        headerView.backgroundColor = UIColor.black
 
        //将头部headView添加到collectionView
        collectionView.addSubview(headerView)
        //插入位置
        collectionView.contentInset = UIEdgeInsets(top: headerViewH, left: 0, bottom: 0, right: 0)
         
        //将Collection View添加到主视图中
        view.addSubview(collectionView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//Collection View数据源协议相关方法
extension ViewController: UICollectionViewDataSource {
    //获取分区数
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return books.count
    }
     
    //获取每个分区里单元格数量
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return books[section].images.count
    }
     
    //返回每个单元格视图
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取重用的单元格
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier:
            CellIdentifier, for: indexPath) as! MyCollectionViewCell
        //设置内部显示的图片
        cell.imageView.image = UIImage(named: books[indexPath.section].images[indexPath.item])
        return cell
    }
     
    //分区的header
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind
        kind: String, at indexPath: IndexPath) -> UICollectionReusableView {
        // 获取重用的分组头
        if let header = collectionView.dequeueReusableSupplementaryView(ofKind:
            UICollectionElementKindSectionHeader, withReuseIdentifier: HeaderIdentifier,
                                                  for: indexPath) as? MySectionHeader {
            //设置分组标题
            header.titleLabel.text = books[indexPath.section].title
            return header
        }
        fatalError("获取重用视图失败!")
    }
     
}
 
//Collection View样式布局协议相关方法
extension ViewController: UICollectionViewDelegate, UICollectionViewDelegateFlowLayout {
    //返回分组头大小
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        referenceSizeForHeaderInSection section: Int) -> CGSize {
        return CGSize(width: collectionView.bounds.width, height: 45.0)
    }
     
    //返回单元格大小
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        let itemWidth = (collectionView.bounds.width - 5)/3
        let itemHeight = itemWidth / 3 * 4
        return CGSize(width: itemWidth, height: itemHeight)
    }
     
    //每个分组的内边距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        insetForSectionAt section: Int) -> UIEdgeInsets {
        return UIEdgeInsets.zero
    }
     
    //单元格的行间距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 2.0
    }
     
    //单元格横向的最小间距
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
        return 0.0
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1599.html