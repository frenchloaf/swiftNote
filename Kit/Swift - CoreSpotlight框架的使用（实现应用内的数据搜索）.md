# Swift - CoreSpotlight框架的使用（实现应用内的数据搜索）

2017-02-27发布：hangge阅读：1046

过去 **Spotlight** 功能有限，我们只能通过搜索应用名称来打开指定 **App**，或者搜索系统应用中的内容（如信息，联系人，邮件等）。对于我们 **App** 中的内容搜索，**Spotlight** 就无能为力了。

自在 **iOS9** 起，苹果放开了权限。允许开发者将应用中任意内容设置成可以被 **Spotlight** 索引到，以及用户在选择了搜索内容时会发生什么。比如我在系统 **Spotlight** 搜索框中输入“**iPhone7**”，可以看到 **UC** 中的内容被搜索出来了。

[![原文:Swift - CoreSpotlight框架的使用（实现应用内的数据搜索）](https://www.hangge.com/blog_uploads/201702/2017020916262755509.png)](https://www.hangge.com/blog/cache/detail_1553.html#)



## 一、Core Spotlight介绍

### 1，基本概念

- **Core Spotlight** 框架是 **iOS9** 提供的一组新的 **API**，用来帮助我们建立起我们应用中内容的索引，并实现指向应用内的深层链接。
- **Core Spotlight** 创建的索引存储在设备上，不与 **Apple** 共享，也不能被其他应用或者设备访问。
- **Core Spotlight** 创建的索引最好在几千的数量级别之下。索引太多很有可能会带来性能问题。

### 2，建立索引的相关类

- **CSSearchableItemAttributeSet**：索引属性集合，也即是索引的内容本身。集合中可以存储以下属性：**title**、**contentDescription**、**thumbnailData**、**rating**、**keywords**。下图显示 **Spotlight** 是如何通过这些属性展示搜索结果的。（如果未设置缩略图则默认显示应用图标）

[![原文:Swift - CoreSpotlight框架的使用（实现应用内的数据搜索）](https://www.hangge.com/blog_uploads/201702/201702091700307281.png)](https://www.hangge.com/blog/cache/detail_1553.html#)

- **CSSearchableItem**：用来表示一个被索引的条目。其在构建的时候需要传入一个 **CSSearchableItemAttributeSet** 对象。



## 二、Core Spotlight使用样例

### 1，索引的创建

（1）下面代码实现当程序启动后，自动新增**5**条索引记录。

```
import` `UIKit``import` `CoreSpotlight``import` `MobileCoreServices` `class` `ViewController``: ``UIViewController` `{``  ` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``    ` `    ``//判断设备支持情况``    ``guard ``CSSearchableIndex``.isIndexingAvailable() ``else` `{``      ``print``(``"该设备不支持添加Spotlight搜索!"``)``      ``return``    ``}``    ` `    ``//创建索引集合``    ``var` `items = [``CSSearchableItem``]()``    ``for` `i ``in` `1..<7 {``      ``//创建索引属性``      ``let` `attributeSet = ``CSSearchableItemAttributeSet``(itemContentType: kUTTypeData``        ``as` `String``)``      ``attributeSet.title = ``"hangge.com 系列文章\(i)"``      ``attributeSet.contentDescription = ``"这个是一段简单的描述"``      ``attributeSet.keywords = [``"article\(i)"``, ``"swift"``]``      ``attributeSet.thumbnailData = ``UIImagePNGRepresentation``(``UIImage``(named: ``"a"``)!)``      ``//创建索引项``      ``let` `item = ``CSSearchableItem``(uniqueIdentifier: ``"\(i)"``,``        ``domainIdentifier: ``"group1"``, attributeSet: attributeSet)``      ``items.append(item)``    ``}``    ` `    ``//保存新增的索引``    ``let` `sIndex = ``CSSearchableIndex``.``default``()``    ``sIndex.indexSearchableItems(items, completionHandler: { (error) ``in``      ``if` `error != ``nil` `{``        ``print``(error!.localizedDescription)``      ``} ``else` `{``        ``print``(``"索引添加成功!"``)``      ``}``    ``})``  ``}``  ` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```

（2）程序运行后退出，然后使用 **Spotlight** 搜索结果如下：
[![原文:Swift - CoreSpotlight框架的使用（实现应用内的数据搜索）](https://www.hangge.com/blog_uploads/201702/2017020920385672199.png)](https://www.hangge.com/blog/cache/detail_1553.html#)[![原文:Swift - CoreSpotlight框架的使用（实现应用内的数据搜索）](https://www.hangge.com/blog_uploads/201702/2017020920390349696.png)](https://www.hangge.com/blog/cache/detail_1553.html#)



### 2，索引的删除

（1）删除全部索引

```swift
let sIndex = CSSearchableIndex.default()
sIndex.deleteAllSearchableItems(completionHandler: { (error) in
    if error != nil {
        print(error!.localizedDescription)
    } else {
        print("删除成功！")
    }
})
```


（2）根据 **domain**（分组范围）删除索引

```swift
let sIndex = CSSearchableIndex.default()
sIndex.deleteSearchableItems(withDomainIdentifiers: ["group1"], completionHandler: {
    (error) in
    if error != nil {
        print(error!.localizedDescription)
    } else {
        print("删除成功！")
    }
})
```


（3）根据 **ID** 删除索引

```swift
let sIndex = CSSearchableIndex.default()
sIndex.deleteSearchableItems(withIdentifiers: ["1","2"], completionHandler: { (error) in
    if error != nil {
        print(error!.localizedDescription)
    } else {
        print("删除成功！")
    }
})
```

###  3，搜索结果的点击响应

当用户点击搜索结果时会自动打开这个应用，并调用 **AppDelegate** 中的 **continue userActivity** 方法。我们可以在该方法中获取点击项的索引 **ID**，从而进行后续操作，比如：打开相应的页面或执行相关操作。

```swift
import UIKit
import CoreSpotlight
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
    func application(_ application: UIApplication, continue userActivity: NSUserActivity,
                     restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
        if userActivity.activityType == CSSearchableItemActionType {
            let uniqueIdentifier = userActivity
                .userInfo?[CSSearchableItemActivityIdentifier] as? String
            print("点击的索引ID为：\(uniqueIdentifier)")
            //执行后续操作.....
        }
        return true
    }
 
    func application(_ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?)
        -> Bool {
        return true
    }
 
    func applicationWillResignActive(_ application: UIApplication) {
    }
 
    func applicationDidEnterBackground(_ application: UIApplication) {
    }
 
    func applicationWillEnterForeground(_ application: UIApplication) {
    }
 
    func applicationDidBecomeActive(_ application: UIApplication) {
    }
 
    func applicationWillTerminate(_ application: UIApplication) {
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1553.html