# Swift - 如何对浮点数进行取余（取模）

2017-01-09发布：hangge阅读：12649

### 1，问题描述

过去 **Swift** 中的取模运算符（**%**）对任何数值类型都是有效的，不管是整型还是浮点型（**Float**、**Double**、**CGFloat**）。但到了 **Swift3**，取余算法是不能作用于浮点型的，否则就会报“**‘%’ is unavailable: Use truncatingRemainder instead** ”错误。

[![原文:Swift - 如何对浮点数进行取余（取模）](https://www.hangge.com/blog_uploads/201611/2016112517220392819.png)](https://www.hangge.com/blog/cache/detail_1457.html#)

###  2，解决办法

使用 **truncatingRemainder** 方法进行浮点数取余

```swift
let value1 = 5.5
let value2 = 2.2
 
let result = value1.truncatingRemainder(dividingBy: value2)
```

注意方法返回值仍然是浮点型，运行效果如下： 

[![原文:Swift - 如何对浮点数进行取余（取模）](https://www.hangge.com/blog_uploads/201611/201611251731085317.png)](https://www.hangge.com/blog/cache/detail_1457.html#)