# Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）

2016-08-08发布：hangge阅读：6069

有时候在我们应用中需要用到表格（**tableView**）的多选功能。其实 **tableView** 已自带了多种多选功能，不用借助第三方组件也可以实现。下面分别进行介绍。（本文代码已升级至 **Swift3**）

**方法1，自定义一个数组保存选中项的索引（非编辑状态）**

（1）我们先定义一个数组，表格在非编辑状态时，点击某个单元格便将其索引添加到这个数组中。同时将单元格尾部打勾表示选中状态。再次点击原来选中的单元格，则取消选中状态，并将索引从数组中移除。

（2）点击导航栏上的“确定”按钮，即可获取到所有选中项的索引以及对应的值，并打印出来。

   [![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080714051788764.png)](https://www.hangge.com/blog/cache/detail_1320.html#)    [![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080714052582468.png)](https://www.hangge.com/blog/cache/detail_1320.html#)

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = ["条目1","条目2","条目3","条目4","条目5"]
     
    //存储选中单元格的索引
    var selectedIndexs = [Int]()
     
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
        self.tableView!.register(UITableViewCell.self,
                                      forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1;
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "SwiftCell"
        //同一形式的单元格重复使用，在声明时已注册
        let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                 for: indexPath) as UITableViewCell
         
        cell.textLabel?.text = self.items[indexPath.row]
         
        //判断是否选中（选中单元格尾部打勾）
        if selectedIndexs.contains(indexPath.row) {
            cell.accessoryType = .checkmark
        } else {
            cell.accessoryType = .none
        }
         
        return cell
    }
     
    // UITableViewDelegate 方法，处理列表项的选中事件
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        //判断该行原先是否选中
        if let index = selectedIndexs.index(of: indexPath.row){
            selectedIndexs.remove(at: index) //原来选中的取消选中
        }else{
            selectedIndexs.append(indexPath.row) //原来没选中的就选中
        }
         
        ////刷新该行
        self.tableView?.reloadRows(at: [indexPath], with: .automatic)
    }
     
    //确定按钮点击
    @IBAction func btnClick(_ sender: AnyObject) {
        print("选中项的索引为：", selectedIndexs)
        print("选中项的值为：")
        for index in selectedIndexs {
            print(items[index])
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**方法2，将allowsMultipleSelection设置为true（非编辑状态）**
前面的样例，表格实际上还是单选的。只不过我们定义了一个数值来保存选中的单元格索引，从而实现多选的功能。
下面还是实现同样的功能，只不过这次将表格设置成允许多选（**allowsMultipleSelection** 为 **true**），这样我们也就不用再另外定义数组来存储选中项索引了。
[![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080718262757032.png)](https://www.hangge.com/blog/cache/detail_1320.html#)[![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080718263464440.png)](https://www.hangge.com/blog/cache/detail_1320.html#)

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = ["条目1","条目2","条目3","条目4","条目5"]
     
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
        self.tableView!.register(UITableViewCell.self,
                                      forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //设置允许单元格多选
        self.tableView!.allowsMultipleSelection = true
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1;
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "SwiftCell"
        //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath) as UITableViewCell
        cell.textLabel?.text = self.items[indexPath.row]
             
        //判断是否选中（选中单元格尾部打勾）
        if tableView.indexPathsForSelectedRows?.index(of: indexPath) != nil{
            cell.accessoryType = .checkmark
        } else {
            cell.accessoryType = .none
        }
             
        return cell
    }
     
    // UITableViewDelegate 方法，处理列表项的选中事件
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let cell = self.tableView?.cellForRow(at: indexPath)
        cell?.accessoryType = .checkmark
    }
     
    //处理列表项的取消选中事件
    func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath) {
        let cell = self.tableView?.cellForRow(at: indexPath)
        cell?.accessoryType = .none
    }
     
    //确定按钮点击
    @IBAction func btnClick(_ sender: AnyObject) {
        var selectedIndexs = [Int]()
         
        if let selectedItems = tableView!.indexPathsForSelectedRows {
            for indexPath in selectedItems {
                selectedIndexs.append(indexPath.row)
            }
        }
         
        print("选中项的索引为：", selectedIndexs)
        print("选中项的值为：")
        for index in selectedIndexs {
            print(items[index])
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



**方法3，allowsMultipleSelectionDuringEditing设置为true（编辑状态）**
这个样例同上面那个有点类似，只不过是表格进入编辑状态下才可以多选。
（1）下面样例表格默认情况下无法进行多选。
（2）长按表格进入编辑状态，这时单元格前面会出现选择框。点击即可进行单元格的选择与取消。
（3）点击导航栏上的“删除”按钮，即可将选中的单元格都删除。



[![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080719315787997.png)](https://www.hangge.com/blog/cache/detail_1320.html#)[![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/201608071932034525.png)](https://www.hangge.com/blog/cache/detail_1320.html#)[![原文:Swift - tableView的单元格多选功能的实现（获取多选值、多选删除）](https://www.hangge.com/blog_uploads/201608/2016080719320924151.png)](https://www.hangge.com/blog/cache/detail_1320.html#)

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource,
    UIGestureRecognizerDelegate {
     
    var items:[String] = ["条目1","条目2","条目3","条目4","条目5"]
     
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
        self.tableView!.register(UITableViewCell.self,
                                      forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //表格在编辑状态下允许多选
        self.tableView?.allowsMultipleSelectionDuringEditing = true
         
        //绑定对长按的响应
        let longPress = UILongPressGestureRecognizer(target:self,
            action:#selector(ViewController.tableviewCellLongPressed(gestureRecognizer:)))
        //代理
        longPress.delegate = self
        longPress.minimumPressDuration = 1.0
        //将长按手势添加到需要实现长按操作的视图里
        self.tableView!.addGestureRecognizer(longPress)
    }
     
    //单元格长按事件响应
    func tableviewCellLongPressed(gestureRecognizer:UILongPressGestureRecognizer)
    {
        if (gestureRecognizer.state == .ended)
        {
            print("UIGestureRecognizerStateEnded");
            //在正常状态和编辑状态之间切换
            if(self.tableView!.isEditing == false) {
                self.tableView!.setEditing(true, animated:true)
            }
            else {
                self.tableView!.setEditing(false, animated:true)
            }
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1;
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "SwiftCell"
        //同一形式的单元格重复使用，在声明时已注册
        let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
        cell.textLabel?.text = self.items[indexPath.row]
        return cell
    }
     
    //删除按钮点击
    @IBAction func btnClick(_ sender: AnyObject) {
        //获取选中项索引
        var selectedIndexs = [Int]()
        if let selectedItems = tableView!.indexPathsForSelectedRows {
            for indexPath in selectedItems {
                selectedIndexs.append(indexPath.row)
            }
        }
         
        //删除选中的数据
        items.removeAt(indexes:selectedIndexs)
        //重新加载数据
        self.tableView?.reloadData()
        //退出编辑状态
        self.tableView!.setEditing(false, animated:true)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension Array {
    //Array方法扩展，支持根据索引数组删除
    mutating func removeAt(indexes: [Int]) {
        for i in indexes.sorted(by: >) {
            self.remove(at: i)
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1320.html