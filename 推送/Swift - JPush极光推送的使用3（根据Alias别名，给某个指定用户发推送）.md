# Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）

2016-07-13发布：hangge阅读：14073

**一、别名（alias）介绍**

（1）我们可以给每一个安装了应用程序的用户，取不同别名来标识（比如可以使用用户账号的 **userid** 来作为别名）。
（2）以后给某个特定用户推送消息时，就可以用此别名来指定。

（3）每个用户只能指定一个别名。所以同一个设备，新设置的别名会覆盖旧的。
（4）如果要删除已有的别名，只要将别名设置为空字符串即可。
（5）系统不限定一个别名只能指定一个用户。如果一个别名被指定到了多个用户，当给指定这个别名发消息时，服务器端API会同时给这多个用户发送消息。

二、别名使用要求

（1）有效的别名组成：字母（区分大小写）、数字、下划线、汉字。
（2）限制：**alias** 命名长度限制为 **40** 字节。（判断长度需采用 **UTF-8** 编码）


**三、别名使用的样例说明**
下面是一个给指定用户发送消息的样例。

**1，iOS客户端界面**

客户端在启动后，我们可以在输入框中填写 **alias** 别名，点击“注册别名”按钮后，便调用 **API** 将该设备与这个别名关联起来。
注意：这么做只是为了演示而已，实际项目中应该是 **App** 在后台就自动去注册别名（比如在登录成功后），而不是由用户去干预。
    [![原文:Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）](https://www.hangge.com/blog_uploads/201607/2016071121024676894.png)](https://www.hangge.com/blog/cache/detail_1271.html#)    [![原文:Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）](https://www.hangge.com/blog_uploads/201607/2016071121025461630.png)](https://www.hangge.com/blog/cache/detail_1271.html#)
**2，服务端界面**

我们输入别名、消息文本，点击“推送”按钮后，即可给指定的用户发送消息。

[![原文:Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）](https://www.hangge.com/blog_uploads/201607/2016071121042930919.png)](https://www.hangge.com/blog/cache/detail_1271.html#)

[![原文:Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）](https://www.hangge.com/blog_uploads/201607/2016071121044715843.png)](https://www.hangge.com/blog/cache/detail_1271.html#)



**3，客户端显示效果**

可以看到使用该别名的设备能收到消息，而其他的设备是收不到的。

[![原文:Swift - JPush极光推送的使用3（根据Alias别名，给某个指定用户发推送）](https://www.hangge.com/blog_uploads/201607/2016071121053938019.png)](https://www.hangge.com/blog/cache/detail_1271.html#)

**四、完整代码**

**1，客户端代码**

（1）**AppDelegate.swift**（这个同之前文章里的一样，没有改变。本文代码已升级至 **Swfit3**）

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var textField: UITextField!
    @IBOutlet weak var textView: UITextView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //按钮点击
    @IBAction func btnTouchUp(_ sender: AnyObject) {
        //获取别名
        let alias = textField.text
        //注册别名
        JPUSHService.setAlias(alias,
                              callbackSelector: #selector(tagsAliasCallBack(resCode:tags:alias:)),
                              object: self)
    }
     
    //别名注册回调
    func tagsAliasCallBack(resCode:CInt, tags:NSSet, alias:NSString) {
        textView.text = "响应结果：\(resCode)"
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


（2）**ViewController.swift**（别名注册相关）

```
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var textField: UITextField!
    @IBOutlet weak var textView: UITextView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    //按钮点击
    @IBAction func btnTouchUp(_ sender: AnyObject) {
        //获取别名
        let alias = textField.text
        //注册别名
        JPUSHService.setAlias(alias,
                              callbackSelector: #selector(tagsAliasCallBack(resCode:tags:alias:)),
                              object: self)
    }
     
    //别名注册回调
    func tagsAliasCallBack(resCode:CInt, tags:NSSet, alias:NSString) {
        textView.text = "响应结果：\(resCode)"
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，服务端代码（index.php）**

```php
<?
//引入代码
require 'JPush/autoload.php';
 
use JPush\Client as JPush;
 
if(isset($_POST["message"])){
  //初始化
  $app_key = "7b528338438ec719495768fd";
  $master_secret = "32da4a2c06d27b26d12c5628";
  $client = new JPush($app_key, $master_secret);
 
  //简单的推送样例
  $result = $client->push()
      ->setPlatform('ios', 'android')
      ->addAlias($_POST["alias"])
      ->setNotificationAlert($_POST["message"])
      ->options(array(
          "apns_production" => true  //true表示发送到生产环境(默认值)，false为开发环境
        ))
      ->send();
 
  echo 'Result=' . json_encode($result);
}
?>
<html>
  <head>
  </head>
  <body>
    <form action="index.php" method="post">
      别名：<input type="text" name="alias"/><br>
      消息：<input type="text" name="message"/>
      <button type="submit">推送广播通知</button>
    </form>
  </body>
</html>
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1271.html