# Swift - socket不定长消息发送接收的几种方法 （以聊天室为例）

2016-08-31发布：hangge阅读：2543

（本文代码已升级至Swift4）

使用 **socket** 可以很方便地实现客服端和服务器的长连接，并进行双向的数据通信。但有时我们发送的数据包长度并不是固定的（比如做一个聊天系统），通常的做法是在数据包前面添加个包头信息，让接收方知道整个发送包的长度。也就是说接收方先收这个固定长度的包头信息，然后再根据包头信息里面定义的实际长度来接收包数据。
下面通过一个聊天室程序演示 **socket** 发送/读取不定长消息包的几种方法。**socket** 通信这里我们使用了一个封装好的 **socket** 库（[SwiftSocket](https://github.com/swiftsocket/SwiftSocket)）。

  [![原文:Swift - socket不定长消息发送接收的几种方法 （以聊天室为例）](https://www.hangge.com/blog_uploads/201506/2015060816525547406.png)](https://www.hangge.com/blog/cache/detail_1335.html#)  [![原文:Swift - socket不定长消息发送接收的几种方法 （以聊天室为例）](https://www.hangge.com/blog_uploads/201506/2015060816530391544.png)](https://www.hangge.com/blog/cache/detail_1335.html#)


**方法1：每条消息发送两个包**
第**1**个包长度固定为**4**个字节，里边记录的是后面发送的实际消息包的长度。

第**2**个包才是实际的消息包。

接收方先接收第一个固定长度的数据包，通过第一个包知道了数据长度，从而进行读取实际的消息包。这种方式我原来写过相关的文章，地址是：[Swift - 使用socket进行通信（附聊天室样例）](https://www.hangge.com/blog/cache/detail_756.html)


（1）发送数据相关代码如下：

```
//发送消息``func` `sendMessage(msgtosend:[``String``:``String``]){``  ``let` `msgdata=try? ``JSONSerialization``.data(withJSONObject: msgtosend,``                              ``options: .prettyPrinted)``  ``var` `len:``Int32``=``Int32``(msgdata!.count)``  ``let` `data = ``Data``(bytes: &len, count: 4)``  ``_ = ``self``.socketClient!.send(data: data)``  ``_ = ``self``.socketClient!.send(data:msgdata!)``}
```


（2）接收数据相关代码如下：

```
//解析收到的消息``func` `readMsg()->[``String``:``Any``]?{``  ``//read 4 byte int as type``  ``if` `let` `data=``self``.tcpClient!.read(4){``    ``if` `data.count==4{``      ``let` `ndata=``NSData``(bytes: data, length: data.count)``      ``var` `len:``Int32``=0``      ``ndata.getBytes(&len, length: data.count)``      ``if` `let` `buff=``self``.tcpClient!.read(``Int``(len)){``        ` `        ``let` `msgd = ``Data``(bytes: buff, count: buff.count)``        ``let` `msgi = (try! ``JSONSerialization``.jsonObject(with: msgd,``              ``options: .mutableContainers)) ``as``! [``String``:``Any``]``        ``return` `msgi``      ``}``    ``}``  ``}``  ``return` `nil``}
```

***\*
\**方法2：每条消息只发送1个包。不过这个包增加了包头信息。**
这个其实是上面方式的整合。发送方将消息长度做为包头信息（比如固定是**4**个字节大小）添加到消息包前面。
接收方先读取前**4**个字节的内容，再根据获取到的长度信息读取剩余的消息。



（1）客户端这边（**ViewController.swift**）发送消息的时候将包头拼在消息包前面，这样每次只发一个数据包。（接收数据方式不变）

```
//发送消息``func` `sendMessage(msgtosend:[``String``:``String``]){``  ``//消息数据包``  ``let` `msgdata=try? ``JSONSerialization``.data(withJSONObject: msgtosend,``                      ``options: .prettyPrinted)``  ``//消息数据包大小``  ``var` `len:``Int32``=``Int32``(msgdata!.count)``  ``//消息包头数据``  ``var` `data = ``Data``(bytes: &len, count: 4)``  ``//包头后面添加正真的消息数据包``  ``data.append(msgdata!)``  ``//发送合并后的数据包``  ``_ = ``self``.socketClient!.send(data: data)``}
```


（2）服务端（**MyTcpSocketServer.swift**）同样也是发送一个包含包头信息的数据包。（接收数据方式不变）

```
//发送消息``func` `sendMsg(msg:[``String``:``Any``]){``  ``//消息数据包``  ``let` `jsondata=try? ``JSONSerialization``.data(withJSONObject: msg, options:``    ``JSONSerialization``.``WritingOptions``.prettyPrinted)``  ``//消息数据包大小``  ``var` `len:``Int32``=``Int32``(jsondata!.count)``  ``//消息包头数据``  ``var` `data = ``Data``(bytes: &len, count: 4)``  ``//包头后面添加正真的消息数据包``  ``data.append(jsondata!)``  ``//发送合并后的数据包``  ``_ = ``self``.tcpClient!.send(data: data)``}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_756.zip](https://www.hangge.com/blog_uploads/201711/2017110715074566519.zip)



**方法3：每条消息只发送1个包。同时不增加包头信息。**
这种方式发送方直接将消息包发送出去，不再额外添加任何包头信息。
接受方由于不知道消息的实际长度，便需要进行循环读取。每次读固定字节数（比如**1024**个字节），如果某次读取到的字节数小于**1024**个字节，则表示读取完毕。



（1）首先修改 **ytcpsocket.swift**，给 **TCPClient** 类添加 **readAll()** 方法，用来实现自动循环读取全部的消息，而不用指定预期的数据长度。

```swift
/*
 * 不指定长度来读取数据
 */
public func readAll(timeout:Int = -1)->[UInt8]?{
    //内容缓冲区
    var data:[UInt8] = [UInt8]()
    repeat{
        if let tempData = read(1024, timeout: timeout) {
            //将每次读取到的数据添加到缓冲区
            data = data + tempData
            //读取结束
            if tempData.count < 1024 {
                break
            }
        }else{
            break
        }
    }while true
    return data
}
```


（2）客户端管理类 **ChatUser**（**MyTcpSocketServer.swift**）修改
接收数据的方式修改了，同时发送数据前不再需要提供消息长度，直接发消息包即可。

```swift
//客户端管理类（便于服务端管理所有连接的客户端）
class ChatUser:NSObject{
    var tcpClient:TCPClient?
    var username:String=""
    var socketServer:MyTcpSocketServer?
     
    //解析收到的消息
    func readMsg()->[String:Any]?{
        if let buff = self.tcpClient!.readAll(){
            let msgd = Data(bytes: buff, count: buff.count)
            let msgi = (try! JSONSerialization.jsonObject(with: msgd,
                            options: .mutableContainers)) as! [String:Any]
            return msgi
        }
        return nil
    }
     
    //循环接收消息
    func messageloop(){
        while true{
            if let msg=self.readMsg(){
                self.processMsg(msg: msg)
            }else{
                self.removeme()
                break
            }
        }
    }
     
    //处理收到的消息
    func processMsg(msg:[String:Any]){
        if msg["cmd"] as! String=="nickname"{
            self.username=msg["nickname"] as! String
        }
        self.socketServer!.processUserMsg(user: self, msg: msg)
    }
     
    //发送消息
    func sendMsg(msg:[String:Any]){
        let jsondata=try? JSONSerialization.data(withJSONObject: msg, options:
            JSONSerialization.WritingOptions.prettyPrinted)
        _ = self.tcpClient!.send(data: jsondata!)
    }
     
    //移除该客户端
    func removeme(){
        self.socketServer!.removeUser(u: self)
    }
     
    //关闭连接
    func kill(){
        _ = self.tcpClient!.close()
    }
}
```


（3）同样的，客户端代码（**ViewController.swift**）这边发送与接收数据也可以简化。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //消息输入框
    @IBOutlet weak var textFiled: UITextField!
    //消息输出列表
    @IBOutlet weak var textView: UITextView!
     
    //socket服务端封装类对象
    var socketServer:MyTcpSocketServer?
    //socket客户端类对象
    var socketClient:TCPClient?
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //启动服务器
        socketServer = MyTcpSocketServer()
        socketServer?.start()
         
        //初始化客户端，并连接服务器
        processClientSocket()
    }
     
    //初始化客户端，并连接服务器
    func processClientSocket(){
        socketClient=TCPClient(addr: "localhost", port: 8080)
         
        DispatchQueue.global(qos: .background).async {
            //用于读取并解析服务端发来的消息
            func readmsg() -> [String:Any]?{
                if let buff=self.socketClient!.readAll(){
                    let msgd = Data(bytes: buff, count: buff.count)
                    let msgi = (try! JSONSerialization.jsonObject(with: msgd,
                                    options: .mutableContainers)) as! [String:Any]
                    return msgi
                }
                return nil
            }
             
            //连接服务器
            let (success,msg)=self.socketClient!.connect(timeout: 5)
            if success{
                DispatchQueue.main.async {
                    self.alert(msg: "connect success", after: {
                    })
                }
                 
                //发送用户名给服务器（这里使用随机生成的）
                let msgtosend=["cmd":"nickname","nickname":"游客\(Int(arc4random()%1000))"]
                self.sendMessage(msgtosend: msgtosend)
                 
                //不断接收服务器发来的消息
                while true{
                    if let msg=readmsg(){
                        DispatchQueue.main.async {
                            self.processMessage(msg: msg)
                        }
                    }else{
                        DispatchQueue.main.async {
                            //self.disconnect()
                        }
                        break
                    }
                }
            }else{
                DispatchQueue.main.async {
                    self.alert(msg: msg,after: {
                    })
                }
            }
        }
    }
     
    //“发送消息”按钮点击
    @IBAction func sendMsg(_ sender: AnyObject) {
        let content=textFiled.text!
        let message=["cmd":"msg","content":content]
        self.sendMessage(msgtosend: message)
        textFiled.text=nil
    }
     
    //发送消息
    func sendMessage(msgtosend:[String:String]){
        let msgdata=try? JSONSerialization.data(withJSONObject: msgtosend,
                                                                options: .prettyPrinted)
        _ = self.socketClient!.send(data:msgdata!)
    }
     
    //处理服务器返回的消息
    func processMessage(msg:[String:Any]){
        let cmd:String=msg["cmd"] as! String
        switch(cmd){
        case "msg":
            self.textView.text = self.textView.text +
                (msg["from"] as! String) + ": " + (msg["content"] as! String) + "\n"
        default:
            print(msg)
        }
    }
     
    //弹出消息框
    func alert(msg:String,after:()->(Void)){
        let alertController = UIAlertController(title: "",
                                                message: msg,
                                                preferredStyle: .alert)
        self.present(alertController, animated: true, completion: nil)
         
        //1.5秒后自动消失
        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1.5) {
            alertController.dismiss(animated: false, completion: nil)
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_756.zip](https://www.hangge.com/blog_uploads/201711/2017110716452532653.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1335.html