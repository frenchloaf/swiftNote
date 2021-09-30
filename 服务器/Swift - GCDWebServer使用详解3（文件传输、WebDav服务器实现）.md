# Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）

2017-03-20发布：hangge阅读：3605

## 一、实现文件的上传下载服务

**GCDWebUploader** 是 **GCDWebServer** 的子类，它提供了一个现成的 **HTML5** 形式的文件上传下载器。**GCDWebUploader** 自带 **UI** 界面，让用户可以在浏览器里上传，下载，删除文件，以及在 **iOS** 应用的沙盒中创建目录文件夹。



### 1，安装配置

（1）除了之前提到的 **GCDWebServer**，我们还需要把 **GCDWebUploader** 这个子文件夹添加到我们项目中来。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021816573267899.png)](https://www.hangge.com/blog/cache/detail_1562.html#)



（2）同时将 **GCDWebUploader.h** 添加到桥接头文件中来。

```
#``import` `"GCDWebServer.h"``#``import` `"GCDWebServerDataResponse.h"``#``import` `"GCDWebUploader.h"
```

### 2，样例代码

在我们程序启动的时候会自动运行个 **HTTP** 服务，端口 **8080**。用户访问这个地址时，会显示一个进行文件管理的页面。

```
import` `UIKit` `class` `ViewController``: ``UIViewController` `{` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()` `    ``//默认上传目录是App的用户文档目录``    ``let` `documentsPath = ``NSHomeDirectory``() + ``"/Documents"``    ``let` `webUploader = ``GCDWebUploader``(uploadDirectory: documentsPath)``    ``webUploader.start(withPort: 8080, bonjourName: ``"Web Based Uploads"``)``    ``print``(``"服务启动成功，使用你的浏览器访问：\(webUploader.serverURL)"``)``  ``}` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```



### 3，效果图

我这里使用模拟器运行程序，然后使用电脑浏览器访问 **http://localhost:8080** 就可以看到效果了。在这里我们可以进行文件的上传、下载、重命名、删除，以及文件夹的创建、删除、重命名等操作。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021817104664797.png)](https://www.hangge.com/blog/cache/detail_1562.html#)



## 二、实现一个WebDAV服务器

前面我们通过 **GCDWebUploader** 来实现一个基于 **HTTP** 的文件服务器，用户通过浏览器可以进行文件的上传下载等操作。
这次我们通过 **GCDWebServer** 来实现一个 **WebDAV** 服务器，让用户可以使用任意的 **WebDAV** 客户端，比如： **Transmit** (**Mac**)，**ForkLift** (**Mac**) 或者 **CyberDuck** (**Mac** / **Windows**)，来访问我们 **App** 的沙盒目录文件。

### 1，安装配置

（1）首先我们需要在：**Target** > **Build Phases** > **Link Binary With Libraries** 中添加动态库 **libxml2**

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021911480090006.png)](https://www.hangge.com/blog/cache/detail_1562.html#)



（2）在 **Target** > **Build Settings** > **HEADER_SEARCH_PATHS** 中把 **$(SDKROOT)/usr/include/libxml2** 添加到 **header search paths**

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021911514012275.png)](https://www.hangge.com/blog/cache/detail_1562.html#)



（3）除了 **GCDWebServer**，我们还需要把 **GCDWebDAVServer** 这个子文件夹添加到我们项目中来。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021911553267839.png)](https://www.hangge.com/blog/cache/detail_1562.html#)



（4）同时将 **GCDWebDAVServer.h** 添加到桥接头文件中来。

```
#``import` `"GCDWebServer.h"``#``import` `"GCDWebServerDataResponse.h"``#``import` `"GCDWebUploader.h"``#``import` `"GCDWebDAVServer.h"
```



### 2，样例代码

当我们程序启动的时候会自动运行个 **WebDAV** 服务，端口 **8080**。

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //默认上传目录是App的用户文档目录
        let documentsPath = NSHomeDirectory() + "/Documents"
        let webUploader = GCDWebUploader(uploadDirectory: documentsPath)
        webUploader.start(withPort: 8080, bonjourName: "Web Based Uploads")
        print("服务启动成功，使用你的浏览器访问：\(webUploader.serverURL)")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**： ![img](https://www.hangge.com/blog/admin0822/include/edit/sysimage/icon16/zip.gif)[hangge_1562.zip](https://www.hangge.com/blog_uploads/201809/2018092809544662794.zip)



### 3，效果图

我这里使用模拟器运行程序，然后使用 **WebDAV** 客户端 **Transmit** 进行连接。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/2017021912052746071.png)](https://www.hangge.com/blog/cache/detail_1562.html#)


连接成功后，可以看到这个 **App** 的 **Documents** 目录下的所有文件，并可以进行相关操作。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201702/201702191205356480.png)](https://www.hangge.com/blog/cache/detail_1562.html#)

如果是 **Windows** 系统，可以使用 **Cyberduck** 这个客户端工具进行连接操作。

[![原文:Swift - GCDWebServer使用详解3（文件传输、WebDav服务器实现）](https://www.hangge.com/blog_uploads/201711/2017110714563647521.png)](https://www.hangge.com/blog/cache/detail_1562.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1562.html