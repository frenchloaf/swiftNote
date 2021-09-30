# Swift - 扩展String增加字符串截取方法subString(start:, length:)

2017-04-03发布：hangge阅读：3972

（本文代码已升级至Swift4）

在 **Swift** 开发时，通过 **String** 的 **substring** 可以实现字符串的截取，不过由于其参数类型是 **String.Index** 或 **Range**，有时使用起来会比较麻烦。 



### 1，扩展String

这里对 **String** 进行扩展，新增一个 **subString** 方法。直接可以根据起始位置（**Int** 类型）和需要的长度（**Int** 类型），来截取出子字符串。

```swift
extension String {
    //根据开始位置和长度截取字符串
    func subString(start:Int, length:Int = -1) -> String {
        var len = length
        if len == -1 {
            len = self.count - start
        }
        let st = self.index(startIndex, offsetBy:start)
        let en = self.index(st, offsetBy:len)
        return String(self[st ..< en])
    }
}
```



### 2，使用样例

```swift
let str1 = "欢迎访问hangge.com"
let str2 = str1.subString(start: 4, length: 6)
print("原字符串：\(str1)")
print("截取出的字符串：\(str2)")
```

运行结果如下：

[![原文:Swift - 扩展String增加字符串截取方法subString(start:, length:)](https://www.hangge.com/blog_uploads/201703/2017030717204015671.png)](https://www.hangge.com/blog/cache/detail_1585.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1585.html