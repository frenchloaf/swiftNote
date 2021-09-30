# Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）

2016-06-12发布：hangge阅读：8645

（本文代码已升级至Swift3）

我之前写过一篇文章：[Swift - URL schemes的使用样例（如：在Safari中打开App）](https://www.hangge.com/blog/cache/detail_1042.html)。介绍通过自定义的 **URL Scheme**，实现从外部浏览器或外部应用打开我们的应用。

同样的，如果想从本地应用中跳转到其他的第三方应用并传值。同样是通过 **URL Scheme** 实现。

**一，使用样例**

常用的第三方应用都定义了不同的 **URL Scheme**，我们通过 **UIApplication.shared.open()** 方法打开相应的链接，即可跳转到对应的 **App** 中。（**iOS10** 以下的系统则是使用 **UIApplication.shared.openURL()** 方法）
**
**

**1，打开淘宝**

下面样例点击按钮后，会自动跳转到淘宝App中。由于我们还传递了一个商品链接参数，那么跳转过来后就会自动打开该商品页面。

[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201606/2016061211071770335.png)](https://www.hangge.com/blog/cache/detail_1141.html#)[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201606/2016061211072272833.png)](https://www.hangge.com/blog/cache/detail_1141.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func click(_ sender: AnyObject) {
        let urlString = "taobao://item.taobao.com/item.htm?id=22671596473"
        if let url = URL(string: urlString) {
            //根据iOS系统版本，分别处理
            if #available(iOS 10, *) {
                UIApplication.shared.open(url, options: [:],
                                          completionHandler: {
                                            (success) in
                })
            } else {
                UIApplication.shared.openURL(url)
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**2，打开百度地图**



下面样例点击按钮后，会自动跳转到百度地图App中。由于我们还传递了地址作为参数，那么跳转过来后就会自动定位到该位置。

（注意：由于参数中带有中文，我们这里使用 **addingPercentEncoding** 方法对其转义一下。）

[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201606/2016061211072957552.png)](https://www.hangge.com/blog/cache/detail_1141.html#)[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201606/2016061211073594272.png)](https://www.hangge.com/blog/cache/detail_1141.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func click(_ sender: AnyObject) {
        let urlStr = "baidumap://map/geocoder?address=北京市海淀区上地信息路9号奎科科技大厦"
        let encodeUrlString = urlStr.addingPercentEncoding(withAllowedCharacters:
            .urlQueryAllowed)
        if let url = URL(string: encodeUrlString!) {
            //根据iOS系统版本，分别处理
            if #available(iOS 10, *) {
                UIApplication.shared.open(url, options: [:],
                                          completionHandler: {
                                            (success) in
                })
            } else {
                UIApplication.shared.openURL(url)
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**3，打开微信扫一扫** 

下面样例点击按钮后，会自动跳转到微信App中。同时自动打开扫一扫界面。 

[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201610/2016101716143534886.png)](https://www.hangge.com/blog/cache/detail_1141.html#)[![原文:Swift - 打开第三方应用，并传递参数（附常用App的URL Scheme）](https://www.hangge.com/blog_uploads/201610/2016101716144415547.png)](https://www.hangge.com/blog/cache/detail_1141.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func click(_ sender: AnyObject) {
        let urlString = "weixin://scanqrcode"
        if let url = URL(string: urlString) {
            //根据iOS系统版本，分别处理
            if #available(iOS 10, *) {
                UIApplication.shared.open(url, options: [:],
                                          completionHandler: {
                                            (success) in
                })
            } else {
                UIApplication.shared.openURL(url)
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**二，常见应用的URL Scheme**

**1，系统默认应用**

| **名称**      | **URL Scheme**               | **Bundle identifier** |
| ------------- | ---------------------------- | --------------------- |
| **Safari**    | http://                      |                       |
| **maps**      | http://maps.google.com       |                       |
| **Phone**     | tel://                       |                       |
| **SMS**       | sms://                       |                       |
| **Mail**      | mailto://                    |                       |
| **iBooks**    | ibooks://                    |                       |
| **App Store** | itms-apps://itunes.apple.com |                       |
| **Music**     | music://                     |                       |
| **Videos**    | videos://                    |                       |

**2，常用第三方软件**

| **名称**             | **URL Scheme**     | **Bundle identifier**                           |
| -------------------- | ------------------ | ----------------------------------------------- |
| **QQ**               | mqq://             |                                                 |
| **微信**             | weixin://          |                                                 |
| **腾讯微博**         | TencentWeibo://    |                                                 |
| **淘宝**             | taobao://          |                                                 |
| **支付宝**           | alipay://          |                                                 |
| **微博**             | sinaweibo://       |                                                 |
| **weico微博**        | weico://           |                                                 |
| **QQ浏览器**         | mqqbrowser://      | com.tencent.mttlite                             |
| **uc浏览器**         | ucbrowser://       | com.ucweb.iphone.uc com.ucweb.iphone.lowversion |
| **海豚浏览器**       | dolphin://         | com.dolphin.browser.iphone.chinese              |
| **欧朋浏览器**       | ohttp://           | com.oupeng.mini                                 |
| **搜狗浏览器**       | SogouMSE://        | com.sogou.SogouExplorerMobile                   |
| **百度地图**         | baidumap://        | com.baidu.map                                   |
| **Chrome**           | googlechrome://    |                                                 |
| **优酷**             | youku://           |                                                 |
| **京东**             | openapp.jdmoble:// |                                                 |
| **人人**             | renren://          |                                                 |
| **美团**             | imeituan://        |                                                 |
| **1号店**            | wccbyihaodian://   |                                                 |
| **我查查**           | wcc://             |                                                 |
| **有道词典**         | yddictproapp://    |                                                 |
| **知乎**             | zhihu://           |                                                 |
| **点评**             | dianping://        |                                                 |
| **微盘**             | sinavdisk://       |                                                 |
| **豆瓣fm**           | doubanradio://     |                                                 |
| **网易公开课**       | ntesopen://        |                                                 |
| **名片全能王**       | camcard://         |                                                 |
| **QQ音乐**           | qqmusic://         |                                                 |
| **腾讯视频**         | tenvideo://        |                                                 |
| **豆瓣电影**         | doubanmovie://     |                                                 |
| **网易云音乐**       | orpheus://         |                                                 |
| **网易新闻**         | newsapp://         |                                                 |
| **网易应用**         | apper://           |                                                 |
| **网易彩票**         | ntescaipiao://     |                                                 |
| **有道云笔记**       | youdaonote://      |                                                 |
| **多看**             | duokan-reader://   |                                                 |
| **全国空气质量指数** | dirtybeijing://    |                                                 |
| **百度音乐**         | baidumusic://      |                                                 |
| **下厨房**           | xcfapp://          |                                                 |


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1141.html