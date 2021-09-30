# Swift - 真机调试正常，打包成IPA安装后一启动就闪退的问题解决

2016-07-06发布：hangge阅读：3964

**问题描述：**

最近开发一个企业级的 **iOS** 应用。在模拟器上面运行是的正常，不打包直接连真机进行调试也是没问题。但是打包成 **ipa** 发布到服务器上，手机通过网页下载安装。App启动后就直接闪退。

**问题原因：**

这个其实是程序确实有bug（不管是个人应用、还是企业应用）。虽然平时使用模拟器调试是没问题，但用的都是 **debug** 模式，这个和 **release** 模式还是有区别的。

**问题解决：**

（1）在选择菜单中的“**Product**”->“**Scheme**”->“**Edit Scheme...**”

[![原文:Swift - 真机调试正常，打包成IPA安装后一启动就闪退的问题解决](https://www.hangge.com/blog_uploads/201607/2016070414470725747.png)](https://www.hangge.com/blog/cache/detail_1260.html#)



（2）将 **Build Configuration** 设置成 **Release**。

[![原文:Swift - 真机调试正常，打包成IPA安装后一启动就闪退的问题解决](https://www.hangge.com/blog_uploads/201607/201607041452178006.png)](https://www.hangge.com/blog/cache/detail_1260.html#)



（3）再次使用模拟器运行程序，你就会发现模拟器也会出现闪退。通过查看崩溃日志，发现报错信息是“dyld: Library not loaded: @rpath/libswiftAVFoundation.dylib Referenced from......”，具体截图如下：

[![原文:Swift - 真机调试正常，打包成IPA安装后一启动就闪退的问题解决](https://www.hangge.com/blog_uploads/201607/2016070415011535303.png)](https://www.hangge.com/blog/cache/detail_1260.html#)



（4）要解决这个问题只要在“**Build Settings**”-> “**Linking**”->“**Runpath Search Paths**”->“**Release**”节点下，

添加 **@executable_path/Frameworks** 即可（原来这个为空）。

[![原文:Swift - 真机调试正常，打包成IPA安装后一启动就闪退的问题解决](https://www.hangge.com/blog_uploads/201607/2016070415033254035.png)](https://www.hangge.com/blog/cache/detail_1260.html#)



（5）再次编译运行或者打包安装，就不会再出现闪退的情况了。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1260.html