# Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）

2016-06-16发布：hangge阅读：9167

（本文代码已升级至Swift4）



我之前写过一篇文章：[Swift - 使用ALAssetsLibrary获取相簿里所有图片，视频（附样例）](https://www.hangge.com/blog/cache/detail_763.html)。介绍了如何使用 **AssetsLibrary** 框架来读取并显示系统中的所有照片。

几年以来，相机应用和照片应用发生了显著的变化，增加了许多新特性，包括按时刻来组织照片的方式。但与此同时，**AssetsLibrary** 框架却没有跟上步伐。所以到了**iOS 9**后，**AssetsLibrary** 框架要被彻底废除。
作为替代，苹果自 **iOS 8** 起提供了一个更现代化的框架 **PhotoKit**。本文将使用 **PhotoKit** 来实现同之前文章一样的功能：读取并显示设备中的所有照片。

**1，Photos框架的特点**

淡化了照片库中 **URL** 的概念，改之使用一个标志符来唯一代表一个资源。同时这个标志符还是动态变化的，也就是说即使同一个资源，每次启动应用后获取的标志符和上一次都是不一样的。


**2，获取原图**
通过 **PHImageManager.default().requestImageData()** 这个方法可以获取图片的缩略图或者原图。如果要获取原图，即不对照片大小进行修改或裁剪，那么方法里参数要进行如下设置：

 **targetSize** 设成 **PHImageManagerMaximumSize** 

 **contentMode** 设成 **.default** （其实 **targetSize** 如果是 **PHImageManagerMaximumSize** 的话，**contentMode** 不管设置成什么都会视作 **.default** ）

**3，获取缩略图**
（1）与 **AssetsLibrary** 相比，**PhotoKit** 没有直接的提供缩略图方法。我们需要指定需要获取的缩略图尺寸，系统会按照你的要求来返回接近该尺寸的图像（会提供刚好大于或等于该尺寸的缩略图，没有的话则返回原图）。
（2）前面提到，**PHImageManager.default().requestImageData()** 既可以用来获取原图，也可以用来获取缩略图。这里我们使用 **PHCachingImageManager**，它是 **PHImageManager** 的子类，所以获取图片的方法是一样的。但这个可以用于缓存 **PHAsset**，这样可以快速获取照片或视频。在 **UICollectionViewController** 中使用比较适合。



**4，样例说明**

（1）启动程序后会获取相机胶卷中所有图片的缩略图，并在 **UICollectionViewController** 中显示出来。
（2）图片按照创建时间倒序排列（最近拍的照片在最上面）。
（3）点击缩略图，可以查看照片的原图以及图片的相关信息。  

**5，效果图如下**

   [![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201606/2016061514132545587.png)](https://www.hangge.com/blog/cache/detail_1233.html#)   [![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201606/2016061514133010749.png)](https://www.hangge.com/blog/cache/detail_1233.html#)



**6，Info.plist配置**

由于苹果安全策略更新，在使用Xcode8开发时，需要在 **Info.plist** 配置请求照片相的关描述字段（**Privacy - Photo Library Usage Description**）

[![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201610/2016101717222541521.png)](https://www.hangge.com/blog/cache/detail_1233.html#)


**7，详细代码**
**--- 首页 CollectionViewController.swift ---**

```swift
import UIKit
import Photos
 
class CollectionViewController: UICollectionViewController {
     
    ///取得的资源结果，用了存放的PHAsset
    var assetsFetchResults:PHFetchResult<PHAsset>?
     
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
         
        //申请权限
        PHPhotoLibrary.requestAuthorization({ (status) in
            if status != .authorized {
                return
            }
             
            //则获取所有资源
            let allPhotosOptions = PHFetchOptions()
            //按照创建时间倒序排列
            allPhotosOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate",
                                                                 ascending: false)]
            //只获取图片
            allPhotosOptions.predicate = NSPredicate(format: "mediaType = %d",
                                                     PHAssetMediaType.image.rawValue)
            self.assetsFetchResults = PHAsset.fetchAssets(with: PHAssetMediaType.image,
                                                     options: allPhotosOptions)
             
            // 初始化和重置缓存
            self.imageManager = PHCachingImageManager()
            self.resetCachedAssets()
             
            //collection view 重新加载数据
            DispatchQueue.main.async{
                self.collectionView?.reloadData()
            }
        })
    }
     
    //重置缓存
    func resetCachedAssets(){
        self.imageManager.stopCachingImagesForAllAssets()
    }
     
    // CollectionView行数
    override func collectionView(_ collectionView: UICollectionView,
                            numberOfItemsInSection section: Int) -> Int {
        return self.assetsFetchResults?.count ?? 0
    }
     
    // 获取单元格
    override func collectionView(_ collectionView: UICollectionView,
                    cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        // storyboard里设计的单元格
        let identify:String = "DesignViewCell"
        // 获取设计的单元格，不需要再动态添加界面元素
        let cell = (self.collectionView?.dequeueReusableCell(
            withReuseIdentifier: identify, for: indexPath))! as UICollectionViewCell
         
        if let asset = self.assetsFetchResults?[indexPath.row] {
            //获取缩略图
            self.imageManager.requestImage(for: asset, targetSize: assetGridThumbnailSize,
                        contentMode: PHImageContentMode.aspectFill,
                        options: nil) { (image, nfo) in
                        (cell.contentView.viewWithTag(1) as! UIImageView).image = image
                        print(image ?? "")
            }
        }
         
        return cell
    }
     
    // 单元格点击响应
    override func collectionView(_ collectionView: UICollectionView,
                                 didSelectItemAt indexPath: IndexPath) {
        let myAsset = self.assetsFetchResults![indexPath.row]
         
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
            + "位置：\(String(describing: myAsset.location))\n"
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

**7，图像的裁剪与传输策略**
上面样例运行后会发现虽然相册中的图片只有7张，但 **collectionView** 页面却加载了14次：

[![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201607/2016072722405784683.png)](https://www.hangge.com/blog/cache/detail_1233.html#)


这是由于图像管理器的优化策略，即它在将图像的高质量版本递送给你之前，先传递一个较低质量的版本过来。这样就会让页面UI更加顺畅。

当然这些策略也是可以通过 **requestImageData()** 方法中的 **options** 参数进行设置的（类型是 **PHImageRequestOptions**。本文样例都是传的都是 **nil**，即使用默认设置。）



**8，PHImageRequestOptions对象常用属性说明**

**（1）resizeMode 属性**
可以设置为 **.exact** (返回图像必须和目标大小相匹配)，**.fast** (比 **.exact** 效率更高，但返回图像可能和目标大小不一样，接近且稍大与目标大小) 或者 **.none**（默认值）。

比如：我们如果将其设置成 **.exact**，那么输出结果如下：

[![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201607/2016072723030890384.png)](https://www.hangge.com/blog/cache/detail_1233.html#)



**（2）deliveryMode 属性**
前面提到的在得到高质量的图片之前，会先传输个低质量的策略就是由这个属性设置。一共有三种方式：
**.opportunistic**（默认值）
**.fastFormat**（如果你想要更快的加载速度，且可以牺牲一点图像质量，可以设成这个。）
**.highQualityFormat**（如果你只想要高质量的图像，并且可以接受更长的加载时间，那么就设成这个。）
比如我们将 **deliveryMode** 设置 **.highQualityFormat**，那么可以看到图片只加载了7次，之前不会再先获得低质量的图片了。

[![原文:Swift - 使用PhotoKit获取照片1（获取所有照片缩略图、原图及其信息）](https://www.hangge.com/blog_uploads/201607/201607272312368030.png)](https://www.hangge.com/blog/cache/detail_1233.html#)

**（3）synchronous 属性**
默认是 **false**，表示 **requestImageData()** 方法是异步操作。如果设置成 **true**，这个就变成同步的了。
注意：当 **synchronous** 设为 **true** 时，**deliveryMode** 属性就会被忽略，并被当作 **.highQualityFormat** 来处理。

**9，源码下载**：![img](https://www.hangge.com/blog/admin0822/include/edit/sysimage/icon16/zip.gif)[hangge_1233.zip](https://www.hangge.com/blog_uploads/201810/2018102310572077725.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1233.html