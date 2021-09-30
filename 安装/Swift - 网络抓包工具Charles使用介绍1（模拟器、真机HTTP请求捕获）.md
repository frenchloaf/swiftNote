# Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）

2016-12-02发布：hangge阅读：3476

**App** 中难免会需要进行网络请求，本文介绍一个 **Mac** 下好用的 **HTTP**/**HTTPS** 抓包工具：**Charles**。它能拦截所有的网络请求，方便我们程序的开发与调试。比如：检查发送的参数是否正确，服务端返回的数据是否正确等等。

## 一、Charles的介绍与安装

**Charles** 是在 **Mac** 下常用的截取网络封包的工具。通过将自己设置成系统的网络访问代理服务器，使得所有的网络访问请求都通过它来完成，从而实现了网络封包的截取和分析。

（虽然是收费软件，可以免费试用**30**天。但试用期过后仍然可以继续使用，只是每次启动时将会有**10**秒种的延时，每次使用时间不能超过**30**分钟。功能是没有任何限制的。）



### 1，Charles的功能特点

- 支持 **SSL** 代理。可以截取分析 **SSL** 的请求。
- 支持流量控制。可以模拟慢速网络以及等待时间（**latency**）较长的请求。
- 支持 **AJAX** 调试。可以自动将 **json** 或 **xml** 数据格式化，方便查看。
- 支持 **AMF** 调试。可以将 **Flash Remoting** 或 **Flex Remoting** 信息格式化，方便查看。
- 支持重发网络请求，方便后端调试。
- 支持修改网络请求参数。
- 支持网络请求的截获并动态修改。
- 检查 **HTML**，**CSS** 和 **RSS** 内容是否符合 **W3C** 标准。

### 2，Charles的安装

只需到官网下载最新版的 **Charles** 安装包（**dmg**后缀的文件）。打开后将 **Charles** 拖到 **Application** 目录下即完成安装。
官网下载地址：https://www.charlesproxy.com/download/

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111114035938113.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



## 二、Charles的使用

**Charles** 可以抓取所有应用的所有网络请求。本文主要介绍进行 **iOS** 开发时，相关的网络请求抓取。比如我们 **App** 在模拟器中调试时，又或者使用真机运行时，如何抓取 **App** 的网络请求。



### 1，模拟器调试

（1）启动 **Charles** 后，右键点击菜单程序图标，选择 “**macOs Proxy**”将 **Charles** 设置成系统代理。

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111114142726291.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



（2）我们在程序中写个简单的网络请求测试代码。

```
let` `parameters:[``String` `: ``Any``] = [``  ``"foo"``: [1,2,3],``  ``"bar"``: [``    ``"baz"``: ``"qux"``  ``]``]` `Alamofire``.request(``"http://www.hangge.com"``, parameters: parameters)
```


（3）运行后可以发现 **Charles** 已经成功的捕获到请求，我们可以查看请求的 **header**，参数，响应等等信息。

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111114580985441.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



### 2，真机调试

（1）首先确保 **iPhone** 和 **Mac** 是在同一个网段下。

（2）进入 **iPhone**，进行 **HTTP** 代理设置，从关闭改为手动，在服务器位置输入 **Mac** 的 **IP** 地址，端口 **8888**。

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116462531783.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



（3）接着在 **iPhone** 上打开要抓包的 **App**，**Charles** 这边就可以抓取到设备上的网络请求了（注意 **Charles** 在这过程中会弹出个确认框，我们点击允许就可以了）。

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111116493368286.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



## 三、过滤网络请求

通常情况下，我们并不需要对所有网络请求都进行监控。有下面2种办法可以过滤网络请求，从而只监控指定目录服务器的请求。

### 1，从捕获结果中过滤

将选项卡切换到“**Sequence**”，在 **Filter** 输入框中填入需要过滤出来的服务器地址。比如：[hangge.com](http://www.hangge.com/)

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201612/2016120209332791719.png)](https://www.hangge.com/blog/cache/detail_1434.html#)

###  2，只截取指定网站的封包

（1）在 **Charles** 的菜单栏选择“**Proxy**” ->“**Recording Settings...**”

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111117184550999.png)](https://www.hangge.com/blog/cache/detail_1434.html#)



（2）在 **Include** 栏中填入需要监控的协议，主机地址，端口号。这样就可以只截取目标网站的封包了。（这里端口我使用通配符 ***** 表示所有端口，也可以指定某个端口，比如 **80**）

[![原文:Swift - 网络抓包工具Charles使用介绍1（模拟器、真机HTTP请求捕获）](https://www.hangge.com/blog_uploads/201611/2016111117214929478.png)](https://www.hangge.com/blog/cache/detail_1434.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1434.html