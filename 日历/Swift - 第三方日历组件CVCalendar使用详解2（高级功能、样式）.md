# Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）

2017-01-04发布：hangge阅读：5152

**相关文章系列：**
[Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog/cache/detail_1504.html)
Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）[当前文章]


**CVCalendar** 是一款超好用的第三方日历组件，不仅功能强大，而且可以方便地进行样式自定义。前文我演示了 **CVCalendar** 一些基本用法，本文接着介绍组件的各种 **API** 功能，以及如何实现样式的修改。

## 三、高级用法

### 1，日历翻页时的响应函数

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    func didShowPreviousMonthView(_ date: Date) {
        print("翻到了上一个月!")
    }
     
    func didShowNextMonthView(_ date: Date) {
        print("翻到了下一个月!")
    }
}
```

### 2，判断选择的日期是否是今天

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123010131892547.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //日期选择响应
    func didSelectDayView(_ dayView: CVCalendarDayView, animationDidFinish: Bool) {
        let message = dayView.isCurrentDay ? "是" : "否"
        //将选择的日期弹出显示
        let alertController = UIAlertController(title: "选择的日期是否是今天：",
                                                message: message, preferredStyle: .alert)
        let okAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
}
```



### 3，时间段选择

- 通过 **shouldSelectRange()** 方法我们可以设置日历是单选还是允许选择一个时间段。
- 如果是选择时间段的话，第一次点击选择的是开始时间，第二次点击选择的是结束时间。

（1）下面样例在时间段选择完毕后打印出开始时间和结束时间。

  [![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123011143817067.png)](https://www.hangge.com/blog/cache/detail_1505.html#)   [![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123011144539113.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //是否允许选择一个时间段
    func shouldSelectRange() -> Bool {
        return true
    }
     
    //时间段选择完毕
    func didSelectRange(from startDayView: DayView, to endDayView: DayView) {
        // 创建一个日期格式器
        let dformatter = DateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日"
        print("开始时间：\(dformatter.string(from: startDayView.date.convertedDate()!))")
        print("结束时间：\(dformatter.string(from: endDayView.date.convertedDate()!))")
    }
}
```


（2）我们还可以指定一个时间段最多可跨越的天数。下面样例最多选择一个5天的时间段。

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    func maxSelectableRange() -> Int {
        return 5
    }
}
```



### 4，不只显示当前选择月的日期

- 默认情况下，日历翻到哪个月就只显示这个月的所有日期。多余的位置留白。
- 我们可以通过 **shouldShowWeekdaysOut()** 方法返回 **true**。在多出来的空位上补上上个月或下个月的日期，文字颜色是灰色。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123011335265594.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //是否显示非当月的日期
    func shouldShowWeekdaysOut() -> Bool {
        return true
    }
}
```



### 5，初始化时自动标注出某些日期

（1）比如下面样例，我们将2016年12月的头三天用灰色圆形背景标注出来。 

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123010290579096.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //当前日期View是不是需要初始标注
    func preliminaryView(shouldDisplayOnDayView dayView: DayView) -> Bool {
        if !dayView.isHidden && dayView.date != nil {
            //获取该日期视图的年月日
            let year = dayView.date.year
            let month = dayView.date.month
            let day = dayView.date.day
            //判断日期是否符合要求
            if year == 2016 && month == 12 && day >= 1 && day <= 3 {
                return true
            }
        }
        return false
    }
     
    //注日期使用的样式
    func preliminaryView(viewOnDayView dayView: DayView) -> UIView {
        //灰色圆形背景
        let circleView = CVAuxiliaryView(dayView: dayView, rect: dayView.frame,
                                         shape: CVShape.circle)
        circleView.fillColor = .colorFromCode(0xDDDDDD)
        return circleView
    }
}
```

（2）标注日期背景的形状可以修改的。比如下面每天我都使用不同的形状背景。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017595719094.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //当前日期View是不是需要初始标注
    func preliminaryView(shouldDisplayOnDayView dayView: DayView) -> Bool {
        if !dayView.isHidden && dayView.date != nil {
            //获取该日期视图的年月日
            let year = dayView.date.year
            let month = dayView.date.month
            let day = dayView.date.day
            //判断日期是否符合要求
            if year == 2016 && month == 12 && day >= 1 && day <= 3 {
                return true
            }
        }
        return false
    }
     
    //注日期使用的样式
    func preliminaryView(viewOnDayView dayView: DayView) -> UIView {
        var view:CVAuxiliaryView
        switch dayView.date.day {
        case 1:
            //左半圆背景
            view = CVAuxiliaryView(dayView: dayView, rect: dayView.frame,
                                   shape: CVShape.rightFlag)
        case 3:
            //右半圆背景
            view = CVAuxiliaryView(dayView: dayView, rect: dayView.frame,
                                   shape: CVShape.leftFlag)
        default:
            //矩形背景
            view = CVAuxiliaryView(dayView: dayView, rect: dayView.frame,
                                   shape: CVShape.rect)
        }
        view.fillColor = .colorFromCode(0xDDDDDD)
        return view
    }
     
    //日期视图间的间距
    func spaceBetweenDayViews() -> CGFloat {
        //消除
        return 0
    }
}
```



### 6，在日期上添加标记点 

（1）下面样例，我们在12月1、2、3号日期上分别添加1~3个标记小圆点。

- 每个圆点颜色可自由定义。
- 每个日期上最多添加 **3** 个圆点。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123014303834783.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
    //该日期视图是否有标记点
    func dotMarker(shouldShowOnDayView dayView: DayView) -> Bool {
        if !dayView.isHidden && dayView.date != nil {
            //获取该日期视图的年月日
            let year = dayView.date.year
            let month = dayView.date.month
            let day = dayView.date.day
            //判断日期是否符合要求
            if year == 2016 && month == 12 && day >= 1 && day <= 3 {
                return true
            }
        }
        return false
    }
     
    //返回相应日期视图上各个标记点的颜色数组(每个日期最多3个点)
    func dotMarker(colorOnDayView dayView: DayView) -> [UIColor] {
        switch dayView.date.day {
        case 1:
            return [UIColor.orange]
        case 2:
            return [UIColor.orange, UIColor.green]
        default:
            return [UIColor.orange, UIColor.green, UIColor.blue]
        }
    }
}
```


（2）我们还可以修改标记点的偏移量和尺寸大小

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123014420890837.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //标记点的偏移量
    func dotMarker(moveOffsetOnDayView dayView: DayView) -> CGFloat {
        return 25
    }
     
    //标记点的尺寸大小
    func dotMarker(sizeOnDayView dayView: DayView) -> CGFloat {
        return 1
    }
}
```


（3）有时我们的标记点并不是一开始就确定的，可能要去请求网络数据，然后根据返回结果来显示标记点。那么在请求完毕后，我们可以手动调用如下方法，日历会自动更新相关的标记点。



```swift
//刷新日历视图（自动调用shouldShowOnDayView方法重新加载标记点）
self.calendarView.contentController.refreshPresentedMonth()
```



### 7，禁止滚动到某日期前的时间

- 默认情况下左右滑动即可自由切换显示上一个月或下一个月日历。
- 我们可以通过 **disableScrollingBeforeDate()** 方法返回一个时间，便无法滚动到该时间之前的日历。

下面样例，我们只能查看本月或者后面的日历，之前的日历无法查看：

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //无法滚动到指定日期前的日历
    func disableScrollingBeforeDate() -> Date {
        return Date()
    }
}
```



### 8，设置可选时间范围

下面样例设置一时间范围（前天到今天），只有该范围内的日期可以点击选择，在该时间范围外的日期点击无效，且文字为灰色。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/201612301639092365.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //最早的可选时间（前天）
    func earliestSelectableDate() -> Date {
        var dayComponents = DateComponents()
        dayComponents.day = -3
        let lastDate = currentCalendar.date(byAdding: dayComponents, to: Date())
        return lastDate!
    }
     
    //最迟的可选时间（今天）
    func latestSelectableDate() -> Date {
         return Date()
    }
}
```



## 四、星期栏样式自定义

### 1，统一修改星期栏文字颜色和背景颜色

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017542955249.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //星期栏文字颜色
    func dayOfWeekTextColor() -> UIColor {
        return UIColor.white
    }
     
    //星期栏背景颜色
    func dayOfWeekBackGroundColor() -> UIColor {
        return UIColor.orange
    }
}
```



### 2，单独修改星期栏文字

下面将周末两天文字改成红色，其他时间为黑色。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017145973286.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //星期文字颜色设置
    func dayOfWeekTextColor(by weekday: Weekday) -> UIColor {
        return weekday == .sunday || weekday == .saturday ?
            UIColor(red: 1.0, green: 0, blue: 0, alpha: 1.0) : UIColor.black
    }
}
```



### 3，修改星期栏文字显示类型 

（1）**normal**：正常模式

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017344659364.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //星期栏文字显示类型
    func weekdaySymbolType() -> WeekdaySymbolType {
        //正常
        return .normal
    }
}
```


（2）**short**：缩写模式（默认）

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017371072688.png)](https://www.hangge.com/blog/cache/detail_1505.html#)



（3）**veryShort**：极简模式

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123017384161633.png)](https://www.hangge.com/blog/cache/detail_1505.html#)



## 五、日历样式自定义

要修改日历的默认样式，首先我们要设置相关的样式代理：

```swift
//日历样式代理
self.calendarView.calendarAppearanceDelegate = self
```

接下来我们对默认样式做个修改：

- 加大文字大小
- 选中的日期：文字为白色，背景为紫色
- 周日的日期：文字为红色。选中的话背景变红色，文字变白色。

[![原文:Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog_uploads/201612/2016123019213228146.png)](https://www.hangge.com/blog/cache/detail_1505.html#)

```swift
extension ViewController: CVCalendarViewAppearanceDelegate {
     
    struct Color {
        static let selectionBackground = UIColor(red: 0.2, green: 0.2, blue: 1.0,
                                                 alpha: 1.0)
        static let sundayText = UIColor(red: 1.0, green: 0.2, blue: 0.2, alpha: 1.0)
        static let sundayTextDisabled = UIColor(red: 1.0, green: 0.6, blue: 0.6,
                                                alpha: 1.0)
        static let sundaySelectionBackground = sundayText
    }
     
    //文字字体设置
    func dayLabelFont(by weekDay: Weekday, status: CVStatus, present: CVPresent)
        -> UIFont {
        return UIFont.systemFont(ofSize: 22)
    }
     
    //文字颜色设置
    func dayLabelColor(by weekDay: Weekday, status: CVStatus, present: CVPresent)
        -> UIColor? {
        switch (weekDay, status, present) {
        case (_, .selected, _), (_, .highlighted, _): return UIColor.white
        case (.sunday, .in, _): return Color.sundayText
        case (.sunday, _, _): return Color.sundayTextDisabled
        case (_, .in, _): return UIColor.black
        default: return UIColor.gray
        }
    }
     
    //文字背景色设置
    func dayLabelBackgroundColor(by weekDay: Weekday, status: CVStatus,
                                 present: CVPresent) -> UIColor? {
        switch (weekDay, status, present) {
        case (.sunday, .selected, _),
             (.sunday, .highlighted, _): return Color.sundaySelectionBackground
        case (_, .selected, _),
             (_, .highlighted, _): return Color.selectionBackground
        default: return nil
        }
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1505.html