# Swift - 使用相机拍摄照片

2015-06-19发布：hangge阅读：5625

**1，打开相机拍照**

通过设置图片控制器UIImagePickerController的来源为UIImagePickerControllerSourceType.Camera，便可以打开相机

```swift
import UIKit
 
class ViewController: UIViewController, UIImagePickerControllerDelegate,
UINavigationControllerDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
    }   
    //拍照
    @IBAction func fromPhotograph(sender: AnyObject) {
        if UIImagePickerController.isSourceTypeAvailable(.Camera){
            //创建图片控制器
            let picker = UIImagePickerController()
            //设置代理
            picker.delegate = self
            //设置来源
            picker.sourceType = UIImagePickerControllerSourceType.Camera
            //允许编辑
            picker.allowsEditing = true
            //打开相机
            self.presentViewController(picker, animated: true, completion: { () -> Void in
                 
            })
        }else{
            println("找不到相机")
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，调用前置，后置摄像头**

相机默认使用后置摄像头，可以设置UIImagePickerControllerCameraDevice类型来使用前置摄像头或后置摄像头。

像iTouch设备不具备前置摄像头，我们可以事先判断下是否支持前置。

```swift
//如果有前置摄像头则调用前置摄像头
if UIImagePickerController.isCameraDeviceAvailable(UIImagePickerControllerCameraDevice.Front){
    picker.cameraDevice = UIImagePickerControllerCameraDevice.Front
}
```


**3，设置闪光灯**
通过cameraFlashMode属性可以设置闪光灯：开启/关闭/自动

```swift
//开启闪光灯
picker.cameraFlashMode = UIImagePickerControllerCameraFlashMode.On
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_770.html