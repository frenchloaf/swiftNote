# Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）

2017-02-04发布：hangge阅读：3186

在 **iOS9.0** 之前, 我们只能通过 **AddressBook** 框架来获取通讯录联系人信息。但 **AddressBook framework** 语法很奇怪，同时也十分难用。所以苹果从 **iOS9.0** 开始推出的全新的联系人框架 **Contacts FrameWork** 作为替代，同时将原来的 **AddressBook** 给废弃掉。


**Contacts FrameWork** 同样包含两种通讯录的访问方式：

- **ContactsUI.framework** 框架 ： 通过系统提供的通讯录交互界面来访问（替代原先的 **AddressBookUI.framework**）
- **Contacts.framework** 框架 ： 没有界面，通过代码来获取所有联系人信息（替代原先的 **AddressBook.framework**）

本文演示前者（**ContactsUI.framework** 框架）的使用。



## 一、ContactsUI.framework介绍

### 1，通讯录访问介绍

使用 **ContactsUI** 时，系统已经为我们封装好了一套联系人的 **UI** 界面，使用十分方便，主要新增的 **controller** 有两个：

- **CNContactPickerViewController**：展示联系人列表的 **controller**
- **CNContactViewController**：展示联系人详细信息的 **controller**

下面演示如何使用 **CNContactPickerViewController** 来实现通讯录联系人选择功能。

###  2，多值属性标签的本地名称

在多值属性中包含了 **label**（标签）、**value**（值）和 **ID** 等部分，其中标签和值都是可以重复的，而 **ID** 是不能重复的。

对于标签，我们可以使用 **CNLabeledValue<NSString>.localizedString(forLabel:)** 方法转为本地标签名（即能看得懂的标签名，比如 **work**、**home**）。否则打印出来的是 **_$!<Home>!$_**，**_$!<Work>!$_** 这样的数据。



## 二、单选联系人

### 1，效果图

（1）点击页面上的“**选择联系人**”按钮后，便会自动弹出系统联系人选择器。

（2）同时在选择器中，我们将那些 **email** 地址为空的联系人设为不可选状态（置灰）

（3）选择一个联系人后，自动关闭选择器。同时在控制台中将联系人的姓名、以及他所有的电话打印出来。

  [![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011713555461676.png)](https://www.hangge.com/blog/cache/detail_1522.html#)  [![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011713555911438.png)](https://www.hangge.com/blog/cache/detail_1522.html#)  [![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011713585140705.png)](https://www.hangge.com/blog/cache/detail_1522.html#)

### 

```swift
import UIKit
import ContactsUI
 
class ViewController: UIViewController, CNContactPickerDelegate {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //选择联系人按钮点击
    @IBAction func selectContacts(_ sender: AnyObject) {
        //联系人选择控制器
        let contactPicker = CNContactPickerViewController()
        //设置代理
        contactPicker.delegate = self
        //添加可选项目的过滤条件
        contactPicker.predicateForEnablingContact
            = NSPredicate(format: "emailAddresses.@count > 0", argumentArray: nil)
        //弹出控制器
        self.present(contactPicker, animated: true, completion: nil)
    }
     
    //单选联系人
    func contactPicker(_ picker: CNContactPickerViewController,
                       didSelect contact: CNContact) {
        //获取联系人的姓名
        let lastName = contact.familyName
        let firstName = contact.givenName
        print("选中人的姓：\(lastName)")
        print("选中人的名：\(firstName)")
         
        //获取联系人电话号码
        print("选中人电话：")
        let phones = contact.phoneNumbers
        for phone in phones {
            //获得标签名（转为能看得懂的本地标签名，比如work、home）
            let phoneLabel = CNLabeledValue<NSString>.localizedString(forLabel: phone.label!)
            //获取号码
            let phoneValue = phone.value.stringValue
            print("\(phoneLabel):\(phoneValue)")
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



## 三、多选联系人

### 1，效果图

（1）点击页面上的“**选择联系人**”按钮后，便会自动弹出系统联系人选择器。

（2）这次在选择器中我们可以同时选择多个联系人。

（3）选择完毕后点击“**完成**”按钮关闭选择器。同时在控制台中将所有选中的联系人的姓名、电话打印出来。

[![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011714365322375.png)](https://www.hangge.com/blog/cache/detail_1522.html#)

[![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011714370133125.png)](https://www.hangge.com/blog/cache/detail_1522.html#)

[![原文:Swift - 使用ContactsUI访问通讯录（通过图形界面选择联系人）](https://www.hangge.com/blog_uploads/201701/2017011714370971335.png)](https://www.hangge.com/blog/cache/detail_1522.html#)

###  2，样例代码

多选代码其实和单选差不多，只不过我们将前面的 **CNContactPickerDelegate** 协议单选相关代码改成多选即可。

```swift
import UIKit
import ContactsUI
 
class ViewController: UIViewController, CNContactPickerDelegate {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //选择联系人按钮点击
    @IBAction func selectContacts(_ sender: AnyObject) {
        //联系人选择控制器
        let contactPicker = CNContactPickerViewController()
        //设置代理
        contactPicker.delegate = self
        //添加可选项目的过滤条件
        contactPicker.predicateForEnablingContact
            = NSPredicate(format: "emailAddresses.@count > 0", argumentArray: nil)
        //弹出控制器
        self.present(contactPicker, animated: true, completion: nil)
    }
     
    //多选联系人
    func contactPicker(_ picker: CNContactPickerViewController, didSelect contacts: [CNContact]) {
        print("一共选择了\(contacts.count)个联系人。")
        for contact in contacts {
            print("--------------")
            //获取联系人的姓名
            let lastName = contact.familyName
            let firstName = contact.givenName
            print("选中人的姓：\(lastName)")
            print("选中人的名：\(firstName)")
             
            //获取联系人电话号码
            print("选中人电话：")
            let phones = contact.phoneNumbers
            for phone in phones {
                //获得标签名（转为能看得懂的本地标签名，比如work、home）
                let phoneLabel = CNLabeledValue<NSString>.localizedString(forLabel: phone.label!)
                //获取号码
                let phoneValue = phone.value.stringValue
                print("\(phoneLabel):\(phoneValue)")
            }
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1522.html