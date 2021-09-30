# Swift - 使用EventKit操作"提醒事项"（2、新增、修改、删除提醒）

2016-08-03发布：hangge阅读：1266  

（本文代码已升级至Swift3）   

我在前篇文章中：[Swift - 使用EventKit操作"提醒事项"（1、查询出所有提醒）](https://www.hangge.com/blog/cache/detail_1311.html)，介绍如何通过 **EventKit** 获取所有的提醒事件。本文接着演示如何对提醒进行增删改操作。

**1，添加提醒**

下面样例中，填写提醒内容、选择提醒时期后，点击“保存”即可将提醒添加到系统的“提醒事项”中。

（这里我将日期输入框的 **inputView** 设置成 **UIDatePicker**，这样点击日期文本框的时候底部会自动出现日期选择器来选择时间。）

[![原文:Swift - 使用EventKit操作"提醒事项"（2、新增、修改、删除提醒）](https://www.hangge.com/blog_uploads/201607/2016073113551540914.png)](https://www.hangge.com/blog/cache/detail_1312.html#)[![原文:Swift - 使用EventKit操作"提醒事项"（2、新增、修改、删除提醒）](https://www.hangge.com/blog_uploads/201607/2016073113552098584.png)](https://www.hangge.com/blog/cache/detail_1312.html#)[![原文:Swift - 使用EventKit操作"提醒事项"（2、新增、修改、删除提醒）](https://www.hangge.com/blog_uploads/201607/2016073113552550742.png)](https://www.hangge.com/blog/cache/detail_1312.html#)

```swift
import UIKit
import EventKit
 
class ViewController: UIViewController {
     
    var eventStore: EKEventStore!
    //提醒信息文本框
    @IBOutlet weak var reminderTextField: UITextField!
    //提醒时间文本框
    @IBOutlet weak var dateTextField: UITextField!
    //提醒日期选择器
    var datePicker: UIDatePicker!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        self.eventStore = EKEventStore()
         
        //设置提醒时间文本框使用的日期选择器
        datePicker = UIDatePicker()
        datePicker.addTarget(self, action: #selector(ViewController.addDate),
                             for: .valueChanged)
        datePicker.datePickerMode = .dateAndTime
        dateTextField.inputView = datePicker
    }
     
    //日期选择器响应方法
    func addDate(){
        //日期样式
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy年MM月dd日 HH:mm"
        //更新提醒时间文本框
        self.dateTextField.text = formatter.string(from: datePicker.date)
    }
     
    //保存按钮响应办法
    @IBAction func addReminder(_ sender: AnyObject) {
        //获取"提醒"的访问授权
        self.eventStore.requestAccess(to: .reminder) {
            (granted: Bool, error: Error?) in
             
            //创建提醒条目
            let reminder = EKReminder(eventStore: self.eventStore)
            reminder.title = (self.reminderTextField.text)!
            let dueDateComponents = self.dateComponentFrom(date: self.datePicker.date)
            reminder.dueDateComponents = dueDateComponents
            reminder.calendar = self.eventStore.defaultCalendarForNewReminders()
            //保存提醒事项
            do {
                try self.eventStore.save(reminder, commit: true)
                print("保存成功！")
            }catch{
                print("创建提醒失败: \(error)")
            }
        }
    }
     
    //根据NSDate获取对应的DateComponents对象
    func dateComponentFrom(date: Date)-> DateComponents{
        let cal = Calendar.current
        let dateComponents = cal.dateComponents([.minute, .hour, .day, .month, .year],
                                                from: date)
        return dateComponents
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，修改提醒**
修改提醒同新增提醒差不多，我们首先获取到需要修改的 **Reminer** 对象，然后修改其时间或标题。最后调用 **eventStore** 的 **save** 方法保存即可。
下面样例将所有的提醒标题都修改成“欢迎访问hangge.com”



[![原文:Swift - 使用EventKit操作"提醒事项"（2、新增、修改、删除提醒）](https://www.hangge.com/blog_uploads/201607/201607311414139725.png)](https://www.hangge.com/blog/cache/detail_1312.html#)

```swift
//获取"提醒"的访问授权
self.eventStore.requestAccess(to: .reminder) {
    (granted: Bool, error: Error?) -> Void in
     
    if granted{
        // 获取授权后，我们可以得到所有的提醒事项
        let predicate = self.eventStore.predicateForReminders(in: nil)
        self.eventStore.fetchReminders(matching: predicate, completion: {
            (reminders: [EKReminder]?) -> Void in
             
            //遍历所有提醒并修改
            for reminder in reminders! {
                reminder.title = "欢迎访问hangge.com"
                reminder.calendar = self.eventStore.defaultCalendarForNewReminders()
                //保存提醒事项
                do {
                    try self.eventStore.save(reminder, commit: true)
                    print("保存成功！")
                }catch{
                    print("保存失败: \(error)")
                }
            }
        })
    }else{
        print("获取提醒失败！需要授权允许对提醒事项的访问。")
    }
}
```


**3，删除提醒**
我们首先获取到需要删除的 **Reminer** 对象，然后调用 **eventStore** 的 **remove** 方法即可。
下面样例将系统里所有的提醒删除。

```swift
//获取"提醒"的访问授权
self.eventStore.requestAccess(to: .reminder) {
    (granted: Bool, error: Error?) in
     
    if granted{
        // 获取授权后，我们可以得到所有的提醒事项
        let predicate = self.eventStore.predicateForReminders(in: nil)
        self.eventStore.fetchReminders(matching: predicate, completion: {
            (reminders: [EKReminder]?) -> Void in
             
            //遍历所有提醒并删除
            for reminder in reminders! {
                //保存提醒事项
                do {
                    try self.eventStore.remove(reminder, commit: true)
                    print("删除成功！")
                }catch{
                    print("删除失败: \(error)")
                }
            }
        })
    }else{
        print("获取提醒失败！需要授权允许对提醒事项的访问。")
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1312.html