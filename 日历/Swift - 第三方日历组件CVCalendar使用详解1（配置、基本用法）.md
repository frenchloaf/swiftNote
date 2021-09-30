# Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）

2017-01-02发布：hangge阅读：5961

**相关文章系列：**
Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）[当前文章]
[Swift - 第三方日历组件CVCalendar使用详解2（高级功能、样式）](https://www.hangge.com/blog/cache/detail_1505.html)


**CVCalendar** 是一款超好用的第三方日历组件，不仅功能强大，而且可以方便地进行样式自定义。同时，**CVCalendar** 还提供月视图、周视图两种展示模式，我们可以根据需求自由选择使用。

## 一、安装配置

（1）从 **GitHub** 上下载最新的代码：https://github.com/Mozharovsky/CVCalendar
（2）将下载下来的源码包中 **CVCalendar.xcodeproj** 拖拽至你的工程中 

[![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/2016122920330510725.png)](https://www.hangge.com/blog/cache/detail_1504.html#)

（3）**工程** -> **General** -> **Embedded Binaries** 项，把 **iOS** 版的 **framework** 添加进来：**CVCalendar.framework**

[![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/2016123009344894997.png)](https://www.hangge.com/blog/cache/detail_1504.html#)

（4）最后，在需要使用 **CVCalendar** 的地方 **import** 进来就可以了

```swift
import CVCalendar
```



## 二、基本用法

### 1，月视图使用样例 

（1）效果图

- 初始化的时候自动显示当月日历，且“**今天**”的日期文字是红色的。
- 顶部导航栏标题显示当前日历的年、月信息，日历左右滑动切换的时候，标题内容也会随之改变。
- 点击导航栏右侧的“**今天**”按钮，日历又会跳回到当前日期。
- 点击日历上的任一日期时间后，该日期背景色会变蓝色（如果是今天则变红色）。同时我们在日期选择响应中，将选择的日期弹出显示。

  [![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/2016123009532223883.png)](https://www.hangge.com/blog/cache/detail_1504.html#)  [![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/2016123009533079247.png)](https://www.hangge.com/blog/cache/detail_1504.html#)  [![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/201612300953376435.png)](https://www.hangge.com/blog/cache/detail_1504.html#)

（2）样例代码

- 日历组件分为：**CVCalendarMenuView** 和 **CVCalendarView** 两部分。前者是显示星期的菜单栏，后者是日期表格视图。这二者的位置和大小我们可以随意调整设置。
- 组件提供了许多代理协议让我进行样式调整或功能响应，我们可以选择使用。但其中 **CVCalendarViewDelegate**, **CVCalendarMenuViewDelegate** 这两个协议是必须的。

```swift
import UIKit
import CVCalendar
 
class ViewController: UIViewController {
    //星期菜单栏
    private var menuView: CVCalendarMenuView!
     
    //日历主视图
    private var calendarView: CVCalendarView!
     
    var currentCalendar: Calendar!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        currentCalendar = Calendar.init(identifier: .gregorian)
         
        //初始化的时候导航栏显示当年当月
        self.title = CVDate(date: Date(), calendar: currentCalendar).globalDescription
         
        //初始化星期菜单栏
        self.menuView = CVCalendarMenuView(frame: CGRect(x:0, y:80, width:300, height:15))
         
        //初始化日历主视图
        self.calendarView = CVCalendarView(frame: CGRect(x:0, y:110, width:300,
                                                         height:450))
         
        //星期菜单栏代理
        self.menuView.menuViewDelegate = self
         
        //日历代理
        self.calendarView.calendarDelegate = self
         
        //将菜单视图和日历视图添加到主视图上
        self.view.addSubview(menuView)
        self.view.addSubview(calendarView)
    }
     
    //今天按钮点击
    @IBAction func todayButtonTapped(_ sender: AnyObject) {
        let today = Date()
        self.calendarView.toggleViewWithDate(today)
    }
     
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
         
        //更新日历frame
        self.menuView.commitMenuViewUpdate()
        self.calendarView.commitCalendarViewUpdate()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //视图模式
    func presentationMode() -> CalendarMode {
        //使用月视图
        return .monthView
    }
     
    //每周的第一天
    func firstWeekday() -> Weekday {
        //从星期一开始
        return .monday
    }
     
    func presentedDateUpdated(_ date: CVDate) {
        //导航栏显示当前日历的年月
        self.title = date.globalDescription
    }
     
    //每个日期上面是否添加横线(连在一起就形成每行的分隔线)
    func topMarker(shouldDisplayOnDayView dayView: CVCalendarDayView) -> Bool {
        return true
    }
     
    //切换月的时候日历是否自动选择某一天（本月为今天，其它月为第一天）
    func shouldAutoSelectDayOnMonthChange() -> Bool {
        return false
    }
     
    //日期选择响应
    func didSelectDayView(_ dayView: CVCalendarDayView, animationDidFinish: Bool) {
        //获取日期
        let date = dayView.date.convertedDate()!
        // 创建一个日期格式器
        let dformatter = DateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日"
        let message = "当前选择的日期是：\(dformatter.string(from: date))"
        //将选择的日期弹出显示
        let alertController = UIAlertController(title: "", message: message,
                                                preferredStyle: .alert)
        let okAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1504.zip](https://www.hangge.com/blog_uploads/201612/2016123010010845853.zip)

### 2，周视图使用样例

同月视图模式相比，周视图日历区域只有一行（每次显示7天日期）。其它方面和月视图相比差别不大，也都是左右滑动切换显示下一周、下一周日期。

[![原文:Swift - 第三方日历组件CVCalendar使用详解1（配置、基本用法）](https://www.hangge.com/blog_uploads/201612/2016123020002796493.png)](https://www.hangge.com/blog/cache/detail_1504.html#)

```swift
import UIKit
import CVCalendar
 
class ViewController: UIViewController {
    //星期菜单栏
    private var menuView: CVCalendarMenuView!
     
    //日历主视图
    private var calendarView: CVCalendarView!
     
    var currentCalendar: Calendar!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        currentCalendar = Calendar.init(identifier: .gregorian)
         
        //初始化的时候导航栏显示当年当月
        self.title = CVDate(date: Date(), calendar: currentCalendar).globalDescription
         
        //初始化星期菜单栏
        self.menuView = CVCalendarMenuView(frame: CGRect(x:0, y:80, width:300, height:15))
         
        //初始化日历主视图
        self.calendarView = CVCalendarView(frame: CGRect(x:0, y:110, width:300,
                                                         height:50))
         
        //星期菜单栏代理
        self.menuView.menuViewDelegate = self
         
        //日历代理
        self.calendarView.calendarDelegate = self
         
        //将菜单视图和日历视图添加到主视图上
        self.view.addSubview(menuView)
        self.view.addSubview(calendarView)
    }
     
    //今天按钮点击
    @IBAction func todayButtonTapped(_ sender: AnyObject) {
        let today = Date()
        self.calendarView.toggleViewWithDate(today)
    }
     
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
         
        //更新日历frame
        self.menuView.commitMenuViewUpdate()
        self.calendarView.commitCalendarViewUpdate()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension ViewController: CVCalendarViewDelegate,CVCalendarMenuViewDelegate {
    //视图模式
    func presentationMode() -> CalendarMode {
        //使用周视图
        return .weekView
    }
     
    //每周的第一天
    func firstWeekday() -> Weekday {
        //从星期一开始
        return .monday
    }
     
    func presentedDateUpdated(_ date: CVDate) {
        //导航栏显示当前日历的年月
        self.title = date.globalDescription
    }
     
    //每个日期上面是否添加横线(连在一起就形成每行的分隔线)
    func topMarker(shouldDisplayOnDayView dayView: CVCalendarDayView) -> Bool {
        return true
    }
     
    //切换周的时候日历是否自动选择某一天（本周为今天，其它周为第一天）
    func shouldAutoSelectDayOnWeekChange() -> Bool {
        return false
    }
     
    //日期选择响应
    func didSelectDayView(_ dayView: CVCalendarDayView, animationDidFinish: Bool) {
        //获取日期
        let date = dayView.date.convertedDate(calendar: currentCalendar)!
        // 创建一个日期格式器
        let dformatter = DateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日"
        let message = "当前选择的日期是：\(dformatter.string(from: date))"
        //将选择的日期弹出显示
        let alertController = UIAlertController(title: "", message: message,
                                                preferredStyle: .alert)
        let okAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1504.html