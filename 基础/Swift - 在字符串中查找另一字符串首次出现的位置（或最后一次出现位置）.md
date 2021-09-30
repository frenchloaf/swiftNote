# Swift - 在字符串中查找另一字符串首次出现的位置（或最后一次出现位置）

2017-03-31发布：hangge阅读：7259

（本文代码已升级至Swift4）

我们知道 **JavaScript** 中有个 **indexOf()** 方法可返回某个指定的字符串值在字符串中首次出现的位置， 而 **Swift** 中却没有提供类似的方法。我们可以通过 **String** 的 **range** 方法来实现一个相同的功能。

### 1，扩展String

这里对 **String** 类做个扩展，新增个 **positionOf** 方法。用来获取内部子字符串第一次、或最后一次出现的位置索引。（如果内部不存在该字符串则返回 **-1**。）

```swift
extension String {
    //返回第一次出现的指定子字符串在此字符串中的索引
    //（如果backwards参数设置为true，则返回最后出现的位置）
    func positionOf(sub:String, backwards:Bool = false)->Int {
        var pos = -1
        if let range = range(of:sub, options: backwards ? .backwards : .literal ) {
            if !range.isEmpty {
                pos = self.distance(from:startIndex, to:range.lowerBound)
            }
        }
        return pos
    }
}
```



### 2，使用样例

```swift
let str1 = "欢迎访问hangge.com。hangge.com做最好的开发者知识平台"
let str2 = "hangge"
print("父字符串：\(str1)")
print("子字符串：\(str2)")
 
let position1 = str1.positionOf(sub: str2)
print("子字符串第一次出现的位置是：\(position1)")
let position2 = str1.positionOf(sub: str2, backwards: true)
print("子字符串最后一次出现的位置是：\(position2)")
```

[![原文:Swift - 在字符串中查找另一字符串首次出现的位置（或最后一次出现位置）](https://www.hangge.com/blog_uploads/201802/2018020717283022408.png)](https://www.hangge.com/blog/cache/detail_1584.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1584.html