# Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）

2017-03-15发布：hangge阅读：4976

## 一、GCDWebServer介绍

**GCDWebServer** 是一个基于 **GCD** 的轻量级服务器框架，用于内嵌到 **MacOS** 或者 **iOS** 系统的应用中提供 **HTTP1.1** 的服务。使用 **GCDWebServer** 我们可以很轻松的在我们的应用中搭建一个 **HTTP** 服务器，让用户可以通过浏览器访问我们应用中的数据。或者使用 **GCDWebServer** 来实现一个无线U盘 **App**。

**Github主页**：https://github.com/swisspol/GCDWebServer

### 1，特点

- 设计优雅，易于使用。仅仅包含**4**个核心类：**server**, **connection**, **request** 和 **response**
- 设计良好的 **API**。头文件注释齐全，非常易于继承和定制个性化需求。
- 事件驱动模型。基于 **GCD** 框架，实现最佳性能和并发。
- 不依赖任何第三方源码。
- 符合新的 **BSD** 许可协议。



### 2，支持的功能

- 针对 **http** 请求，支持完全异步处理
- 针对较大 **HTTP** 请求和响应流，采用内存最优化策略
- 支持解析使用"**application/x-www-form-urlencoded**" 或者 "**multipart/form-data**"编码格式提交的 **html** 表单
- 支持对 **json** 格式的请求或响应进行解析和序列化
- **HTTP** 请求或响应采用分块传输编码
- **HTTP** 请求和响应采用 **gzip** 方式压缩
- 对本地文件的请求支持多种 **HTTP** 类型
- 采用通用、简单的密码保护访问认证机制
- 支持在 **app** 前台、后台或挂起时自动处理事务
- 完全支持 **ipv4** 和 **ipv6**
- 支持文件上传功能。提供通过浏览器实现文件上传和下载的接口。（**GCDWebUploader** -> **GCDWebServer**）
- 支持文件系统服务。**DAV** 不仅被看作 **HTTP** 的扩展，甚至被看作一种网络文件系统。（**GCDWebDAVServer** -> **GCDWebServer**）



### 3，不支持的功能

- 长连接
- **https** 请求



## 二、安装配置

（1）首先我们需要在：**Target** > **Build Phases** > **Link Binary With Libraries** 中添加动态库 **libz**

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017021911372487384.png)](https://www.hangge.com/blog/cache/detail_1555.html#)


（2）将 **GCDWebServer** 整个源码包下载下来，然后将其中的 **GCDWebServer** 子文件夹添加到我们项目中来。

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017021809090557446.png)](https://www.hangge.com/blog/cache/detail_1555.html#)



（3）创建并配置个桥接头文件，并将相关的库引入进来。

```swift
#import "GCDWebServer.h"
#import "GCDWebServerDataResponse.h"
```



## 三、实现一个简单的HTTP服务器

下面程序启动的时候会自动运行个 **HTTP** 服务，端口 **8080**。当用户访问这个服务时，会返回一个显示欢迎信息的 **html** 页面。

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let webServer = GCDWebServer()
         
        webServer?.addDefaultHandler(forMethod: "GET", request: GCDWebServerRequest.self,
                                     processBlock: {request in
            let html = "<html><body>欢迎访问 <b>hangge.com</b></body></html>"
            return GCDWebServerDataResponse(html: html)
             
        })
         
        webServer?.start(withPort: 8080, bonjourName: "GCD Web Server")
        print("服务启动成功，使用你的浏览器访问：\(webServer?.serverURL)")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



我这里使用模拟器运行程序，让后使用电脑浏览器访问 **http://localhost:8080** 就可以看到效果了。

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017021809213556763.png)](https://www.hangge.com/blog/cache/detail_1555.html#)


由于我程序中没有对不同的请求分别做处理，所以无论任何请求服务器都会返回同样的 **HTML** 页面。

## 四、异步响应HTTP请求

上面的样例中我们是同步处理响应 **HTTP** 请求， **GCDWebServer 3.0** 起新增了异步地处理 **HTTP** 请求的特性，方便我们在响应中进行比较耗时的操作（如网路请求、I/O读写操作）。

### 1，样例代码

使用时只需将 **processBlock** 方法改成 **asyncProcessBlock** 方法即可。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let webServer = GCDWebServer()
         
        //异步响应HTTP
        webServer?.addDefaultHandler(forMethod: "GET", request: GCDWebServerRequest.self,
                                     asyncProcessBlock: { (request, completionBlock) in
            //5秒后返回响应（模拟网络请求、I/O读写等耗时操作）
            DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 5) {
                let html = "<html><body>欢迎访问 <b>hangge.com</b></body></html>"
                let response = GCDWebServerDataResponse(html: html)
                //处理结束后回调GCDWebServer
                completionBlock!(response)
            }
        })
         
        webServer?.start(withPort: 8080, bonjourName: "GCD Web Server")
        print("服务启动成功，使用你的浏览器访问：\(webServer?.serverURL)")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

###  2，运行效果

上面代码启动服务后，我们访问 **http://localhost:8080** 要等个 **5** 秒钟才回显示结果。

## 五、HTTP请求重定向

### 1，样例代码

下面实现了一个从“**/**”到“**/index.html**”的页面重定向样例（客户端跳转）。

```swift
webServer?.addHandler(forMethod: "GET", path: "/", request: GCDWebServerRequest.self,
                      processBlock: { (request) -> GCDWebServerResponse? in
    let url = URL(string: "index.html", relativeTo: request?.url)
    return GCDWebServerResponse.init(redirect: url, permanent: false)
})
```

###  2，运行效果

上面代码启动服务后，我们访问 **http://localhost:8080** 时，便自动重定向到 **http://localhost:8080/index.html**。

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017022014140522647.png)](https://www.hangge.com/blog/cache/detail_1555.html#)[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017022014135084758.png)](https://www.hangge.com/blog/cache/detail_1555.html#)

## 六、实现Form表单功能

### 1，样例说明

下面实现一个 **HTTP** 表单，一共用到了两个 **handlers**：

- **GET handler**：产生一个包含一个简单的 **HTML** 表单响应。由于不需要 **HTTP** 请求中的 **body** 信息，因此直接采用 **GCDWebServerRequest** 类。
- **POST handler**：从用户提交的表单中获取值，并将处理结果返回。由于需要得到 **HTTP** 请求中经过 **encode** 后的 **body** 信息中的表单值，就要借助于 **GCDWebServerURLEncodedFormRequest** 类。



### 2，相关配置

由于获取 **form** 表单数据用到了 **GCDWebServerURLEncodedFormRequest**，所以我们需要在桥接头文件中将相关的头文件引入。

```swift
#import "GCDWebServer.h"
#import "GCDWebServerDataResponse.h"
#import "GCDWebServerURLEncodedFormRequest.h"
```



### 3，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let webServer = GCDWebServer()
         
        //处理get请求：/（返回一个提交表单）
        webServer?.addHandler(forMethod: "GET", path: "/", request: GCDWebServerRequest.self,
                              processBlock: { (request) -> GCDWebServerResponse? in
                let html = "<html><body>" +
                            "<form name=\"input\" action=\"/\" method=\"post\" " +
                                "enctype=\"application/x-www-form-urlencoded\"> " +
                            "用户名: <input type=\"text\" name=\"username\">" +
                            "<input type=\"submit\" value=\"提交\">" +
                            "</form>" +
                            "</body></html>"
                return GCDWebServerDataResponse(html: html)
        })
         
        //处理post请求：/（获取提交的表单数据，并返回结果）
        webServer?.addHandler(forMethod: "POST", path: "/",
                              request: GCDWebServerURLEncodedFormRequest.self,
                              processBlock: { (request) -> GCDWebServerResponse? in
                                let formRequest = request as! GCDWebServerURLEncodedFormRequest
                                let value = formRequest.arguments["username"]
                                let html = "<html><body>\(value)</body></html>"
                                return GCDWebServerDataResponse(html: html)
        })
         
        webServer?.start(withPort: 8080, bonjourName: "GCD Web Server")
        print("服务启动成功，使用你的浏览器访问：\(webServer?.serverURL)")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 4，效果图

（1）程序启动后，我们使用电脑浏览器访问 **http://localhost** 会显示出一个 **form** 表单。

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017022015322811296.png)](https://www.hangge.com/blog/cache/detail_1555.html#)


（2）填写内容并提交后，会返回显示刚才我们提交的值。

[![原文:Swift - GCDWebServer使用详解1（介绍、安装配置、HTTP服务实现）](https://www.hangge.com/blog_uploads/201702/2017022015312514922.png)](https://www.hangge.com/blog/cache/detail_1555.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1555.html