# Swift - 多列表格组件的实现（样例4：表格样式美化）

2017-05-15发布：hangge阅读：4117

**相关文章系列：**
[Swift - 多列表格组件的实现（样例1：基本功能的实现）](https://www.hangge.com/blog/cache/detail_1081.html)
[Swift - 多列表格组件的实现（样例2：带排序功能）](https://www.hangge.com/blog/cache/detail_1090.html)
[Swift - 多列表格组件的实现（样例3：表头、列头固定）](https://www.hangge.com/blog/cache/detail_1677.html)
[当前文章] Swift - 多列表格组件的实现（样例4：表格样式美化）

在之前的几篇文章里，我主要演示如何使用 **CollectionView** 实现多列表格组件。而对于表格的样式没有特别设置，只是简单地加个单元格边框，看起来不太美观。

本文对表格样式做个美化，当然功能方面没有修改。

### 1，效果图

- 表头背景使用绿色，文字使用白色。
- 表格内容背景使用白色、灰色交替显示。
- 点击列头进行排序，同时该列内容单元格背景色变成淡蓝色。
- 去掉所有单元格的边框，使表格更加清爽。

  [![原文:Swift - 多列表格组件的实现（样例4：表格样式美化）](https://www.hangge.com/blog_uploads/201705/2017051115514414163.png)](https://www.hangge.com/blog/cache/detail_1678.html#)   [![原文:Swift - 多列表格组件的实现（样例4：表格样式美化）](https://www.hangge.com/blog_uploads/201705/2017051115512770691.png)](https://www.hangge.com/blog/cache/detail_1678.html#)   [![原文:Swift - 多列表格组件的实现（样例4：表格样式美化）](https://www.hangge.com/blog_uploads/201705/201705111551505365.png)](https://www.hangge.com/blog/cache/detail_1678.html#)

### 2，功能说明

- 表头固定，上下拖动滚动条表头位置不变。
- 首列固定，左右拖动滚动条首列位置不变。
- 点击列头标题，内容条目便会根据该列数据进行排序显示（先升序、后降序，依次交替）

### 3，项目代码

**--- UICollectionGridViewController.swift（组件类） ---**

```swift
import Foundation
import UIKit
 
//表格排序协议
protocol UICollectionGridViewSortDelegate: class {
    func sort(colIndex: Int, asc: Bool, rows: [[Any]]) -> [[Any]]
}
 
//多列表格组件（通过CollectionView实现）
class UICollectionGridViewController: UICollectionViewController {
    //表头数据
    var cols: [String]! = []
    //行数据
    var rows: [[Any]]! = []
     
    //排序代理
    weak var sortDelegate: UICollectionGridViewSortDelegate?
     
    //选中的表格列（-1表示没有选中的）
    private var selectedColIdx = -1
    //列排序顺序
    private var asc = true
     
    init() {
        //初始化表格布局
        let layout = UICollectionGridViewLayout()
        super.init(collectionViewLayout: layout)
        layout.viewController = self
        collectionView!.backgroundColor = UIColor.white
        collectionView!.register(UICollectionGridViewCell.self,
                                      forCellWithReuseIdentifier: "cell")
        collectionView!.delegate = self
        collectionView!.dataSource = self
        collectionView!.isDirectionalLockEnabled = true
        //collectionView!.contentInset = UIEdgeInsetsMake(0, 10, 0, 10)
        collectionView!.bounces = false
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("UICollectionGridViewController.init(coder:) has not been implemented")
    }
     
    //设置列头数据
    func setColumns(columns: [String]) {
        cols = columns
    }
     
    //添加行数据
    func addRow(row: [Any]) {
        rows.append(row)
        collectionView!.collectionViewLayout.invalidateLayout()
        collectionView!.reloadData()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    override func viewDidLayoutSubviews() {
        collectionView!.frame = CGRect(x:0, y:0,
                                       width:view.frame.width, height:view.frame.height)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
     
    //返回表格总行数
    override func numberOfSections(in collectionView: UICollectionView) -> Int {
            if cols.isEmpty {
                return 0
            }
            //总行数是：记录数＋1个表头
            return rows.count + 1
    }
     
    //返回表格的列数
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        return cols.count
    }
     
    //单元格内容创建
    override func collectionView(_ collectionView: UICollectionView,
                            cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell",
                                            for: indexPath) as! UICollectionGridViewCell
         
        //设置列头单元格，内容单元格的数据
        if indexPath.section == 0 {
            cell.label.font = UIFont.systemFont(ofSize: 15, weight: UIFontWeightBold)
            cell.label.text = cols[indexPath.row]
            cell.label.textColor = UIColor.white
        } else {
            cell.label.font = UIFont.systemFont(ofSize: 15)
            cell.label.text = "\(rows[indexPath.section-1][indexPath.row])"
            cell.label.textColor = UIColor.black
        }
         
        //表头单元格背景色
        if indexPath.section == 0 {
            cell.backgroundColor = UIColor(red: 0x91/255, green: 0xDA/255,
                                           blue: 0x51/255, alpha: 1)
            //排序列列头显示升序降序图标
            if indexPath.row == selectedColIdx {
                let iconType = asc ? FAType.FALongArrowUp : FAType.FALongArrowDown
                cell.imageView.setFAIconWithName(icon: iconType, textColor: UIColor.white)
            }else{
                cell.imageView.image = nil
            }
        }
        //内容单元格背景色
        else {
            //排序列的单元格背景会变色
            if indexPath.row == selectedColIdx {
                //排序列的单元格背景会变色
                cell.backgroundColor = UIColor(red: 0xCC/255, green: 0xF8/255,
                                               blue: 0xFF/255, alpha: 1)
            }
            //数据区域每行单元格背景色交替显示
            else if indexPath.section % 2 == 0 {
                cell.backgroundColor = UIColor(white: 242/255.0, alpha: 1)
            } else {
                cell.backgroundColor = UIColor.white
            }
        }
         
        return cell
    }
     
    //单元格选中事件
    override func collectionView(_ collectionView: UICollectionView,
                                 didSelectItemAt indexPath: IndexPath) {
        //打印出点击单元格的［行,列］坐标
        print("点击单元格的[行,列]坐标: [\(indexPath.section),\(indexPath.row)]")
        if indexPath.section == 0 && sortDelegate != nil {
            //如果点击的是表头单元格，则默认该列升序排列，再次点击则变降序排列，以此交替
            asc = (selectedColIdx != indexPath.row) ? true : !asc
            selectedColIdx = indexPath.row
            rows = sortDelegate?.sort(colIndex: indexPath.row, asc: asc, rows: rows)
            collectionView.reloadData()
        }
    }
}
```



**--- UICollectionGridViewCell.swift（组件单元格类） ---**

```swift
import UIKit
 
class UICollectionGridViewCell: UICollectionViewCell {
     
    //内容标签
    var label:UILabel!
     
    //箭头图标
    var imageView:UIImageView!
     
    //标签左边距
    var paddingLeft:CGFloat = 5
     
    override init(frame: CGRect) {
        super.init(frame: frame)
         
        self.backgroundColor = UIColor.white
        self.clipsToBounds = true
         
        //添加内容标签
        self.label = UILabel(frame: .zero)
        self.label.textAlignment = .center
        self.addSubview(self.label)
         
        //添加箭头图标
        self.imageView = UIImageView(frame: .zero)
        self.addSubview(self.imageView)
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    override func layoutSubviews() {
        super.layoutSubviews()
     
        label.frame = CGRect(x: paddingLeft, y: 0,
                             width: frame.width - paddingLeft * 2,
                             height: frame.height)
         
        let imageWidth: CGFloat = 14
        let imageHeight: CGFloat = 14
        imageView.frame = CGRect(x:frame.width - imageWidth,
                                 y:frame.height/2 - imageHeight/2,
                                 width:imageWidth, height:imageHeight)
    }
}
```

**--- UICollectionGridViewLayout.swift（布局类） ---**

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
                attributes.frame = CGRect(x:xOffset, y:yOffset, width:itemSize.width,
                                              height:itemSize.height).integral
                 
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

**--- ViewController.swift（测试类） ---**

```swift
import UIKit
 
class ViewController: UIViewController, UICollectionGridViewSortDelegate {
     
    var gridViewController: UICollectionGridViewController!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        gridViewController = UICollectionGridViewController()
        gridViewController.setColumns(columns: ["编号","客户", "消费金额", "消费次数", "满意度"])
        gridViewController.addRow(row: ["No.01","hangge", "100", "8", "60%"])
        gridViewController.addRow(row: ["No.02","张三", "223", "16", "81%"])
        gridViewController.addRow(row: ["No.03","李四", "143", "25", "93%"])
        gridViewController.addRow(row: ["No.04","王五", "75", "2", "53%"])
        gridViewController.addRow(row: ["No.05","韩梅梅", "43", "12", "33%"])
        gridViewController.addRow(row: ["No.06","李雷", "33", "27", "45%"])
        gridViewController.addRow(row: ["No.07","王大力", "33", "22", "15%"])
        gridViewController.addRow(row: ["No.08","蝙蝠侠", "100", "8", "60%"])
        gridViewController.addRow(row: ["No.09","超人", "223", "16", "81%"])
        gridViewController.addRow(row: ["No.10","钢铁侠", "143", "25", "93%"])
        gridViewController.addRow(row: ["No.11","灭霸", "75", "2", "53%"])
        gridViewController.addRow(row: ["No.12","快银", "43", "12", "33%"])
        gridViewController.addRow(row: ["No.13","闪电侠", "33", "27", "45%"])
        gridViewController.addRow(row: ["No.14","绿箭", "33", "22", "15%"])
        gridViewController.addRow(row: ["No.15","绿巨人", "223", "16", "81%"])
        gridViewController.addRow(row: ["No.16","黑寡妇", "143", "25", "93%"])
        gridViewController.addRow(row: ["No.17","企鹅人", "75", "2", "53%"])
        gridViewController.addRow(row: ["No.18","双面人", "43", "12", "33%"])
        gridViewController.addRow(row: ["No.19","奥特曼", "33", "27", "45%"])
        gridViewController.addRow(row: ["No.20","小怪兽s", "33", "22", "15%"])
         
        gridViewController.sortDelegate = self
        view.addSubview(gridViewController.view)
    }
     
    override func viewDidLayoutSubviews() {
        gridViewController.view.frame = CGRect(x:0, y:64, width:view.frame.width,
                                               height:view.frame.height-64)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
     
    //表格排序函数
    func sort(colIndex: Int, asc: Bool, rows: [[Any]]) -> [[Any]] {
        let sortedRows = rows.sorted { (firstRow: [Any], secondRow: [Any])
            -> Bool in
            let firstRowValue = firstRow[colIndex] as! String
            let secondRowValue = secondRow[colIndex] as! String
            if colIndex == 0 || colIndex == 1 {
                //首例、姓名使用字典排序法
                if asc {
                    return firstRowValue < secondRowValue
                }
                return firstRowValue > secondRowValue
            } else if colIndex == 2 || colIndex == 3 {
                //中间两列使用数字排序
                if asc {
                    return Int(firstRowValue)! < Int(secondRowValue)!
                }
                return Int(firstRowValue)! > Int(secondRowValue)!
            }
            //最后一列数据先去掉百分号，再转成数字比较
            let firstRowValuePercent = Int(firstRowValue.substring(to:
                firstRowValue.index(before: firstRowValue.endIndex)))!
            let secondRowValuePercent = Int(secondRowValue.substring(to:
                secondRowValue.index(before: secondRowValue.endIndex)))!
            if asc {
                return firstRowValuePercent < secondRowValuePercent
            }
            return firstRowValuePercent > secondRowValuePercent
        }
        return sortedRows
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1090.zip](https://www.hangge.com/blog_uploads/201805/2018052611110912097.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1678.html