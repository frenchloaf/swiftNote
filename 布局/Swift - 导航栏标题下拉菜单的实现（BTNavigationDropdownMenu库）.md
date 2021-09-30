# Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）

2017-07-17发布：hangge阅读：4810

导航栏中间的文字除了用来显示当前页面的标题外，有时还可以作为下拉菜单的入口，这个在许多新闻类的 **App** 上比较常见。用户点击标题后，下方就会出现下拉选择菜单，点击菜单项即可对内容进行筛选或者跳转到相应的页面。

我们自己可以通过继承 **UIView** 来实现一个下拉菜单组件，也可以直接使用网上一些第三方组件库，本介绍的 **BTNavigationDropdownMenu** 就是其中比较优秀的一个。

### 1，安装配置

（1）进入 **GitHub** 主页将源码包下载到本地，地址：https://github.com/PhamBaTho/BTNavigationDropdownMenu

（2）将 **Source** 文件夹中的所有文件添加到我们项目中了。

[![原文:Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）](https://www.hangge.com/blog_uploads/201707/2017071419170025680.png)](https://www.hangge.com/blog/cache/detail_1654.html#)

### 2，简单的使用样例

（1）效果图

- 点击导航栏标题后，下方会出现下拉菜单。
- 点击菜单项后自动收起下拉菜单，同时标题和页面文字都会改变。
- 下拉菜单的显示和隐藏都有动画效果。

   [![原文:Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）](https://www.hangge.com/blog_uploads/201707/2017071419575174522.png)](https://www.hangge.com/blog/cache/detail_1654.html#)     [![原文:Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）](https://www.hangge.com/blog_uploads/201707/2017071419514049666.png)](https://www.hangge.com/blog/cache/detail_1654.html#)

（2）样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
 
    //界面正中的文本标签
    @IBOutlet weak var contentLabel: UILabel!
     
    //下拉菜单
    var menuView: BTNavigationDropdownMenu!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //导航栏不透明
        self.navigationController?.navigationBar.isTranslucent = false
        //导航栏背景色（下拉菜单栏也用同样的颜色）
        self.navigationController?.navigationBar.barTintColor = UIColor.orange
        //导航栏文字使用白色
        self.navigationController?.navigationBar.titleTextAttributes =
            [NSForegroundColorAttributeName: UIColor.white]
         
        //下拉菜单项
        let items = ["中华料理", "西餐面点", "东南亚菜", "韩国泡菜"]
         
        //创建下拉菜单
        menuView = BTNavigationDropdownMenu(navigationController: self.navigationController,
                                            containerView: self.navigationController!.view,
                                            title: "美食", items: items)
        //单元格文字颜色
        menuView.cellTextLabelColor = UIColor.white
        //单元格背景色
        menuView.cellBackgroundColor = self.navigationController?.navigationBar.barTintColor
        //选中项背景色
        menuView.cellSelectionColor = UIColor(red: 0xff/255, green: 0xca/255,
                                              blue: 0x00/255, alpha: 1)
        //保持选中项的颜色
        menuView.shouldKeepSelectedCellColor = true
    
        //下拉菜单项选中事件响应
        menuView.didSelectItemAtIndexHandler = {(indexPath: Int) -> Void in
            print("当前点击项的索引: \(indexPath)")
            self.contentLabel.text = items[indexPath]
        }
         
        //将下拉菜单设置为titleView
        self.navigationItem.titleView = menuView
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1654.zip](https://www.hangge.com/blog_uploads/201707/2017071420042348699.zip)

### 3，设置默认选中的菜单项

上面样例中，程序初始化后默认是不选中任何菜单项的。如果想要默认就选中某个菜单项，只需在初始化时指定菜单项索引即可。

（1）效果图

默认选中第 **2** 个菜单项，注意导航栏标题也会同步更新。

[![原文:Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）](https://www.hangge.com/blog_uploads/201707/2017071420020612861.png)](https://www.hangge.com/blog/cache/detail_1654.html#)

（2）样例代码

```swift
//创建下拉菜单
menuView = BTNavigationDropdownMenu(title: BTTitle.index(1), items: items)
```

### 4，组件方法

- **menuView.show()**：手动显示下拉菜单
- **menuView.hide()**：手动隐藏下拉菜单
- **menuView.toggle()**：交替切换下拉菜单的显示和隐藏
- **menuView.isShown**：获取下拉菜单当前的显示状态（是否显示）
- **menuView.updateItems(items: [AnyObject])**：更新下拉下单项

### 5，自定义样式

通过修改如下这些 **BTNavigationDropdownMenu** 对象属性，我们可以实现样式的自定义：

- **cellHeight**：单元格高度（默认值：**50**）
- **cellBackgroundColor**：单元格背景色（默认值：**white**）
- **cellSeparatorColor**：单元格分割线颜色（默认值：**darkGray**）
- **cellTextLabelColor**：单元格文字颜色（默认值：**darkGray**）
- **cellTextLabelFont**：单元格文字字体（默认值：**HelveticaNeue-Bold**, **size 17**）
- **navigationBarTitleFont**：导航栏标题文字字体（默认值：**HelveticaNeue-Bold**, **size 17**）
- **cellTextLabelAlignment**：单元格文字对齐方式（默认值：**.left**）
- **cellSelectionColor**：选中单元格的背景色（默认值：**lightGray**）
- **checkMarkImage**：选中单元格使用的勾选图片
- **animationDuration**：展开/收缩下拉菜单动画持续时间（默认值：**0.5s**）
- **arrowImage**：导航栏标题旁边的箭头图片
- **arrowPadding**：导航栏标题与旁边箭头之间的空隙（默认值：**15**）
- **maskBackgroundColor**：下拉菜单展开后背景遮罩的颜色（默认值：**black**）
- **maskBackgroundOpacity**：下拉菜单展开后背景遮罩的透明度（默认值：**0.3**）
- **menuTitleColor**：下拉菜单项文字颜色（默认值：**lightGray**）
- **shouldKeepSelectedCellColor**：是否一直保持选中单元格的颜色（默认值：**false**）
- **shouldChangeTitleText**：选中后是否改变标题文字（默认值：**true**）
- **selectedCellTextLabelColor**：选中单元格的文字颜色（默认值：**darkGray**）
- **arrowTintColor**：箭头颜色（默认值：**white**）



## 附：使用 CocoaPods 安装库后编译报错的问题解决

### 1，问题描述

上面的样例中我是直接把源码下载下来，然后手动添加到项目中。当然同样也可以使用 **CocoaPods** 来添加 **BTNavigationDropdownMenu** 库。

（1）比如我们项目的 **Podfile** 文件内容如下：

```swift
use_frameworks!
 
target 'hangge_1654' do
    pod 'BTNavigationDropdownMenu'
end
```


（2）**pod install** 后打开项目编译，会发现有 **83** 个编译错误。

[![原文:Swift - 导航栏标题下拉菜单的实现（BTNavigationDropdownMenu库）](https://www.hangge.com/blog_uploads/201708/2017083114442777030.png)](https://www.hangge.com/blog/cache/detail_1654.html#)

### 2，问题解决

只要将 **Podfile** 文件内容修改成如下，然后重新 **pod install** 即可。

```swift
use_frameworks!
 
target 'hangge_1654' do
  pod 'BTNavigationDropdownMenu', :git =>
  'https://github.com/PhamBaTho/BTNavigationDropdownMenu.git'
end
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1654.html