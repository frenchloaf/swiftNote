# Swift - 将字符串拆分成数组（把一个字符串分割成字符串数组）

2016-06-13发布：hangge阅读：10723

在Swift中，如果需要把一个字符串根据特定的分隔符拆分（split）成字符串数组，通常有如下两种方法：

**1，使用componentsSeparatedByString()方法**

```swift
let str = "北京、上海、深圳、香港"
print("原始字符串：\(str)")
 
let splitedArray = str.componentsSeparatedByString("、")
print("拆分后的数组：\(splitedArray)")
```

[![原文:Swift - 将字符串拆分成数组（把一个字符串分割成字符串数组）](https://www.hangge.com/blog_uploads/201605/2016052719411121692.png)](https://www.hangge.com/blog/cache/detail_1206.html#)



**2，使用characters.split()方法**

```swift
let str = "北京、上海、深圳、香港"
print("原始字符串：\(str)")
 
let splitedArray = str.characters.split{$0 == "、"}.map(String.init)
print("拆分后的数组：\(splitedArray)")
```