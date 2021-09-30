# Swift - 使用vapor socks库进行socket通信（基于TCP、UDP协议）

2017-04-26发布：hangge阅读：4470

之前我介绍了如何使用 **SwiftSocket** 库实现 **socket** 通信：[Swift - 使用socket进行通信（附聊天室样例）](https://www.hangge.com/blog/cache/detail_756.html)

本文介绍另一个 **socket** 库：**vapor** 的 **socks** 库。个人感觉这个用起来更加简单方便，而且原生就有 **receiveAll** 方法，方便我们发送和接收不定长消息。

## 一、安装配置

（1）从 **Github** 主页下载代码：https://github.com/vapor/socks

（2）将下载下来的 **Socks** 文件夹添加到项目中。

[![原文:Swift - 使用vapor socks库进行socket通信（基于TCP、UDP协议）](https://www.hangge.com/blog_uploads/201703/2017030915411412547.png)](https://www.hangge.com/blog/cache/detail_1588.html#)



## 二、基于TCP协议的socket通信

### 1，效果图

（1）程序运行后会启动一个 **tcp** 服务器，监听 **8080** 端口。

（2）同时还会初始化一个 **tcp** 客户端。在输入框中填写内容后，点击“**发送**”按钮即可将消息发送到服务端（本文样例就是自己发给自己）。

（3）服务端接收到消息后会显示在界面上，同时接收到的消息又返回客户端。

（4）客户端收到反馈消息后同样会将其显示在界面上。

[![原文:Swift - 使用vapor socks库进行socket通信（基于TCP、UDP协议）](https://www.hangge.com/blog_uploads/201703/201703091713521009.png)](https://www.hangge.com/blog/cache/detail_1588.html#)



### 2，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //发送消息输入框
    @IBOutlet weak var textField: UITextField!
     
    //用于显示接收到的消息
    @IBOutlet weak var textView: UITextView!
     
    //TCP服务端
    var server:SynchronousTCPServer!
     
    //TCP客户端
    lazy var client:TCPClient? = {
        //初始化客户端
        let address = InternetAddress(hostname: "127.0.0.1", port: 8080)
        do {
            return try TCPClient(address: address)
        } catch {
            print("Error \(error)")
            return nil
        }
    }()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //启动服务器
        startServer()
    }
     
    //启动服务器
    func startServer() {
        //在后台线程中启动服务器
        DispatchQueue.global(qos: .background).async {
            do {
                //初始化服务器
                self.server = try SynchronousTCPServer(port: 8080)
                
                //在界面上显示启动信息
                DispatchQueue.main.async {
                    let hostname = self.server.address.hostname
                    let address = self.server.address.addressFamily
                    let port = self.server.address.port
                    self.textView.text = "服务器启动，监听："
                                        + "\"\(hostname)\" (\(address)) \(port)\n"
                }
 
                //接收并处理客户端连接
                try self.server.startWithHandler { (client) in
                    self.handleClient(client: client)
                }
            } catch {
                print("Error \(error)")
            }
        }
    }
     
    //处理连接的客户端
    func handleClient(client:TCPClient){
        do {
            while true{
                //获取客户端发送过来的消息：[UInt8]类型
                let data = try client.receiveAll()
                 
                //将接收到的消息转成String类型
                let str = try data.toString()
                //将这个String消息显示到界面上
                DispatchQueue.main.async {
                    self.textView.text = self.textView.text + "服务端接收到消息: \(str)\n"
                }
 
                //将接收到的消息又发回客户端
                try client.send(bytes: data)
                 
                //try client.close() //关闭与客户端链接
            }
        } catch {
            print("Error \(error)")
        }
    }
     
    //发送消息
    @IBAction func sendMessage(_ sender: Any) {
        do {
            let message = self.textField.text
            if message != nil && message != "" {
                try client?.send(bytes: message!.toBytes())
                let str = try client!.receiveAll().toString()
                //将服务端返回的消息显示在界面上
                self.textView.text = self.textView.text + "客户端接收到反馈: \(str)\n"
                //try client.close() //关闭客户端与服务端链接
                 
                //清空输入框
                self.textField.text = ""
            }
        } catch {
            print("Error \(error)")
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1588.zip](https://www.hangge.com/blog_uploads/201703/2017030919010798677.zip)

## 三、基于UDP协议的socket通信

由于 **UDP** 不像 **TCP** 那样需要三次握手，所以不需要建立连接通道，也没有断开连接之说，发送完了也就完了。因此代码简单许多。

### 1，效果图

（1）功能和上面的大体一样。程序运行后会启动一个 **udp** 服务器，监听 **8080** 端口。

（2）同时还会初始化一个 **udp** 客户端。在输入框中填写内容后，点击“**发送**”按钮即可将消息发送到服务端（本文样例就是自己发给自己）。

（3）服务端接收到消息后会显示在界面上，同时接收到的消息又返回客户端。

（4）客户端收到反馈消息后同样会将其显示在界面上。

[![原文:Swift - 使用vapor socks库进行socket通信（基于TCP、UDP协议）](https://www.hangge.com/blog_uploads/201703/201703091713521009.png)](https://www.hangge.com/blog/cache/detail_1588.html#)



### 2，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //发送消息输入框
    @IBOutlet weak var textField: UITextField!
     
    //用于显示接收到的消息
    @IBOutlet weak var textView: UITextView!
     
    //UDP服务端
    var server:SynchronousUDPServer!
     
    //UDP客户端
    lazy var client:UDPClient? = {
        //初始化客户端
        let address = InternetAddress(hostname: "127.0.0.1", port: 8080)
        do {
            return try UDPClient(address: address)
        } catch {
            print("Error \(error)")
            return nil
        }
    }()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //启动服务器
        startServer()
    }
     
    //启动服务器
    func startServer() {
        //在后台线程中启动服务器
        DispatchQueue.global(qos: .background).async {
            do {
                //初始化服务器
                self.server = try SynchronousUDPServer(port: 8080)
                 
                //在界面上显示启动信息
                DispatchQueue.main.async {
                    let hostname = self.server.address.hostname
                    let address = self.server.address.addressFamily
                    let port = self.server.address.port
                    self.textView.text = "服务器启动，监听："
                        + "\"\(hostname)\" (\(address)) \(port)\n"
                }
                 
                //接收并处理客户端消息
                try self.server.startWithHandler(handler: {
                    (received:[UInt8], client: UDPClient) in
                    self.handleClient(received: received, client: client)
                })
            } catch {
                print("Error \(error)")
            }
        }
    }
     
    //处理接收到的客户端消息
    func handleClient(received:[UInt8], client:UDPClient){
        do {
            //将接收到的消息转成String类型
            let str = try received.toString()
            //将这个String消息显示到界面上
            DispatchQueue.main.async {
                self.textView.text = self.textView.text + "服务端接收到消息: \(str)\n"
            }
             
            //将接收到的消息又发回客户端
            try client.send(bytes: received)
        } catch {
            print("Error \(error)")
        }
    }
     
    //发送消息
    @IBAction func sendMessage(_ sender: Any) {
        do {
            let message = self.textField.text
            if message != nil && message != "" {
                try client?.send(bytes: message!.toBytes())
                //获取服务端返回的消息
                let str = try client!.receive().data.toString()
                //将服务端返回的消息显示在界面上
                self.textView.text = self.textView.text + "客户端接收到反馈: \(str)\n"
 
                //清空输入框
                self.textField.text = ""
            }
        } catch {
            print("Error \(error)")
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1588.html