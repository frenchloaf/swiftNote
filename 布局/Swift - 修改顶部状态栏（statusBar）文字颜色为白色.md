# Swift - 修改顶部状态栏（statusBar）文字颜色为白色

2016-05-02发布：hangge阅读：8094

默认情况下，APP顶部“运营商”“电量”等状态栏文字颜色是黑色的，但如果界面背景色比较深的话就会不太好看。

[![原文:Swift - 修改顶部状态栏（statusBar）文字颜色为白色](https://www.hangge.com/blog_uploads/201605/2016050117153228076.png)](https://www.hangge.com/blog/cache/detail_1164.html#)

**下面将状态栏文字颜色改成白色（浅色）**

[![原文:Swift - 修改顶部状态栏（statusBar）文字颜色为白色](https://www.hangge.com/blog_uploads/201605/2016050117154182680.png)](https://www.hangge.com/blog/cache/detail_1164.html#)



1，在 **Info.plist** 中添加如下配置

[![原文:Swift - 修改顶部状态栏（statusBar）文字颜色为白色](https://www.hangge.com/blog_uploads/201605/2016050117161517571.png)](https://www.hangge.com/blog/cache/detail_1164.html#)

```
<``key``>UIViewControllerBasedStatusBarAppearance</``key``>``<``false``/>
```


2，在 **General** -> **Deployment Info** 中，将 **Status Bar Style** 设置成 **Light**。重新运行程序即可看到效果。

[![原文:Swift - 修改顶部状态栏（statusBar）文字颜色为白色](https://www.hangge.com/blog_uploads/201605/2016050117162311214.png)](https://www.hangge.com/blog/cache/detail_1164.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1164.html