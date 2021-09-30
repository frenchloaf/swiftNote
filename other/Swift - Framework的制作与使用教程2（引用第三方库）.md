# Swift - Framework的制作与使用教程2（引用第三方库）

2016-11-11发布：hangge阅读：4615

在我们创建的自定义框架中，也是可以再引用其它第三方的框架库。本文以实现一个网络定时请求的 **framework** 为例，其内部使用到了 **Alamofire**。关于 **Alamofire** 的详细介绍，可以参考我之前写的这篇文章：[Swift - HTTP网络操作库Alamofire使用详解1（配置，以及数据请求）](https://www.hangge.com/blog/cache/detail_970.html)

## 一、framework的制作（引用第三方库 ）

### 1，创建framework工程项目

（1）新建项目的时候选择“**Cocoa Touch Framework**”。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110609244118070.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（2）项目名就叫做“**HanggeSDK**”。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110609265480447.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（3）为了让制作出的 **framework** 在低版本的系统上也能使用，可以在“**General**”->“**Deployment Info**”里设置个较低的发布版本。（这里选择 **9.0**）

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110809512845838.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（4）由于框架中需要使用 **Alamofire**，这里将 **Alamofire.xcodeproj** 添加到项目中来

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110809475633460.png)](https://www.hangge.com/blog/cache/detail_1426.html#)


（5） **General** -> **Linked Frameworks and Libraries** 项，把 **iOS** 版的 **framework** 添加进来： **Alamofire.framework**

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110809490120991.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（6）创建一个功能实现类（**HttpScheduledTask.swift**），代码如下。

```swift
import Foundation
import Alamofire
 
public class HttpScheduledTask {
    //请求地址
    var url:String?
     
    //请求参数
    var params:[String:Any]?
     
    //定时请求时间间隔
    var timeInterval:TimeInterval!
     
    //请求响应回调
    var callBack:(Data)->Void
     
    //定时任务Timer，用于停止定时任务
    var timer:Timer?
     
    //初始化。参数默认为空，时间间隔默认为1秒，默认没用回调处理。
    public init(url:String, params:[String:Any] = [:],timeInterval:TimeInterval = 1,
                callBack:@escaping (Data)->Void = {_ in}) {
        self.url = url
        self.params = params
        self.timeInterval = timeInterval
        self.callBack = callBack
    }
     
    //启动任务
    public func start(){
        //如果之前有定时任务，先停止
        self.timer?.invalidate()
        self.timer = Timer.scheduledTimer(timeInterval: self.timeInterval,
                                          target:self,selector:#selector(onTime),
                                          userInfo:nil,repeats:true)
    }
     
    //时间到，开始请求
    @objc func onTime() {
        Alamofire.request(self.url!, parameters: params).response { (response) in
            if let data = response.data {
                //调用回调函数
                self.callBack(data)
            }
        }
         
    }
     
    //停止任务
    public func stop() {
        self.timer?.invalidate()
    }
}
```

###  2，生成framework库文件

生成的 **framework** 文件是分为模拟器使用和真机使用这两种。

（1）发布编译目标选择“**Generic iOS Device**”后，使用快捷键 **command+B** 或者点击菜单 **Product** > **Build** 编译生成的是真机调试使用的 **framework**。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611335684643.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（2）如果发布编译目标选择的是模拟器，那么编译出来的模拟器使用的 **framework**。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611350036279.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



（3）编译后右键点击项目中生成的 **framework**，选择“**Show in Finder**”，即可打开 **framework** 所在的文件夹。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611362490958.png)](https://www.hangge.com/blog/cache/detail_1426.html#)[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611390965664.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（4）访问上级文件夹，可以看到两种类型的 **framework** 分别放在两个不同的文件夹下。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611405411112.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

## 二、framework的使用

### 1，引入framework

（1）将生成的 **HanggeSDK.framework** 添加到项目中来。（注意：要根据你是使用真机调试还是模拟器调试选择对应的 **framework**）

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/201611061143041821.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（2）接着在“**General**”->“**Embedded Binaries**”中把 **HanggeSDK.framework** 添加进来。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110611475796009.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（3）除了将我们自定义类库 **HanggeSDK** 引入外，还需要将其依赖库也给引入进来（**Alamofire**）。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810051355385.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

否则运行后会报“**dyld: Library not loaded: @rpath/Alamofire.framework/Alamofire**”错误。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810013124804.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

### 2，使用样例

```swift
import UIKit
import HanggeSDK
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //新建一个定时请求任务
        let task1 = HttpScheduledTask(url: "https://httpbin.org/get"){
            data in
            if let utf8Text = String(data: data, encoding: .utf8) {
                print("----- 获取到的数据 -----")
                print(utf8Text)
            }
        }
        //启动定时任务
        task1.start()
        //停止定时任务
        //task1.stop()
         
        /**** 带参数的定时请求任务 *****/
        let task2 = HttpScheduledTask(url: "https://httpbin.org/get",
                                      params: ["name":"hangge.com", "password":123],
                                      timeInterval: 10) { (data) in
            //处理响应
        }
        //启动定时任务
        //task2.start()
        //停止定时任务
        //task2.stop()
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

运行效果如下：

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810161184399.png)](https://www.hangge.com/blog/cache/detail_1426.html#)



源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[HanggeSDK+Sample.zip](https://www.hangge.com/blog_uploads/201611/2016110810171050494.zip)



## 三、将第三方库也打包进framwork中

上面的样例可以看到，如果我们自定义的 **framework** 用到了第三方库。那么在使用这个自定义的 **framework** 的项目中，也需要将这些第三方依赖库给引用进来。
如果嫌麻烦的话，我们也可以把第三方依赖库一起编译打包进 **framewrok** 中。

 （1）点击“**Build Phases**”中左上角的加号，选择“**New Copy Files Phase**”。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810195525194.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（2）在新建的“**Copy Files**”中，将 **Destination** 属性选择“**Framework**”，添加类库（**Alamofire**）。“**Copy only when installing**” 与 “**Code Sign On Copy**” 都不勾选。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810223064890.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（3）重新编译生成新的 **HanggeSDK.framework**，可以看到这个新的 **framework** 体积比原来大很多，说明 **Alamofire** 也被一同打包进了了。

[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810243693774.png)](https://www.hangge.com/blog/cache/detail_1426.html#)[![原文:Swift - Framework的制作与使用教程2（引用第三方库）](https://www.hangge.com/blog_uploads/201611/2016110810244262800.png)](https://www.hangge.com/blog/cache/detail_1426.html#)

（4）我们工程使用这个新的 **framework** 时，就不需要再引入 **Alamofire** 了。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1426.html