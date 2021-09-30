# Swift - 实现NSRange与Range的相互转换

2017-06-23发布：hangge阅读：5792

相较于 **OC**的**NSRange**，**斯威夫特**的**范围**更加复杂，用法也有很大的区别。虽然通常来说我们在**斯威夫特**中使用**范围**就可以了，但有些情况下还是要使用**NSRange**，比如创建富文本的时候。下面演示如何实现**NSRange**与**Range**的相互转换。

### 1，扩展字符串，增加转换方法

为了方便使用，这里对**String**做一个扩展，增加了两个新方法实现的转换。

```swift
extension String {
     
    //Range转换为NSRange
    func nsRange(from range: Range<String.Index>) -> NSRange {
        let from = range.lowerBound.samePosition(in: utf16)
        let to = range.upperBound.samePosition(in: utf16)
        return NSRange(location: utf16.distance(from: utf16.startIndex, to: from),
                       length: utf16.distance(from: from, to: to))
    }
     
    //Range转换为NSRange
    func range(from nsRange: NSRange) -> Range<String.Index>? {
        guard
            let from16 = utf16.index(utf16.startIndex, offsetBy: nsRange.location,
                                     limitedBy: utf16.endIndex),
            let to16 = utf16.index(from16, offsetBy: nsRange.length,
                                   limitedBy: utf16.endIndex),
            let from = String.Index(from16, within: self),
            let to = String.Index(to16, within: self)
            else { return nil }
        return from ..< to
    }
}
```



### 2，使用样例 

（1）下面代码我们先定义一个**NSRange**，将其转为**Range**后对字符串进行截取。

```swift
let nsRange = NSRange(location: 0, length: 3)
 
let str = "hangge.com"
let range = str.range(from: nsRange)
 
let subStr = str.substring(with: range!)
print("截取后的字符串：\(subStr)")
```


（2）运行结果如下：

[![原文:Swift - 实现NSRange与Range的相互转换](https://www.hangge.com/blog_uploads/201706/2017061720144376948.png)](https://www.hangge.com/blog/cache/detail_1718.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1718.html