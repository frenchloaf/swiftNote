# Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）

2017-04-19发布：hangge阅读：10161

在表格 **tableview** 初始化的时候我们可以指定需要使用的 **UITableViewStyle** 样式，可用的样式一共有两种：**.plain** 和 **.grouped**。下面分别对它们做介绍。

## 一、plain模式

### 1，默认样式

- 在 **plain** 模式下，如果 **tableview** 有多个 **section**（分区、分组），组与组之间默认是没有间距的。
- 同时组头或组尾会有 **sticky** 效果（粘性效果、悬停效果）,即表格滚动时组头与组尾会自动停留，而不是跟随单元格一同移动。

[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/201703131528529084.png)](https://www.hangge.com/blog/cache/detail_1598.html#)[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031315290170250.png)](https://www.hangge.com/blog/cache/detail_1598.html#)

```swift
import UIKit

class ViewController: UIViewController , UITableViewDelegate, UITableViewDataSource{
     
    var tableView:UITableView?
     
    //分组头标题
    var articleHeaders:[String]!
    //所有文章标题
    var articleNames:Dictionary<Int, [String]>!
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化数据
        self.articleNames =  [
            0:[String]([
                "1、文本标签（UILabel）的用法",
                "2、按钮（UIButton）的用法",
                "3、文本输入框（UITextField）的用法",
                "4、多行文本输入框（UITextView）的用法",
                "5、开关按钮（UISwitch）的用法",
                "6、分段选择控件（UISegmentedControl）的用法",
                "7、图像控件（UIImageView）的用法",
                ]),
            1:[String]([
                "1、使用占位符文本placeholder添加文本框提示",
                "2、使用autofocus让控件自动获取焦点",
                "3、表单客户端验证",
                "4、日期和时间选择输入",
                "5、颜色选择器",])
        ]
         
        self.articleHeaders = [
            "Swift文章",
            "HTML5文章"
        ]
         
        //创建表视图
        self.tableView = UITableView(frame:self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
    }
     
    //在本例中，有2个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return self.articleHeaders.count
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        let data = self.articleNames[section]
        return data!.count
    }
     
    // UITableViewDataSource协议中的方法，该方法的返回值决定指定分区的头部
    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int)
        -> String? {
            return self.articleHeaders[section]
    }
     
    // UITableViewDataSource协议中的方法，该方法的返回值决定指定分区的尾部
    func tableView(_ tableView:UITableView, titleForFooterInSection section:Int)->String? {
        let data = self.articleNames[section]
        return "有\(data!.count)篇文章"
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            var data = self.articleNames[indexPath.section]
            cell.textLabel?.text = data![indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

### 2，调整分组间的间距

如果需要设置组与组之间的间距，可以通过 **viewForHeaderInSection** 、**viewForFooterInSection** 、**heightForHeaderInSection** 或 **heightForFooterInSection** 这几个方法配合实现。

### 3，去除分组头、分组尾的停留效果

这个通过重写 **tableView** 的 **scrollViewDidScroll** 方法可以实现。要注意的是页面是否有导航控制器，有的话要把自动内边距调整给考虑进去。

（1）分组头部不悬停

```swift
//header不悬停
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    //组头高度
    let sectionHeaderHeight:CGFloat = 30
    //获取是否有默认调整的内边距
    let defaultEdgeTop:CGFloat = navigationController?.navigationBar != nil
        && self.automaticallyAdjustsScrollViewInsets ? 64 : 0
     
    if scrollView.contentOffset.y >= -defaultEdgeTop &&
        scrollView.contentOffset.y <= sectionHeaderHeight - defaultEdgeTop  {
        scrollView.contentInset = UIEdgeInsetsMake(-scrollView.contentOffset.y, 0, 0, 0)
    }
    else if (scrollView.contentOffset.y>=sectionHeaderHeight - defaultEdgeTop) {
        scrollView.contentInset = UIEdgeInsetsMake(-sectionHeaderHeight + defaultEdgeTop,
                                                   0, 0, 0)
    }
}
```


（2）分组尾部不悬停

```swift
//footer不悬停
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    //组尾高度
    let sectionFooterHeight:CGFloat = 30
     
    //获取是否有默认调整的内边距
    let defaultEdgeTop:CGFloat = navigationController?.navigationBar != nil
        && self.automaticallyAdjustsScrollViewInsets ? 64 : 0
     
    let b = scrollView.contentOffset.y + scrollView.frame.height
    let h = scrollView.contentSize.height - sectionFooterHeight
     
    if b <= h {
        scrollView.contentInset = UIEdgeInsetsMake(defaultEdgeTop, 0, -30, 0)
    }else if b > h && b < scrollView.contentSize.height {
         scrollView.contentInset = UIEdgeInsetsMake(defaultEdgeTop, 0, b - h - 30, 0)
    }
}
```


（3）分组头部、尾部均不悬停

```swift
//header、footer均不悬停
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    //组头高度
    let sectionHeaderHeight:CGFloat = 30
    //组尾高度
    let sectionFooterHeight:CGFloat = 30
     
    //获取是否有默认调整的内边距
    let defaultEdgeTop:CGFloat = navigationController?.navigationBar != nil
        && self.automaticallyAdjustsScrollViewInsets ? 64 : 0
     
    //上边距相关
    var edgeTop = defaultEdgeTop
    if scrollView.contentOffset.y >= -defaultEdgeTop &&
        scrollView.contentOffset.y <= sectionHeaderHeight - defaultEdgeTop  {
        edgeTop = -scrollView.contentOffset.y
    }
    else if (scrollView.contentOffset.y>=sectionHeaderHeight - defaultEdgeTop) {
        edgeTop = -sectionHeaderHeight + defaultEdgeTop
    }
     
    //下边距相关
    var edgeBottom:CGFloat = 0
    let b = scrollView.contentOffset.y + scrollView.frame.height
    let h = scrollView.contentSize.height - sectionFooterHeight
     
    if b <= h {
        edgeBottom = -30
    }else if b > h && b < scrollView.contentSize.height {
        edgeBottom = b - h - 30
    }
     
    //设置内边距
    scrollView.contentInset = UIEdgeInsetsMake(edgeTop, 0, edgeBottom, 0)
}
```



## 二、grouped模式

### 1，默认样式

- 在 **grouped** 模式下，如果 **tableview** 有多个 **section**（分区、分组），组与组之间默认是有间距的。
- 而且在表格滚动的同时组头与组尾会随之滚动、不停留，不会有 **sticky** 效果（粘性效果、悬停效果）。

下面分别是：分区头尾均有、只有分区头、分区头尾都没有。这三种情况：
[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031319195488000.png)](https://www.hangge.com/blog/cache/detail_1598.html#)[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031319334699127.png)](https://www.hangge.com/blog/cache/detail_1598.html#)[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031319335959627.png)](https://www.hangge.com/blog/cache/detail_1598.html#)



### 2，去掉多余的间距

（1）在分组头、分组尾都存在时，可以将 **tableview** 最上方的间距给去除。

[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031320375974492.png)](https://www.hangge.com/blog/cache/detail_1598.html#)

```swift
//去除表格上放多余的空隙
self.tableView?.contentInset = UIEdgeInsetsMake(-20, 0, 0, 0)
```


（2）如果只有分组头，没有分组尾，除了将 **tableview** 最上方的间距给去除，还可以将分组尾的高度设置为 **0.01**（不能设为 **0**，否则无效）。同时还要将分组尾设置成一个空的 **UIView**（否则在 **iOS11** 下分组尾高度不会起作用）。

[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031320152095689.png)](https://www.hangge.com/blog/cache/detail_1598.html#)



```swift
//去除表格上放多余的空隙
self.tableView?.contentInset = UIEdgeInsetsMake(-20, 0, 0, 0)
 
//设置分组尾的高度
func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
    return 0.01
}
 
//将分组尾设置为一个空的View
func tableView(_ tableView: UITableView, viewForFooterInSection section: Int) -> UIView? {
    return UIView()
}
```


（3）如果分组头、分组尾均没有，还可以将分组头的高度设置为 **0.01**。

[![原文:Swift - 表格UITableView的plain、grouped两种样式详解（附分组头悬停）](https://www.hangge.com/blog_uploads/201703/2017031320265429375.png)](https://www.hangge.com/blog/cache/detail_1598.html#)



```swift
func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
    return 0.01
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1598.html