# Swift - 获取当前时间的时间戳（时间戳与时间互相转换）

2016-05-25发布：hangge阅读：32296

（本文代码已升级至Swift3）

**1，时间戳** 

时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。



**2，获取当前时间的时间戳**

[![原文:Swift - 获取当前时间的时间戳（时间戳与时间互相转换）](https://www.hangge.com/blog_uploads/201605/2016051914064819142.png)](https://www.hangge.com/blog/cache/detail_1198.html#)

```swift
//获取当前时间
let now = Date()
 
// 创建一个日期格式器
let dformatter = DateFormatter()
dformatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
print("当前日期时间：\(dformatter.string(from: now))")
 
//当前时间的时间戳
let timeInterval:TimeInterval = now.timeIntervalSince1970
let timeStamp = Int(timeInterval)
print("当前时间的时间戳：\(timeStamp)")
```



**3，将时间戳转为日期时间**

**[![原文:Swift - 获取当前时间的时间戳（时间戳与时间互相转换）](https://www.hangge.com/blog_uploads/201605/2016051914191652138.png)](https://www.hangge.com/blog/cache/detail_1198.html#)**

```swift
//时间戳
let timeStamp = 1463637809
print("时间戳：\(timeStamp)")
 
//转换为时间
let timeInterval:TimeInterval = TimeInterval(timeStamp)
let date = Date(timeIntervalSince1970: timeInterval)
 
//格式话输出
let dformatter = DateFormatter()
dformatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
print("对应的日期时间：\(dformatter.string(from: date))")
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1198.html