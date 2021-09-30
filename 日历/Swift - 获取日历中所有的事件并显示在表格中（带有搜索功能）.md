# Swift - 获取日历中所有的事件并显示在表格中（带有搜索功能）

2016-10-18发布：hangge阅读：1515

个人平时喜欢在系统的日历中使用事件来记录一些日常计划。由于有 **iCloud** 同步，这些事件可以自动同步到各个平台上，用起来很方便。

**iOS** 日历还带有搜索功能，可以搜索出一年内的所有事件。但有时侯我想找个很早之前事件（一年前），使用手机搜索就搜不出，而在列表里一个个往前手动翻又太麻烦。所以干脆自己写个 **App**，来加载并查找所有的日历事件。

**1，效果图**

（1）程序启动后，列表自动加载所有的本地日历事件（或 **iCloud** 日历事件），像生日日历、节假日日历、订阅日历等事件都排除。

（2）单元格左侧有个色块，即事件对应的日历分类颜色。

（3）单元格上方还有个搜索栏，可以过滤出标题里包含输入文字的所有事件。

​    [![原文:Swift - 获取日历中所有的事件并显示在表格中（带有搜索功能）](https://www.hangge.com/blog_uploads/201610/2016100616312098825.png)](https://www.hangge.com/blog/cache/detail_1385.html#)     [![原文:Swift - 获取日历中所有的事件并显示在表格中（带有搜索功能）](https://www.hangge.com/blog_uploads/201610/2016100616312665700.png)](https://www.hangge.com/blog/cache/detail_1385.html#)

**2，实现原理**

（1）关于获取日历事件我原来写过一篇文章：[Swift - 使用EventKit获取系统日历事件，添加事件](https://www.hangge.com/blog/cache/detail_644.html)

（2）由于 **EventKit** 没有获取所有日历事件这个方法，只能指定个开始时间，结束时间。然后查询这个时间范围内的事件。这里我就将范围设置成当前时间的前二十年、后二十年（共四十年，这样基本也足够了）



（3）但 **EventKit** 不能直接查询这么大范围的数据，否则返回的数据不全。所有我这里用个 **for** 循环，每次只查询一年的数据，最后再将结果合并起来。

**3，实现步骤**

（1）新建一个自定义单元格（**MyTableViewCell**），同时勾选“**Also create XIB file**”

[![原文:Swift - 获取日历中所有的事件并显示在表格中（带有搜索功能）](https://www.hangge.com/blog_uploads/201610/2016100616094785453.png)](https://www.hangge.com/blog/cache/detail_1385.html#)



（2）打开 **MyTableViewCell.xib**。在里面添加一个 **UIView** 和两个 **UILabel**（分别用来显示分类颜色、事件标题、以及时间），并设置好相关约束。

[![原文:Swift - 获取日历中所有的事件并显示在表格中（带有搜索功能）](https://www.hangge.com/blog_uploads/201610/2016100616125763310.png)](https://www.hangge.com/blog/cache/detail_1385.html#)



（3）**MyTableViewCell.swift** 中代码比较简单。就是和 **xib** 中的 **UI** 组件做个关联。

```swift
import UIKit
 
class MyTableViewCell: UITableViewCell {
 
    //左侧分类标识
    @IBOutlet weak var sideSign: UIView!
     
    //标题标签
    @IBOutlet weak var titleLabel: UILabel!
     
    //时间标签
    @IBOutlet weak var dateLabel: UILabel!
     
    override func awakeFromNib() {
        super.awakeFromNib()
    }
 
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
}
```


（4）主页代码（**ViewController.swift**）

```swift
import UIKit
import EventKit
 
class ViewController: UIViewController {
     
    let eventStore = EKEventStore()
     
    var tableView:UITableView?
     
    //日期格式器
    var dformatter:DateFormatter!
     
    //搜索控制器
    var searchController = UISearchController(searchResultsController: nil)
     
    //事件集合
    var events:[EKEvent] = []
     
    //搜索过滤后的结果集
    var filteredEvents:[EKEvent] = []
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化日期格式器
        dformatter = DateFormatter()
        dformatter.dateFormat = "yyyy年MM月dd日"
         
        //请求日历
        requestCalendars()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UINib(nibName:"MyTableViewCell", bundle:nil),
                                    forCellReuseIdentifier:"myCell")
        self.view.addSubview(self.tableView!)
         
        //配置搜索控制器
        searchController.searchResultsUpdater = self
        definesPresentationContext = true
        searchController.dimsBackgroundDuringPresentation = false
        //搜索状态下隐藏顶部导航栏
        searchController.hidesNavigationBarDuringPresentation = true
         
        tableView!.tableHeaderView = searchController.searchBar
    }
     
    //请求日历
    func requestCalendars() {
        // 请求日历事件
        eventStore.requestAccess(to: .event, completion: {
            granted, error in
            if (granted) && (error == nil) {
                print("granted \(granted)")
                print("error  \(error)")
                 
                //获取本地日历（剔除节假日，生日等其他系统日历）
                let calendars = self.eventStore.calendars(for: .event).filter({
                    (calender) -> Bool in
                    return calender.type == .local || calender.type == .calDAV
                })
                 
                self.events = []
                //获取当前年
                let com = Calendar.current.dateComponents([.year], from: Date())
                let currentYear = com.year!
                print(currentYear)
                 
                // 获取所有的事件（前后20年）
                for i in -20...20 {
                    let startDate = self.startOfMonth(year:currentYear+i, month:1)
                    let endDate = self.endOfMonth(year:currentYear+i, month:12,
                                                  returnEndTime:true)
                    print("查询范围 开始:\(startDate) 结束:\(endDate)")
                     
                    let predicate2 = self.eventStore.predicateForEvents(
                        withStart: startDate, end: endDate, calendars: calendars)
                     
                    if let eV = self.eventStore.events(matching: predicate2) as [EKEvent]!
                    {
                        //将获取到的日历事件添加到集合中
                        self.events.append(contentsOf: eV)
                    }
                }
                 
                print("事件加载完毕!共\(self.events.count)条数据。")
                //重新刷新表格数据
                DispatchQueue.main.async {
                    self.tableView?.reloadData()
                }
            }
        })
    }
     
    //指定年月的开始日期
    func startOfMonth(year: Int, month: Int) -> Date {
        let calendar = Calendar.current
        var startComps = DateComponents()
        startComps.day = 1
        startComps.month = month
        startComps.year = year
        let startDate = calendar.date(from: startComps)!
        return startDate
    }
     
    //指定年月的结束日期
    func endOfMonth(year: Int, month: Int, returnEndTime:Bool = false) -> Date {
        let calendar = Calendar.current
        var components = DateComponents()
        components.month = 1
        if returnEndTime {
            components.second = -1
        } else {
            components.day = -1
        }
        let tem = startOfMonth(year: year, month:month)
        let endOfYear =  calendar.date(byAdding: components, to: tem)!
        return endOfYear
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension ViewController: UITableViewDelegate, UITableViewDataSource{
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1;
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if (self.searchController.isActive) {
            return self.filteredEvents.count
        } else {
            return self.events.count
        }
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell
    {
        let cell:MyTableViewCell = tableView.dequeueReusableCell(withIdentifier: "myCell")
            as! MyTableViewCell
        var event:EKEvent!
        if self.searchController.isActive {
            event = self.filteredEvents[indexPath.row]
        }else{
            event = self.events[indexPath.row]
        }
        cell.titleLabel.text = event.title
        cell.dateLabel.text = dformatter.string(from: event.startDate)
        cell.sideSign.backgroundColor = UIColor(cgColor:event.calendar.cgColor)
        return cell
    }
}
 
extension ViewController: UISearchResultsUpdating{
    //搜索栏文字改变后事件
    func updateSearchResults(for searchController: UISearchController) {
        let searchText = searchController.searchBar.text!
        filteredEvents = events.filter({( event : EKEvent) -> Bool in
            return  event.title.lowercased().contains(searchText.lowercased())
        })
        tableView?.reloadData()
    }
}

```

**4，源码下载**

![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1221.zip](https://www.hangge.com/blog_uploads/201610/2016100819563070845.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1385.html