# Swift - 双问号操作符（??）的介绍（附样例）

2016-12-23发布：hangge阅读：2023

**Swift** 提供了双问号操作符（**??**），英文叫 **Double Question Mark**。可以用来快速对 **nil** 进行条件判断。当我们获取一个可选值（**optional value**）时，如果希望其为 **nil** 的情况下返回一个非 **nil** 值，那么就可以把这个返回值放在 **??** 后面。下面演示几个常见的使用场景。



### 1，可选值不为nil则使用可选值，为nil则使用默认值

比如我们把 **userName** 这个参数值显示在 **label** 中，但希望 **userName** 如果为 **nil** 的话便显示"**未知用户**"。

[![原文:Swift - 双问号操作符（??）的介绍（附样例）](https://www.hangge.com/blog_uploads/201611/2016112415285832478.png)](https://www.hangge.com/blog/cache/detail_1456.html#)

这个我们可以使用三元条件运算来实现：

```swift
var userName:String?
self.label.text = userName != nil ? userName : "未知用户"
```

但使用双问号操作符会更加简单：

```swift
var userName:String?
self.label.text = userName ?? "未知用户"
```

###  2，as? 类型转换后处理nil值

```swift
let message = json["message"] as? String ?? "no message"
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1456.html