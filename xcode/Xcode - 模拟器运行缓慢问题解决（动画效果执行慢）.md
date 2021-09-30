# Xcode - 模拟器运行缓慢问题解决（动画效果执行慢）

2017-03-25发布：hangge阅读：2140

### 1，问题现象

今天刚用 **Swift** 写了个小程序，使用模拟器（**Simulator**）运行时发现运行极其缓慢。启动程序要等半天才显示出主界面。按 **home** 退回桌面，或者滑动切换桌面的过渡动画也十分缓慢，跟慢动作一样。

[![原文:Xcode - 模拟器运行缓慢问题解决（动画效果执行慢）](https://www.hangge.com/blog_uploads/201703/2017030919375530282.png)](https://www.hangge.com/blog/cache/detail_1589.html#)



### 2，问题原因

开始还以为是我电脑或者模拟器出问题了，后来才发现是不小心启用了模拟器的减缓动画功能。

### 3，解决办法

去掉模拟器菜单“**Debug**”->“**Slow Animations**”的勾选即可。

[![原文:Xcode - 模拟器运行缓慢问题解决（动画效果执行慢）](https://www.hangge.com/blog_uploads/201703/2017030919352354138.png)](https://www.hangge.com/blog/cache/detail_1589.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1589.html