# Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）

2016-05-05发布：hangge阅读：6949

（本文代码已升级至Swift3）

市面上有许多新闻客户端提供离线阅读的功能，使用者可以先把新闻页面下载到本地，然后在没有网络的情况下实现离线浏览。或者在使用wifi的情况下缓存一些页面，这样当使用手机网络的时候就不要再进行网络请求了，直接读取本读缓存，不仅提高响应速度，还能节约流量。

本文介绍如何实现将网页缓存到本地，然后再从缓存中读取出来。

### 1，实现原理 

整个应用使用的核心技术是：**URLProtocol** 拦截。通过 **URLProtocol** 拦截，我们可以对所有的网络请求进行捕获并处理。
比如，我们通过 **WebView** 访问一个网页，首先将捕获到的url地址与本地缓存里的数据匹配。如果有这个url对应得缓存数据，就直接使用缓存数据。
如果没找到，再进行网络请求，同时将收到的数据缓存起来，供下次使用。
前面说到，通过 **URLProtocol** 是可以拦截到所有的请求，也就是说不管是html页面，还是里面用到的css、js、图片等文件都是可以拦截并缓存的。

###  2，使用Core Data进行数据持久化存储

本样例使用 **Core Data** 来缓存网页数据，如果对 **Core Data** 不太熟悉的话可以参考我原来写的这篇文章：[Swift - 使用Core Data进行数据持久化存储](https://www.hangge.com/blog/cache/detail_767.html)
（1）创建项目的时候，勾选“**Use Core Data**”

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/201605042018479834.png)](https://www.hangge.com/blog/cache/detail_1118.html#)


（2）打开项目中的 **xcdatamodeld** 文件，添加一个实体“**CachedURLResponse**”，并在并在 **Attribute** 栏目中添加如下属性：

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420195067872.png)](https://www.hangge.com/blog/cache/detail_1118.html#)



**data**：内容数据
**encoding**：响应编码
**mimeType**：响应数据类型
**timestamp**：时间戳（记录缓存的时间）
**url**：请求地址

###  3，创建拦截类

（1）通过继承 **URLProtocol**，我们创建一个拦截类：**MyURLProtocol.swift**

```swift
import UIKit
import CoreData
 
//记录请求数量
var requestCount = 0
 
class MyURLProtocol: URLProtocol , URLSessionDataDelegate, URLSessionTaskDelegate{
     
    //URLSession数据请求任务
    var dataTask:URLSessionDataTask?
    //url请求响应
    var urlResponse: URLResponse?
    //url请求获取到的数据
    var receivedData: NSMutableData?
     
     
    //判断这个 protocol 是否可以处理传入的 request
    override class func canInit(with request: URLRequest) -> Bool {
        //对于已处理过的请求则跳过，避免无限循环标签问题
        if URLProtocol.property(forKey: "MyURLProtocolHandledKey", in: request) != nil {
            return false
        }
        return true
    }
     
    //回规范化的请求（通常只要返回原来的请求就可以）
    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        return request
    }
     
    //判断两个请求是否为同一个请求，如果为同一个请求那么就会使用缓存数据。
    //通常都是调用父类的该方法。我们也不许要特殊处理。
    override class func requestIsCacheEquivalent(_ aRequest: URLRequest,
                                                 to bRequest: URLRequest) -> Bool {
        return super.requestIsCacheEquivalent(aRequest, to:bRequest)
    }
     
    //开始处理这个请求
    override func startLoading() {
        requestCount+=1
        print("Request请求\(requestCount): \(request.url!.absoluteString)")
        //判断是否有本地缓存
        let possibleCachedResponse = self.cachedResponseForCurrentRequest()
        if let cachedResponse = possibleCachedResponse {
            print("----- 从缓存中获取响应内容 -----")
             
            //从本地缓中读取数据
            let data = cachedResponse.value(forKey: "data") as! Data!
            let mimeType = cachedResponse.value(forKey: "mimeType") as! String!
            let encoding = cachedResponse.value(forKey: "encoding") as! String!
             
            //创建一个NSURLResponse 对象用来存储数据。
            let response = URLResponse(url: self.request.url!, mimeType: mimeType,
                                         expectedContentLength: data!.count,
                                         textEncodingName: encoding)
             
            //将数据返回到客户端。然后调用URLProtocolDidFinishLoading方法来结束加载。
            //（设置客户端的缓存存储策略.NotAllowed ，即让客户端做任何缓存的相关工作）
            self.client!.urlProtocol(self, didReceive: response,
                                     cacheStoragePolicy: .notAllowed)
            self.client!.urlProtocol(self, didLoad: data!)
            self.client!.urlProtocolDidFinishLoading(self)
        } else {
            //请求网络数据
            print("===== 从网络获取响应内容 =====")
             
            let newRequest = (self.request as NSURLRequest).mutableCopy() as! NSMutableURLRequest
            //NSURLProtocol接口的setProperty()方法可以给URL请求添加自定义属性。
            //（这样把处理过的请求做个标记，下一次就不再处理了，避免无限循环请求）
            URLProtocol.setProperty(true, forKey: "MyURLProtocolHandledKey",
                                      in: newRequest)
             
            //使用URLSession从网络获取数据
            let defaultConfigObj = URLSessionConfiguration.default
            let defaultSession = Foundation.URLSession(configuration: defaultConfigObj,
                                              delegate: self, delegateQueue: nil)
            self.dataTask = defaultSession.dataTask(with: self.request)
            self.dataTask!.resume()
        }
    }
     
    //结束处理这个请求
    override func stopLoading() {
        self.dataTask?.cancel()
        self.dataTask       = nil
        self.receivedData   = nil
        self.urlResponse    = nil
    }
     
    //URLSessionDataDelegate相关的代理方法
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask,
                    didReceive response: URLResponse,
                            completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
         
        self.client?.urlProtocol(self, didReceive: response,
                                 cacheStoragePolicy: .notAllowed)
        self.urlResponse = response
        self.receivedData = NSMutableData()
        completionHandler(.allow)
    }
     
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask,
                    didReceive data: Data) {
        self.client?.urlProtocol(self, didLoad: data)
        self.receivedData?.append(data)
    }
     
    //URLSessionTaskDelegate相关的代理方法
    func urlSession(_ session: URLSession, task: URLSessionTask
        , didCompleteWithError error: Error?) {
        if error != nil {
            self.client?.urlProtocol(self, didFailWithError: error!)
        } else {
            //保存获取到的请求响应数据
            saveCachedResponse()
            self.client?.urlProtocolDidFinishLoading(self)
        }
    }
     
    //保存获取到的请求响应数据
    func saveCachedResponse () {
        print("+++++ 将获取到的数据缓存起来 +++++")
         
        //获取管理的数据上下文 对象
        let app = UIApplication.shared.delegate as! AppDelegate
        let context = app.persistentContainer.viewContext
         
        //创建NSManagedObject的实例，来匹配在.xcdatamodeld 文件中所对应的数据模型。
        let cachedResponse = NSEntityDescription
            .insertNewObject(forEntityName: "CachedURLResponse",
                                             into: context) as NSManagedObject
        cachedResponse.setValue(self.receivedData, forKey: "data")
        cachedResponse.setValue(self.request.url!.absoluteString, forKey: "url")
        cachedResponse.setValue(Date(), forKey: "timestamp")
        cachedResponse.setValue(self.urlResponse?.mimeType, forKey: "mimeType")
        cachedResponse.setValue(self.urlResponse?.textEncodingName, forKey: "encoding")
         
        //保存（Core Data数据要放在主线程中保存，要不并发是容易崩溃）
        DispatchQueue.main.async(execute: {
            do {
                try context.save()
            } catch {
                print("不能保存：\(error)")
            }
        })
    }
     
    //检索缓存请求
    func cachedResponseForCurrentRequest() -> NSManagedObject? {
        //获取管理的数据上下文 对象
        let app = UIApplication.shared.delegate as! AppDelegate
        let context = app.persistentContainer.viewContext
         
        //创建一个NSFetchRequest，通过它得到对象模型实体：CachedURLResponse
        let fetchRequest = NSFetchRequest<NSFetchRequestResult>()
        let entity = NSEntityDescription.entity(forEntityName: "CachedURLResponse",
                                                       in: context)
        fetchRequest.entity = entity
         
        //设置查询条件
        let predicate = NSPredicate(format:"url == %@", self.request.url!.absoluteString)
        fetchRequest.predicate = predicate
         
        //执行获取到的请求
        do {
            let possibleResult = try context.fetch(fetchRequest)
                as? Array<NSManagedObject>
            if let result = possibleResult {
                if !result.isEmpty {
                    return result[0]
                }
            }
        }
        catch {
            print("获取缓存数据失败：\(error)")
        }
        return nil
    }
}
```

（2）在 **AppDelegate.swift** 中的 **didFinishLaunchingWithOptions** 方法里将其注册一下。
这样当运行程序后，**MyURLProtocol** 将会处理每一个请求然后交付给 **URL Loading System**。包括代码直接调用 **loading system**，以及许多系统组件依赖于加载框架的URL,比如 **UIWebView**。

```swift
import UIKit
import CoreData
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
 
    func application(application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        // Override point for customization after application launch.
        //注册URL Loading System协议，让每一个请求都会经过MyURLProtocol处理
        URLProtocol.registerClass(MyURLProtocol.self)
        return true
    }
 
    //................
```

###  4，测试页面

（1）我们在 **textField** 中输入 **url**，点击“确定”后会使用下方的 **webView** 加载对应的页面。

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420204916239.png)](https://www.hangge.com/blog/cache/detail_1118.html#)



（2）页面对应代码如下，这个同以前我们使用 **webView** 没什么不同。（实际上程序在后台已经默默地进行请求的拦截与缓存等操作了）

```swift
import UIKit
 
class ViewController: UIViewController , UITextFieldDelegate {
     
    //网址输入框
    @IBOutlet var textField: UITextField!
    @IBOutlet var webView: UIWebView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //确定按钮点击
    @IBAction func buttonGoClicked(_ sender: UIButton) {
        if self.textField.isFirstResponder {
            self.textField.resignFirstResponder()
        }
        self.sendRequest()
    }
     
    //键盘确定按钮点击
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        self.sendRequest()
        return true
    }
     
    //请求页面
    func sendRequest() {
        if let text = self.textField.text {
            let url = URL(string:text)
            let request = URLRequest(url:url!)
            self.webView.loadRequest(request)
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

###  5，开始测试

（1）访问[www.hangge.com](http://www.hangge.com/)

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420221064020.png)](https://www.hangge.com/blog/cache/detail_1118.html#)



（2）可以看到控制台会打印出所有的request请求，由于第一次访问，所以本地都没有找到缓存数据。程序则通过网络请求数据，并缓存起来。

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420232014278.png)](https://www.hangge.com/blog/cache/detail_1118.html#)



（3）接着再多访问几个页面（不管是点击页面跳转，还是重新输入url请求都可以）
  [![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420263846461.png)](https://www.hangge.com/blog/cache/detail_1118.html#)  [![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420264793595.png)](https://www.hangge.com/blog/cache/detail_1118.html#)

（4）接着关闭网络，访问之前访问过的页面。可以看到在没有网络的情况下页面仍能加载出来。

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420221064020.png)](https://www.hangge.com/blog/cache/detail_1118.html#)

（5）查看控制台发现，这些request请求都从本地的缓存中取得数据（不断网络的话也会从缓存中取数据）

[![原文:Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog_uploads/201605/2016050420353497085.png)](https://www.hangge.com/blog/cache/detail_1118.html#)

**源码下载：**![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1118.zip](https://www.hangge.com/blog_uploads/201705/2017050816444338113.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1118.html