# Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程

2016-07-05发布：hangge阅读：5381

**1，企业账号介绍**
（1）使用企业开发账户，我们可以发布一个 **ipa** 放到网上，所有人（包括越狱及非越狱设备）都可以直接通过链接下载安装，而不需要通过 **AppStore** 下载，也不需要安装任何证书。
（2）当然，使用企业账号发布的 **iOS** 应用是不能提交到 **AppStore** 上的。而且企业级开发账号也比个人帐号更贵些（299刀/年）。
（3）既然叫企业帐号，就说明是用来开发企业自己的内部应用，给自己的员工使用的。所以不要用企业号做大规模应用分发的一个渠道，否则有可能会被苹果封账号。



**2，IPA打包**

（1）首先要上苹果开发者中心，生成发布证书和相关配置文件。然后下载到本地安装下，这个我就不具体说明了。

（2）打开项目，在“**General**”->“**Team**”中选择团队名称。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321343473694.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（3）在“**Build Settings**” -> “ **Code Signing** ”区域中选择发布证书。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321180845717.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（4）发布编译目标选择“**Generic iOS Device**”

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321205299164.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（5）顶部菜单选择“**Product**”->“**Archive**”

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321224911299.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（6）在弹出的界面中点击“**Export ...**” 进入打包方式选择界面。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321244933085.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（7）选择“**Sava for Enterprise Deployment**”

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321270266431.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（8）选择对应的企业帐号，然后继续即可。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321384424674.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（9）接下来就是安装设备的要求选择。我们选择第一项（默认项），让所有设备都可以安装。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070321400959525.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（10）接下来是确认页面，我们可以核对下各个配置是否正确。同时勾选下方的“**Include manifest for over-the-air Installation**”，表示生成 **.ipa** 文件的同时还会生成 **.plist** 文件。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070509320681737.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（11）接下来配置 **.plist** 文件的相关信息：应用名、发布地址、图标地址、大图地址。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070322040418547.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



（12）然后选择点击“**Export**”就可以导出.ipa安装包及其相应的 **.plist** 文件。

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070322092615728.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



**3，将文件部署到服务器**

（1）首先这个网站要支持 **HTTPS** 协议，用来访问下载 **.plist** 文件。

我们可以自己申请证书来配置，也可以使用我之前介绍的傻瓜化安装工具来部署：[StartEncrypt - 一键部署启用HTTPS服务](https://www.hangge.com/blog/cache/detail_1258.html)


（2）除了**.ipa**、**.plist** 这两个文件。我们还需要提供两个图片（就是配置 .**plist** 信息的时候填写的）

一个尺寸是 **57 X 57** 像素，用来显示下载和安装过程中的图标。

一个尺寸是 **512 X 512** 像素，用来在 **iTunes** 中显示。

（3）同时，我们再创建一个 **html** 页面供用户访问。用户通过点击这个网页上的链接触发 **App** 的下载与安装。

```html
<DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>hangge.com</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
</head>
<body>
  <a href="itms-services://?action=download-manifest&url=https://www.hangge.com/ios/manifest.plist">点击开始安装App</a>
</body>
</html>
```




最后，我们将这5个文件一起放到服务器根路径下的ios目录中。（这个根据你在 **.plist** 文件里的配置路径来放置）

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070410471028665.png)](https://www.hangge.com/blog/cache/detail_1259.html#)



**4，下载安装**

（1）使用手机浏览器访问安装页面：https://www.hangge.com/ios/index.html

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070411140897369.png)](https://www.hangge.com/blog/cache/detail_1259.html#)

（2）点击安装链接，会弹出确认提示框

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/2016070411142057135.png)](https://www.hangge.com/blog/cache/detail_1259.html#)

（3）确定后，即可开始安装

[![原文:Swift - 企业账号进行IPA的打包、分发、下载安装的详细流程](https://www.hangge.com/blog_uploads/201607/201607041114295281.png)](https://www.hangge.com/blog/cache/detail_1259.html#)

（4）如果是 **iOS9** 以上的版本，启动 **App** 时会提示“未受信任的企业级开发者”。

只要在手机系统里“设置”->“通用”->“设备管理”->“企业级应用”中，点击信任即可。

**无法安装问题：**

有时我们把 **IPA** 放到服务器上，手机却死活安装不了。一直提示无法安装。可以试试如下方法处理。

（1）可能你第一次提交到服务器的 **.plist** 文件有误，手机无法安装。后面即使修改了并将其覆盖，由于客户端对这个文件会有缓存就会造成还是安装不成功。可以将 **.plist** 文件改个名字再试试。

（2）如果手机的版本太低，而编译时指定的发布版本又太高，也会无法安装。可以在“**General**”->“**Deployment Target**”中设置成低版本。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1259.html