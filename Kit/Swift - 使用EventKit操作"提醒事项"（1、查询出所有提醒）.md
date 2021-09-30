# Swift - 使用EventKit操作"提醒事项"（1、查询出所有提醒）

2016-08-01发布：hangge阅读：1325

（本文代码已升级至Swift3）

我原来写过一篇文章，介绍如何对系统日历事件（**Event**）进行操作（原文地址：[Swift - 使用EventKit获取系统日历事件，添加事件](https://www.hangge.com/blog/cache/detail_644.html)）    

接下来演示如何使用 **EventKit** 对系统里的提醒事项（**Reminder**）进行操作，本文先介绍如何获取系统里所有的提醒。

**1，效果图**

程序启动后会把所有的提醒事项加载出来，并显示在表格中。（第一次启动会需要访问授权）

   [![原文:Swift - 使用EventKit操作"提醒事项"（1、查询出所有提醒）](https://www.hangge.com/blog_uploads/201607/2016073110310547174.png)](https://www.hangge.com/blog/cache/detail_1311.html#)   [![原文:Swift - 使用EventKit操作"提醒事项"（1、查询出所有提醒）](https://www.hangge.com/blog_uploads/201607/2016073110311076419.png)](https://www.hangge.com/blog/cache/detail_1311.html#)

**2，配置info.plist**

从 iOS10 起，苹果增强对用户的安全和隐私的保护。在申请很多私有权限的时候都需要添加描述。我们这里需要访问提醒事项，那就需要在 info.plist 添加相关的描述字段。

[![原文:Swift - 使用EventKit操作"提醒事项"（1、查询出所有提醒）](https://www.hangge.com/blog_uploads/201704/2017041320231883214.png)](https://www.hangge.com/blog/cache/detail_1311.html#)



**3，样例代码**

```swift
import UIKit
import EventKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var eventStore: EKEventStore!
    var reminders: [EKReminder]!
    var tableView:UITableView?
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        self.view.addSubview(self.tableView!)
         
        // 在取得提醒之前，需要先获取授权
        self.eventStore = EKEventStore()
        self.reminders = [EKReminder]()
        self.eventStore.requestAccess(to: .reminder) {
            (granted: Bool, error: Error?) in
             
            if granted{
                // 获取授权后，我们可以得到所有的提醒事项
                let predicate = self.eventStore.predicateForReminders(in: nil)
                self.eventStore.fetchReminders(matching: predicate, completion: {
                    (reminders: [EKReminder]?) -> Void in
                     
                    self.reminders = reminders
                    print(self.reminders.count)
                    DispatchQueue.main.async{
                        self.tableView?.reloadData()
                    }
                })
            }else{
                print("获取提醒失败！需要授权允许对提醒事项的访问。")
            }
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.reminders.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        let cell = UITableViewCell(style: .subtitle,
                                   reuseIdentifier: "myCell")
        let reminder:EKReminder! = self.reminders![indexPath.row]
         
        //提醒事项的内容
        cell.textLabel?.text = reminder.title
         
        //提醒事项的时间
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
        if let dueDate = reminder.dueDateComponents?.date{
            cell.detailTextLabel?.text = formatter.string(from: dueDate)
        }else{
            cell.detailTextLabel?.text = "N/A"
        }
        return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1311.html