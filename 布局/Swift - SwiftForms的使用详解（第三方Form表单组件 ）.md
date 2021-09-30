# Swift - SwiftForms的使用详解（第三方Form表单组件 ）

2017-06-27发布：hangge阅读：5120

## 一、基本介绍

### 1，什么是SwiftForms?

**SwiftForms** 是一个强大且灵活的第三方 **Swift** 组件库。它可以轻松地通过几行代码创建复杂表单，同时也能很方便地定制单元格显示样式。

### 2，安装配置

（1）从 **GitHub** 上下载最新的代码：https://github.com/ortuman/SwiftForms

（2）将下载下来的源码包中 **SwiftFormsApplication.xcodeproj** 拖拽至你的工程中

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017061920314227390.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



（3）工程 -> **General** -> **Embedded Binaries** 项，把 **SwiftForms.framework** 添加进来。 

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017061920310439433.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



（4）最后，在需要使用 **SwiftForms** 的地方 **import** 进来就可以了。

```
import` `SwiftForms
```



## 二、使用说明

**SwiftForms** 使用起来十分简单，只需让视图控制器继承 **FormViewController**。然后定义一个 **FormDescriptor** 实例，最后再定义 **FormDescriptor** 相关的 **section** 及其 **rows** 即可。

### 1，效果图

（1）这里实现一个最简单的登录表单，一共两个分区（**Section**）：

- 第一个分区有两个表单项，分别可以输入普通文字和密码。
- 第二个分区只有一个按钮。

  [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510591014843.png)](https://www.hangge.com/blog/cache/detail_1720.html#)   [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510591925614.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

（2）点击登录按钮，如果当前表单正在编辑输入则停止编辑。同时获取最新的表单值，并打印出来。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062511005019148.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



### 2，实现步骤

（1）首先继承 **FormViewController** 创建一个视图控制器（**MyFormViewController**），其内容如下：

```swift
import UIKit
import SwiftForms
 
class MyFormViewController: FormViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.title = "hangge.com"
         
        //创建form实例
        let form = FormDescriptor()
        form.title = "Example form"
         
        //第一个section分区
        let section1 = FormSectionDescriptor(headerTitle: nil, footerTitle: nil)
        //用户名
        var row = FormRowDescriptor(tag: "name", type: .text, title: "用户名")
        row.configuration.cell.appearance =
            ["textField.placeholder" : "手机号/邮箱号" as AnyObject,
            "textField.textAlignment" : NSTextAlignment.right.rawValue as AnyObject]
 
        section1.rows.append(row)
        //密码
        row = FormRowDescriptor(tag: "pass", type: .password, title: "密码")
        row.configuration.cell.appearance["textField.textAlignment"]
            = NSTextAlignment.right.rawValue as AnyObject
        section1.rows.append(row)
         
        //第二个section分区
        let section2 = FormSectionDescriptor(headerTitle: nil, footerTitle: nil)
        //提交按钮
        row = FormRowDescriptor(tag: "button", type: .button, title: "登录")
        row.configuration.button.didSelectClosure = { _ in
            self.submit()
        }
        section2.rows.append(row)
         
        //将两个分区添加到form中
        form.sections = [section1, section2]
        self.form = form
    }
     
    //登录按钮点击
    func submit() {
        //取消当前编辑状态
        self.view.endEditing(true)
         
        //将表单中输入的内容打印出来
        let message = self.form.formValues().description
        print(message)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```




（2）打开 **Main.storyboard**，删除默认的 **View Controller**。拖入一个 **Table View Controller**，并勾选“**Is Initial View Controller**”

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017061921164448657.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

（3）由于页面有多个分区，这里将 **tableView** 的 **style** 设置成 **Grouped** 会更加好看些。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/201706251055007698.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



（4）最后将这个 **Table View Controller** 的 **Custom Class** 设置为我们前面定义的 **MyFormViewController** 即可。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017061921175452832.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1720.zip](https://www.hangge.com/blog_uploads/201706/2017062510561844820.zip)

## 三、自带的表单元素类型

前面我只介绍了 **3** 种简单的表单项，**SwiftForms** 其实提供了许多常用的表单组件（分别对应不同的 **type**）供我们使用，下面分别进行介绍。

### 1，普通的文本标签（label）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062119090221768.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "test", type: .label, title: "这时一个文本标签单元格")
row.configuration.cell.placeholder = "右侧文字"
```



### 2，普通的文本输入（text）

（1）输入时，这个使用的是最普通的虚拟键盘。即 **textField.keyboardType = .default**

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/201706211914468048.png)](https://www.hangge.com/blog/cache/detail_1720.html#)[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062119145965863.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "test", type: .text, title: "用户名")
row.configuration.cell.placeholder = "你的工号"
```


（2）可以设置默认值，以及输入框文字靠右显示。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062414174634502.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
row.configuration.cell.appearance["textField.textAlignment"]
    = NSTextAlignment.right.rawValue as AnyObject
row.value = "A007" as AnyObject
```



### 3，特殊的文本输入

根据类型的不同，会自动使用相应的虚拟键盘。

- **.number**：数字输入。使用数字键盘（**keyboardType** 为 **.numberPad**）
- **.decimal**：带小数的数字输入。使用带小数点的数字键盘（**keyboardType** 为 **.decimalPad**）
- **.numbersAndPunctuation**：数字和标点符号输入。使用带数字和标点的键盘（**keyboardType** 为 **.numbersAndPunctuation**）
- **.name**：名字输入。使用普通键盘（**keyboardType** 为 **.default**），关闭自动纠错，同时单词首字母大写。
- **.phone**：电话输入。使用电话键盘（**keyboardType** 为 **.phonePad**）
- **.url**：**URL** 地址输入。使用 **URL** 键盘（**keyboardType** 为 **.URL**），且关闭自动纠错，首字母不大写。
- **.email**：邮箱地址输入。使用 **Email** 键盘（**keyboardType** 为 **.emailAddress**），且关闭自动纠错，首字母不大写。
- **.asciiCapable**：**ASCII** 编码输入。使用 **ASCII** 键盘（**keyboardType** 为 **.asciiCapable**），且关闭自动纠错，首字母不大写。
- **.password**：密码输入。使用普通键盘名，不过 **textField** 使用密文显示，且再次编辑自动清空原先里面的内容。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/201706212015367335.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
var row = FormRowDescriptor(tag: "test", type: .decimal, title: "本月消费")
```



### 4，按钮（button）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062120213053065.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
var row = FormRowDescriptor(tag: "test", type: .button, title: "提交")
row.configuration.button.didSelectClosure = { _ in
    //取消当前编辑状态
    self.view.endEditing(true)
}
```



### 5，开关（booleanSwitch）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062120270450476.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
let row1 = FormRowDescriptor(tag: "test1", type: .booleanSwitch, title: "消息通知")
row1.value = true as AnyObject
let row2 = FormRowDescriptor(tag: "test2", type: .booleanSwitch, title: "自动回复")
```



### 6，勾选（booleanCheck）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062120285126496.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
let row1 = FormRowDescriptor(tag: "test1", type: .booleanCheck, title: "电影")
let row2 = FormRowDescriptor(tag: "test2", type: .booleanCheck, title: "音乐")
row2.value = true as AnyObject
```



### 7，分段选择（segmentedControl） 

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062320320375108.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
//创建分段选择单元
var row = FormRowDescriptor(tag: "test1", type: .segmentedControl, title: "测试等级")
//设置分段值
row.configuration.selection.options = [1,2,3] as [AnyObject]
//设置分段值对应的文字
row.configuration.selection.optionTitleClosure = { value in
    guard let option = value as? Int else { return "" }
    switch option {
    case 1: return "初级"
    case 2: return "中级"
    case 3: return "高级"
    default: return "" }
}
//设置默认值
row.value = 2 as AnyObject
//设置了默认值，分段选择控件的选中项也要同步修改
row.configuration.cell.appearance["segmentedControl.selectedSegmentIndex"] = 1
```



### 8，选择框（picker）

   [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062414311187094.png)](https://www.hangge.com/blog/cache/detail_1720.html#)    [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062414312614390.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
//创建选择框单元
var row = FormRowDescriptor(tag: "test1", type: .picker, title: "测试等级")
row.configuration.cell.showsInputToolbar = true
//设置选项值
row.configuration.selection.options = [1,2,3] as [AnyObject]
//设置选项值值对应的文字
row.configuration.selection.optionTitleClosure = { value in
    guard let option = value as? Int else { return "" }
    switch option {
    case 1: return "初级"
    case 2: return "中级"
    case 3: return "高级"
    default: return "" }
}
//设置默认值
row.value = 2 as AnyObject
```



### 9，日期时间选择（date、time、dateAndTime）

（1）只有日期（**date**）

  [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062509583978299.png)](https://www.hangge.com/blog/cache/detail_1720.html#)    [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062509585427514.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "birthday", type: .date, title: "生日")
```


（2）只有时间（**time**）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510025036802.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



```swift
var row = FormRowDescriptor(tag: "birthday", type: .time, title: "每日更新时间")
```


（3）既有日期又有时间（**dateAndTime**）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510062369268.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "birthday", type: .dateAndTime, title: "结束")
```



### 10，步进器（stepper）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510130537140.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "count", type: .stepper, title: "计数")
//允许的最大值
row.configuration.stepper.maximumValue = 200.0
//允许的最小值
row.configuration.stepper.minimumValue = 20.0
//每次增减的值
row.configuration.stepper.steps = 2.0
```



### 11，滑块（slider）

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510200042581.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "count", type: .slider, title: "背景音乐")
//允许的最大值
row.configuration.stepper.maximumValue = 200.0
//允许的最小值
row.configuration.stepper.minimumValue = 0.0
```



### 12，多项选择（multipleSelector） 

点击单元格后会跳转新页面，列出所有的选项进行多选。

  [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510395122028.png)](https://www.hangge.com/blog/cache/detail_1720.html#)   [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510401034681.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "interest", type: .multipleSelector, title: "兴趣爱好")
row.configuration.selection.options = ([1, 2, 3, 4] as [Int]) as [AnyObject]
row.configuration.selection.allowsMultipleSelection = true
row.configuration.selection.optionTitleClosure = { value in
    guard let option = value as? Int else { return "" }
    switch option {
    case 1:
        return "听音乐"
    case 2:
        return "看电影"
    case 3:
        return "玩游戏"
    case 4:
        return "读书"
    default:
        return ""
    }
}
```



### 13，多行文本输入（multilineText）

  [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510503425297.png)](https://www.hangge.com/blog/cache/detail_1720.html#)   [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062510504295299.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
var row = FormRowDescriptor(tag: "text", type: .multilineText, title: "备注")
```



## 三、表单元素值改变响应 

有时我们想要在表单项的值改变（比如改变了 **segmentedControl**、**picker** 的选择项）时进行一些相应的操作，只需使用对应 **cell** 的 **didUpdateClosure** 回调方法即可。

### 1，效果图

这里我们使用步进器（**stepper**）为例。当点击加号、减号改变 **stepper** 的值时，控制台会把当前的值打印出来。

  [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201712/2017121215152475102.png)](https://www.hangge.com/blog/cache/detail_1720.html#)    [![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201712/2017121215161690450.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

### 2，样例代码

（高亮部分表示对应的响应回调）

```swift
var row = FormRowDescriptor(tag: "count", type: .stepper, title: "计数")
//允许的最大值
row.configuration.stepper.maximumValue = 200.0
//允许的最小值
row.configuration.stepper.minimumValue = 20.0
//每次增减的值
row.configuration.stepper.steps = 2.0
//该表单元素值改变时的响应
row.configuration.cell.didUpdateClosure = { row in
    print("--- 值发生改变 ---")
    print("type：\(row.type)")
    print("tag：\(row.tag)")
    print("title：\(row.title!)")
    print("value：\(row.value!)")
}
```



## 四、外观样式的修改

### 1，设置样式

表单中每一行描述都有一个配置字典（**row.configuration.cell.appearance**），通过它我们可以定制单元格的外观和行为。

（1）这里以设置分段选择（**segmentedControl**）的样式为例，效果图如下：

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062320375994858.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



（2）代码如下：

```swift
//标题文字样式修改
row.configuration.cell.appearance["titleLabel.font"] = UIFont.boldSystemFont(ofSize: 30.0)
//分段选择控件颜色修改
row.configuration.cell.appearance["segmentedControl.tintColor"] = UIColor.red
```



### 2，单元格头部添加图标

这个同普通的 **tableView** 一样，只要在 **cellForRowAt** 方法中给需要的 **cell** 设置图片即可。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062520365852437.png)](https://www.hangge.com/blog/cache/detail_1720.html#)

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
    -> UITableViewCell {
    //获取默认单元格
    let cell = super.tableView(tableView, cellForRowAt: indexPath)
    //如果是第一个单元格则添加邮件图标
    if indexPath.section == 0 && indexPath.row == 0 {
        cell.imageView?.image = UIImage(named: "mail.png")
    }
    return cell
}
```



## 五、创建自定义单元格

虽然 **SwiftForms** 已经提供了各种类型的单元格供我们使用，但有时可能还不能满足需求。通过继承 **FormBaseCell**，我们可以很方便的实现我们自己的表单 **cell**。继承后记得要实现如下两个方法：

- **configure**：这个只有在 **cell** 创建完毕后会调用一次。
- **update**：每次 **cell** 内容需要刷新的时候都会调用。

### 1，效果图

这里自定义一个简单的表单 **cell**，其内部左侧显示一个圆形的图标，右侧是文本输入框。

[![原文:Swift - SwiftForms的使用详解（第三方Form表单组件 ）](https://www.hangge.com/blog_uploads/201706/2017062619112872383.png)](https://www.hangge.com/blog/cache/detail_1720.html#)



### 2，组件代码（MyImageHeadCell.swift）

```swift
import UIKit
import SwiftForms
 
class MyImageHeadCell: FormBaseCell {
     
    //左侧图片视图
    open let headImageView = UIImageView()
     
    //右侧输入框
    open let textField  = UITextField()
     
    //初始化配置（只会在单元格创建完毕后调用一次）
    override func configure() {
        super.configure()
         
        //去除选中样式
        selectionStyle = .none
         
        //图片保持比例缩放
        headImageView.contentMode = .scaleAspectFill
        //设置遮罩
        headImageView.layer.masksToBounds = true
     
        //输入框文字样式
        textField.font = UIFont.preferredFont(forTextStyle: UIFontTextStyle.body)
        //输入框内容编辑响应时间
        textField.addTarget(self, action: #selector(MyImageHeadCell.editingChanged(_:)),
                            for: .editingChanged)
         
        //将组件添加到视图中
        contentView.addSubview(headImageView)
        contentView.addSubview(textField)
    }
     
    //每次需要更新单元格是都会调用
    override func update() {
        super.update()
         
        //设置输入框文字
        textField.text = rowDescriptor?.value as? String
        textField.placeholder = rowDescriptor?.configuration.cell.placeholder
        textField.isSecureTextEntry = false
        textField.clearButtonMode = .whileEditing
    }
     
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
         
        //圆形图标直径为单元格高度的3/4
        let diameter = super.frame.height / 4 * 3
        //设置图标的尺寸、位置
        headImageView.frame.size = CGSize(width: diameter, height: diameter)
        headImageView.center = CGPoint(x:diameter, y: super.frame.height / 2)
        //设置圆角半径(宽度的一半)，显示成圆形。
        headImageView.layer.cornerRadius = diameter / 2
         
        //设置输入框的尺寸和位置
        textField.frame = CGRect(x: diameter * 2, y: 0,
                                 width: super.frame.width - diameter * 2,
                                 height: super.frame.height)
    }
 
    //输入框内容编辑
    internal func editingChanged(_ sender: UITextField) {
        guard let text = sender.text, text.characters.count > 0
            else { rowDescriptor?.value = nil; update(); return }
        rowDescriptor?.value = text as AnyObject
    }
}
```



### 3，测试代码

```swift
import UIKit
import SwiftForms
 
class MyFormViewController: FormViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.title = "hangge.com"
         
        //创建form实例
        let form = FormDescriptor()
        form.title = "Example form"
         
        //第一个section分区
        let section1 = FormSectionDescriptor(headerTitle: nil, footerTitle: nil)
         
        //第1个单元格
        var row = FormRowDescriptor(tag: "windows", type: .unknown, title: "Windows")
        row.configuration.cell.appearance["headImageView.image"] =
            UIImage(named: "windows.png") as AnyObject
        row.configuration.cell.cellClass = MyImageHeadCell.self
        section1.rows.append(row)
         
        //第2个单元格
        row = FormRowDescriptor(tag: "ruby", type: .unknown, title: "Ruby")
        row.configuration.cell.appearance["headImageView.image"] =
            UIImage(named: "ruby.png") as AnyObject
        row.configuration.cell.cellClass = MyImageHeadCell.self
        section1.rows.append(row)
         
         
        //第二个section分区
        let section2 = FormSectionDescriptor(headerTitle: nil, footerTitle: nil)
        //提交按钮
        row = FormRowDescriptor(tag: "button", type: .button, title: "提交")
        row.configuration.button.didSelectClosure = { _ in
            self.submit()
        }
        section2.rows.append(row)
         
        //将两个分区添加到form中
        form.sections = [section1, section2]
        self.form = form
    }
     
    //登录按钮点击
    func submit() {
        //取消当前编辑状态
        self.view.endEditing(true)
         
        //将表单中输入的内容打印出来
        let message = self.form.formValues().description
        print(message)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



## 六、创建自定义的选择控制器（Selector Controllers）

**SwiftForms** 自带了一个选择控制器 **FormOptionsSelectorController**，像前面提到的多项选择 **cell** 就用到了这个选择控制器，点击表单 **cell** 后就会跳转到新页面进行多选操作。

为了满足一些特殊的业务需求，可能需要实现自定义的选择控制器。我们只要遵守 **FormSelector** 协议，即可通过 **cell** 对象显示这个选择控制器，更改属性值。同时还可以根据用户的操作，改变表单行值（**row value**）。具体写法我就不举例了，大家可以参考 **FormOptionsSelectorController.swift**

注意：定义好选择控制器类之后，别忘了设置 **row.configuration.selection.controllerClass**，使其能使用我们自定义的选择器控制器。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1720.html