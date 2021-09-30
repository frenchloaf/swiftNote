# Swift - 如何连续dismiss 2个VC视图控制器（以及直接跳回根视图）

2016-12-14发布：hangge阅读：6214

我们知道通过 **present** 和 **dismiss** 方法可以进行页面（**ViewController**）跳转。其中 **present** 是加载新的模态视图，而 **dismiss** 是退出当前视图，回到上一个视图。

[![原文:Swift - 如何连续dismiss 2个VC视图控制器（以及直接跳回根视图）](https://www.hangge.com/blog_uploads/201611/2016110915025364621.png)](https://www.hangge.com/blog/cache/detail_1430.html#)

但有时我们并不想要一级一级地往回跳，比如需要跨级跳转，或者直接跳回到根页面上。下面通过样例分别进行演示。

## 一、连续dissmiss两个视图

比如下面样例，我们在**C**页面中想要直接跳回到**A**页面。

[![原文:Swift - 如何连续dismiss 2个VC视图控制器（以及直接跳回根视图）](https://www.hangge.com/blog_uploads/201611/2016110915132030079.png)](https://www.hangge.com/blog/cache/detail_1430.html#)

代码如下：

```swift
self.presentingViewController?.presentingViewController?.dismiss(animated: true, completion: nil)
```



## 二、直接跳回到根视图

比如下面样例，我们在**F**页面上想直接跳回到最底层页面**A**。有两种实现方法。 

[![原文:Swift - 如何连续dismiss 2个VC视图控制器（以及直接跳回根视图）](https://www.hangge.com/blog_uploads/201611/2016110915190453715.png)](https://www.hangge.com/blog/cache/detail_1430.html#)

**1，循环调用 presentingViewController 获取根VC，再dissmiss**

```swift
//获取根VC
var rootVC = self.presentingViewController
while let parent = rootVC?.presentingViewController {
    rootVC = parent
}
//释放所有下级视图
rootVC?.dismiss(animated: true, completion: nil)
```


**2，直接通过 window.rootViewController 获取根VC，再dissmiss**

```swift
self.view.window?.rootViewController?.dismiss(animated: true, completion: nil)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1430.html