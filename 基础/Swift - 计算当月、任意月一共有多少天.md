# Swift - 计算当月、任意月一共有多少天

2016-06-06发布：hangge阅读：2869

**1，获取当前月天数**

```swift
//计算当月天数
func getDaysInCurrentMonth() -> Int {
    let calendar = NSCalendar.currentCalendar()
     
    let date = NSDate()
    let nowComps = calendar.components([.Year, .Month, .Day], fromDate: date)
    let year =  nowComps.year
    let month = nowComps.month
     
    let startComps = NSDateComponents()
    startComps.day = 1
    startComps.month = month
    startComps.year = year
     
    let endComps = NSDateComponents()
    endComps.day = 1
    endComps.month = month == 12 ? 1 : month + 1
    endComps.year = month == 12 ? year + 1 : year
     
    let startDate = calendar.dateFromComponents(startComps)!
    let endDate = calendar.dateFromComponents(endComps)!
     
    let diff = calendar.components(.Day, fromDate: startDate, toDate: endDate,
                                   options: .MatchFirst)
    return diff.day
}

```

测试代码：

```swift
let days = getDaysInCurrentMonth()
print("本月有\(days)天")
```

[![原文:Swift - 计算当月、任意月一共有多少天](https://www.hangge.com/blog_uploads/201606/2016060610043037364.png)](https://www.hangge.com/blog/cache/detail_1222.html#)

**2，获取指定年月的天数**

```swift
//计算指定月天数
func getDaysInMonth( year: Int, month: Int) -> Int
{
    let calendar = NSCalendar.currentCalendar()
     
    let startComps = NSDateComponents()
    startComps.day = 1
    startComps.month = month
    startComps.year = year
     
    let endComps = NSDateComponents()
    endComps.day = 1
    endComps.month = month == 12 ? 1 : month + 1
    endComps.year = month == 12 ? year + 1 : year
     
    let startDate = calendar.dateFromComponents(startComps)!
    let endDate = calendar.dateFromComponents(endComps)!
     
    let diff = calendar.components(.Day, fromDate: startDate, toDate: endDate,
                                   options: .MatchFirst)
    return diff.day
}
```

测试代码：

```swift
let days = getDaysInMonth(2016, month: 2)
print("2016年2月有\(days)天")
```

[![原文:Swift - 计算当月、任意月一共有多少天](https://www.hangge.com/blog_uploads/201606/201606061008587516.png)](https://www.hangge.com/blog/cache/detail_1222.html#)