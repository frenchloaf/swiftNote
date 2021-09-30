# Swift - 监听照片库里的变化（自动获取最新添加的图片）

2017-03-10发布：hangge阅读：2163

当我们使用微信时会发现它有个预判“**你可能要发送的照片**”的功能，具体操作步骤如下：

- 先打开微信进行聊天，然后将微信退到后台。
- 接着进行一些拍照或者截图操作。
- 再回到微信中，点击输入框旁边的加号发送附件。微信便会自动提示是否需要发送刚刚新增的那张照片，并显示照片的缩略图。（如果刚才新增了多张，则显示最后添加的那张。）

[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201701/2017011016321348195.png)](https://www.hangge.com/blog/cache/detail_1515.html#)

### 1，实现原理

- 要实现这个功能其实很简单 ，就是程序启动后监听照片库中资源的变化。
- 一旦有变化便会触发 **PHPhotoLibraryChangeObserver** 代理的 **photoLibraryDidChange** 方法。
- 我们在 **photoLibraryDidChange** 方法中可以判断具体的变化类型，比如时新增、修改还是删除，然后提示给用户。

###  2，效果图

（1）程序启动的时候获取当前所有照片资源，并添加监听。这时界面上什么都不显示。

[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201701/2017011017234530351.png)](https://www.hangge.com/blog/cache/detail_1515.html#)

（2）双击 **home** 键退回桌面，进入到相册中。我们删除 **1** 张照片，修改 **1** 张照片，新增两张照片（一张截屏图片、一张拍照图片）

[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201701/2017011017202018284.png)](https://www.hangge.com/blog/cache/detail_1515.html#)

（3）再次回到刚才的应用，会发现控制台打印出刚才照片库中的所有变化。同时界面上显示出最新添加的那张照片的缩略图。

[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201701/2017011017211582677.png)](https://www.hangge.com/blog/cache/detail_1515.html#)[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201701/2017011017224720342.png)](https://www.hangge.com/blog/cache/detail_1515.html#)

### 3，样例代码

（1）**Info.plist** 配置
由于苹果安全策略更新，在使用 **Xcode8** 开发时，需要在 **Info.plist** 配置请求照片相的关描述字段（**Privacy - Photo Library Usage Description**）

[![原文:Swift - 监听照片库里的变化（自动获取最新添加的图片）](https://www.hangge.com/blog_uploads/201610/2016101717222541521.png)](https://www.hangge.com/blog/cache/detail_1515.html#)


（2）**ViewController.swift**

```swift
import UIKit
import Photos
 
class ViewController: UIViewController {
     
    //取得的资源结果，用来存放的PHAsset
    var assetsFetchResults:PHFetchResult<PHAsset>!
     
    //带缓存的图片管理对象
    var imageManager:PHCachingImageManager!
     
    //用于显示缩略图
    var imageView: UIImageView!
     
    //缩略图大小
    var assetGridThumbnailSize:CGSize!
     
    override func viewDidLoad() {
        //imageView初始化
        imageView = UIImageView()
        imageView.frame = CGRect(x:20, y:40, width:100, height:100)
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        self.view.addSubview(imageView)
         
        //初始化和重置缓存
        self.imageManager = PHCachingImageManager()
         
        //计算我们需要的缩略图大小
        let scale = UIScreen.main.scale
        assetGridThumbnailSize = CGSize(width: imageView.frame.width*scale ,
                                        height: imageView.frame.height*scale)
         
        //申请权限
        PHPhotoLibrary.requestAuthorization({ (status) in
            if status != .authorized {
                return
            }
             
            //启动后先获取目前所有照片资源
            let allPhotosOptions = PHFetchOptions()
            allPhotosOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate",
                                                                 ascending: false)]
            allPhotosOptions.predicate = NSPredicate(format: "mediaType = %d",
                                                     PHAssetMediaType.image.rawValue)
            self.assetsFetchResults = PHAsset.fetchAssets(with: .image,
                                                          options: allPhotosOptions)
            print("--- 资源获取完毕 ---")
             
            //监听资源改变
            PHPhotoLibrary.shared().register(self)
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//PHPhotoLibraryChangeObserver代理实现，图片新增、删除、修改开始后会触发
extension ViewController:PHPhotoLibraryChangeObserver{
     
    //当照片库发生变化的时候会触发
    func photoLibraryDidChange(_ changeInstance: PHChange) {
        //获取assetsFetchResults的所有变化情况，以及assetsFetchResults的成员变化前后的数据
        guard let collectionChanges = changeInstance.changeDetails(for:
            self.assetsFetchResults as! PHFetchResult<PHObject>) else { return }
         
        DispatchQueue.main.async {
            //获取最新的完整数据
            if let allResult = collectionChanges.fetchResultAfterChanges
                as? PHFetchResult<PHAsset>{
                self.assetsFetchResults = allResult
            }
             
            if !collectionChanges.hasIncrementalChanges || collectionChanges.hasMoves{
                return
            }else{
                print("--- 监听到变化 ---")
                //照片删除情况
                if let removedIndexes = collectionChanges.removedIndexes,
                    removedIndexes.count > 0{
                    print("删除了\(removedIndexes.count)张照片")
                }
                //照片修改情况
                if let changedIndexes = collectionChanges.changedIndexes,
                    changedIndexes.count > 0{
                    print("修改了\(changedIndexes.count)张照片")
                }
                //照片新增情况
                if let insertedIndexes = collectionChanges.insertedIndexes,
                    insertedIndexes.count > 0{
                    print("新增了\(insertedIndexes.count)张照片")
                    print("将最新一张照片的缩略图显示在界面上。")
                     
                    //获取最后添加的图片资源
                    let asset = self.assetsFetchResults[insertedIndexes.first!]
                    //获取缩略图
                    self.imageManager.requestImage(for: asset,
                                            targetSize: self.assetGridThumbnailSize,
                                            contentMode: .aspectFill, options: nil) {
                                                (image, nfo) in
                                                self.imageView.image = image
                    }
                }
            }
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1515.html