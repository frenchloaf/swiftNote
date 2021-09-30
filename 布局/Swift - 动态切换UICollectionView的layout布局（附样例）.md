# Swift - 动态切换UICollectionView的layout布局（附样例）

2017-03-24发布：hangge阅读：8549

在项目中，有时我们会对 **UICollectionView** 提供多套不同的布局样式，从而实现不同的展示效果。用户可以根据需求，自由切换使用不同的布局样式。这个通过 **collectionView** 的  **setCollectionViewLayout** 方法就可以实现。

### 1，setCollectionViewLayout方法介绍

（1）使用 **setCollectionViewLayout** 设置新布局后，只会调整单元格的位置，不会刷新单元格数据和样式。如果单元格内容有变化，则还需要调用 **reloadData()** 方法重新加载。

（2）通过 **setCollectionViewLayout** 方法的 **animated** 参数可以指定布局变化时，是否会播放动画（即每个单元格会自动移动到新的位置，并有动画过渡效果）

### 2，效果图

（1）页面初始化时，**collectionView** 默认使用自定义的复杂布局（一大两小）

（2）点击导航栏的“**切换**”按钮，可以使 **collectionView** 在复杂布局和普通的 **flow** 流式布局间相互切换。

（3）两种布局互相切换时还会有动画效果。

  [![原文:Swift - 动态切换UICollectionView的layout布局（附样例）](https://www.hangge.com/blog_uploads/201703/2017031314052444340.png)](https://www.hangge.com/blog/cache/detail_1594.html#)   [![原文:Swift - 动态切换UICollectionView的layout布局（附样例）](https://www.hangge.com/blog_uploads/201703/2017031314053799756.png)](https://www.hangge.com/blog/cache/detail_1594.html#)

### 3，样例代码

注意：下面代码中 **CustomLayout** 便是我们自定义的复杂布局类，具体内容可以参考我的这篇文章：[Swift - 使用网格（UICollectionView）的自定义布局实现复杂页面](https://www.hangge.com/blog/cache/detail_591.html)

```swift
import UIKit
 
class ViewController: UIViewController {
    //普通的flow流式布局
    var flowLayout:UICollectionViewFlowLayout!
    //自定义的线性布局
    var customLayout:CustomLayout!
     
    var collectionView:UICollectionView!
     
    //重用的单元格的Identifier
    let CellIdentifier = "myCell"
     
    //课程名称和图片，每一门课程用字典来表示
    let courses = [
        ["name":"Swift","pic":"swift.png"],
        ["name":"Xcode","pic":"xcode.png"],
        ["name":"Java","pic":"java.png"],
        ["name":"PHP","pic":"php.png"],
        ["name":"JS","pic":"js.png"],
        ["name":"React","pic":"react.png"],
        ["name":"Ruby","pic":"ruby.png"],
        ["name":"HTML","pic":"html.png"],
        ["name":"C#","pic":"c#.png"]
    ]
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化Collection View
        initCollectionView()
    }
     
    private func initCollectionView() {
        //初始化flow布局
        flowLayout = UICollectionViewFlowLayout()
        flowLayout.itemSize = CGSize(width: 100, height: 100)
         
        //初始化自定义布局
        customLayout = CustomLayout()
         
        //初始化Collection View
        collectionView = UICollectionView(frame: view.bounds,
                                          collectionViewLayout: customLayout)
         
        //Collection View代理设置
        collectionView.delegate = self
        collectionView.dataSource = self
        collectionView.backgroundColor = .white
         
        //注册重用的单元格
        let cellXIB = UINib.init(nibName: "MyCollectionViewCell", bundle: Bundle.main)
        collectionView.register(cellXIB, forCellWithReuseIdentifier: CellIdentifier)
         
        //将Collection View添加到主视图中
        view.addSubview(collectionView)
    }
     
    //切换布局样式
    @IBAction func changeLayout(_ sender: Any) {
        self.collectionView.collectionViewLayout.invalidateLayout()
        
        //交替切换新布局
        let newLayout = collectionView.collectionViewLayout
            .isKind(of: CustomLayout.self) ? flowLayout : customLayout
        collectionView.setCollectionViewLayout(newLayout, animated: true)
        //将滚动条移动到顶部
        let indexPath = IndexPath(row: 0, section: 0)
        self.collectionView.scrollToItem(at: indexPath, at:.top, animated: false)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//Collection View数据源协议相关方法
extension ViewController: UICollectionViewDataSource {
    //获取分区数
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 1
    }
     
    //获取每个分区里单元格数量
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return courses.count
    }
     
    //返回每个单元格视图
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取重用的单元格
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier:
            CellIdentifier, for: indexPath) as! MyCollectionViewCell
        cell.label.text = courses[indexPath.item]["name"]
        //设置内部显示的图片
        cell.imageView.image = UIImage(named: courses[indexPath.item]["pic"]!)
        return cell
    }
}
 
//Collection View样式布局协议相关方法
extension ViewController: UICollectionViewDelegate {
     
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1594.zip](https://www.hangge.com/blog_uploads/201703/2017031619494540533.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1594.html