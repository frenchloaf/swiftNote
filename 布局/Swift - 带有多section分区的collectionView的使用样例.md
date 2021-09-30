# Swift - 带有多section分区的collectionView的使用样例

2017-03-27发布：hangge阅读：11112

**UICollectionView** 组件和 **UITableView** 组件一样，也是可以设置多个 **section**（分区、分组）。每个 **section** 我们可以设置不同的 **header** 和 **footer**，同时每个分区内还可以显示不同数量和内容的单元格。

虽然我们也可以在 **tableViewCell** 中嵌入 **collectionView** 来实现同样的效果，但会更加麻烦些，特别是如果每个 **collectionView** 内单元格数量不同，**collectionView** 还要实现动态高度。

所以如果能满足需求的话，还是建议使用多 **section** 的 **collectionView** 来实现。

### 1，效果图

（1）**collectView** 的每一个分区对应一个月份的图书列表。

（2）分区头显示月份标题。分区尾背景设为灰色，内容为空。

（3）分区内单元格显示当月所有书籍封面图片，数量不定，分区高度自适应。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/201703111009459942.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



### 2，实现步骤

（1）打开 **StoryBoard**，删除默认的 **View Controller**。拖入一个 **Collection View Controller**，并将其设置为初始 **VC**。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109125662192.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（2）选中这个 **Scene**，点击菜单“**Editor**”->“**Embed In**”->“**Navigation Controller**”添加一个导航控制器。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109154423321.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（3）选中 **Collection View**，在其属性面板中保持 **Layout** 属性为 **Flow** 不变。勾选上“**Section Header**”和“**Section Footer**”，分别增加分区头和分区尾。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109174193710.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（4）调整好头部的尺寸，并将其 **Identifier** 设置为 **HeaderView**

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109570647491.png)](https://www.hangge.com/blog/cache/detail_1592.html#)

（5）往头部中拖入一个 **Label** 并设置好相关约束。同时将 **Label** 的 **Tag** 设置为 **1**。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109245969688.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（6）调整好分区尾部的尺寸，将其背景色设为灰色。同时将其 **Identifier** 设置为 **FooterView**。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109593421873.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（7）调整好单元格尺寸，将其 **Identifier** 设置为 **MyCell**

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031110032565990.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（8）往单元格里拖入一个 **ImageView**，并设置好相关约束。同时将 **ImageView** 的 **Tag** 设为 **1**。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109332596414.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（9）选中 **Collection View**，在尺寸样式面包中设置好内边距和单元格间距。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109370195366.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（10）新建一个 **CollectionViewController** 类（继承 **UICollectionViewController**）。并在 **StoryBoard** 中作绑定。

[![原文:Swift - 带有多section分区的collectionView的使用样例](https://www.hangge.com/blog_uploads/201703/2017031109405367984.png)](https://www.hangge.com/blog/cache/detail_1592.html#)



（11）**CollectionViewController.swift** 完整代码如下

```swift
import UIKit
 
//每月书籍
struct BookPreview {
    var title:String
    var images:[String]
}
 
class CollectionViewController: UICollectionViewController {
     
    //所有书籍数据
    let books = [
        BookPreview(title: "五月新书", images: ["0.jpg", "1.jpg","2.jpg", "3.jpg",
                                               "4.jpg","5.jpg","6.jpg"]),
        BookPreview(title: "六月新书", images: ["7.jpg", "8.jpg", "9.jpg"]),
        BookPreview(title: "七月新书", images: ["10.jpg", "11.jpg", "12.jpg", "13.jpg"])
    ]
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //获取分区数
    override func numberOfSections(in collectionView: UICollectionView) -> Int {
        return books.count
    }
     
    //获取每个分区里单元格数量
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return books[section].images.count
    }
     
    //分区的header与footer
    override func collectionView(_ collectionView: UICollectionView,
                                 viewForSupplementaryElementOfKind kind: String,
                                 at indexPath: IndexPath) -> UICollectionReusableView {
        var reusableview:UICollectionReusableView!
         
        //分区头
        if kind == UICollectionElementKindSectionHeader{
            reusableview = collectionView.dequeueReusableSupplementaryView(ofKind: kind,
                                        withReuseIdentifier: "HeaderView", for: indexPath)
            //设置头部标题
            let label = reusableview.viewWithTag(1) as! UILabel
            label.text = books[indexPath.section].title
        }
        //分区尾
        else if kind == UICollectionElementKindSectionFooter{
            reusableview = collectionView.dequeueReusableSupplementaryView(ofKind: kind,
                                        withReuseIdentifier: "FooterView", for: indexPath)
             
        }
        return reusableview
    }
     
    //返回每个单元格视图
    override func collectionView(_ collectionView: UICollectionView,
                            cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取单元格
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "MyCell",
                                                      for: indexPath)
        //设置单元格中的图片
        let imageView = cell.viewWithTag(1) as! UIImageView
        imageView.image = UIImage(named: books[indexPath.section].images[indexPath.item])
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1592.zip](https://www.hangge.com/blog_uploads/201703/2017031110082952246.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1592.html