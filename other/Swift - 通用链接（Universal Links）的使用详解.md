# Swift - 通用链接（Universal Links）的使用详解

2017-03-06发布：hangge阅读：6186

## 一、通用链接介绍

通用链接（**Universal Links**）是 **iOS9** 推出的一项功能。如果我们的应用支持通用链接，那么就能够方便的通过传统的 **HTTP** 链接来启动 **APP**（只要设备上已经安装了这个 **App**，不需要额外做任何判断)，或者打开网页（如果 **iOS** 设备上没有安装该 **App**）

###  1，通用链接与URL Scheme的区别

（1）**URL Scheme** 使用介绍

- 在 **iOS9** 之前，对于从各种从浏览器（**Safari**、**UIWebView**、**WKWebView**）中唤醒（启动）**App** 的需求，我们通常只能使用 **URL Scheme**。我之前也写过不少相关文章（[点击查看](https://www.hangge.com/blog/cache/detail_1042.html)）。
- 使用时我们首先要在 **App** 中注册 **Scheme**，然后网页中使用特定格式的链接，比如：**<a href="hangge://www.hangge.com/app">打开APP</a>** 
- 而且每当我们要唤醒某个 **App** 时，还需要提前判断系统中是否安装了能够响应此 **Scheme** 的 **App**。否则处理不好常常会造成空白页或者跳转不了问题。
- 如果设备中有这个 **App**，那么打开前还会弹出个提示框询问是否要使用该 **App** 打开。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201602/2016020115314724523.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（2）通用链接（**Universal Links**） 使用介绍

- 通用链其实就是一条普通的 **http** 链接。比如：**https://www.hangge.com/app**
- 如果我们的 **App** 支持通用链接，且设备中安装了该 **App**。那么当用户点击链接，就会直接进入到我们的 **App** 中了。
- 如果设备中没有支持这个链接的 **App**，那么点击链接后还是继续进到链接对应的 **html** 页面中。完全不需要我们人为判断是去打开 **App**，还是打开 **Web** 页面。
- 同时通过通用链接进入到 **App** 这个过程是不会弹出提示框的，整个流程十分顺畅。（如果手机安装了“**知乎**”，我们随便在浏览器中搜索一个知乎页面点击就可以看到效果。）



### 2，通用链接的优点

- **唯一性**：不像自定义的 **scheme**，因为它使用标准的 **http**/**https** 链接到你的 **web** 站点，所以它不会被其它的 **app** 所声明。另外，**Custom URL scheme** 因为是自定义的协议，所以在没有安装 **app** 的情况下是无法直接打开的，而 **universal links** 本身是一个 **HTTP**/**HTTPS** 链接，所以有更好的兼容性。
- **安全**：当用户的手机上安装了你的 **app**，那么 **iOS** 将去你的网站上去下载你上传上去的说明文件(这个说明文件声明了你的 **app** 可以打开哪些类型的 **http** 链接)。因为只有你自己才能上传文件到你网站的根目录，所以你的网站和你的 **app** 之间的关联是安全的。
- **可变**：当用户手机上没有安装你的 **app** 的时候，**Universal Links** 也能够工作。如果你愿意，在没有安装你的 **app** 的时候，用户点击链接，会在 **safari** 中展示你网站的内容。
- **简单**：一个 **URL** 链接，可以同时作用于网站和 **app**。
- **私有**：其它 **app** 可以在不需要知道你的 **app** 是否安装了的情况下和你的 **app** 相互通信。

###  3，使用条件

- 有一个注册的域名
- 通过 **SSL** 访问域名（即使用 **HTTPS** 请求）
- 支持上传一个 **JSON** 文件到你的域名
- 至少 **iOS 9 beta 2** 版本。
- 至少 **Xcode 7 beta 2**。



## 二、相关配置

### 1，创建apple-app-site-association文件

首先创建一个名为 **apple-app-site-association** 的文件（注意没有后缀），其内容是 **json** 格式的数据。假设内容如下：

```json
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "MNARL96.com.hangge.demo",
                "paths": [ "NOT /blog/support/*", "/blog/*", "/app/"]
            },
            {
                "appID": "ABCDEFG12.com.hangge.demo",
                "paths": [ "*" ]
            }
        ]
    }
}
```



**相关参数说明**：

- **appID**：由 **TeamID.BundleId** 组成。**TeamID** 可以从苹果开发账号页面中查看，**BundleId** 就是 **Bundle Identifier**，可以直接在工程里查看。
- **paths**：其设定一个我们的 **App** 支持的路径列表，只有这些指定的路径的链接，才能被 **App** 所处理。
  （注意：**path** 是大小写敏感。***** 通配符表示任意路径。**NOT** 关键字表示禁止的部分。数组内优先级从左到右逐渐降低。）

###  2，上传apple-app-site-association文件

（1）首先在网站的根目录下新建一个 **.well-known** 文件夹，并将 **apple-app-site-association** 文件上传到该文件夹中。

如果服务器是 **windows** 系统，可以在命令提示符（**cmd**）中使用命令来创建文件夹：

```
md .well-known
```

**apple-app-site-association 文件保存的位置**：

- 在 **iOS9.3** 以前，该文件需要上传到网站的根目录下。
- 从 **iOS9.3** 起，苹果更改了通用链接的请求文件的路径，变成了在 **.well-known** 目录下。


（2）如果使用的是 **IIS** 服务器，为了让 **apple-app-site-association** 文件能被访问到，需要 **MIME** 类型配置。否则会报 **404** 错误。

- 文件扩展名：**.**
- MIME类型：**application/json**

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021414480531242.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（3）我们使用浏览器测试下是否能访问到。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021519563127405.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（4）当然苹果为了方便开发者，也提供了一个网页来验证我们编写的这个 **apple-app-site-association** 是否合法有效。

验证地址：https://search.developer.apple.com/appsearch-validation-tool/

### 3，Xcode相关配置

（1）首先就是将工程配置中的 **Associated Domains** 打开，并填写上我们想支持的域名（前缀是 **applinks**）。**App** 会在第一次启动的时候从这里填写的域名中请求 **apple-app-site-association** 文件。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021415033492786.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（2）配置后会发现我们工程中会自动添加了一个 **.entitlements** 文件。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021415052455120.png)](https://www.hangge.com/blog/cache/detail_1554.html#)



同时在开发者中心里可以看到，**Associated Domains** 是启用状态。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021415151898994.png)](https://www.hangge.com/blog/cache/detail_1554.html#)



## 三、测试使用

编译部署 **App** 后，我们就可以测试下通用链接是否生效了。这里提供三种方法。

### 1，使用备忘录测试

（1）我们在 **iOS** 设备自带的备忘录中添加两条链接，其中一条不符合我们在 **apple-app-site-association** 文件配置的规则，另一条符合规则。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520484821073.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（2）如果点击不符合规则的链接，则像原来一样，会使用 **safari** 浏览器打开这个链接。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520491471903.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（3）如果点击是符合通用规则的链接，则会自动跳转到我们的 **App** 中。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520494165912.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

如果长按这个链接，在出现的弹出菜单中还有“**在XXX中打开**”这个选项。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520510347237.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

### 2，使用邮件测试

给自己的邮箱发封邮件，正文带上要测试的链接。使用手机“**邮件**”App接收并点击测试。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520514967612.png)](https://www.hangge.com/blog/cache/detail_1554.html#)



### 3，使用浏览器测试

（1）将需要测试的地址放在一个网页中，并使用 **safari** 浏览器访问这个页面（注意：直接在浏览器地址栏中打开测试地址是没有用的）。这里我访问 [hangge.com](http://hangge.com/)

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520524179924.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（2）随便点击一个链接（都是符合我前面配置的规则的）打开新页面。这时在出现的网页上用手指下滑，会发现页面上方出现 在"**XXX**"应用中打开　这个提示信息。

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520530995867.png)](https://www.hangge.com/blog/cache/detail_1554.html#)

（3）点击“**打开**”即可跳转到我们的 **App** 中。

**为什么在Safari中点击链接不能直接进入App?**

上面样例可以看到我们点击一个链接后还是使用 **Safari** 打开，只不过新页面下拉多了个使用App打开的选项。

这是由于从 **iOS9.3** 起，通用链接不支持域内跳转了，跳转前后的两个 **domain** 必须是不同的，否则只会 **Safari** 打开。
前面的样例中，我跳转前跳转后都是在 **hangge.com** 这个域名下，自然不会进入 **App**。如果我是使用百度搜索到 **hangge.com** 的页面，然后点击链接，由于前后域不一样则会自动进入到 **App** 中。

## 四、在App中添加相关的响应处理

虽然用户点击某个链接，已经可以直接可以进入到我们的 **App** 中。但我们还是需要获取到用户进来的链接，然后根据链接来处理、展示给用户相应的信息。

###  在AppDelegate中实现相关的响应方法

下面代码实现，当用户通过链接进入到我们的 **App** 中时，将这个链接打印出来。

```swift
import UIKit
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
     
    var window: UIWindow?
     
    func application(_ application: UIApplication, continue userActivity: NSUserActivity,
                     restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
        //判断是从Universal Links进来的链接
        if userActivity.activityType == NSUserActivityTypeBrowsingWeb {
            let webpageURL = userActivity.webpageURL
            print("点击的链接是：\(webpageURL)")
            //进行后续的处理
            //......
        }
        return true
    }
     
    func application(_ application: UIApplication, didFinishLaunchingWithOptions
        launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        return true
    }
     
    func applicationWillResignActive(_ application: UIApplication) {
    }
     
    func applicationDidEnterBackground(_ application: UIApplication) {
    }
     
    func applicationWillEnterForeground(_ application: UIApplication) {
    }
     
    func applicationDidBecomeActive(_ application: UIApplication) {
    }
     
    func applicationWillTerminate(_ application: UIApplication) {
    }
}
```

**运行结果如下**：

[![原文:Swift - 通用链接（Universal Links）的使用详解](https://www.hangge.com/blog_uploads/201702/2017021520423679785.png)](https://www.hangge.com/blog/cache/detail_1554.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1554.html