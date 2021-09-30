# Swift - 日志框架XCGLogger的使用详解

2016-12-21发布：hangge阅读：6236

前面我写过两篇文章演示如何封装一个日志函数，实现日志的格式化输出，以及日志文件的保存。

[Swift - 实现日志输出的封装1（显示出调用的文件名、方法、行号）](https://www.hangge.com/blog/cache/detail_1417.html)
[Swift - 实现日志输出的封装2（同步记录到文件中去）](https://www.hangge.com/blog/cache/detail_1482.html)

除了自己实现外，网上还有许多功能更加强大的第三方日志框架供我们使用。本文介绍其中一个比较优秀的日志框架：**XCGLogger**

## 一、XCGLogger的介绍与配置

### 1，什么是XCGLogger

- **XCGLogger** 是一个 **debug** 日志框架，可用于 **Swift** 项目中。
- 使用 **XCGLogger** ，除了可以将日志详细信息输出到控制器台外，还可以输出到指定的文件中去。
- 虽然使用起来同 **NSLog()** 或 **print()** 差不多，但 **XCGLogger** 会附带更多的额外信息，比如：日期、函数名、文件名和行号。

比如我们可以使用 **XCGLogger** 输出如下这种最简单的日志信息：

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/2016122009183228481.png)](https://www.hangge.com/blog/cache/detail_1418.html#)


也可以通过配置，输出下面这种完整的日志信息：

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/2016122009184337605.png)](https://www.hangge.com/blog/cache/detail_1418.html#)





### 2，XCGLogger的安装与配置

（1）从 **GitHub** 上下载最新的代码：https://github.com/DaveWoodCom/XCGLogger
（2）将下载下来的源码包中 **XCGLogger.xcodeproj** 拖拽至你的工程中

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/2016121514135897025.png)](https://www.hangge.com/blog/cache/detail_1418.html#)


（3）工程 -> **General** -> **Embedded Binaries** 项，把 **iOS** 版的 **framework** 添加进来：**XCGLogger.framework**

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/2016121514150930445.png)](https://www.hangge.com/blog/cache/detail_1418.html#)


（4）最后，在需要使用 **XCGLogger** 的地方 **import** 进来就可以了

```
import` `XCGLogger
```



## 二、基本用法

### 1，XCGLogger初始化

（1）我们可以在 **AppDelegate.swift** 中定义一个全局的日志对象，方便使用。
（2）同时对这个日志对象进行初始化。

- 这里我选择打印所有的日志信息（级别、方法名、文件名、行号）。
- 日志除了会打印在控制台中外，还会同步输出到文件中（**Library/Caches/log.txt**）。
- 同时日志在控制台打印和文件输出的消息级别一样，都是 **debug** 级。

```swift
import UIKit
import XCGLogger
 
let log = XCGLogger.default
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
    func application(_ application: UIApplication, didFinishLaunchingWithOptions
        launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
         
        //日志文件地址
        let cachePath = FileManager.default.urls(for: .cachesDirectory,
                                                 in: .userDomainMask)[0]
        let logURL = cachePath.appendingPathComponent("log.txt")
 
        //日志对象设置
        log.setup(level: .debug, showThreadName: true, showLevel: true,
                  showFileNames: true, showLineNumbers: true,
                  writeToFile: logURL, fileLevel: .debug)
         
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



### 2，日志输出

通过 **XCGLogger** 内置的方法，我们可以输出各种消息级别的日志信息。比如下面样例我们打印从低到高各种级别的日志。

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        log.verbose("一条verbose级别消息：程序执行时最详细的信息。")
        log.debug("一条debug级别消息：用于代码调试。")
        log.info("一条info级别消息：常用与用户在console.app中查看。")
        log.warning("一条warning级别消息：警告消息，表示一个可能的错误。")
        log.error("一条error级别消息：表示产生了一个可恢复的错误，用于告知发生了什么事情。")
        log.severe("一条severe error级别消息：表示产生了一个严重错误。程序可能很快会奔溃。")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 3，运行效果

控制台输出如下。可以看到由于我们前面将日志显示级别设置成 **debug**，所以 **verbose** 级别的日志消息就不会打印。 

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/2016121515113411918.png)](https://www.hangge.com/blog/cache/detail_1418.html#)



## 三、更高级的初始化方法

通常情况下我们只需要像上面样例一样，使用 **setup** 方法初始化 **XCGLogger** 对象相关配置即可。方便简单，代码也少。
当然 **XCGLogger** 也支持更加灵活的配置，使得我们日志记录会更加可控。比如：想让控制台显示的信息与日志文件里记录的信息不一样。



```swift
//创建一个logger对象
let log = XCGLogger(identifier: "advancedLogger", includeDefaultDestinations: false)
 
//控制台输出
let systemDestination = AppleSystemLogDestination(identifier: "advancedLogger.systemDestination")
 
//设置控制台输出的各个配置项
systemDestination.outputLevel = .debug
systemDestination.showLogIdentifier = false
systemDestination.showFunctionName = true
systemDestination.showThreadName = true
systemDestination.showLevel = true
systemDestination.showFileName = true
systemDestination.showLineNumber = true
systemDestination.showDate = true
 
//logger对象中添加控制台输出
log.add(destination: systemDestination)
 
//日志文件地址
let cachePath = FileManager.default.urls(for: .cachesDirectory,
                                         in: .userDomainMask)[0]
let logURL = cachePath.appendingPathComponent("log.txt")
//文件出输出
let fileDestination = FileDestination(writeToFile: logURL,
                                      identifier: "advancedLogger.fileDestination")
 
//设置文件输出的各个配置项
fileDestination.outputLevel = .debug
fileDestination.showLogIdentifier = false
fileDestination.showFunctionName = true
fileDestination.showThreadName = true
fileDestination.showLevel = true
fileDestination.showFileName = true
fileDestination.showLineNumber = true
fileDestination.showDate = true
 
//文件输出在后台处理
fileDestination.logQueue = XCGLogger.logQueue
 
//logger对象中添加控制台输出
log.add(destination: fileDestination)
 
//开始启用
log.logAppDetails()
```



## 四、过滤日志信息

### 1，按文件名过滤

（1）下面代码不显示 **AppDelegate.swift** 中输出的日志，其它文件日志都会显示

```swift
log.filters = [FileNameFilter(excludeFrom: ["AppDelegate.swift"],
                              excludePathWhenMatching: true)]
```


（2）下面代码只显示 **AppDelegate.swift** 中输出的日志，其它文件日志都不显示

```swift
log.filters = [FileNameFilter(includeFrom: ["AppDelegate.swift"],
                              excludePathWhenMatching: true)]
```



### 2，按标签过滤

（1）使用自定义的标签字符串。
比如我们在输出一条 **debug** 消息时，附上一个自定义的标签（**sensitive**）

```swift
let tags = XCGLogger.Constants.userInfoKeyTags
log.debug("这里进行用户身份验证。",userInfo: [tags: "sensitive"])
```

下面设置分别是不显示这个标签的日志，以及只显示这个标签的日志。

```swift
//下面代码不显示标签为“sensitive”的日志，其它日志都会显示
log.filters = [TagFilter(excludeFrom: ["sensitive"])]
 
//下面代码只显示标签为“sensitive”的日志，其它日志都不会显示
log.filters = [TagFilter(includeFrom: ["sensitive"])]
```


（2）通过扩展 **Tag** 和 **Dev** 这两个标签结构体
**Tag** 和 **Dev** 是 **XCGLogger** 自带的，我们这里通过扩展，在其上面添加一个些自定义标签。

```swift
extension Tag {
    static let sensitive = Tag("sensitive")
    static let ui = Tag("ui")
    static let data = Tag("data")
}
 
extension Dev {
    static let dave = Dev("dave")
    static let sabby = Dev("sabby")
}
```

我们日志输出时可以混合使用多个标签：

```swift
//设置多个标签
log.debug("A tagged log message", userInfo: Dev.dave | Tag.sensitive)
 
//设置单个标签
log.debug("Another tagged log message", userInfo: Dev.dave.dictionary)
```

下面根据便签进行日志过滤：

```swift
//下面代码不显示标签只为“dave”或“sensitive”的日志，其它日志都会显示
log.filters = [TagFilter(excludeFrom: [Dev.dave, Tag.sensitive])]
 
//下面代码只显示标签为“dave”或“sensitive”的日志，其它日志都不会显示
log.filters = [TagFilter(includeFrom: [Dev.dave, Tag.sensitive])]
```



## 五、选择性的执行代码

有时我们日志输出前要进行一些计算或操作，再将结果通过日志打印出来。由于这些操作与程序实际运行无关，只是用于日志打印，那么我们可以将其放置在日志方法闭包中。这样既保证代码的干净，同时如果日志不输出的话也不会浪费系统资源。
比如下面样例，如果我们将日志消息级别调到 **debug** 以上，那么下面日志不会输出，且内部的循环操作自然也不会执行。

```swift
log.debug {
    var total = 0.0
    for receipt in receipts {
        total += receipt.total
    }
     
    return "Total of all receipts: \(total)"
}
```



## 六、日期格式化

默认日志输出的格式是“**2016-12-15 20:06:27.923**”这种形式。当然我们也可以改成任意的自定义的格式。

```swift
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "MM/dd/yyyy hh:mma"
dateFormatter.locale = Locale.current
log.dateFormatter = dateFormatter
```



## 七、自动切换配置

通过使用 **Swift** 的编译标志（**build flags**）,我们可以在程序调试、生产版本中自动使用不同的日志级别和日志条目。

```swift
#if DEBUG
log.setup(level: .debug, showThreadName: true, showLevel: true,
          showFileNames: true, showLineNumbers: true)
#else
log.setup(level: .severe, showThreadName: true, showLevel: true,
          showFileNames: true, showLineNumbers: true)
#endif
```



## 八、在后台进行日志处理

默认情况下，日志输出是在调用它们的进程里进行的。这样可以确保日志消息可以立刻显示，方便我们使用断点调试时可以立刻看到结果。
但在发布的生产版本中，如果日志也在当前线程中执行会对性能造成一些影响。我们可以指定日志目标进程使用哪个执行队列中。

### 1，下面两种方法都是实现将日志文件输出放在后台线程中进行：

```swift
//方式1
fileDestination.logQueue = XCGLogger.logQueue
 
//方式2
fileDestination.logQueue = DispatchQueue.global(qos: .background)
```



### 2，也可以通过build flags来实现程序在调试、生产自动使用不同线程

```swift
#if DEBUG
    log.setup(level: .debug, showThreadName: true, showLevel: true,
              showFileNames: true, showLineNumbers: true)
#else
    log.setup(level: .severe, showThreadName: true, showLevel: true,
              showFileNames: true, showLineNumbers: true)
    if let consoleLog = log.logDestination(XCGLogger.Constants.baseConsoleDestinationIdentifier)
        as? ConsoleDestination {
        consoleLog.logQueue = XCGLogger.logQueue
    }
#endif
```



## 九、实现日志文件的增量记录

默认情况下，日志文件的输出的是覆盖模式的。也就是说，我们指定了一个日志文件路径，程序每次重新启动时，如果这个文件原来就存在，其内容也会被覆盖。

- 我们可以在日志配置时将 **shouldAppend** 参数设置为 **true**，这样如果原来的文件存在，也不会重新覆盖。而是将新日志信息添加到文件尾部。
- 同时 **appendMarker** 可以设置个字符串，其在每次程序启动时会自动添加到日志文件中。方便我们区分开每次程序运行的日志。



```swift
//创建一个logger对象
let log = XCGLogger(identifier: "advancedLogger", includeDefaultDestinations: false)
 
//控制台输出
let systemDestination = AppleSystemLogDestination(identifier: "advancedLogger.systemDestination")
 
//设置各个配置项
systemDestination.outputLevel = .debug
systemDestination.showLogIdentifier = false
systemDestination.showFunctionName = true
systemDestination.showThreadName = true
systemDestination.showLevel = true
systemDestination.showFileName = true
systemDestination.showLineNumber = true
systemDestination.showDate = true
 
//logger对象中添加控制台输出
log.add(destination: systemDestination)
 
//日志文件地址
let cachePath = FileManager.default.urls(for: .cachesDirectory,
                                         in: .userDomainMask)[0]
let logURL = cachePath.appendingPathComponent("log.txt")
//文件出输出
let fileDestination = FileDestination(writeToFile: logURL,
                                      identifier: "advancedLogger.fileDestination",
                                      shouldAppend: true, appendMarker: "-- Relauched App --")
 
//设置各个配置项
fileDestination.outputLevel = .debug
fileDestination.showLogIdentifier = false
fileDestination.showFunctionName = true
fileDestination.showThreadName = true
fileDestination.showLevel = true
fileDestination.showFileName = true
fileDestination.showLineNumber = true
fileDestination.showDate = true
 
//文件输出在后台处理
fileDestination.logQueue = XCGLogger.logQueue
 
//logger对象中添加控制台输出
log.add(destination: fileDestination)
 
//开始启用
log.logAppDetails()
```


比如我们这里一共启动了三次应用，日志文件的内容如下：

[![原文:Swift - 日志框架XCGLogger的使用详解](https://www.hangge.com/blog_uploads/201612/201612161101011377.png)](https://www.hangge.com/blog/cache/detail_1418.html#)



## 十，实现日志文件的转储

上面的样例中，我们都是将日志记录到 **Caches** 文件夹下的 **log.txt**。如果是增量日志的话，那个这个文件就会越来越大。所以我们可以通过 **rotateFile(to:)** 方法实现日志的转储。比如：每隔一段时间，或每当日志文件到一定大小时调用下该方法，将日志内容转储到另一个文件中。（注意：转储后原来日志文件里的内容会清空。）
具体转储操作的步骤如下：

- 首先通过 **XCGLogger** 的 **destination(withIdentifier:)** 方法获取 **FileDestination** 对象。
- 再调用 **FileDestination** 对象的 **rotateFile(to:)** 方法将日志文件转储到指定路径。
- **rotateFile(to:)** 方法执行成功后会返回 **true**。根据结果我们可以进行一些后续操作。比如：压缩、邮件发送日志文件等等。

下面代码我们将日志文件转储到用户文档目录下：

```swift
let fileDestination = log.destination(withIdentifier: "advancedLogger.fileDestination")
    as! FileDestination
 
//获取用户文档目录
let documentPath = FileManager.default.urls(for: .documentDirectory,
                                         in: .userDomainMask)[0]
let logURL = documentPath.appendingPathComponent("log.txt")
 
//将当前的日志文件复制到用户文档目录中去
fileDestination.rotateFile(to: logURL)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1418.html