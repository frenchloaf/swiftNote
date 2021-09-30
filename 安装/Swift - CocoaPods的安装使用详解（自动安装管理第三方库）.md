# Swift - CocoaPods的安装使用详解（自动安装管理第三方库）

2016-09-12发布：hangge阅读：5097

我们开发的时候，常常需要引入一些第三方库（比如：**Alamofire**、**SwiftyJSON** 等等）。过去的做法是把这些库下载下来，并引入到工程中。如果有依赖其他库的话，还要手动将这些依赖库给添加进来。关键是如果这些第三方库后面有更新的话，我们还要先把项目中原来的库给删除。再重复前面的步骤。这样就很麻烦了。

而使用 **CocoaPods** 以后，这些工作我们都不需要做。只需做好配置工作，安装更新这些第三方库，**CocoaPods** 都会自动帮我们做好。（本文代码现已升级至Swift3）

**一、CocoaPods介绍**

（1）**CocoaPods** 是 **ios** 开发中的第三方库管理的工具。目的是让我们能自动化的、集中的、直观的管理第三方开源库。

（2）**CocoaPods** 能够自动解决库与库之间的依赖关系，下载库的源代码。并创建一个 **Xcode** 的 **workspace** 来将这些第三方库和我们的工程连接起来，供我们开发使用。

**二、安装CocoaPods**

**1，将gem升级为最新版本**

```bash
sudo gem update --system
```



**2，运行如下命令安装CocoaPods**

```bash
sudo gem install -n /usr/local/bin cocoapods
```

接着执行如下命令，如果没有报错，就说明一切安装成功了。（这个执行后要等上一段时间，**Cocoapods** 在将它的信息下载到 **~/.cocoapods** 里。大概有100多兆。）

```bash
pod setup
```


**3，安装后执行如下命令查看版本**

```bash
pod --version
```

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091108514529200.png)](https://www.hangge.com/blog/cache/detail_1358.html#)


**4，以后要更新升级CocoaPods，执行如下命令**

```bash
sudo gem update cocoapods
```

***\*
\**三、CocoaPods安装失败的原因和解决办法**
**1，执行install后一直没反应，无法安装**
有时你会发现执行 **install** 命令后一直没有反应。这是因为 **Ruby** 的默认源使用的是 **rubygems.org**。由于国内网络原因，访问这上面的资源文件会间歇性连接失败。

（1）我们可以执行如下命令，将源改成使用淘宝提供的 **rubygems.org** 镜像 

```bash
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
```

（1）由于淘宝镜像已停止维护，所以不再使用淘宝镜像。而是改为执行如下命令，将源改成使用 **RubyGems** 镜像。

```bash
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
```


（2）接着执行下面命令，确保只有 **gems.ruby-china.org** （确保旧版本镜像都要移除，比如原先如果有淘宝镜像也要删除。）

```bash
gem sources -l
```

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201612/2016121116313364008.png)](https://www.hangge.com/blog/cache/detail_1358.html#)



（3）最后再使用 **install** 就能成功安装 **CocoaPods** 了。

**2，Ruby版本太低**
如果本机 **Ruby** 的版本过低也可能导致安装失败。 

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091014281820647.png)](https://www.hangge.com/blog/cache/detail_1358.html#)

（1）安装 **rvm**（**rvm** 可以让我们拥有多个版本的 **Ruby**，并且可以在多个版本之间自由切换。）

```bash
curl -L get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
```

（2）完毕后执行如下命令，如果出现版本号则说明 **rvm** 安装成功。

```
rvm -v
```

（3）执行 **rvm list known** 命令可以列出所有 **ruby** 可安装的版本信息。 

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091014415345001.png)](https://www.hangge.com/blog/cache/detail_1358.html#)


（4）我们安装一个 **ruby** 版本

```bash
rvm install 2.2.4
```

（5）并将其设置为默认版本。这时我们应该就可以正常安装 **CocoaPods** 了。

```bash
rvm use 2.2.4 --default
```

（6）使用 **rvm list** 命令可以查看已安装的 **ruby**

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091108461496100.png)](https://www.hangge.com/blog/cache/detail_1358.html#)

（7）如果要卸载一个已安装 **ruby** 版本，执行如下命令。

```bash
rvm remove 2.1.4
```

***\*
\**四、CocoaPods的使用**
下面我们创建一个工程项目（假设工程名是 **hangge_1358**），演示如果使用 **CocoaPods** 自动下载、导入、配置第三方库。

**1，创建Podfile**

首先进入到工程的根目录下，创建空白的**Podfile** 文件。

```bash
cd /Users/hangge/Documents/Code/hangge_1358
touch Podfile
```

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091109030664023.png)](https://www.hangge.com/blog/cache/detail_1358.html#)

**2，编辑Podfile**

我们在 **Podfile** 文件中写上需要引入的第三方库，这里以 **Alamofire**、**SwiftyJSON** 这两个库为例。
（注意：**Alamofire** 自**4.0**版本起，**SwiftyJSON** 自**3.0**版本起，都可以支持 **Swift3**。）

```bash
platform :ios, '9.0'
use_frameworks!
 
target 'hangge_1358' do
    pod 'Alamofire', '~> 4.0'
    pod 'SwiftyJSON', '~> 3.0'
end
```


**3，开始导入库**
（1）执行下面命令，开始导入前面配置的第三方库。

```bash
cd /Users/hangge/Documents/Code/hangge_1358
pod install
```

（2）执行成功后。看到工程根目录下多了三个新文件：**hangge_1358.xcworkspace**、**Podfile.lock** 文件、**Pods** 目录。

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091110414973682.png)](https://www.hangge.com/blog/cache/detail_1358.html#)

**4，打开新生成的hangge_1358.xcworkspace**

（1）往后我们就需要使用这个新生成的 **hangge_1358.xcworkspace** 文件来开发。因为原来的工程（**hangge_1358.xcodeproj**）设置已经被更改了，如果我们直接打开原来的工程文件去编译就会报错。

（2）打开后可以看到里面除了我们项目工程，还有一个 **Pods** 工程。整个结构还是很清晰明了的。

[![原文:Swift - CocoaPods的安装使用详解（自动安装管理第三方库）](https://www.hangge.com/blog_uploads/201609/2016091110181066927.png)](https://www.hangge.com/blog/cache/detail_1358.html#)

**5，测试代码**

开发时，我们只需要在使用的时候 **import** 一下需要的库就可以了。

```swift
import UIKit
import Alamofire
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        Alamofire.request("http://www.hangge.com")
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


**6，新增或移除第三方库**
以后如果需要在工程中导入新库，或者移除原有的库。还是先编辑 **Podfile** 文件，再执行 **install** 命令。

```bash
pod install
```


**7，更新第三方库**
如果引用的库没有改变，只是想要将这些库更新到最新版本，则执行 **update** 命令。

```bash
pod update
```


**五、导入Objective-C写的第三方库**
上面的样例我们是创建工 **Swift** 工程，并导入 **Swift** 语言写的第三方库。如果想要在 **Swift** 工程中引入 **OC** 写的第三方库，操作也是一样的。而且我们项目中不需要像过去一样创建桥接头文件，把Objective-C头文件引用进来。

这里以 **OC** 写的图片缓存库：**SDWebImage** 为例说明。
**1，编辑Podfile文件**

```bash
platform :ios, '9.0'
use_frameworks!
 
target 'hangge_1358' do
    pod 'Alamofire', '~> 4.0'
    pod 'SwiftyJSON', '~> 3.0'
    pod 'SDWebImage'
end
```

**2，执行安装命令**

```bash
pod install
```

**3，代码中直接将SDWebImage库import就能使用**

```swift
import SDWebImage
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let imageView = UIImageView()
        imageView.frame = CGRect(x:10, y:10, width:130, height:27.5)
        let url = URL(string: "http://www.hangge.com/blog/images/logo.png")
        imageView.sd_setImage(with: url)
        self.view.addSubview(imageView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1358.html