# Swift - 去除Xcode8控制台中多余的打印信息

2016-10-06发布：hangge阅读：1087

**1，问题描述**

将 **Xcode** 升级成 **8.0** 后，发现只要一运行程序，控制台就自动打印输出一堆信息，看着不爽。如果应用里还有我们自己的打印日志，那混在一起也不方便查看。

[![原文:Swift - 去除Xcode8控制台中多余的打印信息](https://www.hangge.com/blog_uploads/201609/2016092314245620133.png)](https://www.hangge.com/blog/cache/detail_1371.html#)



**2，解决办法**

（1）点击菜单的：**Product** -> **Scheme** -> **Edit Scheme...**（或者使用快捷键 **command + shift + <**）

[![原文:Swift - 去除Xcode8控制台中多余的打印信息](https://www.hangge.com/blog_uploads/201609/2016092314225325025.png)](https://www.hangge.com/blog/cache/detail_1371.html#)

（2）在 **Run** -> **Arguments** -> **Environment Variables** 里边添加 **OS_ACTIVITY_MODE ＝ disable** 即可。

[![原文:Swift - 去除Xcode8控制台中多余的打印信息](https://www.hangge.com/blog_uploads/201609/201609231430239223.png)](https://www.hangge.com/blog/cache/detail_1371.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1371.html