# Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）

2016-07-12发布：hangge阅读：4573

在前文中：[Swift - JPush极光推送的使用1（配置、简单的推送测试样例）](https://www.hangge.com/blog/cache/detail_1268.html)。介绍了移动客户端如何配置使用 **JPush SDK**。当时是登录到极光控制台，来发送推送消息进行测试的。但通常在实际项目中，我们会有专门的应用服务器来管理所有的用户，设备、消息等。

极光提供了完善的 **REST API** 接口，以及各个服务器语言专用的 **SDK**（**JAVA**、**Python**、**C#**、**PHP**、**Nodejs**、**Ruby**）。我们服务器后台只要调用一下相应的接口就可以进行消息推送了。

下面演示后台使用 **PHP SDK** ，给所有人发送消息。（客户端还是使用前文那个，这里就不再说明了。）

**1，使用PHP SDK推送广播消息** 

虽然我们使用 **REST API** 来推送也是可以的，但使用 **JPush PHP SDK** 的话会更加方便快捷。

（1）首先到“资源下载”页面下载服务端 **SDK**。地址：https://docs.jiguang.cn/jpush/resources/#sdk_1

[![原文:Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）](https://www.hangge.com/blog_uploads/201607/2016071110465989468.png)](https://www.hangge.com/blog/cache/detail_1270.html#)


（2）将压缩包解压，里面除了接口文件外还提供了样例代码。我们使用时，只需要把压缩包里的 **src** 文件夹，以及 **autoload.php** 文件上传到服务器项目目录下。（我这里是放到 **JPush** 文件夹下。）

[![原文:Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）](https://www.hangge.com/blog_uploads/201610/2016103117253589299.png)](https://www.hangge.com/blog/cache/detail_1270.html#)



（3）在需要使用 **JPush** 的文件中，引入“**autoload.php**”即可。

```php
require 'JPush/autoload.php';
```


（4）下面是一个完整的页面代码
注意：**setOptions** 方法里的第四个参数。用来指定 **APNs** 是否是生产环境。**true** 表示推送到生产环境，**false** 表示推送到开发环境。

```php+HTML
<?
//引入代码
require 'JPush/autoload.php';
 
use JPush\Client as JPush;
 
if(isset($_POST["message"])){
  //初始化
  $app_key = "7b528331738ec719165798fd";
  $master_secret = "32da4e2c06ac7b55da2c9228";
  $client = new JPush($app_key, $master_secret);
 
  //简单的推送样例
  $result = $client->push()
      ->setPlatform('ios', 'android')
      ->addAllAudience()
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
      <input type="text" name="message"/>
      <button type="submit">推送广播通知</button>
    </form>
  </body>
</html>
```


**2，运行测试**

（1）将上面的 **php** 页面上传到服务器并访问，页面显示如下：

[![原文:Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）](https://www.hangge.com/blog_uploads/201607/201607111435503353.png)](https://www.hangge.com/blog/cache/detail_1270.html#)



（2）在输入框中填写需要发送的通知消息，点击“推送广播通知”按钮。即可调用 **JPust API** 接口，将消息推送到所有的设备上。同时推送完毕后，页面上会显示接口调用的返回结果。

[![原文:Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）](https://www.hangge.com/blog_uploads/201607/2016071114405180350.png)](https://www.hangge.com/blog/cache/detail_1270.html#)



（3）推送完毕后，手机这边也成功地显示出通知消息。

[![原文:Swift - JPush极光推送的使用2（PHP调用服务接口给所有人发推送）](https://www.hangge.com/blog_uploads/201607/2016071114484727705.png)](https://www.hangge.com/blog/cache/detail_1270.html#)



**3，更完整的推送示例**

上面演示的只是 **php** 调用 **sdk** 的一个最简单的样例。其实 **JPush SDK** 提供的 **API** 接口配置还是很丰富的，比如下面是一个完整的推送示例。其中包含指定 **Platform**，指定 **Alias**，**Tag**，指定 **iOS**，**Android notification**，指定 **Message** 等。

（对于向指定的 **Alias**、**Tag** 设备推送通知，后面我还会写两篇相关文章来说明。）

```php
//简单的推送样例
$result = $client->push()
     ->setPlatform('ios', 'android')
     ->addAlias('alias1')
     ->addTag(['tag1', 'tag2'])
     //->addAllAudience()
     ->setNotificationAlert('Hello, JPush')
     ->iosNotification('hello', [
       'sound' => 'sound',
       'badge' => '+1',
       'extras' => [
         'key' => 'value'
       ]
     ])
     ->androidNotification('hello', [
       'title' => 'title',
       'extras' => [
         'key' => 'value'
       ]
     ])
     ->message('Hello JPush', [
       'title' => 'Hello',
       'content_type' => 'text',
       'extras' => [
         'key' => 'value'
       ]
     ])
     ->options(array(
       "apns_production" => true,
       'sendno' => 100,
       'time_to_live' => 100,
       //'big_push_duration' => 5
       //'override_msg_id' => 100,
     ))
     ->send();
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1270.html