# Swift - 在TableViewCell中获取父TableView（附：获取任意类型的父View）

2016-12-28发布：hangge阅读：3581

## 一、在TableViewCell里获取对应的TableView

有时我们需要在自定义的单元格（**tableViewCell**）中获取其所在的表格（**tableView**）对象。除了可以从外部把 **tableView** 传入到 **cell** 中去外，还可以通过循环遍历 **cell** 的 **superview** 来得到其所在的父 **tableView**。

###  1，扩展UITableViewCell

为方便使用，这里对 **UITableViewCell** 进行扩展，添加个方法用来获取其所在的 **tableView**。

```swift
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



### 2，使用样例

我们在每个单元格尾部添加一个按钮，点击后打印出这个按钮所在单元格的 **indexPath**。（先得到 **tableView**，再根据 **tableView** 获取到当前 **cell** 所在的 **indexPath**）

（1）效果图

  [![原文:Swift - 在TableViewCell中获取父TableView（附：获取任意类型的父View）](https://www.hangge.com/blog_uploads/201612/2016122709363075555.png)](https://www.hangge.com/blog/cache/detail_1499.html#)   [![原文:Swift - 在TableViewCell中获取父TableView（附：获取任意类型的父View）](https://www.hangge.com/blog_uploads/201612/2016122709363680483.png)](https://www.hangge.com/blog/cache/detail_1499.html#)

（2）样例代码

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
        self.tableView!.allowsSelection = false
        //创建一个重用的单元格
        self.tableView!.register(MyTableCell.self, forCellReuseIdentifier: "tableCell")
        self.view.addSubview(self.tableView!)
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
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
                as! MyTableCell
             
            cell.textLabel?.text = "条目\(indexPath.row)"
            return cell
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//单元格类
class MyTableCell: UITableViewCell {
 
    var button:UIButton!
     
    //初始化
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
         
        button = UIButton(frame:CGRect(x:0, y:0, width:40, height:25))
        button.setTitle("点击", for:.normal) //普通状态下的文字
        button.backgroundColor = UIColor.orange
        button.layer.cornerRadius = 5
        button.titleLabel?.font = UIFont.systemFont(ofSize: 13)
        //按钮点击
        button.addTarget(self, action:#selector(tapped(_:)), for:.touchUpInside)
        self.addSubview(button)
    }
     
    //布局
    override func layoutSubviews() {
        super.layoutSubviews()
        button.center = CGPoint(x: bounds.size.width - 35, y: bounds.midY)
    }
     
    //按钮点击事件响应
    func tapped(_ button:UIButton){
        let tableView = superTableView()
        let indexPath = tableView?.indexPath(for: self)
        print("indexPath：\(indexPath!)")
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

原文出自：www.hangge.com  转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1499.html
```



## 二、获取任意类型的父View 

我们可以对上面的方法做个改进，通过扩展 **UIView**，让我们可以获得任意视图对象（**View**）所在的指定类型的父视图。



### 1，扩展View

这里对 **UIView** 进行扩展，根据传入的类型，循环遍历获取该类型的父 **View**。

```swift
extension UIView {
    //返回该view所在的父view
    func superView<T: UIView>(of: T.Type) -> T? {
        for view in sequence(first: self.superview, next: { $0?.superview }) {
            if let father = view as? T {
                return father
            }
        }
        return nil
    }
}
```



### 2，使用样例

这里对上面样例代码做个修改，之前我们在单元格中这么得到该 **cell** 所在的 **tableView**：

```swift
let tableView = superTableView()
```

现在改成这样：

```swift
let tableView = superView(of: UITableView.self)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1499.html