# Swift - Framework的制作与使用教程3（与Objective-C混合编程）

2016-11-14发布：hangge阅读：4321

在前面两篇文章中，我介绍了如何制作并使用一个纯 **Swift** 写的 **Framework**。但有时我们在过去的开发中积累了大量的 **OC** 代码，需要在新建的 **Swift Framework** 中使用，如果将原来的 **OC** 代码全部改写成 **Swift** 的也不太现实。或者我们需要在用到一些其它的第三方 **OC** 库。那么我们自定义的框架中就要实现 **Swift** 与 **OC** 的混编了。

在一般的工程项目中，要想让 **Swift** 能调用 **OC** 代码，只需创建一个桥街头文件。在桥接头文件中，**import** 任何你想暴露给 **Swift** 的 **Objective-C** 头文件。最后在 **Build Settings** -> **Objective-C Bridging header** 配置好这个桥街头文件即可。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710481621852.png)](https://www.hangge.com/blog/cache/detail_1427.html#)


而 **Framework** 无法使用桥街头，如果在 **Framework** 工程中也这么操作的话，会发现无法编译成功，报“**using bridging headers with framework targets is unsupported**”错误。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710463754473.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

本文演示实现如何在 **Swift framework** 中使用 **OC** 代码。

## 一、framework的制作（Swift与OC混编）

**KissXML** 是一个 **OC** 写的 **XML** 解析库，我原来写过相关的文章：[Swift - 解析XML格式数据（分别使用GDataXML和KissXML）](https://www.hangge.com/blog/cache/detail_646.html)。

这里我将 **KissXML** 封装到我们的 **framework** 中，并对其做个扩展方便使用。

### 1，创建framework工程项目

（1）新建项目的时候选择“**Cocoa Touch Framework**”。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110609244118070.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（2）项目名就叫做“**HanggeSDK**”。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110609265480447.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（3）为了让制作出的 **framework** 在低版本的系统上也能使用，可以在“**General**”->“**Deployment Info**”里设置个较低的发布版本。（这里选择 **8.0**）

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611133371048.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（4）将 **KissXML** 的库文件导入进来。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710021816193.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（5）在 **Build Phases** -> **Headers** 中我们可以看到 **KissXML** 的相关头文件已经在 **Project** 标签下了（如果没有，点下方的加号添加）。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710042459745.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（6）将需要暴露给 **Swift** 的 **Objective-C** 的头文件，拖动到 **Public** 标签下。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710323945400.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（7）在“**项目名.h**”（本样例是 **HanggeSDK.h**）文件中，将需要访问的头文件 **import** 进来。

```objective-c
#import <UIKit/UIKit.h>
 
//! Project version number for HanggeSDK.
FOUNDATION_EXPORT double HanggeSDKVersionNumber;
 
//! Project version string for HanggeSDK.
FOUNDATION_EXPORT const unsigned char HanggeSDKVersionString[];
 
// In this header, you should import all the public headers of your framework
// using statements like #import <HanggeSDK/PublicHeader.h>
 
 
#import "DDXML.h"
#import "DDXMLElementAdditions.h"

原文出自：www.hangge.com  转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1427.html
```

（8）由于 **KissXML** 用到了 **libxml** 库，我们还需在 **build phases** -> **Link Binary With Libraries** 中，点击“**+**”添加“**libxml2.2.tbd**”

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710184285006.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（9）添加进来后，接着在 **build setting** -> **Header Search Paths** 里添加 **${SDK_DIR}/usr/include/libxml2**

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710174833814.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（10）最后我们对 **KissXML** 做个扩展，使其通过索引即可访问子节点。新建个 **Swift** 文件（**KissXMLExtension.swift**），其内容如下：

```swift
import Foundation
 
extension DDXMLElement {
    //通过索引获取子节点
    public subscript(key: String) -> DDXMLElement {
        get {
            let r = self.forName(key)
            return r!
        }
    }
}
```



### 2，生成framework库文件

生成的 **framework** 文件是分为模拟器使用和真机使用这两种。

（1）发布编译目标选择“**Generic iOS Device**”后，使用快捷键 **command+B** 或者点击菜单 **Product** > **Build** 编译生成的是真机调试使用的 **framework**。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611335684643.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（2）如果发布编译目标选择的是模拟器，那么编译出来的模拟器使用的 **framework**。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611350036279.png)](https://www.hangge.com/blog/cache/detail_1427.html#)



（3）编译后右键点击项目中生成的 **framework**，选择“**Show in Finder**”，即可打开 **framework** 所在的文件夹。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611362490958.png)](https://www.hangge.com/blog/cache/detail_1427.html#)[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611390965664.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（4）访问上级文件夹，可以看到两种类型的 **framework** 分别放在两个不同的文件夹下。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611405411112.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

## 二、framework的使用

### 1，引入framework

（1）将生成的 **HanggeSDK.framework** 添加到项目中来。（注意：要根据你是使用真机调试还是模拟器调试选择对应的 **framework**）

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/201611061143041821.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（2）接着在“**General**”->“**Embedded Binaries**”中把 **HanggeSDK.framework** 添加进来。

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110611475796009.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

### 2，使用样例

（1）**users.xml** 文件中存放着人员信息集合。

```xml
<?xml version="1.0" encoding="utf-8"?>
<Users>
    <User id="101">
        <name>航歌</name>
        <tel>
            <mobile>1234567</mobile>
            <home>025-8100000</home>
        </tel>
    </User>
    <User id="102">
        <name>hangge</name>
        <tel>
            <mobile>8989889</mobile>
            <home>025-8122222</home>
        </tel>
    </User>
</Users>
```


（2）下面代码读取这个 **xml** 文件，并遍历其中的数据。

可以看到项目中我们只需要 **import HanggeSDK** 就可以使用 **KissXML** 的 **OC** 类，而不用再添加桥接头什么的了。

```swift
import UIKit
import HanggeSDK
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let label:UILabel = UILabel(frame:CGRect(x: 100, y: 100, width: 300, height: 100));
        label.text = "输出结果在控制台"
        self.view.addSubview(label)
        //测试Swift调用Object的XML库功能
        testXML()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
     
    func testXML() {
        //获取xml文件路径
         
        let file = Bundle.main.path(forResource: "users", ofType: "xml")
        let url = URL(fileURLWithPath: file!)
        //获取xml文件内容
        let xmlData = try! Data(contentsOf: url)
         
        //构造XML文档
        let doc = try!  DDXMLDocument(data: xmlData, options:0)
         
        //利用XPath来定位节点（XPath是XML语言中的定位语法，类似于数据库中的SQL功能）
        let users = try! doc.nodes(forXPath: "//User") as! [DDXMLElement]
        for user in users {
            let uid = user.attribute(forName: "id")!.stringValue
            let uname = user["name"].stringValue
             
            let mobile = user["tel"]["mobile"].stringValue
            let home = user["tel"]["home"].stringValue
             
            print("User: uid:\(uid!),uname:\(uname!),mobile:\(mobile!),home:\(home!)")
        }
    }
}
```


（3）运行结果如下：

[![原文:Swift - Framework的制作与使用教程3（与Objective-C混合编程）](https://www.hangge.com/blog_uploads/201611/2016110710420867032.png)](https://www.hangge.com/blog/cache/detail_1427.html#)

（4）源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[HanggeSDK+Sample.zip](https://www.hangge.com/blog_uploads/201611/2016110710432483921.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1427.html