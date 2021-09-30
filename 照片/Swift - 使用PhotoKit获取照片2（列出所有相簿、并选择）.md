# Swift - 使用PhotoKit获取照片2（列出所有相簿、并选择）

2016-06-19发布：hangge阅读：4927

（本文代码已升级至Swift3）

在前面一篇文章中：[Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog/cache/detail_1233.html)。介绍了如何使用 **Photos** 框架来获取相机胶卷中的所有图片。

本文在起基础上做个功能改进，添加选择照片相簿的功能。

**1，样例说明**

（1）首先通过 **tableView** 将系统中的所有智能相簿，以及用户自定义的相簿通过表格的形式展示出来。

（2）相簿按照内部包含的图片数量进行降序排列。同时如果某个相簿内部没有任何图片，则将其过滤掉不显示。

（3）点击某个相簿，则会展示出该相簿下所有照片的缩略图。

（4）其他功能同前一篇文章一样（包括点击缩略图显示原图，以及图片信息）

**2，效果图如下**

  [![原文:Swift - 使用PhotoKit获取照片2（列出所有相簿、并选择）](https://www.hangge.com/blog_uploads/201606/2016061517113190426.png)](https://www.hangge.com/blog/cache/detail_1234.html#)  [![原文:Swift - 使用PhotoKit获取照片2（列出所有相簿、并选择）](https://www.hangge.com/blog_uploads/201606/2016061517114539626.png)](https://www.hangge.com/blog/cache/detail_1234.html#)  [![原文:Swift - 使用PhotoKit获取照片2（列出所有相簿、并选择）](https://www.hangge.com/blog_uploads/201606/2016061517120550603.png)](https://www.hangge.com/blog/cache/detail_1234.html#)

**3，详细代码**

**--- 相簿列表首页 TableViewController.swift ---**

```swift
import UIKit
import Photos
 
//相簿列表项
class AlbumItem {
    //相簿名称
    var title:String?
    //相簿内的资源
    var fetchResult:PHFetchResult<PHAsset>
     
    init(title:String?,fetchResult:PHFetchResult<PHAsset>){
        self.title = title
        self.fetchResult = fetchResult
    }
}
 
class TableViewController: UITableViewController {
    //相簿列表项集合
    var items:[AlbumItem] = []
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
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
            }
        })
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
            if assetsFetchResult.count > 0{
                let title = titleOfAlbumForChinse(title: c.localizedTitle)
                items.append(AlbumItem(title: title,
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
     
    //表格分区数
    override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //表格单元格数量
    override func tableView(_ tableView: UITableView,
                            numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //设置单元格内容
    override func tableView(_ tableView: UITableView,
                            cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        //为了提供表格显示性能，已创建完成的单元需重复使用
        let identify:String = "myCell"
        //同一形式的单元格重复使用，在声明时已注册
        let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                 for: indexPath) as UITableViewCell
        let item = self.items[indexPath.row]
        let titleLabel = cell.contentView.viewWithTag(1) as! UILabel
        titleLabel.text = item.title
        let countLabel = cell.contentView.viewWithTag(2) as! UILabel
        countLabel.text = "(\(item.fetchResult.count))"
        return cell
    }
     
    //页面跳转
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        //如果是跳转到展示相簿缩略图页面
        if segue.identifier == "showPhotos"{
             
            guard let collectionViewController = segue.destination
                as? CollectionViewController,
                let cell = sender as? UITableViewCell else{
                    return
            }
             
            guard let indexPath = self.tableView.indexPath(for: cell) else { return }
             
            //获取选中的相簿信息
            let item = self.items[indexPath.row]
            //设置标题
            collectionViewController.title = item.title
            //传递相簿内的图片资源
            collectionViewController.assetsFetchResults = item.fetchResult
 
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**--- 缩略图展示页 CollectionViewController.swift ---**
（注意：高亮部分表示相较于前文，修改过的地方）

```swift
import UIKit
import Photos
 
class CollectionViewController: UICollectionViewController {
     
    ///取得的资源结果，用了存放的PHAsset
    var assetsFetchResults:PHFetchResult<PHAsset>!
     
    ///缩略图大小
    var assetGridThumbnailSize:CGSize!
     
    /// 带缓存的图片管理对象
    var imageManager:PHCachingImageManager!
     
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
         
        //根据单元格的尺寸计算我们需要的缩略图大小
        let scale = UIScreen.main.scale
        let cellSize = (self.collectionViewLayout as! UICollectionViewFlowLayout).itemSize
        assetGridThumbnailSize = CGSize(width:cellSize.width*scale ,
                                        height:cellSize.height*scale)
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        // 如果没有传入值 则获取所有资源
        if assetsFetchResults == nil {
            //则获取所有资源
            let allPhotosOptions = PHFetchOptions()
            //按照创建时间倒序排列
            allPhotosOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate",
                                                                 ascending: false)]
            //只获取图片
            allPhotosOptions.predicate = NSPredicate(format: "mediaType = %d",
                                                     PHAssetMediaType.image.rawValue)
            assetsFetchResults = PHAsset.fetchAssets(with: PHAssetMediaType.image,
                                                     options: allPhotosOptions)
        }
         
        // 初始化和重置缓存
        self.imageManager = PHCachingImageManager()
        self.resetCachedAssets()
    }
     
    //重置缓存
    func resetCachedAssets(){
        self.imageManager.stopCachingImagesForAllAssets()
    }
     
    // CollectionView行数
    override func collectionView(_ collectionView: UICollectionView,
                            numberOfItemsInSection section: Int) -> Int {
        return self.assetsFetchResults.count
    }
     
    // 获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                    cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = (self.collectionView?.dequeueReusableCell(
            withReuseIdentifier: identify, for: indexPath))! as UICollectionViewCell
         
        let asset = self.assetsFetchResults[indexPath.row]
        //获取缩略图
        self.imageManager.requestImage(for: asset, targetSize: assetGridThumbnailSize,
                                       contentMode: PHImageContentMode.aspectFill,
                                       options: nil) { (image, nfo) in
                                        (cell.contentView.viewWithTag(1) as! UIImageView)
                                            .image = image
                                        print(image)
        }
        return cell
    }
     
    // 单元格点击响应
    override func collectionView(_ collectionView: UICollectionView,
                                 didSelectItemAt indexPath: IndexPath) {
        let myAsset = self.assetsFetchResults[indexPath.row]
         
        //这里不使用segue跳转（建议用segue跳转）
        let detailViewController = UIStoryboard(name: "Main", bundle: nil)
            .instantiateViewController(withIdentifier: "detail")
            as! ImageDetailViewController
        detailViewController.myAsset = myAsset
         
        // navigationController跳转到detailViewController
        self.navigationController!.pushViewController(detailViewController,
                                                      animated:true)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```


**--- 详情页 ImageDetailViewController.swift ---**
（注意：这个完全没有改动。）

```swift
import UIKit
import Photos
 
class ImageDetailViewController: UIViewController {
    //选中的图片资源
    var myAsset:PHAsset!
    //用于显示图片信息
    @IBOutlet weak var textView: UITextView!
    //用于显示原图
    @IBOutlet weak var imageView: UIImageView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取文件名
        PHImageManager.default().requestImageData(for: myAsset, options: nil,
                                                                 resultHandler: {
                                                                    _, _, _, info in
            self.title = (info!["PHImageFileURLKey"] as! NSURL).lastPathComponent
        })
         
        //获取图片信息
        textView.text = "日期：\(myAsset.creationDate!)\n"
            + "类型：\(myAsset.mediaType.rawValue)\n"
            + "位置：\(myAsset.location)\n"
            + "时长：\(myAsset.duration)\n"
         
        //获取原图
        PHImageManager.default().requestImage(for: myAsset,
                         targetSize: PHImageManagerMaximumSize , contentMode: .default,
                         options: nil, resultHandler: {
                            (image, _: [AnyHashable : Any]?) in
                            self.imageView.image = image
        })
         
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```


**4，源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1233.zip](https://www.hangge.com/blog_uploads/201701/2017010712373278870.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1234.html