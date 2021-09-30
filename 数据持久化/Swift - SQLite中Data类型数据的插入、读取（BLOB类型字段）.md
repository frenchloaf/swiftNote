# Swift - SQLite中Data类型数据的插入、读取（BLOB类型字段）

2017-03-08发布：hangge阅读：2725

我原来写一过一篇文章介绍如何使用第三方的 **SQLiteBD** 库来操作 **SQLite** 数据库（[点击查看](https://www.hangge.com/blog/cache/detail_645.html)）。当时存取的都是一些数字、字符串这样的基本数据类型，有网友问如果想存个图片进去应该怎么做。本文演示如何实现 **Data** 类型的数据存取。

### 1，实现原理

（1）首先我们建表的时候，用于保存 **Data** 数据的字段要使用大数据类型，比如：**BLOB**（二进制数据）

（2）读取操作同其他数据类型的读取没什么差别。不过插入的时候要注意，不能直接将数据拼接到 **sql** 语句中，而是要使用预处理语句：

```swift
let sql = "insert into t_image(idata) values(?)"
db.execute(sql: sql, parameters:[imageData])
```

###  2，效果图

（1）程序启动后会自动判断是否存在图片表，没有的话就创建一张。表字段很简单，就一个 **ID** 主键，和一个 **BLOB** 类型的字段（用于存储图片数据）

（2）点击“**保存**”按钮，将项目中的 **0.png** 这张图片存储到图片表中。

（3）点击“**读取**”按钮，从图片表中取出图片数据，并显示在 **imageview** 中。

  [![原文:Swift - SQLite中Data类型数据的插入、读取（BLOB类型字段）](https://www.hangge.com/blog_uploads/201703/2017030617292566558.png)](https://www.hangge.com/blog/cache/detail_1578.html#)    [![原文:Swift - SQLite中Data类型数据的插入、读取（BLOB类型字段）](https://www.hangge.com/blog_uploads/201703/2017030617293462642.png)](https://www.hangge.com/blog/cache/detail_1578.html#)

###  3，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var imageView: UIImageView!
     
    var db:SQLiteDB!
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取数据库实例
        db = SQLiteDB.shared
        //打开数据库
        _ = db.openDB()
        //如果表还不存在则创建表（其中uid为自增主键）
        let result = db.execute(sql: "create table if not exists t_image(uid integer primary key,idata blob)")
        print("表创建完毕：\(result)")
    }
     
    //点击保存
    @IBAction func saveData(_ sender: Any) {
        //获取图片并转换为Data
        let imageURL =  Bundle.main.url(forResource: "0", withExtension: "png")!
        let imageData = try! Data(contentsOf: imageURL)
        //将Data数据插入到数据库
        let sql = "insert into t_image(idata) values(?)"
        let result = db.execute(sql: sql, parameters:[imageData])
        print("数据保存完毕：\(result)")
    }
 
    //点击加载
    @IBAction func loadData(_ sender: Any) {
        let data = db.query(sql: "select * from t_image")
        if data.count > 0 {
            //获取最后一行数据显示
            let image = data[data.count - 1]
            if let imgData = image["idata"] as? Data {
                self.imageView.image = UIImage(data: imgData)
                print("数据读取完毕")
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1578.zip](https://www.hangge.com/blog_uploads/201704/201704251658531927.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1578.html