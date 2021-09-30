# Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）

2016-07-08发布：hangge阅读：5493

（本文代码已升级至Swift4）

有时我们需要从系统相册中选择视频录像，来进行编辑或者上传操作，这时使用 **UIImagePickerController** 就可以实现。
默认情况下，**UIImagePickerController** 打开系统“照片”后允许用户选择所有的媒体文件（不管是照片还是录像），我们可以通过 **mediaTypes** 属性设置。让其只显示视频录像。



**1，样例说明**
（1）下面样例点击“选择视频”按钮后，会自动打开相册选择视频。
（2）由于设置了 **mediaTypes**，所有的图片都会过滤掉，只留下视频选择。
（3）选择完毕，系统会自动将视频复制一个到应用的 **tmp** 文件夹（临时文件夹）下。我们可以直接对这个文件进行操作，而不会影响到系统相册中的原视频。
（4）本样例选择后，就直接使用 **AVPlayerViewController** 进行播放。

**2，效果图**

 [![原文:Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016051816581261488.png)](https://www.hangge.com/blog/cache/detail_1192.html#) [![原文:Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016051816582034898.png)](https://www.hangge.com/blog/cache/detail_1192.html#) [![原文:Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016051816582835046.png)](https://www.hangge.com/blog/cache/detail_1192.html#)



可以看到选择后，视频会被复制到 **tmp** 目录下：

[![原文:Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/201605181659526528.png)](https://www.hangge.com/blog/cache/detail_1192.html#)



选择完毕后自动播放该视频：

[![原文:Swift - 从相册中选择视频（过滤掉照片，使用UIImagePickerController）](https://www.hangge.com/blog_uploads/201605/2016051817003066513.png)](https://www.hangge.com/blog/cache/detail_1192.html#)



**3，样例代码**

```swift
import UIKit
import MobileCoreServices
import AssetsLibrary
import AVKit
import AVFoundation
 
class ViewController: UIViewController,  UIImagePickerControllerDelegate,
UINavigationControllerDelegate{
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建一个ContactAdd类型的按钮
        let button:UIButton = UIButton(type:.system)
        button.frame = CGRect(x:10, y:150, width:100, height:30)
        button.setTitle("选择视频", for:.normal)
        button.addTarget(self, action:#selector(selectVideo), for:.touchUpInside)
        self.view.addSubview(button)
    }
     
    //选择视频
    @objc func selectVideo() {
        if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
            //初始化图片控制器
            let imagePicker = UIImagePickerController()
            //设置代理
            imagePicker.delegate = self
            //指定图片控制器类型
            imagePicker.sourceType = .photoLibrary
            //只显示视频类型的文件
            imagePicker.mediaTypes = [kUTTypeMovie as String]
            //不需要编辑
            imagePicker.allowsEditing = false
            //弹出控制器，显示界面
            self.present(imagePicker, animated: true, completion: nil)
        }
        else {
            print("读取相册错误")
        }
    }
     
    //选择视频成功后代理
    func imagePickerController(_ picker: UIImagePickerController,
                               didFinishPickingMediaWithInfo info: [String : Any]) {
        //获取视频路径（选择后视频会自动复制到app临时文件夹下）
        let videoURL = info[UIImagePickerControllerMediaURL] as! URL
        let pathString = videoURL.relativePath
        print("视频地址：\(pathString)")
         
        //图片控制器退出
        self.dismiss(animated: true, completion: {})
         
        //播放视频文件
        reviewVideo(videoURL)
    }
     
    //视频播放
    func reviewVideo(_ videoURL: URL) {
        //定义一个视频播放器，通过本地文件路径初始化
        let player = AVPlayer(url: videoURL)
        let playerViewController = AVPlayerViewController()
        playerViewController.player = player
        self.present(playerViewController, animated: true) {
            playerViewController.player!.play()
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1192.html