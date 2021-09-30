# Swift - 判断两个日期是不是同一天的几个办法

2016-11-30发布：hangge阅读：3241

在开发中，我们有时会需要比较两个日期时间（**Date**），判断它们是不是在同一天。下面演示几种判断方法。

[![原文:Swift - 判断两个日期是不是同一天的几个办法](https://www.hangge.com/blog_uploads/201611/2016110816102240826.png)](https://www.hangge.com/blog/cache/detail_1423.html#)

### 1，格式化成字符串比较

下面方法将两个日期格式化成只包含年月日的字符串，再比较两个字符串是否相等。

```swift
//初始化日期格式器
let dformatter = DateFormatter()
dformatter.dateFormat = "yyyyMMdd"
 
//开始比较
if dformatter.string(from: date1) == dformatter.string(from: date2) {
    print("它们是同一天")
}else {
    print("它们不是同一天")
}
```



### 2，取出日期的年、月、日部分,分别进行比较

```swift
let calendar = Calendar.current
let comp1 = calendar.dateComponents([.year,.month,.day], from: date1)
let comp2 = calendar.dateComponents([.year,.month,.day], from: date2)
 
//开始比较
if comp1.year == comp2.year && comp1.month == comp2.month && comp1.day == comp2.day {
    print("它们是同一天")
}else {
    print("它们不是同一天")
}
```



### 3，使用Calendar的isDate方法进行判断（推荐）

这个是 **Swift3** 新增的方法，使用方便，效率也最高。

```swift
if Calendar.current.isDate(date1, inSameDayAs: date2) {
    print("它们是同一天")
}else {
    print("它们不是同一天")
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1423.html