# Swift - 解决表格中TextField,TextView编辑时，输入框被键盘遮挡的问题

2017-01-24发布：hangge阅读：5310

通常我们可以通过自定义单元格，并在单元格内部添加 **UITextField** 或 **UITextView** 来实现可编辑列表。但有时会发现，当我们编辑 **tableView** 中的输入框内容时，出现的软键盘会将输入框遮挡住。

比如下面样例：

（1）表格中一共有 **20** 条记录。自定义单元格中使用的是 **UITextField**（**UITextView** 同理）

（2）当我们想要编辑“**条目8**”这个单元格内容时，点击该单元格内容，出现的虚拟键盘会把输入框给挡住。“条目8”中的输入框并没有自动移动到可视区域。

（3）而且由于虚拟键盘的存在，我们如果将滚动条滚到底会发现底部的单元格也被键盘给挡住了。

  [![原文:Swift - 解决表格中TextField,TextView编辑时，输入框被键盘遮挡的问题](https://www.hangge.com/blog_uploads/201612/2016122714025031128.png)](https://www.hangge.com/blog/cache/detail_1498.html#)  [![原文:Swift - 解决表格中TextField,TextView编辑时，输入框被键盘遮挡的问题](https://www.hangge.com/blog_uploads/201612/2016122714025877528.png)](https://www.hangge.com/blog/cache/detail_1498.html#)  [![原文:Swift - 解决表格中TextField,TextView编辑时，输入框被键盘遮挡的问题](https://www.hangge.com/blog_uploads/201612/2016122714030495469.png)](https://www.hangge.com/blog/cache/detail_1498.html#)

要解决输入框被遮挡的问题有很多办法，下面分别介绍。由于有的方法即适用于 **UITextField** 也适用于 **UITextView**，而有的方法对于这两种控件稍微有些区别，所以下面我对单元格使用 **UITextField** 或 **UITextView** 这两种情况分别进行说明。

## 一、单元格中使用UITextField

假设我们有如下自定义单元格，其内部使用的是 **UITextField**。

```swift
import UIKit 
//单元格类
class MyTextFieldCell: UITableViewCell, UITextFieldDelegate {
    //单元格内部标签（可输入）
    var label:UITextField!
     
    //初始化
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
         
        super.init(style: style, reuseIdentifier: reuseIdentifier)
         
        //初始化文本标签
        label = UITextField(frame: CGRect.null)
        label.textColor = UIColor.black
        label.font = UIFont.systemFont(ofSize: 16)
         
        //设置文本标签代理
        label.delegate = self
        label.contentVerticalAlignment = .center
        //添加文本标签
        addSubview(label)
    }
     
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
        label.frame = CGRect(x: 15, y: 0, width: bounds.size.width - 15,
                             height: bounds.size.height)
    }
     
    //键盘回车
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return false
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```



### 1，视图控制器为UITableViewController情况

如果 **tableView** 所在的视图控制器是 **UITableViewController** 的话，那我们就不需要特别处理即可放心使用，系统会自动处理键盘遮挡的问题。

```swift
import UIKit
 
class TableViewController: UITableViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建一个重用的单元格
        self.tableView!.register(MyTextFieldCell.self, forCellReuseIdentifier: "tableCell")
    }
     
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int)
        -> Int {
        return 20
    }
     
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //创建一个重用的单元格
        let cell = tableView
            .dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
            as! MyTextFieldCell
        cell.label.text = "条目\(indexPath.row)"
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 2，视图控制器为UIViewController情况

如果 **tableView** 所在的视图控制器是 **UIViewController** 的话，这里有两种办法防止键盘遮挡。



- **方法1**：在原来的 **UIViewController** 内部再添加一层 **UITableViewController**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(MyTextFieldCell.self, forCellReuseIdentifier: "tableCell")
        self.view.addSubview(self.tableView!)
         
        //添加一个UITableViewController
        let tableVC = UITableViewController.init(style: .plain)
        tableVC.tableView = self.tableView
        self.addChildViewController(tableVC)
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 20
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //创建一个重用的单元格
            let cell = tableView
                .dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
                as! MyTextFieldCell
             
            cell.label.text = "条目\(indexPath.row)"
            return cell
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



**方法2**：监听键盘通知，在键盘出现或消失的时候修改**tableView** 的 **contentInset** 和 **scrollIndicatorInsets**

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(MyTextFieldCell.self, forCellReuseIdentifier: "tableCell")
        self.view.addSubview(self.tableView!)
         
        //监听键盘弹出通知
        NotificationCenter.default
            .addObserver(self,selector: #selector(keyboardWillShow(_:)),
                         name: NSNotification.Name.UIKeyboardWillShow, object: nil)
        //监听键盘隐藏通知
        NotificationCenter.default
            .addObserver(self,selector: #selector(keyboardWillHide(_:)),
                         name: NSNotification.Name.UIKeyboardWillHide, object: nil)
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 20
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //创建一个重用的单元格
            let cell = tableView
                .dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
                as! MyTextFieldCell
             
            cell.label.text = "条目\(indexPath.row)"
            return cell
    }
     
    // 键盘显示
    func keyboardWillShow(_ notification: Notification) {
        let userInfo = (notification as NSNotification).userInfo!
        //键盘尺寸
        let keyboardSize = (userInfo[UIKeyboardFrameBeginUserInfoKey]
            as! NSValue).cgRectValue
        var contentInsets:UIEdgeInsets
        //判断是横屏还是竖屏
        let statusBarOrientation = UIApplication.shared.statusBarOrientation
        if UIInterfaceOrientationIsPortrait(statusBarOrientation) {
            contentInsets = UIEdgeInsetsMake(64.0, 0.0, (keyboardSize.height), 0.0);
        } else {
            contentInsets = UIEdgeInsetsMake(64.0, 0.0, (keyboardSize.width), 0.0);
        }
        //tableview的contentview的底部大小
        self.tableView!.contentInset = contentInsets;
        self.tableView!.scrollIndicatorInsets = contentInsets;
    }
     
    // 键盘隐藏
    func keyboardWillHide(_ notification: Notification) {
        //还原tableview的contentview大小
        let contentInsets:UIEdgeInsets = UIEdgeInsetsMake(64.0, 0.0, 0, 0.0);
        self.tableView!.contentInset = contentInsets
        self.tableView!.scrollIndicatorInsets = contentInsets
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



## 二、单元格中使用UITextView



假设我们有一个自定义单元格 **MyTextViewCell**，其内部使用的是 **UITextView**。（同上面的自定义单元格 **MyTextFieldCell** 相比，就是把里面的 **UITextField** 替换成 **UITextView**。这里就不再写详细代码了）



### 1，视图控制器为UITableViewController情况

如果 **tableView** 所在的视图控制器是 **UITableViewController** 的话，系统会自动处理键盘遮挡的问题。这个同上面使用 **UITextField** 的情况一样，不再说明。

### 2，视图控制器为UIViewController情况

如果 **tableView** 所在的视图控制器是 **UIViewController** 的话，同样有两种办法防止键盘遮挡。

- **方法1**：在原来的 **UIViewController** 内部再添加一层 **UITableViewController**。这个同上面使用 **UITextField** 的情况一样，不再说明。
- **方法2**：监听键盘通知，除了在键盘出现或消失的时候修改 **tableView** 的 **contentInset** 和 **scrollIndicatorInsets**以外，我们还需要添加一个 **textView** 编辑响应事件，即 **textView** 开始编辑时 **tableView** 要将当前单元格滚动到可视区域。

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(MyTextViewCell.self, forCellReuseIdentifier: "tableCell")
        self.view.addSubview(self.tableView!)
         
        //添加一个UITableViewController
        let tableVC = UITableViewController.init(style: .plain)
        tableVC.tableView = self.tableView
        self.addChildViewController(tableVC)
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 20
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //创建一个重用的单元格
            let cell = tableView
                .dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
                as! MyTextViewCell
             
            cell.label.text = "条目\(indexPath.row)"
            return cell
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//单元格类
class MyTextViewCell: UITableViewCell, UITextViewDelegate {
    //单元格内部标签（可输入）
    var label:UITextView!
     
    //初始化
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
         
        super.init(style: style, reuseIdentifier: reuseIdentifier)
         
        //初始化文本标签
        label = UITextView(frame: CGRect.null)
        label.textColor = UIColor.black
        label.font = UIFont.systemFont(ofSize: 16)
         
        //设置文本标签代理
        label.delegate = self
        //添加文本标签
        addSubview(label)
    }
     
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
        label.frame = CGRect(x: 15, y: 0, width: bounds.size.width - 15,
                             height: bounds.size.height)
    }
     
    //开始编辑
    func textViewDidBeginEditing(_ textView: UITextView) {
        //获取tablveView
        let tableView = superTableView()
        //获取当前cell所在的indexPath
        let indexPath = tableView?.indexPath(for: self)
        //将当前cell滚动到可视区域
        tableView?.scrollToRow(at: indexPath!, at: .none, animated: true)
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
 
extension UITableViewCell {
    //返回cell所在的UITableView
    func superTableView() -> UITableView? {
        for view in sequence(first: self.superview, next: { $0?.superview }) {
            if let tableView = view as? UITableView {
                return tableView
            }
        }
        return nil
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1498.html