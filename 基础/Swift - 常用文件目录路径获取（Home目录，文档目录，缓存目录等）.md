# Swift - 常用文件目录路径获取（Home目录，文档目录，缓存目录等）

2015-06-15发布：hangge阅读：16121

iOS应用程序只能在自己的目录下进行文件的操作，不可以访问其他的存储空间，此区域被称为沙盒。下面介绍常用的程序文件夹目录：

**1，Home目录  ./**

整个应用程序各文档所在的目录

```swift
//获取程序的Home目录
let homeDirectory = NSHomeDirectory()
```

**2，Documnets目录  ./Documents**

用户文档目录，苹果建议将程序中建立的或在程序中浏览到的文件数据保存在该目录下，iTunes备份和恢复的时候会包括此目录

```swift
//方法1
let documentPaths = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.DocumentDirectory,
    NSSearchPathDomainMask.UserDomainMask, true)
let documnetPath = documentPaths[0] as! String
 
//方法2
let ducumentPath2 = NSHomeDirectory() + "/Documents"
```

**3，Library目录  ./Library**

这个目录下有两个子目录：Caches 和 Preferences

Library/Preferences目录，包含应用程序的偏好设置文件。不应该直接创建偏好设置文件，而是应该使用NSUserDefaults类来取得和设置应用程序的偏好。

Library/Caches目录，主要存放缓存文件，iTunes不会备份此目录，此目录下文件不会再应用退出时删除



```swift
//Library目录－方法1
let libraryPaths = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.LibraryDirectory,
    NSSearchPathDomainMask.UserDomainMask, true)
let libraryPath = libraryPaths[0] as! String
 
//Library目录－方法2
let libraryPath2 = NSHomeDirectory() + "/Library"
 
 
//Cache目录－方法1
let cachePaths = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.CachesDirectory,
    NSSearchPathDomainMask.UserDomainMask, true)
let cachePath = cachePaths[0] as! String
 
//Cache目录－方法2
let cachePath2 = NSHomeDirectory() + "/Library/Caches"
```

**4，tmp目录  ./tmp**

用于存放临时文件，保存应用程序再次启动过程中不需要的信息，重启后清空。

```swift
//方法1
let tmpDir = NSTemporaryDirectory()
 
//方法2
let tmpDir2 = NSHomeDirectory() + "/tmp"
```


**5，程序打包安装的目录 NSBundle.mainBundle()**
工程打包安装后会在NSBundle.mainBundle()路径下，该路径是只读的，不允许修改。
所以当我们工程中有一个SQLite数据库要使用，在程序启动时，我们可以把该路径下的数据库拷贝一份到Documents路径下，以后整个工程都将操作Documents路径下的数据库。

```swift
//声明一个Documents下的路径
var dbPath = NSHomeDirectory() + "/Documents/hanggeDB.sqlite"
//判断数据库文件是否存在
if !NSFileManager.defaultManager().fileExistsAtPath(dbPath){
    //获取安装包内数据库路径
    var bundleDBPath:String? = NSBundle.mainBundle().pathForResource("hanggeDB", ofType: "sqlite")
    //将安装包内数据库拷贝到Documents目录下
    NSFileManager.defaultManager().copyItemAtPath(bundleDBPath!, toPath: dbPath, error: nil)
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_765.html