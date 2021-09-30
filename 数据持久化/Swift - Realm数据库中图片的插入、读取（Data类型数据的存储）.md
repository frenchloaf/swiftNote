# Swift - Realm数据库中图片的插入、读取（Data类型数据的存储）

2017-04-12发布：hangge阅读：3752

（本文代码已升级至Swift4）

我原来写一过一篇文章介绍如何使用 **Realm** 数据库（[点击查看](https://www.hangge.com/blog/cache/detail_891.html)）。当时存取的都是一些数字、字符串这样的基本数据类型，有网友问如果想存个图片（**Image**）进去应该怎么做。本文演示如何实现 **Data** 类型的数据存取。



### 1，实现原理

（1）**Realm** 支持 **Data** 类型的属性，我们要做的就是将图片转换为 **Data** 类型，再进行存储即可。

（2）**Data** 类型的属性读取操作同其他数据类型的读取没什么差别。只要注意不要超过 **16MB** 即可。

###  2，效果图

（1）点击“**保存**”按钮，将项目中的 **0.png** 这张图片存储到 **Realm** 数据库中。

（2）点击“**读取**”按钮，从库中取出最新保存的那张图片，并显示在 **imageview** 中。

  [![原文:Swift - Realm数据库中图片的插入、读取（Data类型数据的存储）](https://www.hangge.com/blog_uploads/201703/2017030617292566558.png)](https://www.hangge.com/blog/cache/detail_1641.html#)    [![原文:Swift - Realm数据库中图片的插入、读取（Data类型数据的存储）](https://www.hangge.com/blog_uploads/201703/2017030617293462642.png)](https://www.hangge.com/blog/cache/detail_1641.html#)

###  3，样例代码

```swift
import UIKit
import RealmSwift
 
class ViewController: UIViewController {
    //用于显示图片
    @IBOutlet weak var imageView: UIImageView!
     
    //使用默认的数据库
    let realm = try! Realm()
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //点击保存
    @IBAction func saveData(_ sender: Any) {
        //获取图片并转换为Data
        let imageURL =  Bundle.main.url(forResource: "0", withExtension: "png")!
        let imageData = try! Data(contentsOf: imageURL)
        //将Data数据放到实体对象中
        let portrait = HeadPortrait()
        portrait.data = imageData
        //数据持久化操作（类型记录也会自动添加的）
        try! realm.write {
            realm.add(portrait)
        }
        print("数据保存完毕!")
    }
     
    //点击加载
    @IBAction func loadData(_ sender: Any) {
        //获取所有头像图片（根据插入时间倒序排列）
        let portraits = realm.objects(HeadPortrait.self).sorted(byKeyPath: "date", ascending: false)
        //将最新一张图片显示出来
        if portraits.count > 0 {
            if let imgData = portraits[0].data {
                self.imageView.image = UIImage(data: imgData)
            }
        }
        print("数据读取完毕!")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//用户头像
class HeadPortrait:Object {
    //图片数据
    @objc dynamic var data:Data?
     
    //创建时间
    @objc dynamic var date = Date()
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1641.html