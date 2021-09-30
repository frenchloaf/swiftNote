# Swift - 使用UIImagePickerController从相册选择照片并展示

2015-06-19发布：hangge阅读：16111

（本文代码已升级至Swift4）

**1，UIImagePickerController介绍**

（1）选择相册中的图片或者拍照，都是通过UIImagePickerController控制器实例化一个对象，然后通过self.present方法推送出界面显示。

（2）使用present的类需要实现UIImagePickerControllerDelegate,UINavigationControllerDelegate两个代理。

（3）UIImagePickerController可以通过isSourceTypeAvailable方法来判断设备是否支持照相机/图片库/相册功能。如果支持，可以通过sourceType属性来设置图片控制器的显示类型。

**2，下面通过一个样例，演示如何使用UIImagePickerController**

（1）点击“选择照片”，自动打开相册选择照片

（2）照片选中后，返回原界面并加载照片原图，同时控制台会打印照片的info信息

（3）如果选择照片前打开“编辑”开关，选中照片后会先进入照片编辑页面

**3，效果图如下：**

[![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201506/2015061910560482241.png)](https://www.hangge.com/blog/cache/detail_769.html#) [![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201506/2015061910561363775.png)](https://www.hangge.com/blog/cache/detail_769.html#) [![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201506/2015061910562330693.png)](https://www.hangge.com/blog/cache/detail_769.html#)

[![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201506/2015061910563832696.png)](https://www.hangge.com/blog/cache/detail_769.html#) [![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201506/2015061910565554781.png)](https://www.hangge.com/blog/cache/detail_769.html#)

**4，样例实现**

（1）Info.plist配置
由于苹果安全策略更新，在使用Xcode8开发时，需要在Info.plist配置请求照片相的关描述字段（Privacy - Photo Library Usage Description）

[![原文:Swift - 使用UIImagePickerController从相册选择照片并展示](https://www.hangge.com/blog_uploads/201610/2016101717222541521.png)](https://www.hangge.com/blog/cache/detail_769.html#)

（2）样例代码如下：

```swift
import UIKit
 
class ViewController: UIViewController, UIImagePickerControllerDelegate,
UINavigationControllerDelegate {
     
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var editSwitch: UISwitch!
     
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
            picker.sourceType = UIImagePickerController.SourceType.photoLibrary
            //设置是否允许编辑
            picker.allowsEditing = editSwitch.isOn
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
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        //查看info对象
        print(info)
         
        //显示的图片
        let image:UIImage!
        if editSwitch.isOn {
            //获取编辑后的图片
            image = info[.editedImage] as? UIImage
        }else{
            //获取选择的原图
            image = info[.originalImage] as? UIImage
        }
         
        imageView.image = image
        //图片控制器退出
        picker.dismiss(animated: true, completion: {
            () -> Void in
        })
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin0822/include/edit/sysimage/icon16/zip.gif)[hangge_769.zip](https://www.hangge.com/blog_uploads/201812/2018121810121258981.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_769.html