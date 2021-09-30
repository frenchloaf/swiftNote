# Swift - 字符串的替换与过滤（附：过滤emoji表情符号）

2017-04-17发布：hangge阅读：5773

开发中常常会遇到需要将 **String** 字符串中的特殊符号给过滤掉，或者将某些子字符串替换成其他的内容，下面通过样例进行演示。

## 一、字符串替换

### 1，简单的替换

下面将字符串中所有的 **com** 替换成 **COM**。

[![原文:Swift - 字符串的替换与过滤（附：过滤emoji表情符号）](https://www.hangge.com/blog_uploads/201704/2017041114251592236.png)](https://www.hangge.com/blog/cache/detail_1647.html#)

```swift
//原始字符串
let str1 = "欢迎访问hangge.com.com.com"
//替换后的字符串
let str2 = str1.replacingOccurrences(of: "com", with: "COM")
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```



### 2，使用正则表达式替换

（1）为方便使用，我们这里对 **String** 做个扩展，增加正则替换相关方法。

```swift
import Foundation
 
extension String {
    //返回字数
    var count: Int {
        let string_NS = self as NSString
        return string_NS.length
    }
     
    //使用正则表达式替换
    func pregReplace(pattern: String, with: String,
                     options: NSRegularExpression.Options = []) -> String {
        let regex = try! NSRegularExpression(pattern: pattern, options: options)
        return regex.stringByReplacingMatches(in: self, options: [],
                                              range: NSMakeRange(0, self.count),
                                              withTemplate: with)
    }
}
```


（2）下面代码我们将字符串中所有的英文字母替换成下划线。

[![原文:Swift - 字符串的替换与过滤（附：过滤emoji表情符号）](https://www.hangge.com/blog_uploads/201704/2017041114431453624.png)](https://www.hangge.com/blog/cache/detail_1647.html#)

```swift
//原始字符串
let str1 = "欢迎访问hangge.com"
//替换后的字符串
let str2 = str1.pregReplace(pattern: "[a-zA-Z]", with: "_")
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```



## 二、字符串过滤

这个同样可以通过字符串替换的方法实现，即将需要过滤掉的字符串替换成空字符串。



### 1，简单的过滤

下面将字符串中所有的 **com** 过滤掉。

[![原文:Swift - 字符串的替换与过滤（附：过滤emoji表情符号）](https://www.hangge.com/blog_uploads/201704/2017041114473430850.png)](https://www.hangge.com/blog/cache/detail_1647.html#)

```swift
//原始字符串
let str1 = "欢迎访问hangge.com.com.com"
//替换后的字符串
let str2 = str1.replacingOccurrences(of: "com", with: "")
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```



### 2，使用正则表达式过滤 

（1）为方便使用，我们这里对 **String** 做个扩展，增加正则替换相关方法。

```swift
//具体方法见上方“正则表达式替换”部分
```


（2）下面代码将字符串中所有的表情符号给过滤掉。



[![原文:Swift - 字符串的替换与过滤（附：过滤emoji表情符号）](https://www.hangge.com/blog_uploads/201704/2017041114561083743.png)](https://www.hangge.com/blog/cache/detail_1647.html#)

```swift
//原始字符串
let str1 = "欢迎🆚访问💓😄hangge.com"
//判断表情的正则表达式
let pattern = "[\\ud83c\\udc00-\\ud83c\\udfff]|[\\ud83d\\udc00-\\ud83d\\udfff]|[\\u2600-\\u27ff]"
//替换后的字符串
let str2 = str1.pregReplace(pattern: pattern, with: "")
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1647.html