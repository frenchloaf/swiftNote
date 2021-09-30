# Swift - 修改tableView分组（section）头部、尾部的字体颜色和大小

2016-10-14发布：hangge阅读：10288

我原来写过一篇关于表格（**tableView**）分组（**section**）的文章：[Swift - 使用表格组件（UITableView）实现分组列表](https://www.hangge.com/blog/cache/detail_557.html)。默认情况下，每个 **section** 头、尾的样式如下：

[![原文:Swift - 修改tableView分组（section）头部、尾部的字体颜色和大小](https://www.hangge.com/blog_uploads/201610/2016100920535333149.png)](https://www.hangge.com/blog/cache/detail_1391.html#)

如果想要改变分组头尾里文字的颜色、大小样式。通常有两种方法。

**1，通过UIAppearance协议统一修改（UIAppearance Protocol）**

使用这种方式会将 **App** 中所有的 **tableView** 的分组头部、尾部文本标签样式进行统一修改。比如下面代码，我们将标签颜色改成黑色斜体，并加大字号。

**注意**：这种方法只有在 **tableView** 不超过屏幕的情况下可用。如果超过屏幕长度（出现滚动条），就会因为重用问题导致设置失效（又还原成默认样式）。



```swift
override func viewDidLoad() {
    super.viewDidLoad()
    //......
     
    //设置分区头尾的文字颜色：黑色
    UILabel.appearance(whenContainedInInstancesOf: [UITableViewHeaderFooterView.self])
        .textColor = UIColor.black
    //设置分区头尾的文字样式：15号斜体
    UILabel.appearance(whenContainedInInstancesOf: [UITableViewHeaderFooterView.self])
        .font = UIFont.italicSystemFont(ofSize: 15)
}
```

效果图如下：

[![原文:Swift - 修改tableView分组（section）头部、尾部的字体颜色和大小](https://www.hangge.com/blog_uploads/201610/2016100920553987900.png)](https://www.hangge.com/blog/cache/detail_1391.html#)

### 2，在 willDisplayHeaderView 和 willDisplayFooterView 这两个协议方法中修改

通过 **willDisplayHeaderView** 和 **willDisplayFooterView** 方法可以分别获取到即将要显示的分组头视图和分组尾视图，从而对它内部的文字样式进行修改。

比如下面代码将分组头部文字改成橙色，并居中显示。

```swift
//分组头即将要显示
func tableView(_ tableView: UITableView, willDisplayHeaderView view: UIView,
               forSection section: Int) {
    guard let header = view as? UITableViewHeaderFooterView else { return }
    header.textLabel?.textColor = UIColor.orange
    header.textLabel?.font = UIFont.boldSystemFont(ofSize: 18)
    header.textLabel?.frame = header.frame
    header.textLabel?.textAlignment = .center
}
```

效果图如下：

[![原文:Swift - 修改tableView分组（section）头部、尾部的字体颜色和大小](https://www.hangge.com/blog_uploads/201810/2018102317020434212.png)](https://www.hangge.com/blog/cache/detail_1391.html#)



**3，使用viewForHeaderInSection或viewForFooterInSection单独修改**
通过 **viewForHeaderInSection** 和 **viewForFooterInSection** 方法可以单独为每一个表格，甚至每个分组自定义不同的头尾视图，自然可以实现文字的样式自定义。
比如下面代码将分组头部改成白色文字，黑色背景。

```swift
//返回分区头部视图
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    let headerView = UIView()
    headerView.backgroundColor = UIColor.black
    let titleLabel = UILabel()
    titleLabel.text = self.adHeaders?[section]
    titleLabel.textColor = UIColor.white
    titleLabel.sizeToFit()
    titleLabel.center = CGPoint(x: self.view.frame.width/2, y: 20)
    headerView.addSubview(titleLabel)
    return headerView
}
 
//返回分区头部高度
func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
    return 40
}
```

效果图如下：

[![原文:Swift - 修改tableView分组（section）头部、尾部的字体颜色和大小](https://www.hangge.com/blog_uploads/201610/201610092058114791.png)](https://www.hangge.com/blog/cache/detail_1391.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1391.html