# Swift - 检查项目中是否使用了IDFA（附：使用友盟统计时IDFA选项的勾选）

2017-06-05发布：hangge阅读：2132

### 1，IDFA介绍

（1）**IDFA** 是苹果 **iOS 6** 新增的广告标示符，英文全称是 **Identifier for Advertising**，中文叫广告标示符。

（2）**IDFA** 可以简单理解为 **iPhone** 的设备身份证，适用于对外：例如广告推广，换量等跨应用的用户追踪等。

（3）**IDFA** 存储在用户 **iOS** 系统上，同一设备上的所有程序获取到的 **IDFA** 是相同的。但是有几种情况下，会重新生成广告标示符。

- 用户完全重置系统（(设置程序 -> 通用 -> 还原 -> 还原位置与隐私) ，这个广告标示符会重新生成。
- 用户明确的还原广告(设置程序-> 通用 -> 关于本机 -> 广告 -> 还原广告标示符) ，那么广告标示符也会重新生成。

（4）同时 **iOS 10** 开始提供禁止广告跟踪功能，勾选这个功能的设备无法读取到 **IDFA**。

### 2，检查项目中是否包含IDFA

我们将应用提交到 **App Store** 时，**iTunes Connect** 会询问我们是否使用了广告标识符，以及具体的使用情况。

如果我们项目代码或者引用的第三方 **SDK** 库用到了 **IDFA**，就需要如实勾选。否则可以会无法审核通过。我们可以使用如下操作来检查。

（1）打开“**终端**”，进入到项目文件夹。

[![原文:Swift - 检查项目中是否使用了IDFA（附：使用友盟统计时IDFA选项的勾选）](https://www.hangge.com/blog_uploads/201703/2017032010010753264.png)](https://www.hangge.com/blog/cache/detail_1608.html#)



（2）执行如下命令。（注意最后有个点号不要漏了）

```
grep -r advertisingIdentifier .
```

（3）由于我的项目中使用到了 **IDFA** 版的有盟统计 **SDK**，检查结果如下。

[![原文:Swift - 检查项目中是否使用了IDFA（附：使用友盟统计时IDFA选项的勾选）](https://www.hangge.com/blog_uploads/201703/2017032010063354095.png)](https://www.hangge.com/blog/cache/detail_1608.html#)



## 使用IDFA版的友盟，提交时如何选择防止被AppStore拒绝

- 友盟提供了 **IDFA** 版（标准版）和不含 **IDFA** 版两个 **SDK**，两个 **SDK** 在数据上并没有差异。
- 建议优先使用标准版 **SDK**，采集 **IDFA** 是为了防止今后因为苹果可能禁止目前使用的 **openudid** 而造成的数据波动。

如果我们的应用使用友盟标准 **SDK** 而未集成任何广告服务，但需要跟踪广告带来的激活行为，按照下图填写 **Appstore** 中的 **IDFA** 选项：

[![原文:Swift - 检查项目中是否使用了IDFA（附：使用友盟统计时IDFA选项的勾选）](https://www.hangge.com/blog_uploads/201703/2017032010130228550.png)](https://www.hangge.com/blog/cache/detail_1608.html#)



如果仍因为采集 **IDFA** 被 **Appstore** 审核拒绝，建议集成任意一家广告或选用友盟无 **IDFA** 版 **SDK**。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1608.html