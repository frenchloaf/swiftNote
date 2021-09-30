# Swift - 在单元格里的按钮点击事件中获取对应的单元格以及索引路径

2016-12-30发布：hangge阅读：10821

有时我们需要在**的tableView**的自定义单元格中添加一些按钮，并在按钮点击方法中进行一些业务逻辑。比如下面样例，在每个单元格尾部都有一个加号按钮，点击后会把该单元格的数字加到总数上，并在标题中显示。

[![翻译：Swift - 在单元格里的按钮点击事件中获取的单元格以及索引路径](https://www.hangge.com/blog_uploads/201612/201612271106581725.png)](https://www.hangge.com/blog/cache/detail_1500.html#)

整个页面以及单元格自定义是在**故事板**中实现的，同时将**按钮**的点击事件绑定到**VC** 中。需要我们在点击事件中要先按钮得到自己的**单元格**，再这个获取**单元**格里的文本标签，将标签内容转换成数字与最终相加。

[![翻译：Swift - 在单元格里的按钮点击事件中获取的单元格以及索引路径](https://www.hangge.com/blog_uploads/201612/2016122711102858597.png)](https://www.hangge.com/blog/cache/detail_1500.html#)

### 1，在点击事件中获取的单元格

我们添加一个**superUITableViewCell**方法通过遍历循环**按钮**的**上海华**来获取其对应的**细胞**这个原理和我之前写的文章类似：[斯威夫特-在TableViewCell中获取父的TableView（附：获取任意类型的父视图）](https://www.hangge.com/blog/cache/detail_1499.html)

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    @IBOutlet weak var tableView: UITableView!
   
    //总数
    var sum = 0
     
    override func loadView() {
        super.loadView()
         
        self.title = "总数：\(sum)"
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.tableView.delegate = self
        self.tableView.dataSource = self
        self.tableView.allowsSelection = false
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
            let label = cell.viewWithTag(1) as! UILabel
            label.text = "\(indexPath.row)"
            return cell
    }
     
    //单元格中的按钮点击
    @IBAction func tapAddButton(_ sender: AnyObject) {
        let btn = sender as! UIButton
        let cell = superUITableViewCell(of: btn)!
        let label = cell.viewWithTag(1) as! UILabel
        sum = sum + (label.text! as NSString).integerValue
        self.title = "总数：\(sum)"
    }
     
    //返回button所在的UITableViewCell
    func superUITableViewCell(of: UIButton) -> UITableViewCell? {
        for view in sequence(first: of.superview, next: { $0?.superview }) {
            if let cell = view as? UITableViewCell {
                return cell
            }
        }
        return nil
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 2、功能改进

（1）我们可以通过扩展**UIView**，让我们可视化的任意视图对象（**View**）各自指定类型的父视图。

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

（2）在**按钮**点击事件中可以这么获取到对应的**单元格**：

```swift
//单元格中的按钮点击
@IBAction func tapAddButton(_ sender: AnyObject) {
    let btn = sender as! UIButton
    let cell = btn.superView(of: UITableViewCell.self)!
    let label = cell.viewWithTag(1) as! UILabel
    sum = sum + (label.text! as NSString).integerValue
    self.title = "总数：\(sum)"
}
```

### 3，在点击事件中获取的indexPath

获取到**cell**后，通过**tableView**的**indexPath(for:)**方法获取对应的**indexPath**。



```swift
//单元格中的按钮点击
@IBAction func tapAddButton(_ sender: AnyObject) {
    let btn = sender as! UIButton
    let cell = btn.superView(of: UITableViewCell.self)!
    let indexPath = tableView.indexPath(for: cell)
    print("indexPath：\(indexPath!)")
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1499.zip](https://www.hangge.com/blog_uploads/201612/2016122711181134128.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1500.html