# Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）

2016-07-25发布：hangge阅读：2891

**App Store** 自2016年6月1日开始实施全新策略，所有提交至苹果 **App Store** 的 **iOS** 应用申请必须要兼容面向硬件识别和网络路由的最新互联网协议：**IPv6-only** 标准。

**一、IPv4与IPv6介绍**

**1，二者的区别**

（1）**IPv4** 是互联网协议（**Internet Protocol**，**IP**）的第四版，也是第一个被广泛使用，目前运用最多的互联网技术协议。

**IPv4** 地址格式是这个样子：**123.58.25.46**。

（2）**IPv6** 是 **IPv4** 的下一个版本 。**IPv6** 地址长度为 128 位，地址空间增加了 2^128-2^32 个，它在提高安全性方面相比前代有着较大的提升。此外，身份认证和隐私权也是 **IPv6** 的关键特性。

**IPv6** 地址格式是这个样子：**2001:da8:215:4009:250:56ff:fe97:40c7** 。

**2，什么是IPV6-Only支持**
（1）目前当我们用 iOS 设备连接上 **Wifi**、**4G**、**3G** 等网络时，设备被分配的地址均是 **IPV4** 地址。但是随着运营商和企业逐渐部署 **IPV6 DNS64/NAT64** 网络之后，设备被分配的地址会变成 **IPV6** 的地址，而这些网络就是所谓的 **IPV6-Only** 网络，并且仍然可以通过此网络去获取 **IPV4** 地址提供的内容。 

（2）这里说的支持 **IPV6-Only** 网络，其实就是说让应用在 **IPv6 DNS64/NAT64** 网络环境下仍然能够正常运行。

（3）但由于我们目前的实际网络环境仍然是 **IPV4** 网络，所以应用需要能够同时保证 **IPV4** 和 **IPV6** 环境下的可用性。

**二、如何让应用支持IPV6-Only**

**1，不要用IP地址**

比如我们与服务器进行数据请求。要使用域名（如[www.hangge.com](http://www.hangge.com/)），而不是使用硬编码的 IPv4地址（**123.58.25.46**）。

**2，使用高级的网络API**

这些高级的网络 **API** 包括：**CFNetwork**、**NSURL**、**NSURLSession**、**NSURLRequest**、**NSURLConnection**、**WebKit**。这些高级的API不仅便于使用，而且很多底层的像适配 **IPv6** 的工作都已经帮我们做好了，我们可以放心使用。

而对于一些内部是封装使用高级API的第三方库：比如 **Reachability**、**Alamofire**、最新版的 **AFNetWorking**。我们自然也不需要做什么，就可以兼容 **IPv6**。

**3，让底层的socket API同时支持IPV4和IPV6**

如果我们应用中使用了长连接，那肯定会使用到底层 **socket API**。这就需要我们手动来判断当前网络来生成对应 **IP** 格式。

推荐使用谷歌的开源库 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)。这个是支持 **IPv6** 的。

**三、搭建一个IPv6测试环境**

前面讲了这么多，不如在本地搭建一个 **IPv6** 网络测试环境。让 **App** 在这个环境下跑跑看，测试下有没有问题。

（1）首先你要有台通过网线上网的 **Mac** 电脑（注意是通过有线方式，不能是 **WiFi** 方式）
（2）打开“**系统偏好设置**”，按住“**Option**”键的同时点击“**共享**”

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/2016072416471342077.png)](https://www.hangge.com/blog/cache/detail_1303.html#)

（3）会发现在共享界面中多了个“**创建 NAT64 网络**”的复选框，勾选它。同时开启互联网共享功能。

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/2016072421161742648.png)](https://www.hangge.com/blog/cache/detail_1303.html#)



（4）这样我们就使用 **Mac** 做了一个 **NAT64** 网络热点。

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/2016072421162471921.png)](https://www.hangge.com/blog/cache/detail_1303.html#)



（5）最后用 **iPhone** 连接这个 **Wi-Fi** 热点，测试程序即可。注意：要把手机设置成飞行模式（先点飞行，再点WiFi）。防止手机使用蜂窝移动网络，如果有代理什么的也要去掉。

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/2016072421184273812.png)](https://www.hangge.com/blog/cache/detail_1303.html#)

（6）这里写一个很简单的测试样例，通过IP地址来获取数据（**202.108.22.5** 是百度搜索首页的 **IP**）

```swift
//创建NSURL对象
let urlString:String="http://202.108.22.5"
let url:NSURL! = NSURL(string:urlString)
//创建请求对象
let request:NSURLRequest = NSURLRequest(URL: url)
let session = NSURLSession.sharedSession()
let dataTask = session.dataTaskWithRequest(request,
               completionHandler: {(data, response, error) -> Void in
                if error != nil{
                    print(error?.code)
                    print(error?.description)
                }else{
                    let str = NSString(data: data!, encoding: NSUTF8StringEncoding)
                    print(str)
                }
}) as NSURLSessionTask
```




（7）使用 **iOS8** 系统的手机测试下，可以发现使用 **ip** 是无法请求到数据的（为什么用 **iOS8** 后面会说明）。

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/201607261044134960.png)](https://www.hangge.com/blog/cache/detail_1303.html#)



（8）如果改成通过域名的话，便可以获取到数据。

```swift
//let urlString:String="http://202.108.22.5"
let urlString:String="http://www.baidu.com"
```

[![原文:Swift - 如何让应用支持IPv6-only网络（附：搭建IPv6测试环境）](https://www.hangge.com/blog_uploads/201607/2016072610461737415.png)](https://www.hangge.com/blog/cache/detail_1303.html#)

**注意：为什么我手机在 IPv6 NAT64环境下，使用IP地址也能获取到数据？**

你的手机肯定版本是 **iOS9** 的。如果是 **iOS8.4** 及以下版本肯定是不能正常访问。

**苹果官方解释如下：**
In iOS 9 and OS X 10.11 and later, NSURLSession and CFNetwork automatically synthesize IPv6 addresses from IPv4 literals locally on devices operating on DNS64/NAT64 networks. However, you should still work to rid your code of IP address literals.

大意就是虽然 **iOS9** 自动会将 **IPV4** 地址合成 **IPV6** 地址，让其在 **DNS64/NAT64** 网络上运行。但你仍然需要把这种写死的IP地址给去掉（比如改成域名）。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1303.html