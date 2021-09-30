# Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）

2016-12-05发布：hangge阅读：1564

在前文中：[Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog/cache/detail_1434.html)。我介绍了如何使用 **Charles** 抓取普通的 **HTTP** 请求。但如果请求的是 **HTTPS** 地址的话，如下下面样例：

```swift
let parameters:[String : Any] = [
    "foo": [1,2,3],
    "bar": [
        "name": "hangge"
    ]
]
 
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters,
                    encoding: JSONEncoding.default)
```

会发现捕获到请求中，**request** 和 **response** 信息都是乱码。其实如果想要抓取 **HTTPS** 请求的话，只需要安装相关的 **SSL** 证书即可。



### 1，Charles 根证书配置 

（1）选择 **Charles** 菜单“**Help**”->“**SSL Proxying**”->“**Install Charles Root Certificate**”

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111115160311930.png)](https://www.hangge.com/blog/cache/detail_1435.html#)

（2）接着会自动弹出“**钥匙串访问**”窗口。默认情况下 **Charles Proxy CA** 这个证书是不被信任的，我们右键选择“**显示简介**”。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111115185759721.png)](https://www.hangge.com/blog/cache/detail_1435.html#)

（3）将“**使用此证书时：**”改成“**始终信任**”

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111115340371150.png)](https://www.hangge.com/blog/cache/detail_1435.html#)



（4）关闭后可以看到 **Charles** 已经是信任的了。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111115353493641.png)](https://www.hangge.com/blog/cache/detail_1435.html#)

### 2，Charles设置

（1）选择菜单中“**Proxy**”->“**SSL Proxying Settings...**”

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116041986211.png)](https://www.hangge.com/blog/cache/detail_1435.html#)



（2）在设置窗口中，勾选“**Enable SSL Proxying**”。同时把要抓取的 **HTTPS** 请求的域名地址和端口填上。（比如这里要抓取本文最开始样例里的地址）

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/201611111607047667.png)](https://www.hangge.com/blog/cache/detail_1435.html#)



（3）当然我们也可以使用通配符 *****。比如下面设置，就表示抓取所有地址与端口的 **HTTPS** 请求。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116120326436.png)](https://www.hangge.com/blog/cache/detail_1435.html#)

###  3，模拟器配置

（1）首先确保模拟器已经启动。选择Charles菜单“**Help**”->“**SSL Proxying**”->“**Install Charles Root Certificate in iOS Simulators**”

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116154918136.png)](https://www.hangge.com/blog/cache/detail_1435.html#)

（2）上面选择后就会自动安装配置好模拟器证书。我们再此运行测试程序，会发现 **Charles** 已经能成功捕获并解析出请求数据了。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116225418342.png)](https://www.hangge.com/blog/cache/detail_1435.html#)



### 4，真机配置

（1）打开手机浏览器，访问如下地址：**charlesproxy.com/getssl**

（2）接着会弹出下面页面，点击“**安装**”即可。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116565385438.png)](https://www.hangge.com/blog/cache/detail_1435.html#)


（3）后面就很普通的 **HTTP** 请求抓取一样了，进入 **iPhone** 里的 **HTTP** 代理设置，将关闭改为手动，在服务器位置输入 **Mac** 的 **IP** 地址，端口 **8888**。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116462531783.png)](https://www.hangge.com/blog/cache/detail_1435.html#)



（4）最后在 **iPhone** 上打开要抓包的 **App**，**Charles** 这边就可以抓取到设备上的网络请求了。注意 **Charles** 在这过程中会弹出个确认框，我们点击允许就可以了。

[![原文:Swift - 网络抓包工具Charles使用介绍2（HTTPS请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116493368286.png)](https://www.hangge.com/blog/cache/detail_1435.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1435.html