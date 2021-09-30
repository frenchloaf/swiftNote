# Swift - 实现日志输出的封装1（显示出调用的文件名、方法、行号）

2016-12-16发布：hangge阅读：2636

在开发调试程序时，我们少不了使用 **print** 方法进行日志打印。当然简单地调试使用 **print** 方法就够了，但如果日志输出的地方很多，就不好区分出每条日志具体是在哪里打印的。

本文对日志打印功能做个封装，自动实现日志信息的格式化。

### 1，效果图

从下图可以看出，控制台除了输出我们指定的日志内容外，还会自动记录日志触发点的文件名、函数名、行号。

[![原文:Swift - 实现日志输出的封装1（显示出调用的文件名、方法、行号）](https://www.hangge.com/blog_uploads/201612/2016121417105854141.png)](https://www.hangge.com/blog/cache/detail_1417.html#)

### 2，日志封装代码

由于 **Swift** 支持全局函数，全局函数可以在当前所在的命名空间下随意调用。所以我们只需要定义一个全局函数即可。

```swift
//封装的日志输出功能（T表示不指定日志信息参数类型）
func HGLog<T>(_ message:T, file:String = #file, function:String = #function,
           line:Int = #line) {
    #if DEBUG
        //获取文件名
        let fileName = (file as NSString).lastPathComponent
        //打印日志内容
        print("\(fileName):\(line) \(function) | \(message)")
    #endif
}
```

### 3，使用样例

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        HGLog("程序启动！")
        doSomething()
    }
     
    func doSomething() {
        HGLog("欢迎访问hangge.com")
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1417.html