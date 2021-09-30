# Swift - 获取（当月、当年、任意月|年）的第一天、最后一天日期

2016-06-07发布：hangge阅读：5477

（本文代码已升级至Swift4）



### **1，得到当前月第一天、最后一天日期**

```swift
//本月开始日期
func startOfCurrentMonth() -> Date {
    let date = Date()
    let calendar = NSCalendar.current
    let components = calendar.dateComponents(
        Set<Calendar.Component>([.year, .month]), from: date)
    let startOfMonth = calendar.date(from: components)!
    return startOfMonth
}
 
//本月结束日期
func endOfCurrentMonth(returnEndTime:Bool = false) -> Date {
    let calendar = NSCalendar.current
    var components = DateComponents()
    components.month = 1
    if returnEndTime {
        components.second = -1
    } else {
        components.day = -1
    }
     
    let endOfMonth =  calendar.date(byAdding: components, to: startOfCurrentMonth())!
    return endOfMonth
}
```

测试代码：

```swift
//创建一个日期格式器
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
 
//当月第一天日期
let startDate = startOfCurrentMonth()
print("本月开始时间：", dateFormatter.string(from: startDate as Date))
 
//当月最后一天日期1
let endDate1 = endOfCurrentMonth()
print("本月结束时间1：", dateFormatter.string(from: endDate1))
 
//当月最后一天日期2
let endDate2 = endOfCurrentMonth(returnEndTime: true)
print("本月结束时间2：", dateFormatter.string(from: endDate2))
```

[![原文:Swift - 获取（当月、当年、任意月|年）的第一天、最后一天日期](https://www.hangge.com/blog_uploads/201606/201606061244076652.png)](https://www.hangge.com/blog/cache/detail_1223.html#)


**
**

### **2，得到当年第一天、最后一天日期**

```swift
//本年开始日期
func startOfCurrentYear() -> Date {
    let date = Date()
    let calendar = NSCalendar.current
    let components = calendar.dateComponents(Set<Calendar.Component>([.year]), from: date)
    let startOfYear = calendar.date(from: components)!
    return startOfYear
}
 
//本年结束日期
func endOfCurrentYear(returnEndTime:Bool = false) -> Date {
    let calendar = NSCalendar.current
    var components = DateComponents()
    components.year = 1
    if returnEndTime {
        components.second = -1
    } else {
        components.day = -1
    }
     
    let endOfYear = calendar.date(byAdding: components, to: startOfCurrentYear())!
    return endOfYear
}
```

测试代码：

```swift
//创建一个日期格式器
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
 
//本年度第一天日期
let startDate = startOfCurrentYear()
print("本年度开始时间：", dateFormatter.string(from: startDate))
 
//本年度最后一天日期1
let endDate1 = endOfCurrentYear()
print("本年度结束时间1：", dateFormatter.string(from: endDate1))
 
//本年度最后一天日期2
let endDate2 = endOfCurrentYear(returnEndTime: true)
print("本年度结束时间2：", dateFormatter.string(from: endDate2))
```

[![原文:Swift - 获取（当月、当年、任意月|年）的第一天、最后一天日期](https://www.hangge.com/blog_uploads/201606/2016060612452269736.png)](https://www.hangge.com/blog/cache/detail_1223.html#)





### **3，得到指定年月的第一天、最后一天日期**

```swift
//指定年月的开始日期
func startOfMonth(year: Int, month: Int) -> Date {
    let calendar = NSCalendar.current
    var startComps = DateComponents()
    startComps.day = 1
    startComps.month = month
    startComps.year = year
    let startDate = calendar.date(from: startComps)!
    return startDate
}
 
//指定年月的结束日期
func endOfMonth(year: Int, month: Int, returnEndTime:Bool = false) -> Date {
    let calendar = NSCalendar.current
    var components = DateComponents()
    components.month = 1
    if returnEndTime {
        components.second = -1
    } else {
        components.day = -1
    }
     
    let endOfYear = calendar.date(byAdding: components,
                                  to: startOfMonth(year: year, month:month))!
    return endOfYear
}
```

测试代码：

```swift
//创建一个日期格式器
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
 
//当月第一天日期
let startDate = startOfMonth(year: 2016, month: 8)
print("2016年8月的开始时间：", dateFormatter.string(from: startDate))
 
//当月最后一天日期1
let endDate1 = endOfMonth(year: 2016, month: 8)
print("2016年8月的结束时间1：", dateFormatter.string(from: endDate1))
 
//当月最后一天日期2
let endDate2 = endOfMonth(year: 2016, month: 8, returnEndTime: true)
print("2016年8月的结束时间2：", dateFormatter.string(from: endDate2))
```

[![原文:Swift - 获取（当月、当年、任意月|年）的第一天、最后一天日期](https://www.hangge.com/blog_uploads/201606/2016060612300843515.png)](https://www.hangge.com/blog/cache/detail_1223.html#)