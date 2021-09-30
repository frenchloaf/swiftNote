# Swift - 使用Contacts访问通讯录2（添加、修改、删除联系人）

2017-02-08发布：hangge阅读：2081

在前文中：[Swift - 使用Contacts访问通讯录1（纯代码获取联系人）](https://www.hangge.com/blog/cache/detail_1523.html)。我介绍了如何使用 **Contacts.framework** 框架来获取通讯录里的联系人。本文接着演示如何对通讯录进行新增、修改、删除联系人操作。
（注意：这些操作同查询一样，首先需要发起授权请求。并且在 **Info.plist** 配置好请求通讯录的相关描述字段。） 

### 1，添加新联系人

下面程序启动后，自动往通讯录中新增一个联系人。这里除了设置联系人姓名、电话等基本信息外，还给其添加了个头像。

```swift
import UIKit
import Contacts
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //申请权限
        CNContactStore().requestAccess(for: .contacts) { (isRight, error) in
            if isRight {
                //授权成功添加数据。
                self.addContact()
            }
        }
    }
     
    //新增一个联系人
    func addContact() {
        //创建通讯录对象
        let store = CNContactStore()
         
        //创建CNMutableContact类型的实例
        let contactToAdd = CNMutableContact()
         
        //设置姓名
        contactToAdd.familyName = "张"
        contactToAdd.givenName = "飞"
         
        //设置昵称
        contactToAdd.nickname = "fly"
         
        //设置头像
        let image = UIImage(named: "fei")!
        contactToAdd.imageData = UIImagePNGRepresentation(image)
         
        //设置电话
        let mobileNumber = CNPhoneNumber(stringValue: "18510002000")
        let mobileValue = CNLabeledValue(label: CNLabelPhoneNumberMobile,
                                         value: mobileNumber)
        contactToAdd.phoneNumbers = [mobileValue]
         
        //设置email
        let email = CNLabeledValue(label: CNLabelHome, value: "feifei@163.com" as NSString)
        contactToAdd.emailAddresses = [email]
         
        //添加联系人请求
        let saveRequest = CNSaveRequest()
        saveRequest.add(contactToAdd, toContainerWithIdentifier: nil)
         
        do {
            //写入联系人
            try store.execute(saveRequest)
            print("保存成功!")
        } catch {
            print(error)
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
```

运行后打开设备通讯录，可以发现联系人已添加成功。

   [![原文:Swift - 使用Contacts访问通讯录2（添加、修改、删除联系人）](https://www.hangge.com/blog_uploads/201701/2017011911105484849.png)](https://www.hangge.com/blog/cache/detail_1524.html#)    [![原文:Swift - 使用Contacts访问通讯录2（添加、修改、删除联系人）](https://www.hangge.com/blog_uploads/201701/2017011911110978628.png)](https://www.hangge.com/blog/cache/detail_1524.html#)



### 2，编辑修改联系人

先获取所有联系人并进行遍历，根据联系人姓名或者电话来判断是否要修改。这里我们将“张飞”的昵称修改成“小飞飞”。

```
import UIKit
import Contacts
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //申请权限
        CNContactStore().requestAccess(for: .contacts) { (isRight, error) in
            if isRight {
                //授权成功修改数据。
                self.editContact()
            }
        }
    }
     
    func editContact() {
        //创建通讯录对象
        let store = CNContactStore()
         
        //获取Fetch,并且指定要获取联系人中的什么属性
        let keys = [CNContactFamilyNameKey, CNContactGivenNameKey, CNContactNicknameKey]
         
        //创建请求对象，需要传入一个(keysToFetch: [CNKeyDescriptor]) 包含'CNKeyDescriptor'类型的数组
        let request = CNContactFetchRequest(keysToFetch: keys as [CNKeyDescriptor])
         
        //遍历所有联系人
        do {
            try store.enumerateContacts(with: request, usingBlock: {
                (contact : CNContact, stop : UnsafeMutablePointer<ObjCBool>) -> Void in
                 
                let mutableContact = contact.mutableCopy() as! CNMutableContact
                 
                //获取姓名
                let lastName = mutableContact.familyName
                let firstName = mutableContact.givenName
                //判断是否符合要求
                if lastName == "张" && firstName == "飞"{
                    //设置昵称
                    mutableContact.nickname = "小飞飞"
                     
                    //修改联系人请求
                    let request = CNSaveRequest()
                    request.update(mutableContact)
                    do {
                        //修改联系人
                        try store.execute(request)
                        print("修改成功!")
                    } catch {
                        print(error)
                    }
                }
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



### 3，删除联系人

同样先获取所有联系人并进行遍历，根据联系人姓名或者电话来判断是否要删除。这里我们将“张飞”这个联系人删除。

```
import UIKit
import Contacts
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //申请权限
        CNContactStore().requestAccess(for: .contacts) { (isRight, error) in
            if isRight {
                //授权成功删除数据。
                self.deleteContact()
            }
        }
    }
     
    func deleteContact() {
        //创建通讯录对象
        let store = CNContactStore()
         
        //获取Fetch,并且指定要获取联系人中的什么属性
        let keys = [CNContactFamilyNameKey, CNContactGivenNameKey, CNContactNicknameKey]
         
        //创建请求对象，需要传入一个(keysToFetch: [CNKeyDescriptor]) 包含'CNKeyDescriptor'类型的数组
        let request = CNContactFetchRequest(keysToFetch: keys as [CNKeyDescriptor])
         
        //遍历所有联系人
        do {
            try store.enumerateContacts(with: request, usingBlock: {
                (contact : CNContact, stop : UnsafeMutablePointer<ObjCBool>) -> Void in
                 
                let mutableContact = contact.mutableCopy() as! CNMutableContact
 
                //获取姓名
                let lastName = mutableContact.familyName
                let firstName = mutableContact.givenName
                //判断是否符合要求
                if lastName == "张" && firstName == "飞"{
                    //删除联系人请求
                    let request = CNSaveRequest()
                    request.delete(mutableContact)
                     
                    do {
                        //执行操作
                        try store.execute(request)
                        print("删除成功!")
                    } catch {
                        print(error)
                    }
                }
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


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1524.html