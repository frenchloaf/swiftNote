# Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）

2016-05-23发布：hangge阅读：2751

（本文代码已升级至Swift3）

本文演示如何制作一个可以编辑单元格内容的表格（**UITableView**）。

**1，效果图**

（1）默认状态下，表格不可编辑，当点击单元格的时候会弹出提示框显示选中的内容。

​    [![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/201605101352306604.png)](https://www.hangge.com/blog/cache/detail_1175.html#)     [![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/2016051013530164041.png)](https://www.hangge.com/blog/cache/detail_1175.html#)

（2）点击导航栏右侧编辑按钮，表格进入可以编辑状态

[![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/2016052916295133798.png)](https://www.hangge.com/blog/cache/detail_1175.html#)

（3）这时我们可以删除表格项。

[![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/2016052916300636496.png)](https://www.hangge.com/blog/cache/detail_1175.html#)

（4）也可以拖动调整单元格的顺序。

[![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/2016052916301884662.png)](https://www.hangge.com/blog/cache/detail_1175.html#)

（5）然后就是本文的重点，在编辑状态下。直接点击单元格，即可在当前页面下直接编辑修改单元格中的内容。

[![原文:Swift - 可编辑表格样例（可直接编辑单元格中内容、移动删除单元格）](https://www.hangge.com/blog_uploads/201605/201605101356203737.png)](https://www.hangge.com/blog/cache/detail_1175.html#)

**2，单元格编辑功能讲解**

（1）通过自定义 **UITableViewCell**，在其内部添加一个 **textField** 来实现可编辑的cell。

（2）通过改变 **userInteractionEnabled** 属性，可以让 **textField** 在可编辑与只读两个状态间切换。

（3）在编辑状态下要加大 **textField** 的左边距，因为左侧会出现个删除按钮图标。

（4）同时在编辑 **cell** 内容的时候，由于键盘会弹出挡到后面的表格。所以我们在键盘出现的时候，要通过改变 **edgeInsets** 的办法在底部加大 **tableview** 的 **contentview** 大小。

而结束编辑隐藏键盘时，需要还原 **tableview** 的 **contentview** 的尺寸。

**3，程序代码**

--- **MyTableViewCell.swift**（自定义单元格类）---

```swift
import UIKit
 
//表格数据实体类
class ListItem: NSObject {
    var text: String
     
    init(text: String) {
        self.text = text
    }
}
 
//单元格类
class MyTableViewCell: UITableViewCell, UITextFieldDelegate {
    //单元格内部标签（可输入）
    let label:UITextField
    //单元格左边距
    var leftMarginForLabel: CGFloat = 15.0
     
    //单元格数据
    var listItem:ListItem? {
        didSet {
            label.text = listItem!.text
        }
    }
     
    //单元格是否可编辑
    var labelEditable:Bool? {
        didSet {
            label.isUserInteractionEnabled = labelEditable!
            //如果是可以编辑的话，要加大左边距（因为左边有个删除按钮）
            leftMarginForLabel = labelEditable! ? 45.0 : 15.0
            self.setNeedsLayout()
        }
    }
     
    //初始化
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        //初始化文本标签
        label = UITextField(frame: CGRect.null)
        label.textColor = UIColor.black
        label.font = UIFont.systemFont(ofSize: 16)
         
        super.init(style: style, reuseIdentifier: reuseIdentifier)
         
        //设置文本标签代理
        label.delegate = self
        label.contentVerticalAlignment = UIControlContentVerticalAlignment.center
        //添加文本标签
        addSubview(label)
    }
     
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
        label.frame = CGRect(x: leftMarginForLabel, y: 0,
                             width: bounds.size.width - leftMarginForLabel,
                             height: bounds.size.height)
    }
     
    //键盘回车
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return false
    }
     
    //结束编辑
    func textFieldShouldEndEditing(_ textField: UITextField) -> Bool {
        if listItem != nil {
            listItem?.text = textField.text!
        }
        return true
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```



--- **ViewController.swift**（主类）---

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate,
UITableViewDataSource{
     
    //表格
    var tableView:UITableView?
    //表格数据
    var listItems = [ListItem]()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化数据
        listItems = [ListItem(text: "这是条目1"), ListItem(text: "这是条目2"),
                     ListItem(text: "这是条目3"), ListItem(text: "这是条目4"),
                     ListItem(text: "这是条目5"), ListItem(text: "这是条目6"),
                     ListItem(text: "这是条目7"), ListItem(text: "这是条目8"),
                     ListItem(text: "这是条目9"), ListItem(text: "这是条目10"),
                     ListItem(text: "这是条目11"), ListItem(text: "这是条目12"),
                     ListItem(text: "这是条目13"), ListItem(text: "这是条目14"),
                     ListItem(text: "这是条目15"), ListItem(text: "这是条目16"),
                     ListItem(text: "这是条目17")]
         
        //创建表视图
        self.tableView = UITableView(frame:self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(MyTableViewCell.self, forCellReuseIdentifier: "tableCell")
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
     
    //导航栏编辑按钮点击
    @IBAction func editBarBtnClick(_ sender: UIBarButtonItem) {
        //在正常状态和编辑状态之间切换
        if(self.tableView!.isEditing == false){
            self.tableView!.setEditing(true, animated:true)
            sender.title = "保存"
        }
        else{
            self.tableView!.setEditing(false, animated:true)
            sender.title = "编辑"
        }
        //重新加载表数据（改变单元格输入框编辑/只读状态）
        self.tableView?.reloadData()
    }
     
    //在本例中，有1个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return listItems.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell
    {
        let cell = tableView
            .dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
            as! MyTableViewCell
        //设置单元格内容
        let item = listItems[(indexPath as NSIndexPath).row]
        cell.listItem = item
        cell.accessoryType = UITableViewCellAccessoryType.disclosureIndicator
        //内容标签是否可编辑
        cell.labelEditable = tableView.isEditing
        return cell
    }
     
    // UITableViewDelegate 方法，处理列表项的选中事件
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)
    {
        self.tableView!.deselectRow(at: indexPath, animated: true)
        let itemString = listItems[(indexPath as NSIndexPath).row].text
        let alertController = UIAlertController(title: "提示!",
                                                message: "你选中了【\(itemString)】",
            preferredStyle: .alert)
        let okAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
     
    //是否有删除功能
    func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath)
        -> UITableViewCellEditingStyle
    {
        if(self.tableView!.isEditing == false){
            return UITableViewCellEditingStyle.none
        }else{
            return UITableViewCellEditingStyle.delete
        }
    }
     
    //删除提示
    func tableView(_ tableView: UITableView,
                   titleForDeleteConfirmationButtonForRowAt indexPath: IndexPath)
        -> String? {
            return "确定删除？"
    }
     
    //编辑完毕（这里只有删除操作）
    func tableView(_ tableView: UITableView,
                   commit editingStyle: UITableViewCellEditingStyle,
                   forRowAt indexPath: IndexPath) {
        if(editingStyle == UITableViewCellEditingStyle.delete)
        {
            self.listItems.remove(at: (indexPath as NSIndexPath).row)
            self.tableView!.reloadData()
            print("你确认了删除按钮")
        }
    }
     
    //在编辑状态，可以拖动设置cell位置
    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        return true
    }
     
    //移动cell事件
    func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath,
                   to toIndexPath: IndexPath) {
        if fromIndexPath != toIndexPath{
            //获取移动行对应的值
            let itemValue:ListItem = listItems[(fromIndexPath as NSIndexPath).row]
            //删除移动的值
            listItems.remove(at: (fromIndexPath as NSIndexPath).row)
            //如果移动区域大于现有行数，直接在最后添加移动的值
            if (toIndexPath as NSIndexPath).row > listItems.count{
                listItems.append(itemValue)
            }else{
                //没有超过最大行数，则在目标位置添加刚才删除的值
                listItems.insert(itemValue, at:(toIndexPath as NSIndexPath).row)
            }
        }
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
     
    //页面移除时
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidAppear(animated)
        //取消键盘监听通知
        NotificationCenter.default.removeObserver(self)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1175.zip](https://www.hangge.com/blog_uploads/201612/2016122510475649322.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1175.html