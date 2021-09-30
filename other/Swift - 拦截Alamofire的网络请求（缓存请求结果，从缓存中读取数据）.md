# Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）

2016-05-09发布：hangge阅读：5900

在之前的文章：[Swift - 将网页缓存到本地（抓取html页面，并实现离线浏览）](https://www.hangge.com/blog/cache/detail_1118.html)中，介绍了如何通过 **NSURLProtocol** 拦截实现网页的缓存。
当时使用 **WebView** 做演示，通过创建一个拦截类，我们可以将 **WebView** 发起的所有网络请求进行拦截，并将捕获到的url地址与本地缓存里的数据匹配。如果有这个url对应得缓存数据，就直接使用缓存数据。 如果没找到，再进行网络请求，同时将收到的数据缓存起来，供下次使用。
有网友问了，如果是用第三方的框架发起请求（比如 **Alamofire**），这样也能自动拦截吗？
当然是没问题的，只需要简单的配置下就可以了。

**1，演示样例**
我们在 **textField** 中输入 **url**，点击“确定”后会使用 **Alamofire** 进行网络请求，将获取到的数据显示在下方的 **textView** 中。 

[![原文:Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）](https://www.hangge.com/blog_uploads/201605/2016050717364978897.png)](https://www.hangge.com/blog/cache/detail_1171.html#)


**2，使用Core Data进行数据持久化存储**
同[前面的文章](https://www.hangge.com/blog/cache/detail_1118.html)一样，本样例还是使用 **Core Data** 保存缓存数据。定义的实体也是一样的，这里就不再多讲了。

[![原文:Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）](https://www.hangge.com/blog_uploads/201605/2016050420195067872.png)](https://www.hangge.com/blog/cache/detail_1171.html#)


**3，创建拦截类**

拦截类：**MyURLProtocol.swift**，这个同前文里面的还是一样，不需要改变。

```swift
import UIKit
import CoreData
 
//记录请求数量
var requestCount = 0
 
class MyURLProtocol: NSURLProtocol , NSURLSessionDataDelegate, NSURLSessionTaskDelegate{
     
    //NSURLSession数据请求任务
    var dataTask:NSURLSessionDataTask?
    //url请求响应
    var urlResponse: NSURLResponse?
    //url请求获取到的数据
    var receivedData: NSMutableData?
     
     
    //判断这个 protocol 是否可以处理传入的 request
    override class func canInitWithRequest(request: NSURLRequest) -> Bool {
        //对于已处理过的请求则跳过，避免无限循环标签问题
        if NSURLProtocol.propertyForKey("MyURLProtocolHandledKey",
                                        inRequest: request) != nil {
            return false
        }
        return true
    }
     
    //回规范化的请求（通常只要返回原来的请求就可以）
    override class func canonicalRequestForRequest(request: NSURLRequest) -> NSURLRequest {
        return request
    }
     
    //判断两个请求是否为同一个请求，如果为同一个请求那么就会使用缓存数据。
    //通常都是调用父类的该方法。我们也不许要特殊处理。
    override class func requestIsCacheEquivalent(aRequest: NSURLRequest,
                                                 toRequest bRequest: NSURLRequest)
        -> Bool {
        return super.requestIsCacheEquivalent(aRequest, toRequest:bRequest)
    }
     
    //开始处理这个请求
    override func startLoading() {
        requestCount+=1
        print("Request请求\(requestCount): \(request.URL!.absoluteString)")
        //判断是否有本地缓存
        let possibleCachedResponse = self.cachedResponseForCurrentRequest()
        if let cachedResponse = possibleCachedResponse {
            print("----- 从缓存中获取响应内容 -----")
             
            //从本地缓中读取数据
            let data = cachedResponse.valueForKey("data") as! NSData!
            let mimeType = cachedResponse.valueForKey("mimeType") as! String!
            let encoding = cachedResponse.valueForKey("encoding") as! String!
             
            //创建一个NSURLResponse 对象用来存储数据。
            let response = NSURLResponse(URL: self.request.URL!, MIMEType: mimeType,
                                         expectedContentLength: data.length,
                                         textEncodingName: encoding)
             
            //将数据返回到客户端。然后调用URLProtocolDidFinishLoading方法来结束加载。
            //（设置客户端的缓存存储策略.NotAllowed ，即让客户端做任何缓存的相关工作）
            self.client!.URLProtocol(self, didReceiveResponse: response,
                                     cacheStoragePolicy: .NotAllowed)
            self.client!.URLProtocol(self, didLoadData: data)
            self.client!.URLProtocolDidFinishLoading(self)
        } else {
            //请求网络数据
            print("===== 从网络获取响应内容 =====")
             
            let newRequest = self.request.mutableCopy() as! NSMutableURLRequest
            //NSURLProtocol接口的setProperty()方法可以给URL请求添加自定义属性。
            //（这样把处理过的请求做个标记，下一次就不再处理了，避免无限循环请求）
            NSURLProtocol.setProperty(true, forKey: "MyURLProtocolHandledKey",
                                      inRequest: newRequest)
             
            //使用NSURLSession从网络获取数据
            let defaultConfigObj = NSURLSessionConfiguration.defaultSessionConfiguration()
            let defaultSession = NSURLSession(configuration: defaultConfigObj,
                                              delegate: self, delegateQueue: nil)
            self.dataTask = defaultSession.dataTaskWithRequest(newRequest)
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
     
    //NSURLSessionDataDelegate相关的代理方法
    func URLSession(session: NSURLSession, dataTask: NSURLSessionDataTask,
                    didReceiveResponse response: NSURLResponse,
                            completionHandler: (NSURLSessionResponseDisposition) -> Void) {
         
        self.client?.URLProtocol(self, didReceiveResponse: response,
                                 cacheStoragePolicy: .NotAllowed)
        self.urlResponse = response
        self.receivedData = NSMutableData()
        completionHandler(.Allow)
    }
     
    func URLSession(session: NSURLSession, dataTask: NSURLSessionDataTask,
                    didReceiveData data: NSData) {
        self.client?.URLProtocol(self, didLoadData: data)
        self.receivedData?.appendData(data)
    }
     
    //NSURLSessionTaskDelegate相关的代理方法
    func URLSession(session: NSURLSession, task: NSURLSessionTask
        , didCompleteWithError error: NSError?) {
        if error != nil && error!.code != NSURLErrorCancelled {
            self.client?.URLProtocol(self, didFailWithError: error!)
        } else {
            //保存获取到的请求响应数据
            saveCachedResponse()
            self.client?.URLProtocolDidFinishLoading(self)
        }
    }
     
    //保存获取到的请求响应数据
    func saveCachedResponse () {
        print("+++++ 将获取到的数据缓存起来 +++++")
         
        //获得Core Data的NSManagedObjectContext
        let delegate = UIApplication.sharedApplication().delegate as! AppDelegate
        let context = delegate.managedObjectContext
         
        //创建NSManagedObject的实例，来匹配在.xcdatamodeld 文件中所对应的数据模型。
        let cachedResponse = NSEntityDescription
            .insertNewObjectForEntityForName(
                "CachedURLResponse", inManagedObjectContext: context) as NSManagedObject
        cachedResponse.setValue(self.receivedData, forKey: "data")
        cachedResponse.setValue(self.request.URL!.absoluteString, forKey: "url")
        cachedResponse.setValue(NSDate(), forKey: "timestamp")
        cachedResponse.setValue(self.urlResponse?.MIMEType, forKey: "mimeType")
        cachedResponse.setValue(self.urlResponse?.textEncodingName, forKey: "encoding")
         
        //保存（Core Data数据要放在主线程中保存，要不并发是容易崩溃）
        dispatch_async(dispatch_get_main_queue(), {
            do {
                try context.save()
            } catch {
                print("不能保存：\(error)")
            }
        })
    }
     
    //检索缓存请求
    func cachedResponseForCurrentRequest() -> NSManagedObject? {
        //获得managedObjectContext
        let delegate = UIApplication.sharedApplication().delegate as! AppDelegate
        let context = delegate.managedObjectContext
         
        //创建一个NSFetchRequest，通过它得到对象模型实体：CachedURLResponse
        let fetchRequest = NSFetchRequest()
        let entity = NSEntityDescription.entityForName("CachedURLResponse",
                                                       inManagedObjectContext: context)
        fetchRequest.entity = entity
         
        //设置查询条件
        let predicate = NSPredicate(format:"url == %@", self.request.URL!.absoluteString)
        fetchRequest.predicate = predicate
         
        //执行获取到的请求
        do {
            let possibleResult = try context.executeFetchRequest(fetchRequest)
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

不同的是，前文我们在 **AppDelegate.swift** 中的 **didFinishLaunchingWithOptions** 方法里将其注册一下，这里不需要了。

**4，拦截Alamofire请求**
关键点是把我们定义的拦截类添加到 **Session Configuration** 中来（代码高亮部分）

```swift
import UIKit
import Alamofire
 
class ViewController: UIViewController , UITextFieldDelegate {
     
    //网址输入框
    @IBOutlet var textField: UITextField!
    @IBOutlet weak var textView: UITextView!
     
    var manager: Alamofire.Manager?
     
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //添加拦截协议
        let configuration = NSURLSessionConfiguration.defaultSessionConfiguration()
        configuration.protocolClasses!.insert(MyURLProtocol.self, atIndex: 0)
        manager = Manager(configuration: configuration)
    }
     
    //确定按钮点击
    @IBAction func buttonGoClicked(sender: UIButton) {
        if self.textField.isFirstResponder() {
            self.textField.resignFirstResponder()
        }
         self.sendRequest()
    }
     
    //请求数据
    func sendRequest() { 
        if let text = self.textField.text {
            manager?.request(.GET, text)
                .responseString(queue: nil, encoding: NSUTF8StringEncoding) { response in
                    self.textView.text = response.result.value
            }
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**5，开始测试**
（1）由于第一次请求数据时，由于本地都没有找到缓存数据。程序则通过网络请求数据，并缓存起来。

[![原文:Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）](https://www.hangge.com/blog_uploads/201605/2016050717562069640.png)](https://www.hangge.com/blog/cache/detail_1171.html#)


（2）再次访问同一个url，则直接从缓存中读取数据，而不再发起网络请求。

[![原文:Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）](https://www.hangge.com/blog_uploads/201605/2016050717562825221.png)](https://www.hangge.com/blog/cache/detail_1171.html#)



**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1118.zip](https://www.hangge.com/blog_uploads/201605/2016050717564453947.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1171.html