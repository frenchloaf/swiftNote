# Swift - 多列表格组件的实现（样例3：表头、列头固定）

2017-05-12发布：hangge阅读：2222

**相关文章系列：**
[Swift - 多列表格组件的实现（样例1：基本功能的实现）](https://www.hangge.com/blog/cache/detail_1081.html)
[Swift - 多列表格组件的实现（样例2：带排序功能）](https://www.hangge.com/blog/cache/detail_1090.html)
[当前文章] Swift - 多列表格组件的实现（样例3：表头、列头固定）
[Swift - 多列表格组件的实现（样例4：表格样式美化）](https://www.hangge.com/blog/cache/detail_1678.html)


在之前的文章中，我介绍了如何使用 **CollectionView** 来实现一个多列表格组件。当时表头和内容单元格是连在一起，一同随视图滚动的。但如果数据很多，表头滚动上去后，要查看下面数据的类别和字段就会很麻烦。
同样的，如果表格横向的数据列过多，需要左右拖动才能看到全部。那么可能还会需要实现个首列固定功能。即不管表格如何左右滚动，第一列总会显示在屏幕最左侧。
下面分别介绍这两个功能的实现。



## 一、表头固定

### 1，效果图

当表格数据行数超过显示区域时，不管上下如何滚动，表头始终固定并显示在最上方位置。

  [![原文:Swift - 多列表格组件的实现（样例3：表头、列头固定）](https://www.hangge.com/blog_uploads/201705/2017051111095299197.png)](https://www.hangge.com/blog/cache/detail_1677.html#)   [![原文:Swift - 多列表格组件的实现（样例3：表头、列头固定）](https://www.hangge.com/blog_uploads/201705/2017051111101397318.png)](https://www.hangge.com/blog/cache/detail_1677.html#)

### 2，项目代码

**UICollectionGridViewController.swift**（组件类）、**UICollectionGridViewCell.swift**（组件单元格类）、**ViewController.swift**（测试类）同前文一样，这里就不再说明了。

**---** **UICollectionGridViewLayout.swift****（布局类） ---**

代码中高亮部分即实现表头固定功能需要添加或修改的代码：

- **prepare()** 方法：将第一行的所有单元格根据当前内容纵向偏移量设置 **Y** 轴坐标。并调整这些单元的 **zIndex** 使其在最上层，防止滚动时被内容单元格覆盖。
- **shouldInvalidateLayout()** 方法：改为边界发生任何改变时（包括滚动条改变），都应该刷新布局。

```swift
import Foundation
import UIKit
 
//多列表格组件布局类
class UICollectionGridViewLayout: UICollectionViewLayout {
    //记录每个单元格的布局属性
    private var itemAttributes: [[UICollectionViewLayoutAttributes]] = []
    private var itemsSize: [NSValue] = []
    private var contentSize: CGSize = CGSize.zero
    //表格组件视图控制器
    var viewController: UICollectionGridViewController!
     
    //准备所有view的layoutAttribute信息
    override func prepare() {
        if collectionView!.numberOfSections == 0 {
            return
        }
         
        var column = 0
        var xOffset: CGFloat = 0
        var yOffset: CGFloat = 0
        var contentWidth: CGFloat = 0
        var contentHeight: CGFloat = 0
         
        if itemAttributes.count > 0 {
            return
        }
         
        itemAttributes = []
        itemsSize = []
         
        if itemsSize.count != viewController.cols.count {
            calculateItemsSize()
        }
         
        for section in 0 ..< (collectionView?.numberOfSections)! {
            var sectionAttributes: [UICollectionViewLayoutAttributes] = []
            for index in 0 ..< viewController.cols.count {
                let itemSize = itemsSize[index].cgSizeValue
                 
                let indexPath = IndexPath(item: index, section: section)
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                //除第一列，其它列位置都左移一个像素，防止左右单元格间显示两条边框线
                if index == 0{
                    attributes.frame = CGRect(x:xOffset, y:yOffset, width:itemSize.width,
                                              height:itemSize.height).integral
                }else {
                    attributes.frame = CGRect(x:xOffset-1, y:yOffset,
                                              width:itemSize.width+1,
                                              height:itemSize.height).integral
                }
                 
                //如果是第一行（表头）
                if section == 0 {
                    //列头位置固定
                    var frame = attributes.frame
                    frame.origin.y = self.collectionView!.contentOffset.y
                    attributes.frame = frame
                    //列头单元格处于最顶层
                    attributes.zIndex = 1024
                }
                 
                sectionAttributes.append(attributes)
                 
                xOffset = xOffset+itemSize.width
                column += 1
                 
                if column == viewController.cols.count {
                    if xOffset > contentWidth {
                        contentWidth = xOffset
                    }
                     
                    column = 0
                    xOffset = 0
                    yOffset += itemSize.height
                }
            }
            itemAttributes.append(sectionAttributes)
        }
         
        let attributes = itemAttributes.last!.last! as UICollectionViewLayoutAttributes
        contentHeight = attributes.frame.origin.y + attributes.frame.size.height
        contentSize = CGSize(width:contentWidth, height:contentHeight)
    }
     
    //需要更新layout时调用
    override func invalidateLayout() {
        itemAttributes = []
        itemsSize = []
        contentSize = CGSize.zero
        super.invalidateLayout()
    }
     
    // 返回内容区域总大小，不是可见区域
    override var collectionViewContentSize: CGSize {
        get {
            return contentSize
        }
    }
     
    // 这个方法返回每个单元格的位置和大小
    override func layoutAttributesForItem(at indexPath: IndexPath)
        -> UICollectionViewLayoutAttributes? {
        return itemAttributes[indexPath.section][indexPath.row]
    }
     
    // 返回所有单元格位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
            var attributes: [UICollectionViewLayoutAttributes] = []
            for section in itemAttributes {
                attributes.append(contentsOf: section.filter(
                    {(includeElement: UICollectionViewLayoutAttributes) -> Bool in
                        return rect.intersects(includeElement.frame)
                }))
            }
            return attributes
    }
     
    //改为边界发生任何改变时（包括滚动条改变），都应该刷新布局。
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
     
    //计算所有单元格的尺寸（每一列各一个单元格）
    func calculateItemsSize() {
        var remainingWidth = collectionView!.frame.width -
            collectionView!.contentInset.left - collectionView!.contentInset.right
         
        var index = viewController.cols.count-1
        while index >= 0 {
            let newItemSize = sizeForItemWithColumnIndex(columnIndex: index,
                                                         remainingWidth: remainingWidth)
            remainingWidth -= newItemSize.width
            let newItemSizeValue = NSValue(cgSize: newItemSize)
            //由于遍历列的时候是从尾部开始遍历了，因此将结果插入数组的时候都是放人第一个位置
            itemsSize.insert(newItemSizeValue, at: 0)
            index -= 1
        }
    }
     
    //计算某一列的单元格尺寸
    func sizeForItemWithColumnIndex(columnIndex: Int, remainingWidth: CGFloat) -> CGSize {
        let columnString = viewController.cols[columnIndex]
        //根据列头标题文件，估算各列的宽度
        let size = NSString(string: columnString).size(attributes: [
            NSFontAttributeName:UIFont.systemFont(ofSize: 15),
            NSUnderlineStyleAttributeName:NSUnderlineStyle.styleSingle.rawValue
            ])
         
        //修改成所有列都平均分配
        let width = remainingWidth/CGFloat(columnIndex+1)
        //计算好的宽度还要取整，避免偏移
        return CGSize(width: ceil(width), height:size.height + 10)
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1090.zip](https://www.hangge.com/blog_uploads/201705/2017051111261648648.zip)



## 二、表头、首列均固定

### 1，效果图

除了上下滚动表头固定外。左右拖动滚动条，首列也是固定不动的。

   [![原文:Swift - 多列表格组件的实现（样例3：表头、列头固定）](https://www.hangge.com/blog_uploads/201705/2017051114202270987.png)](https://www.hangge.com/blog/cache/detail_1677.html#)    [![原文:Swift - 多列表格组件的实现（样例3：表头、列头固定）](https://www.hangge.com/blog_uploads/201705/2017051114202775894.png)](https://www.hangge.com/blog/cache/detail_1677.html#)

### 2，项目代码

**UICollectionGridViewController.swift**（组件类）、**UICollectionGridViewCell.swift**（组件单元格类）、**ViewController.swift**（测试类）同前文一样，这里就不再说明了。

**---** **UICollectionGridViewLayout.swift****（布局类） ---**

代码中高亮部分即实现表头、列头固定功能需要添加或修改的代码：

- **prepare()** 方法：将第一行的所有单元格根据当前内容纵向偏移量设置 **Y** 轴坐标，第一列的所有单元格根据横向偏移量设置 **X** 轴坐标。并调整这些单元的 **zIndex** 使其在最上层，防止滚动时被内容单元格覆盖。
- **shouldInvalidateLayout()** 方法：改为边界发生任何改变时（包括滚动条改变），都应该刷新布局。
- **sizeForItemWithColumnIndex()** 方法：这里给每个单元格设置个最小宽度，让横向内容区域超出屏幕，从而出现横向滚动条。

```swift
import Foundation
import UIKit
 
//多列表格组件布局类
class UICollectionGridViewLayout: UICollectionViewLayout {
    //记录每个单元格的布局属性
    private var itemAttributes: [[UICollectionViewLayoutAttributes]] = []
    private var itemsSize: [NSValue] = []
    private var contentSize: CGSize = CGSize.zero
    //表格组件视图控制器
    var viewController: UICollectionGridViewController!
     
    //准备所有view的layoutAttribute信息
    override func prepare() {
        if collectionView!.numberOfSections == 0 {
            return
        }
         
        var column = 0
        var xOffset: CGFloat = 0
        var yOffset: CGFloat = 0
        var contentWidth: CGFloat = 0
        var contentHeight: CGFloat = 0
         
        if itemAttributes.count > 0 {
            return
        }
         
        itemAttributes = []
        itemsSize = []
         
        if itemsSize.count != viewController.cols.count {
            calculateItemsSize()
        }
         
        for section in 0 ..< (collectionView?.numberOfSections)! {
            var sectionAttributes: [UICollectionViewLayoutAttributes] = []
            for index in 0 ..< viewController.cols.count {
                let itemSize = itemsSize[index].cgSizeValue
                 
                let indexPath = IndexPath(item: index, section: section)
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                //除第一列，其它列位置都左移一个像素，防止左右单元格间显示两条边框线
                if index == 0{
                    attributes.frame = CGRect(x:xOffset, y:yOffset, width:itemSize.width,
                                              height:itemSize.height).integral
                }else {
                    attributes.frame = CGRect(x:xOffset-1, y:yOffset,
                                              width:itemSize.width+1,
                                              height:itemSize.height).integral
                }
                 
                //将表头、首列单元格置为最顶层
                if section == 0 && index == 0 {
                    attributes.zIndex = 1024
                }else if section == 0 || index == 0 {
                    attributes.zIndex = 1023
                }
                 
                //表头单元格位置固定
                if section == 0 {
                    var frame = attributes.frame
                    frame.origin.y = self.collectionView!.contentOffset.y
                    attributes.frame = frame
                }
                //首列单元格位置固定
                if index == 0 {
                    var frame = attributes.frame
                    frame.origin.x = self.collectionView!.contentOffset.x
                        + collectionView!.contentInset.left
                    attributes.frame = frame
                }
                 
                sectionAttributes.append(attributes)
                 
                xOffset = xOffset+itemSize.width
                column += 1
                 
                if column == viewController.cols.count {
                    if xOffset > contentWidth {
                        contentWidth = xOffset
                    }
                     
                    column = 0
                    xOffset = 0
                    yOffset += itemSize.height
                }
            }
            itemAttributes.append(sectionAttributes)
        }
         
        let attributes = itemAttributes.last!.last! as UICollectionViewLayoutAttributes
        contentHeight = attributes.frame.origin.y + attributes.frame.size.height
        contentSize = CGSize(width:contentWidth, height:contentHeight)
    }
     
    //需要更新layout时调用
    override func invalidateLayout() {
        itemAttributes = []
        itemsSize = []
        contentSize = CGSize.zero
        super.invalidateLayout()
    }
     
    // 返回内容区域总大小，不是可见区域
    override var collectionViewContentSize: CGSize {
        get {
            return contentSize
        }
    }
     
    // 这个方法返回每个单元格的位置和大小
    override func layoutAttributesForItem(at indexPath: IndexPath)
        -> UICollectionViewLayoutAttributes? {
        return itemAttributes[indexPath.section][indexPath.row]
    }
     
    // 返回所有单元格位置属性
    override func layoutAttributesForElements(in rect: CGRect)
        -> [UICollectionViewLayoutAttributes]? {
            var attributes: [UICollectionViewLayoutAttributes] = []
            for section in itemAttributes {
                attributes.append(contentsOf: section.filter(
                    {(includeElement: UICollectionViewLayoutAttributes) -> Bool in
                        return rect.intersects(includeElement.frame)
                }))
            }
            return attributes
    }
     
    //改为边界发生任何改变时（包括滚动条改变），都应该刷新布局。
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
     
    //计算所有单元格的尺寸（每一列各一个单元格）
    func calculateItemsSize() {
        var remainingWidth = collectionView!.frame.width -
            collectionView!.contentInset.left - collectionView!.contentInset.right
         
        var index = viewController.cols.count-1
        while index >= 0 {
            let newItemSize = sizeForItemWithColumnIndex(columnIndex: index,
                                                         remainingWidth: remainingWidth)
            remainingWidth -= newItemSize.width
            let newItemSizeValue = NSValue(cgSize: newItemSize)
            //由于遍历列的时候是从尾部开始遍历了，因此将结果插入数组的时候都是放人第一个位置
            itemsSize.insert(newItemSizeValue, at: 0)
            index -= 1
        }
    }
     
    //计算某一列的单元格尺寸
    func sizeForItemWithColumnIndex(columnIndex: Int, remainingWidth: CGFloat) -> CGSize {
        let columnString = viewController.cols[columnIndex]
        //根据列头标题文件，估算各列的宽度
        let size = NSString(string: columnString).size(attributes: [
            NSFontAttributeName:UIFont.systemFont(ofSize: 15),
            NSUnderlineStyleAttributeName:NSUnderlineStyle.styleSingle.rawValue
            ])
         
        //修改成所有列都平均分配（但宽度不能小于90）
        let width = max(remainingWidth/CGFloat(columnIndex+1), 90)
        //计算好的宽度还要取整，避免偏移
        return CGSize(width: ceil(width), height:size.height + 10)
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1090.zip](https://www.hangge.com/blog_uploads/201705/2017051116373955754.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1677.html