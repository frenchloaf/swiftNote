# Swift - 从相册中选择照片并上传（使用UIImagePickerController）

2016-05-13发布：hangge阅读：16082

（本文代码已升级至Swift3）

选择本地图片并上传是应用开发中一个比较常见的功能。

  [![原文:Swift - 从相册中选择照片并上传（使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016050915092952452.png)](https://www.hangge.com/blog/cache/detail_1174.html#)  [![原文:Swift - 从相册中选择照片并上传（使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016050915093994073.png)](https://www.hangge.com/blog/cache/detail_1174.html#)

我们使用 **UIImagePickerController**（图片选择器）可以很方便的从系统“照片”中选择图片，但我们会发现选择完毕后，通过图片的 **info[UIImagePickerControllerReferenceURL]** 得到的是一个引用路径，格式如下：

```
assets-library:``//asset/asset.PNG?id=90B54369-5E79-433D-B74A-E8E0870EAF27&ext=PNG
```

用这个路径是没法上传文件的。想要把选择的图片上传，通常我们会想到如下两种方式：

## 方法一：先将图片保存到一个临时文件夹下，再上传

下面样例在 **imagePickerController** 选择图片后，使用 **fileManager** 将其复制保存到应用的文档目录下，再将复制过来的图片上传。

```swift
import UIKit
import Alamofire
 
class ViewController: UIViewController, UIImagePickerControllerDelegate,
UINavigationControllerDelegate {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //选取相册
    @IBAction func fromAlbum(_ sender: Any) {
        //判断设置是否支持图片库
        if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
            //初始化图片控制器
            let picker = UIImagePickerController()
            //设置代理
            picker.delegate = self
            //指定图片控制器类型
            picker.sourceType = .photoLibrary
            //弹出控制器，显示界面
            self.present(picker, animated: true, completion: {
                () -> Void in
            })
        }else{
            print("读取相册错误")
        }
    }
     
    //选择图片成功后代理
    func imagePickerController(_ picker: UIImagePickerController,
                               didFinishPickingMediaWithInfo info: [String : Any]) {
         
        //获取选择的原图
        let pickedImage = info[UIImagePickerControllerOriginalImage] as! UIImage
         
        //将选择的图片保存到Document目录下
        let fileManager = FileManager.default
        let rootPath = NSSearchPathForDirectoriesInDomains(.documentDirectory,
                                                .userDomainMask, true)[0] as String
        let filePath = "\(rootPath)/pickedimage.jpg"
        let imageData = UIImageJPEGRepresentation(pickedImage, 1.0)
        fileManager.createFile(atPath: filePath, contents: imageData, attributes: nil)
         
        //上传图片
        if (fileManager.fileExists(atPath: filePath)){
            //取得NSURL
            let imageURL = URL(fileURLWithPath: filePath)
             
            //使用Alamofire上传
            Alamofire.upload(imageURL, to: "http://www.hangge.com/upload.php")
                .responseString { response in
                    print("Success: \(response.result.isSuccess)")
                    print("Response String: \(response.result.value ?? "")")
            }
        }
         
        //图片控制器退出
        picker.dismiss(animated: true, completion:nil)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

不管使用模拟器还是真机调试，运行后可以看到图片上传成功了：

[![原文:Swift - 从相册中选择照片并上传（使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016050915112489550.png)](https://www.hangge.com/blog/cache/detail_1174.html#)



## 方法二：使用PhotoKit获取选择图片的真实路径，再上传

```swift
import Alamofire
import Photos
 
class ViewController: UIViewController, UIImagePickerControllerDelegate,
UINavigationControllerDelegate {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //选取相册
    @IBAction func fromAlbum(_ sender: Any) {
        //判断设置是否支持图片库
        if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
            //初始化图片控制器
            let picker = UIImagePickerController()
            //设置代理
            picker.delegate = self
            //指定图片控制器类型
            picker.sourceType = .photoLibrary
            //弹出控制器，显示界面
            self.present(picker, animated: true, completion: {
                () -> Void in
            })
        }else{
            print("读取相册错误")
        }
    }
     
    //选择图片成功后代理
    func imagePickerController(_ picker: UIImagePickerController,
                               didFinishPickingMediaWithInfo info: [String : Any]) {
 
        //选择图片的引用路径
        let pickedURL = info[UIImagePickerControllerReferenceURL] as! URL
        let fetchResult: PHFetchResult = PHAsset.fetchAssets(withALAssetURLs: [pickedURL],
                                                                            options: nil)
        let asset = fetchResult.firstObject
         
        PHImageManager.default()
            .requestImageData(for: asset!, options: nil, resultHandler: {
            (imageData:Data?, dataUTI:String?, orientation:UIImageOrientation,
            info:[AnyHashable : Any]?) in
                //获取实际路径
                let imageURL = info!["PHImageFileURLKey"] as! URL
                print("路径：",imageURL)
                print("文件名：",imageURL.lastPathComponent)
                 
                //使用Alamofire上传
                Alamofire.upload(imageURL, to: "http://www.hangge.com/upload.php")
                    .responseString { response in
                        print("Success: \(response.result.isSuccess)")
                        print("Response String: \(response.result.value ?? "")")
                }
            })
         
        //图片控制器退出
        picker.dismiss(animated: true, completion:nil)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

使用模拟器运行后，可以看到图片上传成功了：

[![原文:Swift - 从相册中选择照片并上传（使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/201605091550043057.png)](https://www.hangge.com/blog/cache/detail_1174.html#)



但如果使用真机调试的话，虽然我们得到了图片的真实路径和文件名，但还是无法上传。所以上传图片还是建议使用方法一。

[![原文:Swift - 从相册中选择照片并上传（使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016050915501764969.png)](https://www.hangge.com/blog/cache/detail_1174.html#)



## 附录：

（1）本文样例使用 **Alamofire** 上传文件，对于Alamofire不熟悉的可参考我原来写过的几篇文章：

- [Swift - HTTP网络操作库Alamofire使用详解1（配置，以及数据请求）](https://www.hangge.com/blog/cache/detail_970.html)
- [Swift - HTTP网络操作库Alamofire使用详解2（文件上传）](https://www.hangge.com/blog/cache/detail_971.html)

（2）服务端php代码如下：

```php
<?php
/** php 接收流文件
* @param  String  $file 接收后保存的文件名
* @return boolean
*/
function receiveStreamFile($receiveFile){  
    $streamData = isset($GLOBALS['HTTP_RAW_POST_DATA'])? $GLOBALS['HTTP_RAW_POST_DATA'] : '';
    
    if(empty($streamData)){
        $streamData = file_get_contents('php://input');
    }
    
    if($streamData!=''){
        $ret = file_put_contents($receiveFile, $streamData, true);
    }else{
        $ret = false;
    }
   
    return $ret;  
}
  
//定义服务器存储路径和文件名
$receiveFile =  $_SERVER["DOCUMENT_ROOT"]."/uploadFiles/hangge.zip";
$ret = receiveStreamFile($receiveFile);
echo json_encode(array('success'=>(bool)$ret));
?>
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1174.html