# Swift - StoryBoard中实现页面元素三等分（Auto Layout ）

2017-02-02发布：hangge阅读：1458

**Auto Layout** 的本质是用一些约束条件对元素进行约束，从而让他们显示在我们需要他们显示的地方。下面本文演示如何在 **storyboard** 中通过设置约束实现三等分。

### 1，效果图

（1）界面上横向放置 **3** 个 **UIView**，每个 **view** 高度固定，距离上方边距也是固定。

（2）左右两个 **view** 距离两侧边距也是固定。

（3）每个 **view** 的横向间距固定。宽度相等。

（4）上面约束设置完毕后，这个 **3** 个 **view** 在不同尺寸的屏幕下都自适应宽度，从而实现三等分。

- 4寸：

  [![原文:Swift - StoryBoard中实现页面元素三等分（Auto Layout ）](https://www.hangge.com/blog_uploads/201701/2017013120403691278.png)](https://www.hangge.com/blog/cache/detail_1539.html#)

- 5.5寸：

  [![原文:Swift - StoryBoard中实现页面元素三等分（Auto Layout ）](https://www.hangge.com/blog_uploads/201701/2017013120404380130.png)](https://www.hangge.com/blog/cache/detail_1539.html#)

### 2，操作步骤

（1）在 **storyboard** 中放置 **3** 个 **UIView**，并调整好位置尺寸。

[![原文:Swift - StoryBoard中实现页面元素三等分（Auto Layout ）](https://www.hangge.com/blog_uploads/201701/2017013119082786927.png)](https://www.hangge.com/blog/cache/detail_1539.html#)



（2）相关约束设置见如下动画。

[![原文:Swift - StoryBoard中实现页面元素三等分（Auto Layout ）](https://www.hangge.com/blog_uploads/201701/2017013120350858597.gif)](https://www.hangge.com/blog/cache/detail_1539.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1539.html