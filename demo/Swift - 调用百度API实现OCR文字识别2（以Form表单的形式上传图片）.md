# Swift - 调用百度API实现OCR文字识别2（以Form表单的形式上传图片）

2017-06-19发布：hangge阅读：1458

在前文中，我介绍了如何将图片转成 **Base64** 字符串，并调用百度接口进行文字识别（[点击查看](https://www.hangge.com/blog/cache/detail_1712.html)）。如果觉得识别前要先将图片进行编码转换很麻烦，那么可以试试本文介绍的通过 **Form** 表单形式上传图片。

## 一、百度OCR文字识别的三个版本接口

百度OCR文字识别提供了三种版本的接口供我们使用，三个版本的接口地址都是一样，通过 **imagetype** 参数加以区分：

- 参数 **imagetype = 1** 时：调用 **v1** 接口，为原本使用的稳定版本。这个也是默认接口，像上面样例一样，如果不传 **imagetype** 参数就调用 **v1** 接口。
- 参数 **imagetype = 2** 时：调用 **v2** 接口，为升级版本。可以通过页面表单（或者模拟表单）提交的方式进行提交图片。
- 参数 **imagetype = 3** 时：调用 **v3** 接口，可以填写 **URL**。通常用于测试使用。

## 二、以Form表单的形式上传图片，并进行文字识别

使用表单提交的好处是，我们不需要再将图片 **Data** 转成 **Base64** 字符串，只需要附上文件路径即可自动提交。



### 1，效果图

（1）页面上方是我们要识别的一张手机截图，图片格式是 **jpg**。（不知为何，我测试 **png** 格式时一直不成功，报 **10002** 错误。）

（2）通过调用 **API** 接口，将返回的结果解析并显示在下方的文本框中。我们可以得到每个识别区域的位置尺寸，以及该区域里的文字内容。

[![原文:Swift - 调用百度API实现OCR文字识别2（以Form表单的形式上传图片）](https://www.hangge.com/blog_uploads/201706/2017061013395418662.png)](https://www.hangge.com/blog/cache/detail_1713.html#)



### 2，样例代码

（1）这里我使用第三方的网络库 **Alamofire** 来上传 **MultipartFormData** 类型的文件数据（类似于网页上 **Form** 表单里的文件提交） 。
（2）除了要将各种参数放在表单数据里以外 ，还要记得把 **apikey** 添加到 **header** 中。

（3）由于接口返回的是 **JSON** 的数据，为方便解析，我这里仍然使用了 **SwiftyJSON** 这个第三方 **JSON** 库。它与 **Alamofire** 能完美地配合使用，具体可以参考我之前写的这篇文章：[Swift - SwiftyJSON的使用详解（附样例，用于JSON数据处理）](https://www.hangge.com/blog/cache/detail_968.html)

```swift
import UIKit
import Alamofire
 
class ViewController: UIViewController {
     
    //显示识别结果
    @IBOutlet weak var textView: UITextView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
        //获取图片
        let file = Bundle.main.path(forResource: "g1", ofType: "jpg")!
        let fileUrl = URL(fileURLWithPath: file)
        request(fileUrl: fileUrl)
    }
     
    //请求API接口
    func request(fileUrl: URL) {
        //接口地址（使用http：//也可以，记得再info.plist里做相关的配置）
        let httpUrl = "https://apis.baidu.com/idl_baidu/baiduocrpay/idlocrpaid"
        //使用Alamofire发送form提交请求
        Alamofire.upload(multipartFormData:{
                multipartFormData in
                multipartFormData.append("pc".data(using: .utf8)!,
                                         withName: "fromdevice")
                multipartFormData.append("10.10.10.0".data(using: .utf8)!,
                                         withName: "clientip")
                multipartFormData.append("LocateRecognize".data(using: .utf8)!,
                                         withName: "detecttype")
                multipartFormData.append("2".data(using: .utf8)!,
                                         withName: "imagetype")
                multipartFormData.append(fileUrl, withName: "image")
            },
            usingThreshold: UInt64.init(),
            to: httpUrl,
            method:.post,
            headers: ["apikey": "你的apikey..."],
            encodingCompletion: {
                encodingResult in
                switch encodingResult {
                case .success(let upload, _, _):
                    upload.responseJSON { response in
                        print("----- 原始数据 -----\n")
                        print(response.result.value ?? "")
                        //解析数据并显示结果
                        self.showResult(data: response.result.value!)
                    }
                case .failure(let encodingError):
                    print(encodingError)
                }
            })
    }
     
    //解析数据并显示结果
    func showResult(data:Any) {
        var result = ""
        let json = JSON(data)
        //循环每个区域识别出来的内容
        for (_,subJson):(String, JSON) in json["retData"]{
            result.append("-- 区域（")
            result.append("左|\(subJson["rect"]["left"].intValue) ")
            result.append("上|\(subJson["rect"]["top"].intValue) ")
            result.append("宽度|\(subJson["rect"]["width"].intValue) ")
            result.append("高度|\(subJson["rect"]["height"].intValue)）--\n")
            result.append("\(subJson["word"].stringValue)\n\n") //识别的内容
        }
        //在主线程中显示解析的结果
        DispatchQueue.main.async{
            self.textView.text = result
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1711_2.zip](https://www.hangge.com/blog_uploads/201706/2017061119411746618.zip)

## 三、上传大图

（1）前面的样例中，我们上传的图片 **base64_encode** 后不可以超过 **300\*1024**，如果超过 **300k**，则会报 **-10004** 错误。

[![原文:Swift - 调用百度API实现OCR文字识别2（以Form表单的形式上传图片）](https://www.hangge.com/blog_uploads/201706/2017061013524457582.png)](https://www.hangge.com/blog/cache/detail_1713.html#)



（2）如果需要识别更大的图片，则将 **sizetype** 参数为 **big**，此时支持 **1M** 以内的图片识别。需注意：大图(大于 **300K** 小于 **1M**)的识别计费为 **2** 次。同时不论那种状态图片识别不支持 **1M** 以上的。

```swift
multipartFormData.append("big".data(using: .utf8)!, withName: "sizetype")
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1713.html