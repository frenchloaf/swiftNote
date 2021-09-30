# Swift - 实现日志输出的封装2（同步记录到文件中去）

2016-12-19发布：hangge阅读：2740

在上一篇文章中：[Swift - 实现日志输出的封装1（显示出调用的文件名、方法、行号）](https://www.hangge.com/blog/cache/detail_1417.html)。我演示了如何封装实现一个日志输出的全局函数。但由于日志都是直接打印到控制台上，如果程序重启后原先的日志信息就会被清空。

文本对日志封装方法做个功能改进，日志除了输出到控制台外，还会同时写到本地文件中去。

###  1，日志封装代码

- 这里我将日志信息写到应用的 **Caches** 文件夹中（**Library/Caches/log.txt**）
- 为了让日志信息更加丰富，再将日志写到文件中时还会附上调用的日期时间。

```swift
//封装的日志输出功能（T表示不指定日志信息参数类型）
func HGLog<T>(_ message:T, file:String = #file, function:String = #function,
           line:Int = #line) {
    #if DEBUG
        //获取文件名
        let fileName = (file as NSString).lastPathComponent
        //日志内容
        let consoleStr = "\(fileName):\(line) \(function) | \(message)"
        //打印日志内容
        print(consoleStr)
         
        // 创建一个日期格式器
        let dformatter = DateFormatter()
        // 为日期格式器设置格式字符串
        dformatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        // 使用日期格式器格式化当前日期、时间
        let datestr = dformatter.string(from: Date())
         
        //将内容同步写到文件中去（Caches文件夹下）
        let cachePath = FileManager.default.urls(for: .cachesDirectory,
                                                  in: .userDomainMask)[0]
        let logURL = cachePath.appendingPathComponent("log.txt")
        appendText(fileURL: logURL, string: "\(datestr) \(consoleStr)")
    #endif
}
 
//在文件末尾追加新内容
func appendText(fileURL: URL, string: String) {
    do {
        //如果文件不存在则新建一个
        if !FileManager.default.fileExists(atPath: fileURL.path) {
            FileManager.default.createFile(atPath: fileURL.path, contents: nil)
        }
         
        let fileHandle = try FileHandle(forWritingTo: fileURL)
        let stringToWrite = "\n" + string
         
        //找到末尾位置并添加
        fileHandle.seekToEndOfFile()
        fileHandle.write(stringToWrite.data(using: String.Encoding.utf8)!)
         
    } catch let error as NSError {
        print("failed to append: \(error)")
    }
}
```



### 2，使用样例

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



### 3，运行效果

（1）我这里运行了两次程序，可以看到 **Caches** 文件夹下已经生成了日志文件。

[![原文:Swift - 实现日志输出的封装2（同步记录到文件中去）](https://www.hangge.com/blog_uploads/201612/2016121416511694033.png)](https://www.hangge.com/blog/cache/detail_1482.html#)


（2）打开日志文件，其内容如下。

[![原文:Swift - 实现日志输出的封装2（同步记录到文件中去）](https://www.hangge.com/blog_uploads/201612/2016121417133289577.png)](https://www.hangge.com/blog/cache/detail_1482.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1482.html