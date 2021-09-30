# Swift - 使用Contacts访问通讯录1（纯代码获取联系人）

2017-02-06发布：hangge阅读：4830

在 **iOS9.0** 之前, 我们只能通过 **AddressBook** 框架来获取通讯录联系人信息。但 **AddressBook framework** 语法很奇怪，同时也十分难用。所以苹果从 **iOS9.0** 开始推出的全新的联系人框架 **Contacts FrameWork** 作为替代，同时将原来的 **AddressBook** 给废弃掉。


**Contacts FrameWork** 同样包含两种访问通讯录的方式：

- **ContactsUI.framework** 框架 ： 通过系统提供的通讯录交互界面来访问（替代原先的 **AddressBookUI.framework**）
- **Contacts.framework** 框架 ： 没有界面，通过代码来获取所有联系人信息（替代原先的 **AddressBook.framework**）

[前文](https://www.hangge.com/blog/cache/detail_1522.html)中我介绍了 **ContactsUI.framework** 的使用，本文接着演示 **Contacts.framework** 框架的使用。



## 一、Contacts.framework介绍

### 1，Contacts使用步骤

（1）获取用户的授权，如果授权则往下继续进行。

（2）创建通讯录对象

（3）创建联系人请求对象

（4）遍历所有联系人，获取联系人信息

### 2，联系人记录的属性

每一条联系人记录中，都有很多属性，而这些属性又分为单值属性和多值属性。

（1）单值属性是只有一个值的属性：

- **familyName** ：姓
- **givenName** ：名
- **middleName** ：中间名
- **previousFamilyName** ：前缀
- **nameSuffix** ：后缀
- **phoneticFamilyName** ：姓氏汉语拼音或音标
- **phoneticGivenName** ：名字汉语拼音或音标
- **nickname** ：昵称
- **organizationName** ：公司（组织）
- **jobTitle** ：职位
- **departmentName** ：部门
- **note** ：备注

（2）多值属性是包含多个值的集合类型：

- **phoneNumbers** ：电话
- **emailAddresses** ：Email
- **postalAddresses** ：地址
- **urlAddresses** ：URL属性
- **dates** ：纪念日
- **instantMessageAddresses** ：获取即时通讯(IM)
- **socialProfiles** ：社交账号
- **contactRelations** ：亲属关系人



### 3，多值属性标签的本地名称

在多值属性中包含了 **label**（标签）、**value**（值）和 **ID** 等部分，其中标签和值都是可以重复的，而 **ID** 是不能重复的。

对于标签，我们可以使用 **CNLabeledValue<NSString>.localizedString(forLabel:)** 方法转为本地标签名（即能看得懂的标签名，比如 **work**、**home**）。要不然打印出来的是 **_$!<Home>!$_**，**_$!<Work>!$_** 这样的数据。

## 二、获取所有联系人信息

### 1，效果图

（1）程序启动后自动开始请求并获取通讯录信息。

（2）同时将通讯录里的所有联系人信息输出到控制台中。（具体包括联系人的姓名、昵称、公司、职位、部门、备注、电话、email、 地址、纪念日、即时通讯）

[![原文:Swift - 使用Contacts访问通讯录1（纯代码获取联系人）](https://www.hangge.com/blog_uploads/201701/2017011816461873845.png)](https://www.hangge.com/blog/cache/detail_1523.html#)

### 2，Info.plist配置

由于苹果安全策略更新，在使用 **Xcode8** 开发时，需要在 **Info.plist** 配置请求通讯录的相关描述字段（**Privacy - Contacts Usage Description**）

[![原文:Swift - 使用Contacts访问通讯录1（纯代码获取联系人）](https://www.hangge.com/blog_uploads/201701/2017011816352921704.png)](https://www.hangge.com/blog/cache/detail_1523.html#)



### 3，样例代码

```swift
import UIKit
import Contacts
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
        CNContactStore().requestAccess(for: .contacts) { (isRight, error) in
            if isRight {
                //授权成功加载数据。
                self.loadContactsData()
            }
        }
    }
     
    func loadContactsData() {
        //获取授权状态
        let status = CNContactStore.authorizationStatus(for: .contacts)
        //判断当前授权状态
        guard status == .authorized else { return }
         
        //创建通讯录对象
        let store = CNContactStore()
         
        //获取Fetch,并且指定要获取联系人中的什么属性
        let keys = [CNContactFamilyNameKey, CNContactGivenNameKey, CNContactNicknameKey,
                    CNContactOrganizationNameKey, CNContactJobTitleKey,
                    CNContactDepartmentNameKey, CNContactNoteKey, CNContactPhoneNumbersKey,
                    CNContactEmailAddressesKey, CNContactPostalAddressesKey,
                    CNContactDatesKey, CNContactInstantMessageAddressesKey
        ]
         
        //创建请求对象
        //需要传入一个(keysToFetch: [CNKeyDescriptor]) 包含CNKeyDescriptor类型的数组
        let request = CNContactFetchRequest(keysToFetch: keys as [CNKeyDescriptor])
         
        //遍历所有联系人
        do {
            try store.enumerateContacts(with: request, usingBlock: {
                (contact : CNContact, stop : UnsafeMutablePointer<ObjCBool>) -> Void in
                 
                //获取姓名
                let lastName = contact.familyName
                let firstName = contact.givenName
                print("姓名：\(lastName)\(firstName)")
                 
                //获取昵称
                let nikeName = contact.nickname
                print("昵称：\(nikeName)")
                 
                //获取公司（组织）
                let organization = contact.organizationName
                print("公司（组织）：\(organization)")
                 
                //获取职位
                let jobTitle = contact.jobTitle
                print("职位：\(jobTitle)")
                 
                //获取部门
                let department = contact.departmentName
                print("部门：\(department)")
                 
                //获取备注
                let note = contact.note
                print("备注：\(note)")
                 
                //获取电话号码
                print("电话：")
                for phone in contact.phoneNumbers {
                    //获得标签名（转为能看得懂的本地标签名，比如work、home）
                    var label = "未知标签"
                    if phone.label != nil {
                        label = CNLabeledValue<NSString>.localizedString(forLabel:
                            phone.label!)
                    }
                     
                    //获取号码
                    let value = phone.value.stringValue
                    print("\t\(label)：\(value)")
                }
                 
                //获取Email
                print("Email：")
                for email in contact.emailAddresses {
                    //获得标签名（转为能看得懂的本地标签名）
                    var label = "未知标签"
                    if email.label != nil {
                        label = CNLabeledValue<NSString>.localizedString(forLabel:
                            email.label!)
                    }
                     
                    //获取值
                    let value = email.value
                    print("\t\(label)：\(value)")
                }
                 
                //获取地址
                print("地址：")
                for address in contact.postalAddresses {
                    //获得标签名（转为能看得懂的本地标签名）
                    var label = "未知标签"
                    if address.label != nil {
                        label = CNLabeledValue<NSString>.localizedString(forLabel:
                            address.label!)
                    }
                     
                    //获取值
                    let detail = address.value
                    let contry = detail.value(forKey: CNPostalAddressCountryKey) ?? ""
                    let state = detail.value(forKey: CNPostalAddressStateKey) ?? ""
                    let city = detail.value(forKey: CNPostalAddressCityKey) ?? ""
                    let street = detail.value(forKey: CNPostalAddressStreetKey) ?? ""
                    let code = detail.value(forKey: CNPostalAddressPostalCodeKey) ?? ""
                    let str = "国家:\(contry) 省:\(state) 城市:\(city) 街道:\(street)"
                        + " 邮编:\(code)"
                    print("\t\(label)：\(str)")
                }
                 
                //获取纪念日
                print("纪念日：")
                for date in contact.dates {
                    //获得标签名（转为能看得懂的本地标签名）
                    var label = "未知标签"
                    if date.label != nil {
                        label = CNLabeledValue<NSString>.localizedString(forLabel:
                            date.label!)
                    }
                     
                    //获取值
                    let dateComponents = date.value as DateComponents
                    let value = NSCalendar.current.date(from: dateComponents)
                    let dateFormatter = DateFormatter()
                    dateFormatter.dateFormat = "yyyy年MM月dd日 HH:mm:ss"
                    print("\t\(label)：\(dateFormatter.string(from: value!))")
                }
                 
                //获取即时通讯(IM)
                print("即时通讯(IM)：")
                for im in contact.instantMessageAddresses {
                    //获得标签名（转为能看得懂的本地标签名）
                    var label = "未知标签"
                    if im.label != nil {
                        label = CNLabeledValue<NSString>.localizedString(forLabel:
                            im.label!)
                    }
                     
                    //获取值
                    let detail = im.value
                    let username = detail.value(forKey: CNInstantMessageAddressUsernameKey)
                        ?? ""
                    let service = detail.value(forKey: CNInstantMessageAddressServiceKey)
                        ?? ""
                    print("\t\(label)：\(username) 服务:\(service)")
                }
                 
                print("----------------")
                 
            })
        } catch {
            print(error)
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1523.html