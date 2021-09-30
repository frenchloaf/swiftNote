# Swift - 实现UICollectionView分组头悬停效果（方法2：使用iOS9新特性）

2017-05-05发布：hangge阅读：4187

前文提到（[点击查看](https://www.hangge.com/blog/cache/detail_1599.html)），在 **iOS9** 之前，如果要实现 **UICollectionView** 分区头（**section header**）的悬停效果，只能通过自定义布局来实现。

而从 **iOS9** 起，**UICollectionViewFlowLayout**（默认的流式布局）新增了两个属性：

- **sectionHeadersPinToVisibleBounds**：是否将分组头钉在可视区域
- **sectionFootersPinToVisibleBounds**：是否将分组尾钉在可视区域

有了这两个属性，我们不再需要自己实现相关的布局类，就可以实现分组头、分组尾的悬停效果。

（虽然这两个属性很方便，但有时为了实现一些特殊需求，比如想要用户滚动视图的时候逐渐淡化分组头直至消失，还是需要通过自定义布局来实现。）

### 1，效果图

可以看到，随着我滚动单元格视图，分组头都是固定在对应分组可视区域的顶部，不会跟着移动。

  [![原文:Swift - 实现UICollectionView分组头悬停效果（方法2：使用iOS9新特性）](https://www.hangge.com/blog_uploads/201703/2017031415311848458.png)](https://www.hangge.com/blog/cache/detail_1600.html#)   [![原文:Swift - 实现UICollectionView分组头悬停效果（方法2：使用iOS9新特性）](https://www.hangge.com/blog_uploads/201703/2017031415324795275.png)](https://www.hangge.com/blog/cache/detail_1600.html#)

### 2，样例代码

（1）自定义单元格类：**MyCollectionViewCell.swift**（创建的时候生成对应的 **xib** 文件）

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法2：使用iOS9新特性）](https://www.hangge.com/blog_uploads/201703/2017031414420741968.png)](https://www.hangge.com/blog/cache/detail_1600.html#)

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

[![原文:Swift - 实现UICollectionView分组头悬停效果（方法2：使用iOS9新特性）](https://www.hangge.com/blog_uploads/201703/2017031414481359494.png)](https://www.hangge.com/blog/cache/detail_1600.html#)

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



（3）使用样例：**ViewController.swift**

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
         
        //初始化Collection View
        initCollectionView()
    }
 
    private func initCollectionView() {
        //初始化flow布局
        let layout = UICollectionViewFlowLayout()
        //分组头悬停
        layout.sectionHeadersPinToVisibleBounds = true
         
        //初始化Collection View
        let collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
         
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

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1600.zip](https://www.hangge.com/blog_uploads/201703/2017031415284155364.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1600.html