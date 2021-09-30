# Swift - GCDWebServer使用详解2（实现静态、动态网站，Web服务器）

2017-03-17发布：hangge阅读：2459

## 一、实现一个静态的文件目录网站（Static Website）

**GCDWebServer** 内置的处理程序可以递归服务端目录，从而实现一个静态的文件目录浏览功能（只读）。同时我们还可以自由设置“**Cache - Control**”头。

### 1，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let webServer = GCDWebServer()
        webServer?.addGETHandler(forBasePath: "/", directoryPath: NSHomeDirectory(),
                                 indexFilename: nil, cacheAge: 3600,
                                 allowRangeRequests: true)
        webServer?.start(withPort: 8080, bonjourName: "GCD Web Server")
        print("服务启动成功，使用你的浏览器访问：\(webServer?.serverURL)")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 2，运行效果



用模拟器运行程序，然后使用电脑浏览器访问 **http://localhost:8080** 便会显示出 **App** 的沙盒目录。我们可以点击浏览里面各个文件和文件夹，或者对文件进行下载操作。

  [![原文:Swift - GCDWebServer使用详解2（实现静态、动态网站，Web服务器）](https://www.hangge.com/blog_uploads/201702/2017021921134339956.png)](https://www.hangge.com/blog/cache/detail_1563.html#)   [![原文:Swift - GCDWebServer使用详解2（实现静态、动态网站，Web服务器）](https://www.hangge.com/blog_uploads/201702/2017021921134977401.png)](https://www.hangge.com/blog/cache/detail_1563.html#)



## 二、实现动态网站（Dynamic Website）

有时候简单的静态页面浏览并不能满足我们的需求，我们可能需要页面能够根据不同的条件动态显示内容，这时可以借助 **GCDWebServerDataResponse** 类来实现。

**GCDWebServerDataResponse** 继承自 **GCDWebServerResponse** 类，它为我们提供了一套基本的模板技术。即根据模板和一组变量（格式为 **%variable%**）来生成 **HTML** 内容并返回。下面通过一个样例作为演示。

### 1，效果图

（1）程序启动后，我们使用浏览器访问 **http://localhost:8080** 即可显示网站首页内容。（程序会自动重定向到 **index.html** 页面上）

（2）点击网页左侧的图片列表，右侧 **image** 组件会显示响应的图片内容。这里注意的是：左侧的图片列表、以及标题并不是写死在 **html** 页面上的，而是通过程序里的变量填充到模版中生成的。

（3）也就是说整个网站的 **js**、**css**、图片是静态的。而网页是动态的。

[![原文:Swift - GCDWebServer使用详解2（实现静态、动态网站，Web服务器）](https://www.hangge.com/blog_uploads/201702/201702202004444519.png)](https://www.hangge.com/blog/cache/detail_1563.html#)



### 2，网站相关代码

（1）整个项目结构如下，其中 **Website** 文件夹（注意是文件夹，不是组）下放置的是网站中用到的所有页面和资源。

[![原文:Swift - GCDWebServer使用详解2（实现静态、动态网站，Web服务器）](https://www.hangge.com/blog_uploads/201702/2017022019541893765.png)](https://www.hangge.com/blog/cache/detail_1563.html#)



（2）**default.css**

```css
.left {
    float:left;
    width:150px;
    margin-right: 10px;
}
 
.title {
    background-color: #339718;
    color: #FFFFFF;
    padding: 10px;
    text-align: center;
}
 
li {
    cursor:pointer;
}
```


（3）**index.html**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>hangge.com</title>
  <link rel="stylesheet" href="default.css" type="text/css">
  <script src="jquery.js" charset="utf-8"></script>
  <script>
     $(document).ready(function(){
       $("li").click(function(){
         var data = $(this).attr("data");
         $("#image1").attr("src","imgs/"+data);
       });
     });
  </script>
</head>
<body>
   <div class="left">
     <div class="title">
       %var_title%
     </div>
     <ul>
       %var_li%
     <ul>
   </div>
   <img id="image1" src="imgs/1.png">
</body>
</html>
```



### 3，程序代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取网站文件夹路径
        let websitePath = Bundle.main.path(forResource: "Website", ofType: nil)
         
        let webServer = GCDWebServer()
         
        //先设置个默认的handler处理静态文件（比如css、js、图片等）
        webServer?.addGETHandler(forBasePath: "/", directoryPath: websitePath,
                                 indexFilename: nil, cacheAge: 3600,
                                 allowRangeRequests: true)
         
        //再覆盖个新的handler处理动态页面（html页面）
        webServer?.addHandler(forMethod: "GET", pathRegex: "^/.*\\.html$",
                              request: GCDWebServerRequest.self,
                              processBlock: { (request) -> GCDWebServerResponse? in
            let var_li = "<li data='1.png'>图片1</li>" +
                         "<li data='2.png'>图片2</li>" +
                         "<li data='3.png'>图片3</li>" +
                         "<li data='4.png'>图片4</li>"
            let variables = [
                "var_title": "点击切换图片",
                "var_li": var_li,
            ]
            let htmlTemplate = websitePath?.appending(request!.path)
            return GCDWebServerDataResponse(htmlTemplate: htmlTemplate,
                                            variables: variables)
        })
         
        //HTTP请求重定向（/从定向到/index.html）
        webServer?.addHandler(forMethod: "GET", path: "/",
                              request: GCDWebServerRequest.self,
                              processBlock: { (request) -> GCDWebServerResponse? in
                    let url = URL(string: "index.html", relativeTo: request?.url)
                    return GCDWebServerResponse.init(redirect: url, permanent: false)
        })
         
        //服务器启动
        webServer?.start(withPort: 8080, bonjourName: "GCD Web Server")
        print("服务启动成功，使用你的浏览器访问：\(webServer?.serverURL)")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1563.zip](https://www.hangge.com/blog_uploads/201702/2017022020033713828.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1563.html