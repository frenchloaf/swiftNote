# Swift - 使用Carthage来安装管理第三方库

2016-09-18发布：hangge阅读：4536

之前我介绍了如何使用 **CocoaPods** 来管理第三方库（[Swift - CocoaPods的安装使用详解](https://www.hangge.com/blog/cache/detail_1358.html)），本文介绍另一个第三方库管理工具：**Carthage**。

**1，Carthage介绍**

相较于 **CocoaPods** 的悠久历史，**Carthage** 还比较年轻，出现也没几年（自 **Swift** 语言出来后才有的）。它的目标是用最简单的方式来将第三方框架（**frameworks**）添加到我们项目中来。

**GitHub** 主页地址：https://github.com/Carthage/Carthage.git

**2，Carthage与CocoaPods的区别**

（1）使用 **CocoaPods** 我们只需要修改 **Podfile** 文件，**CocoaPods** 会直接创建和修改项目的 **workspace** 配置。也就是说 **CocoaPods** 创建的是高度集成的项目。

而 **Carthage** 的特点是灵活，耦合度不高，使用时不需要集成相应的 **project**，也不会创建新的 **workspace**。仅仅需要依赖打包好的 **framework** 文件即可。

（2）虽然**CocoaPods** 功能更加强大，但因为国内网络问题，很多库更新不下来（所以前文里，我将其改用淘宝提供的镜像源），或者更新了又用不了，或是打包时出现各种问题。

**Carthage** 只需要从 **github** 上下载项目即可，配置简单，使用的第三方库的时候就像使用苹果原生的 **framework** 一样，干净简洁。


**3，安装Carthage**

（1）在“终端”中运行如下命令更新 **homebrew**

```
brew update
```


（2）安装 **Carthage**

```
brew install carthage
```


（3）安装完毕后执行 **carthage version** 命令可查看版本。

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091314375611435.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

（4）以后如果需要更新Carthage版本，则执行下面命令：

```
brew upgrade carthage
```


**4，Carthage的使用**
（1）首先进入到工程的根目录下，创建空白的 **Cartfile** 文件。

```
cd /Users/hangge/Documents/Code/hangge_1359
touch Cartfile
```

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091315564999345.png)](https://www.hangge.com/blog/cache/detail_1359.html#)


（2）打开 **Cartfile** 文件，写入如下内容。（这里还是以使用 **Alamofire**、**SwiftyJSON** 这两个库为例）

```
github "Alamofire/Alamofire" ~> 3.0
github "SwiftyJSON/SwiftyJSON"
```

（3）保存并关闭 **Cartfile** 文件，执行如下命令。

```
carthage update --platform iOS
```

这时 **Carthage** 就会自动为我们下载和编译所需要的第三方库。



（4）命令执行完毕后，在项目文件夹中会创建一个名为 **Carthage** 的文件夹。

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091316471861923.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

（5）而在 **Carthage/Build/iOS** 文件夹下就是刚创建好的 **framework** 文件。

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091409542219520.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

（6）打开我们的工程项目，将上面的两个 **framework** 拖到 **General** -> **Linked Frameworks and Libraries** 中来。

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091409581437648.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

（7）点击配置页的 **Build Phases** 标签坐上角的加号，添加一个 **Run Script**

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091410015569537.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

（8）将新增的 **Run Script** 做如下修改：

[![原文:Swift - 使用Carthage来安装管理第三方库](https://www.hangge.com/blog_uploads/201609/2016091410100263340.png)](https://www.hangge.com/blog/cache/detail_1359.html#)

**Shell** 下方文本区域输入：**/usr/local/bin/carthage copy-frameworks**

**input Files** 中添加需要导入的库：

**$(SRCROOT)/Carthage/Build/iOS/Alamofire.framework**

**$(SRCROOT)/Carthage/Build/iOS/SwiftyJSON.framework**

（9）最后开发时，我们只需要在使用的时候 **import** 一下需要的库就可以了。

```swift
import UIKit
import Alamofire
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        Alamofire.request(.GET, "http://www.hangge.com")
            .responseString { response in
                print("Success: \(response.result.isSuccess)")
                print("Response String: \(response.result.value)")
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，修改或更新第三方组件库**
（1）如果我们项目引用的库没有改变，只是想要将这些库更新到最新版本，则执行 **update** 命令即可。

```
carthage update --platform iOS
```

（2）如果要新增第三方库，或是删除原有的库。我们先编辑修改 **Cartfile** 文件。再执行执行 **update** 命令。

```
carthage update --platform iOS
```

最后打开项目，修改 **General** -> **Linked Frameworks and Libraries** 中的库引用。 以及 **Run Script** 中的 **input Files** 里相关内容即可。 


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1359.html