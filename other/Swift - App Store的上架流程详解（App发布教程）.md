# Swift - App Store的上架流程详解（App发布教程）

2017-04-07发布：hangge阅读：2578

想要将自己开发的 **App** 上架到 **App Store**，我们首先要有一个开发者账号，可以是个人开发者账号、或公司开发者账号（企业账号不能提交 **App Store**）。有了账号以后，按如下步骤提交应用。

### 1，iTunes Connect上填写相关信息

（1）首先我们登录苹果开发者官网：https://developer.apple.com/

（2）登录后点击左侧的“**iTunes Connect**”菜单，在出现的右侧界面中点击“**Go to iTunes Connect**”按钮。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030517333612236.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（3）在打开的页面中，点击“**我的App**”

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030517372714305.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（4）点击左上角的加号，选择“**新建App**”

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030517392218894.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（5）在弹出的小窗口中填写好我们 **App** 的一些基本信息。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030609274187474.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（6）点击“**创建**”后，就进入到详细信息页面。这个可以后面再写，因为上传 **App** 比较慢，这个可以等后面上传的时候再过来写。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030609334087767.png)](https://www.hangge.com/blog/cache/detail_1576.html#)

### 2，提交App

（1）关于证书的生成和相关配置，这里就不再说明了。

（2）发布编译目标选择“**Generic iOS Device**”

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201607/2016070321205299164.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（3）顶部菜单选择“**Product**”->“**Archive**”

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201607/2016070321224911299.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



（4）在弹出的界面中点击“**Upload to App Store...**” 按钮，**Xcode** 便会自动上传我们的 **App**。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030610005518910.png)](https://www.hangge.com/blog/cache/detail_1576.html#)



### 3，完善iTunes Connect上的信息

（1）我们补充好 **App** 的内容介绍、版本信息、价格与销售范围。

注意：构建版本会在我们 **UpLoad to App Store** 成功之后的半个小时内显示出来。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030609374939110.png)](https://www.hangge.com/blog/cache/detail_1576.html#)

（2）一切填写完毕后，点击“**提交以供审核**”即可提交应用。

[![原文:Swift - App Store的上架流程详解（App发布教程）](https://www.hangge.com/blog_uploads/201703/2017030609410018402.png)](https://www.hangge.com/blog/cache/detail_1576.html#)

（3）接下来就耐心等待苹果审核了。这个一般3天内就可以审核完毕，如果没通过也会反馈具体的原因。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1576.html