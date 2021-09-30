# Swift - 相册图片多选功能的实现

2017-01-18发布：hangge阅读：7716

使用 **UIImagePickerController**，我们可以很方便的从系统相册中选择照片。但 **UIImagePickerController** 每次只能选择一张图片，不支持多选。这样如果我们需要一次上传多张图片到服务器，使用 **UIImagePickerController** 效率就会很低。

本文演示如何实现一个多选组件，类似微信发朋友圈那样，可以一次打勾选择多张照片。

## 一、组件介绍

### 1，实现原理

（1）我们通过 **Photos** 框架（**PhotoKit**）读取出所有图片数据，并使用自定义的 **collection view** 将缩略图显示出来，同时实现相关的多选功能。

（2）相簿列表同样是通过 **PhotoKit** 读取出所有相簿信息，并使用 **tableview** 展示出来。

### 2，效果图

（1）点击“**开始选择照片**”按钮，会弹出我们自定义的图片多选组件。默认显示出“**相机胶卷**”里的所有照片。

  [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/201701071847267933.png)](https://www.hangge.com/blog/cache/detail_1512.html#)    [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718473148494.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

（2）点击导航栏左上角的“**相簿**”按钮，即可列出设备中所有相簿，点击便显示出该相簿中的所有照片。

  [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718493697103.png)](https://www.hangge.com/blog/cache/detail_1512.html#)   [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718494298907.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

（3）点击图片缩略图即可在选中和取消选中状态间切换，同时图片右上角的打勾图标在状态改变时也会有弹性变化效果。

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718505630454.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

（4）选择图片的同时下方工具栏也会实时显示出选中的图片数量，这个改变时同样有动画效果。如果超过限制（可用设置），则会弹出消息提示。

  [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718532885368.png)](https://www.hangge.com/blog/cache/detail_1512.html#)   [![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718533313193.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

（5）点击“**完成**”按钮，退出图片选择组件。这里我们在在回调方法中把选中的结果给打印出来。

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010718542196006.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

## 二、组件介绍

整个组件的代码结构如下，下面分别进行介绍。

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010719261288819.png)](https://www.hangge.com/blog/cache/detail_1512.html#)



### 1，相簿列表相关

（1）**HGImagePickerController.swift**（相簿列表页控制器）

- 注意最下面，为方便使用，我扩展了 **UIViewController**，添加了个 **presentHGImagePicker()** 方法，调用后便会出现照片选择组件。
- 虽然上面方法调用时加载的是相簿列表视图。但一开始时会先加载相机胶卷图片，并自动跳转到图片选择视图。

```swift
import UIKit
import Photos
 
//相簿列表项
struct HGImageAlbumItem {
    //相簿名称
    var title:String?
    //相簿内的资源
    var fetchResult:PHFetchResult<PHAsset>
}
 
//相簿列表页控制器
class HGImagePickerController: UIViewController {
    //显示相簿列表项的表格
    @IBOutlet weak var tableView:UITableView!
     
    //相簿列表项集合
    var items:[HGImageAlbumItem] = []
     
    //每次最多可选择的照片数量
    var maxSelected:Int = Int.max
     
    //照片选择完毕后的回调
    var completeHandler:((_ assets:[PHAsset])->())?
     
    //从xib或者storyboard加载完毕就会调用
    override func awakeFromNib() {
        super.awakeFromNib()
         
        //申请权限
        PHPhotoLibrary.requestAuthorization({ (status) in
            if status != .authorized {
                return
            }
             
            // 列出所有系统的智能相册
            let smartOptions = PHFetchOptions()
            let smartAlbums = PHAssetCollection.fetchAssetCollections(with: .smartAlbum,
                                                              subtype: .albumRegular,
                                                              options: smartOptions)
            self.convertCollection(collection: smartAlbums)
             
            //列出所有用户创建的相册
            let userCollections = PHCollectionList.fetchTopLevelUserCollections(with: nil)
            self.convertCollection(collection: userCollections
                as! PHFetchResult<PHAssetCollection>)
             
            //相册按包含的照片数量排序（降序）
            self.items.sort { (item1, item2) -> Bool in
                return item1.fetchResult.count > item2.fetchResult.count
            }
             
            //异步加载表格数据,需要在主线程中调用reloadData() 方法
            DispatchQueue.main.async{
                self.tableView?.reloadData()
                 
                //首次进来后直接进入第一个相册图片展示页面（相机胶卷）
                if let imageCollectionVC = self.storyboard?
                    .instantiateViewController(withIdentifier: "hgImageCollectionVC")
                    as? HGImageCollectionViewController{
                    imageCollectionVC.title = self.items.first?.title
                    imageCollectionVC.assetsFetchResults = self.items.first?.fetchResult
                    imageCollectionVC.completeHandler = self.completeHandler
                    imageCollectionVC.maxSelected = self.maxSelected
                    self.navigationController?.pushViewController(imageCollectionVC,
                                                                  animated: false)
                }
            }
        })
    }
     
    //页面加载完毕
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //设置标题
        title = "相簿"
        //设置表格相关样式属性
        self.tableView.separatorInset = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 0)
        self.tableView.rowHeight = 55
        //添加导航栏右侧的取消按钮
        let rightBarItem = UIBarButtonItem(title: "取消", style: .plain, target: self,
                                           action:#selector(cancel) )
        self.navigationItem.rightBarButtonItem = rightBarItem
    }
     
    //转化处理获取到的相簿
    private func convertCollection(collection:PHFetchResult<PHAssetCollection>){
        for i in 0..<collection.count{
            //获取出但前相簿内的图片
            let resultsOptions = PHFetchOptions()
            resultsOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate",
                                                               ascending: false)]
            resultsOptions.predicate = NSPredicate(format: "mediaType = %d",
                                                   PHAssetMediaType.image.rawValue)
            let c = collection[i]
            let assetsFetchResult = PHAsset.fetchAssets(in: c , options: resultsOptions)
            //没有图片的空相簿不显示
            if assetsFetchResult.count > 0 {
                let title = titleOfAlbumForChinse(title: c.localizedTitle)
                items.append(HGImageAlbumItem(title: title,
                                              fetchResult: assetsFetchResult))
            }
        }
    }
     
    //由于系统返回的相册集名称为英文，我们需要转换为中文
    private func titleOfAlbumForChinse(title:String?) -> String? {
        if title == "Slo-mo" {
            return "慢动作"
        } else if title == "Recently Added" {
            return "最近添加"
        } else if title == "Favorites" {
            return "个人收藏"
        } else if title == "Recently Deleted" {
            return "最近删除"
        } else if title == "Videos" {
            return "视频"
        } else if title == "All Photos" {
            return "所有照片"
        } else if title == "Selfies" {
            return "自拍"
        } else if title == "Screenshots" {
            return "屏幕快照"
        } else if title == "Camera Roll" {
            return "相机胶卷"
        }
        return title
    }
     
    //取消按钮点击
    func cancel() {
        //退出当前视图控制器
        self.dismiss(animated: true, completion: nil)
    }
     
    //页面跳转
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        //如果是跳转到展示相簿缩略图页面
        if segue.identifier == "showImages"{
            //获取照片展示控制器
            guard let imageCollectionVC = segue.destination
                as? HGImageCollectionViewController,
                let cell = sender as? HGImagePickerCell else{
                return
            }
            //设置回调函数
            imageCollectionVC.completeHandler = completeHandler
            //设置标题
            imageCollectionVC.title = cell.titleLabel.text
            //设置最多可选图片数量
            imageCollectionVC.maxSelected = self.maxSelected
            guard  let indexPath = self.tableView.indexPath(for: cell) else { return }
             
            //获取选中的相簿信息
            let fetchResult = self.items[indexPath.row].fetchResult
            //传递相簿内的图片资源
            imageCollectionVC.assetsFetchResults = fetchResult
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//相簿列表页控制器UITableViewDelegate,UITableViewDataSource协议方法的实现
extension HGImagePickerController:UITableViewDelegate,UITableViewDataSource{
    //设置单元格内容
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        //同一形式的单元格重复使用，在声明时已注册
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
            as! HGImagePickerCell
        let item = self.items[indexPath.row]
        cell.titleLabel.text = "\(item.title ?? "") "
        cell.countLabel.text = "（\(item.fetchResult.count)）"
        return cell
    }
     
    //表格单元格数量
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //表格单元格选中
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
 
extension UIViewController {
    //HGImagePicker提供给外部调用的接口，同于显示图片选择页面
    func presentHGImagePicker(maxSelected:Int = Int.max,
                              completeHandler:((_ assets:[PHAsset])->())?)
        -> HGImagePickerController?{
        //获取图片选择视图控制器
        if let vc = UIStoryboard(name: "HGImage", bundle: Bundle.main)
            .instantiateViewController(withIdentifier: "imagePickerVC")
            as? HGImagePickerController{
            //设置选择完毕后的回调
            vc.completeHandler = completeHandler
            //设置图片最多选择的数量
            vc.maxSelected = maxSelected
            //将图片选择视图控制器外添加个导航控制器，并显示
            let nav = UINavigationController(rootViewController: vc)
            self.present(nav, animated: true, completion: nil)
            return vc
        }
        return nil
    }
}
```


（2）**HGImagePickerCell.swift**（相簿列表单元格）

```swift
import UIKit
 
//相簿列表单元格
class HGImagePickerCell: UITableViewCell {
    //相簿名称标签
    @IBOutlet weak var titleLabel:UILabel!
    //照片数量标签
    @IBOutlet weak var countLabel:UILabel!
     
    override func awakeFromNib() {
        super.awakeFromNib()
        self.layoutMargins = UIEdgeInsets.zero
    }
 
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
}
```

###  2，图片选择页面相关

（1）**HGImageCollectionViewController.swift**（图片缩略图集合页控制器）

```swift
import UIKit
import Photos
 
//图片缩略图集合页控制器
class HGImageCollectionViewController: UIViewController {
    //用于显示所有图片缩略图的collectionView
    @IBOutlet weak var collectionView:UICollectionView!
     
    //下方工具栏
    @IBOutlet weak var toolBar:UIToolbar!
 
    //取得的资源结果，用了存放的PHAsset
    var assetsFetchResults:PHFetchResult<PHAsset>?
     
    //带缓存的图片管理对象
    var imageManager:PHCachingImageManager!
     
    //缩略图大小
    var assetGridThumbnailSize:CGSize!
     
    //每次最多可选择的照片数量
    var maxSelected:Int = Int.max
     
    //照片选择完毕后的回调
    var completeHandler:((_ assets:[PHAsset])->())?
     
    //完成按钮
    var completeButton:HGImageCompleteButton!
     
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
         
        //根据单元格的尺寸计算我们需要的缩略图大小
        let scale = UIScreen.main.scale
        let cellSize = (self.collectionView.collectionViewLayout as!
            UICollectionViewFlowLayout).itemSize
        assetGridThumbnailSize = CGSize(width: cellSize.width*scale ,
                                        height: cellSize.height*scale)
    }
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //背景色设置为白色（默认是黑色）
        self.collectionView.backgroundColor = UIColor.white
         
        //初始化和重置缓存
        self.imageManager = PHCachingImageManager()
        self.resetCachedAssets()
         
        //设置单元格尺寸
        let layout = (self.collectionView.collectionViewLayout as!
            UICollectionViewFlowLayout)
        layout.itemSize = CGSize(width: UIScreen.main.bounds.size.width/4-1,
                                 height: UIScreen.main.bounds.size.width/4-1)
        //允许多选
        self.collectionView.allowsMultipleSelection = true
         
        //添加导航栏右侧的取消按钮
        let rightBarItem = UIBarButtonItem(title: "取消", style: .plain,
                                           target: self, action: #selector(cancel))
        self.navigationItem.rightBarButtonItem = rightBarItem
       
        //添加下方工具栏的完成按钮
        completeButton = HGImageCompleteButton()
        completeButton.addTarget(target: self, action: #selector(finishSelect))
        completeButton.center = CGPoint(x: UIScreen.main.bounds.width - 50, y: 22)
        completeButton.isEnabled = false
        toolBar.addSubview(completeButton)
    }
 
    //重置缓存
    func resetCachedAssets(){
        self.imageManager.stopCachingImagesForAllAssets()
    }
     
     
    //取消按钮点击
    func cancel() {
        //退出当前视图控制器
        self.navigationController?.dismiss(animated: true, completion: nil)
    }
     
    //获取已选择个数
    func selectedCount() -> Int {
        return self.collectionView.indexPathsForSelectedItems?.count ?? 0
    }
 
    //完成按钮点击
    func finishSelect(){
        //取出已选择的图片资源
        var assets:[PHAsset] = []
        if let indexPaths = self.collectionView.indexPathsForSelectedItems{
            for indexPath in indexPaths{
                assets.append(assetsFetchResults![indexPath.row] )
            }
        }
        //调用回调函数
        self.navigationController?.dismiss(animated: true, completion: {
            self.completeHandler?(assets)
        })
    }
}
 
//图片缩略图集合页控制器UICollectionViewDataSource,UICollectionViewDelegate协议方法的实现
extension HGImageCollectionViewController:UICollectionViewDataSource
,UICollectionViewDelegate{
    //CollectionView项目
    func collectionView(_ collectionView: UICollectionView,
                        numberOfItemsInSection section: Int) -> Int {
        return self.assetsFetchResults?.count ?? 0
    }
     
    // 获取单元格
    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        //获取storyboard里设计的单元格，不需要再动态添加界面元素
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell",
                                    for: indexPath) as! HGImageCollectionViewCell
        let asset = self.assetsFetchResults![indexPath.row]
        //获取缩略图
        self.imageManager.requestImage(for: asset, targetSize: assetGridThumbnailSize,
                                       contentMode: .aspectFill, options: nil) {
                                        (image, nfo) in
            cell.imageView.image = image
        }
         
        return cell
    }
     
    //单元格选中响应
    func collectionView(_ collectionView: UICollectionView,
                        didSelectItemAt indexPath: IndexPath) {
        if let cell = collectionView.cellForItem(at: indexPath)
            as? HGImageCollectionViewCell{
            //获取选中的数量
            let count = self.selectedCount()
            //如果选择的个数大于最大选择数
            if count > self.maxSelected {
                //设置为不选中状态
                collectionView.deselectItem(at: indexPath, animated: false)
                //弹出提示
                let title = "你最多只能选择\(self.maxSelected)张照片"
                let alertController = UIAlertController(title: title, message: nil,
                                                        preferredStyle: .alert)
                 
                let cancelAction = UIAlertAction(title:"我知道了", style: .cancel,
                                                 handler:nil)
                alertController.addAction(cancelAction)
                self.present(alertController, animated: true, completion: nil)
            }
            //如果不超过最大选择数
            else{
                //改变完成按钮数字，并播放动画
                completeButton.num = count
                if count > 0 && !self.completeButton.isEnabled{
                    completeButton.isEnabled = true
                }
                cell.playAnimate()
            }
        }
    }
     
    //单元格取消选中响应
    func collectionView(_ collectionView: UICollectionView,
                        didDeselectItemAt indexPath: IndexPath) {
        if let cell = collectionView.cellForItem(at: indexPath)
            as? HGImageCollectionViewCell{
             //获取选中的数量
            let count = self.selectedCount()
            completeButton.num = count
             //改变完成按钮数字，并播放动画
            if count == 0{
                completeButton.isEnabled = false
            }
            cell.playAnimate()
        }
    }
}
```


（2）**HGImageCollectionViewCell.swift**（图片缩略图集合页单元格）

```swift
import UIKit
 
//图片缩略图集合页单元格
open class HGImageCollectionViewCell: UICollectionViewCell {
    //显示缩略图
    @IBOutlet weak var imageView:UIImageView!
     
    //显示选中状态的图标
    @IBOutlet weak var selectedIcon:UIImageView!
     
    //设置是否选中
    open override var isSelected: Bool {
        didSet{
            if isSelected {
                selectedIcon.image = UIImage(named: "hg_image_selected")
            }else{
                selectedIcon.image = UIImage(named: "hg_image_not_selected")
            }
        }
    }
     
    //播放动画，是否选中的图标改变时使用
    func playAnimate() {
        //图标先缩小，再放大
        UIView.animateKeyframes(withDuration: 0.4, delay: 0, options: .allowUserInteraction,
                                animations: {
            UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 0.2,
                               animations: {
                self.selectedIcon.transform = CGAffineTransform(scaleX: 0.7, y: 0.7)
            })
            UIView.addKeyframe(withRelativeStartTime: 0.2, relativeDuration: 0.4,
                               animations: {
                self.selectedIcon.transform = CGAffineTransform.identity
            })
            }, completion: nil)
    }
     
    open override func awakeFromNib() {
        super.awakeFromNib()
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
    }
}
```


（3）**HGImageCompleteButton.swift**（照片选择页下方工具栏的“完成”按钮）

```swift
import UIKit
 
//照片选择页下方工具栏的“完成”按钮
class HGImageCompleteButton: UIView {
    //已选照片数量标签
    var numLabel:UILabel!
    //按钮标题标签“完成”
    var titleLabel:UILabel!
     
    //按钮的默认尺寸
    let defaultFrame = CGRect(x:0, y:0, width:70, height:20)
     
    //文字颜色（同时也是数字背景颜色）
    let titleColor = UIColor(red: 0x09/255, green: 0xbb/255, blue: 0x07/255, alpha: 1)
     
    //点击点击手势
    var tapSingle:UITapGestureRecognizer?
     
    //设置数量
    var num:Int = 0{
        didSet{
            if num == 0{
                numLabel.isHidden = true
            }else{
                numLabel.isHidden = false
                numLabel.text = "\(num)"
                playAnimate()
            }
        }
    }
     
    //是否可用
    var isEnabled:Bool = true {
        didSet{
            if isEnabled {
                titleLabel.textColor = titleColor
                tapSingle?.isEnabled = true
            }else{
                titleLabel.textColor = UIColor.gray
                tapSingle?.isEnabled = false
            }
        }
    }
     
    init(){
        super.init(frame:defaultFrame)
     
        //已选照片数量标签初始化
        numLabel = UILabel(frame:CGRect(x: 0 , y: 0 , width: 20, height: 20))
        numLabel.backgroundColor = titleColor
        numLabel.layer.cornerRadius = 10
        numLabel.layer.masksToBounds = true
        numLabel.textAlignment = .center
        numLabel.font = UIFont.systemFont(ofSize: 15)
        numLabel.textColor = UIColor.white
        numLabel.isHidden = true
        self.addSubview(numLabel)
         
        //按钮标题标签初始化
        titleLabel = UILabel(frame:CGRect(x: 20 , y: 0 ,
                                          width: defaultFrame.width - 20,
                                          height: 20))
        titleLabel.text = "完成"
        titleLabel.textAlignment = .center
        titleLabel.font = UIFont.systemFont(ofSize: 15)
        titleLabel.textColor = titleColor
        self.addSubview(titleLabel)
    }
     
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
     
    //用户数字改变时播放的动画
    func playAnimate() {
        //从小变大，且有弹性效果
        self.numLabel.transform = CGAffineTransform(scaleX: 0.1, y: 0.1)
        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.5,
                       initialSpringVelocity: 0.5, options: UIViewAnimationOptions(),
                       animations: {
            self.numLabel.transform = CGAffineTransform.identity
        }, completion: nil)
    }
     
    //添加事件响应
    func addTarget(target: Any?, action: Selector?) {
        //单击监听
        tapSingle = UITapGestureRecognizer(target:target,action:action)
        tapSingle!.numberOfTapsRequired = 1
        tapSingle!.numberOfTouchesRequired = 1
        self.addGestureRecognizer(tapSingle!)
    }
}
```



### 3，相关资源

（1）**HGImage.xcassets**
里面就两个图标图片，分别表示选中和未选中状态。

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010719590326535.png)](https://www.hangge.com/blog/cache/detail_1512.html#)

（2）**HGImage.storyboard**

- 在 **storyboard** 中添加两个 **View Controller Scene**，分别绑定上面的 **HGImagePickerController** 和 **HGImageCollectionViewController**。
- 同时两个 **Scene** 中的 **cell** 分别绑定 **HGImagePickerCell** 和 **HGImageCollectionViewCell**。
- **Image Picker Controller Scene** 的 **Storyboard ID** 设置为 **imagePickerVC**。单元格 **identifier** 设置为 **cell**。
- **Image Collection View Controller Scene** 的 **Storyboard ID** 设置为 **hgImageCollectionVC**。单元格 **identifier** 设置为 **cell**。
- 从 **Image Picker Controller Scene** 的单元格拖动一个 **Segue** 到 **Image Collection View Controller Scene**。类型为 **Show**，**identifier** 为 **showImages**。

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201701/2017010721091921608.png)](https://www.hangge.com/blog/cache/detail_1512.html#)



## 三、使用样例

### 1，Info.plist配置

由于苹果安全策略更新，在使用 **Xcode8** 开发时，需要在 **Info.plist** 配置请求照片相的关描述字段（**Privacy - Photo Library Usage Description**）

[![原文:Swift - 相册图片多选功能的实现](https://www.hangge.com/blog_uploads/201610/2016101717222541521.png)](https://www.hangge.com/blog/cache/detail_1512.html#)



### 2，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
    }
 
    //“开始选择照片”按钮点击
    @IBAction func buttonTapped(_ sender: AnyObject) {
        //开始选择照片，最多允许选择4张
        _ = self.presentHGImagePicker(maxSelected:4) { (assets) in
            //结果处理
            print("共选择了\(assets.count)张图片，分别如下：")
            for asset in assets {
                print(asset)
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1512.zip](https://www.hangge.com/blog_uploads/201708/2017081211132974936.zip)



## 附：实现视频多选功能

要想实现视频文件多选功能的话，只需要把 **HGImagePickerController.swift** 中的资源类型改成视频即可：

```
//转化处理获取到的相簿
private func convertCollection(collection:PHFetchResult<PHAssetCollection>){
    for i in 0..<collection.count{
        //获取出但前相簿内的图片
        let resultsOptions = PHFetchOptions()
        resultsOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate",
                                                           ascending: false)]
        resultsOptions.predicate = NSPredicate(format: "mediaType = %d",
                                               PHAssetMediaType.video.rawValue)
        let c = collection[i]
        let assetsFetchResult = PHAsset.fetchAssets(in: c , options: resultsOptions)
        //没有图片的空相簿不显示
        if assetsFetchResult.count > 0 {
            let title = titleOfAlbumForChinse(title: c.localizedTitle)
            items.append(HGImageAlbumItem(title: title,
                                          fetchResult: assetsFetchResult))
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1512.html