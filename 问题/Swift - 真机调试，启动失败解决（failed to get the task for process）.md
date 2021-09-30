# Swift - 真机调试，启动失败解决（failed to get the task for process）

2016-09-02发布：hangge阅读：1882

**问题描述：**

今天把手机接到电脑上，打算通过 **Xcode** 进行真机调试。编译运行后，直接启动报错。手机白屏，**Xcode** 这边显示报错信息如下：

**Could not launch “航歌”**
process launch failed: failed to get the task for process 9794

[![原文:Swift - 真机调试，启动失败解决（failed to get the task for process）](https://www.hangge.com/blog_uploads/201608/2016081814021446412.png)](https://www.hangge.com/blog/cache/detail_1336.html#)

而使用模拟器运行是没问题的。

**原因分析：**

这是证书问题。真机调试 **project** 和 **targets** 的证书都必须是开发证书，我之前使用发布证书发布程序后忘记改回来了。

**问题解决：**

在“**Build Settings**”->“**Code Signing**”中将 **Distribution** 改成 **Developer** 即可。

[![原文:Swift - 真机调试，启动失败解决（failed to get the task for process）](https://www.hangge.com/blog_uploads/201608/2016081814082973021.png)](https://www.hangge.com/blog/cache/detail_1336.html#)




原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1336.html