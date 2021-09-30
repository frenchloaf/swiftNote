# Swift - 调用百度API实现OCR文字识别1（以Base64字符串形式上传图片）

2017-06-16发布：hangge阅读：3286

如果我们应用中要用到图片的文字识别，自己去实现底层算法显然不太现实。好在百度 **APIStore** 里提供了文字识别的 **API**，我们只需将要识别的图片通过接口发送过去，即可返回识别后的文本。下面通过样例演示如何实现 **OCR** 文字识别。

## 一、百度OCR文字识别企业版

### 1，基本介绍

百度 **OCR** 文字识别企业版依托百度业界领先的 **OCR** 算法，提供了整图文字检测、识别、整图文字识别、整图文字行定位和单字图像识别等功能。

接口文档地址：http://apistore.baidu.com/apiworks/servicedetail/969.html

### 2，收费标准

百度 **OCR** 文字识别企业版不是免费的服务，大概 **50** 元 **1000** 次。但我们一开始可以先试用，可以免费使用 **200** 次，这个足够我们开发测试了。

### 3，支持的图片

（1）图片格式：目前仅支持 **jpg**，**png** 格式

（2）图片尺寸：目前最大尺寸 **1M** 以内的图片识别。

- 小于 **300k** 图片：保持原有收费方案，即一张图片计费 **1** 次。
- 大于 **300k**、小于 **1M** 的图片：一张图片计费 **2** 次（同时要将 **sizetype** 参数设定为 **big**）

## 二、以Base64编码形式上传图片，并进行文字识别

### 1，效果图

（1）页面上方是我们要识别的一张手机截图，图片格式是 **jpg**。（不知为何，我测试 **png** 格式时一直不成功，报 **10002** 错误。）

（2）通过调用 **API** 接口，将返回的结果解析并显示在下方的文本框中。我们可以得到每个识别区域的位置尺寸，以及该区域里的文字内容。

[![原文:Swift - 调用百度API实现OCR文字识别1（以Base64字符串形式上传图片）](https://www.hangge.com/blog_uploads/201706/2017061013395418662.png)](https://www.hangge.com/blog/cache/detail_1712.html#)



### 2，样例代码

（1）调用接口需要用到百度的 **apikey**，没有的话去注册一个：http://apistore.baidu.com/

（2）由于图片数据是和其它参数拼接在一起，这里对图片要进行两次转换处理：

- 先将图片的 **Data** 数据转成 **Base64** 字符串。
- 再将 **Base64** 字符串进行 **URLencode** 编码，确保字符串中只有数字和字母。

（3）由于接口返回的是 **JSON** 的数据，为方便解析，我这里使用了 **SwiftyJSON** 这个第三方 **JSON** 库。其详细用法可以参考我之前写的这篇文章：[Swift - SwiftyJSON的使用详解（附样例，用于JSON数据处理）](https://www.hangge.com/blog/cache/detail_968.html)



```
import UIKit
 
class ViewController: UIViewController {
     
    //显示识别结果
    @IBOutlet weak var textView: UITextView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
        //获取图片
        let file = Bundle.main.path(forResource: "g1", ofType: "jpg")!
        let fileUrl = URL(fileURLWithPath: file)
        let fileData = try! Data(contentsOf: fileUrl)
        //将图片转为base64编码
        let base64 = fileData.base64EncodedString(options: .endLineWithLineFeed)
        //继续将base64字符串urlencode一下，确保只有数字和字母
        let imageString = base64.addingPercentEncoding(withAllowedCharacters:
            .alphanumerics)
       
        //请求接口
        request(imageString: imageString!)
    }
     
    //请求API接口
    func  request(imageString: String) {
        //接口地址（使用http：//也可以，记得再info.plist里做相关的配置）
        let httpUrl = "https://apis.baidu.com/idl_baidu/baiduocrpay/idlocrpaid"
        //参数拼接
        let httpArg = "fromdevice=iPhone&clientip=10.10.10.0&detecttype=LocateRecognize" +
            "&languagetype=CHN_ENG&imagetype=1&image=" + imageString
         
        //创建请求对象
        var request = URLRequest(url: URL(string: httpUrl)!)
        request.timeoutInterval = 6
        request.httpMethod = "POST"
        request.addValue("你的apikey...", forHTTPHeaderField: "apikey")
        request.addValue("application/x-www-form-urlencoded",
                         forHTTPHeaderField: "Content-Type")
        //设置请求内容
        request.httpBody = httpArg.data(using: .utf8)
         
        //使用URLSession发起请求
        let session = URLSession.shared
        let dataTask = session.dataTask(with: request,
                            completionHandler: {(data, response, error) -> Void in
                                if error != nil{
                                    print(error.debugDescription)
                                }else if let d = data{
                                    let str = String(data: d, encoding: .utf8)!
                                    print("----- 原始数据 -----\n\(str)")
                                    //解析数据并显示结果
                                    self.showResult(data: d)
                                }
        }) as URLSessionTask
         
        //使用resume方法启动任务
        dataTask.resume()
    }
     
    //解析数据并显示结果
    func showResult(data:Data) {
        var result = ""
        let json = try! JSON(data: data)
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

**源码下载**： ![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1711_2.zip](https://www.hangge.com/blog_uploads/201706/201706101346482181.zip)

## 三、上传大图

（1）前面的样例中，我们上传的图片 **base64_encode** 后不可以超过 **300\*1024**，如果超过 **300k**，则会报 **-10004** 错误。

[![原文:Swift - 调用百度API实现OCR文字识别1（以Base64字符串形式上传图片）](https://www.hangge.com/blog_uploads/201706/2017061013524457582.png)](https://www.hangge.com/blog/cache/detail_1712.html#)



（2）如果需要识别更大的图片，则将 **sizetype** 参数为 **big**，此时支持 **1M** 以内的图片识别。需注意：大图(大于 **300K** 小于 **1M**)的识别计费为 **2** 次。同时不论那种状态图片识别不支持 **1M** 以上的。

```
//参数拼接
let httpArg = "fromdevice=iPhone&clientip=10.10.10.0&detecttype=LocateRecognize" +
    "&languagetype=CHN_ENG&imagetype=1&image=" + imageString +
    "&sizetype=big"
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1712.html