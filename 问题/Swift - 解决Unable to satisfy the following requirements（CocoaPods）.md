# Swift - 解决Unable to satisfy the following requirements（CocoaPods）

2017-05-31发布：hangge阅读：604

### 1，问题描述

项目需要用到 **SnapKip**，同往常一样，准备使用 **CocoaPods** 来下载安装。

当配置好 **Podfile**，执行“**pod install**”命令却报“**Unable to satisfy the following requirements**”错误。具体信息如下：

[![原文:Swift - 解决Unable to satisfy the following requirements（CocoaPods）](https://www.hangge.com/blog_uploads/201703/2017032011203817493.png)](https://www.hangge.com/blog/cache/detail_1609.html#)



### 2，问题原因

这是由于本地的第三方库的数据和远端的数据不同步，导致 **CocoaPods** 本地的版本库低于远端。只要解决版本不同步的问题即可。

### 3，解决办法

（1）执行 **pod update --verbose** 命令。

[![原文:Swift - 解决Unable to satisfy the following requirements（CocoaPods）](https://www.hangge.com/blog_uploads/201703/2017032011281070483.png)](https://www.hangge.com/blog/cache/detail_1609.html#)



（2）命令执行完毕后，就可以正常 **pod install** 了。

[![原文:Swift - 解决Unable to satisfy the following requirements（CocoaPods）](https://www.hangge.com/blog_uploads/201703/2017032011293152295.png)](https://www.hangge.com/blog/cache/detail_1609.html#)