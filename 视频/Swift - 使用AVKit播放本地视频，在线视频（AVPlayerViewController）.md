# Swift - 使用AVKit播放本地视频，在线视频（AVPlayerViewController）

2016-04-26发布：hangge阅读：12017

（本文代码已升级至Swift4）

原来我们可以使用 **Media Player** 框架 **MPMoviePlayerController** 来播放视频、音频。但自iOS9.0起，这个将会被废除，所以苹果建议使用 **AVKit** 的**AVPlayerViewController** 来代替。



### 1，单独使用 AVPlayer

这个可以在当前视图中添加一个视频播放窗口，位置大小可设置，但不带播放控制器。所以如果需要控制视频播放状态的话，就需要自己在页面上添加按钮，写相应的控制方法了。

[![原文:Swift - 使用AVKit播放本地视频，在线视频（AVPlayerViewController）](https://www.hangge.com/blog_uploads/201604/2016042311044494943.png)](https://www.hangge.com/blog/cache/detail_1107.html#)

```swift
import UIKit
import AVKit
import AVFoundation
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //定义一个视频文件路径
        let filePath = Bundle.main.path(forResource: "hangge", ofType: "mp4")
        let videoURL = URL(fileURLWithPath: filePath!)
        //定义一个视频播放器，通过本地文件路径初始化
        let player = AVPlayer(url: videoURL)
        //设置大小和位置（全屏）
        let playerLayer = AVPlayerLayer(player: player)
        playerLayer.frame = self.view.bounds
        //添加到界面上
        self.view.layer.addSublayer(playerLayer)
        //开始播放
        player.play()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 2，视频播放完毕响应

我们可以定义一个 **AVPlayerItem**，当视频播放完毕后它会发送一个 **AVPlayerItemDidPlayToEndTime** 通知。我们可以监听这个通知，在响应事件中做一些后续工作（比如关闭播放器、播放下一个视频等）

```swift
import UIKit
import AVKit
import AVFoundation
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidLoad()
         
        //定义一个视频文件路径
        let filePath = Bundle.main.path(forResource: "hangge", ofType: "mp4")
        let videoURL = URL(fileURLWithPath: filePath!)
        //定义一个playerItem，并监听相关的通知
        let playerItem = AVPlayerItem(url: videoURL)
        NotificationCenter.default.addObserver(self,
                           selector: #selector(playerDidFinishPlaying),
                           name: NSNotification.Name.AVPlayerItemDidPlayToEndTime,
                           object: playerItem)
        //定义一个视频播放器，通过playerItem径初始化
        let player = AVPlayer(playerItem: playerItem)
        //设置大小和位置（全屏）
        let playerLayer = AVPlayerLayer(player: player)
        playerLayer.frame = self.view.bounds
        //添加到界面上
        self.view.layer.addSublayer(playerLayer)
        //开始播放
        player.play()
    }
     
    //视频播放完毕响应
    @objc func playerDidFinishPlaying() {
        print("播放完毕!")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```





### 3，使用AVPlayerViewController

这个类似于原来的 **MPMoviePlayerController**，使用时是会切换到新的视图控制器页面上，同时会有播放、暂停、进度选择等播放控制器。

[![原文:Swift - 使用AVKit播放本地视频，在线视频（AVPlayerViewController）](https://www.hangge.com/blog_uploads/201604/2016042311045622179.png)](https://www.hangge.com/blog/cache/detail_1107.html#)

```swift
import UIKit
import AVKit
import AVFoundation
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidLoad()
         
        //定义一个视频文件路径
        let filePath = Bundle.main.path(forResource: "hangge", ofType: "mp4")
        let videoURL = URL(fileURLWithPath: filePath!)
        //定义一个视频播放器，通过本地文件路径初始化
        let player = AVPlayer(url: videoURL)
        let playerViewController = AVPlayerViewController()
        playerViewController.player = player
        self.present(playerViewController, animated: true) {
            playerViewController.player!.play()
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 4，播放在线视频

前面两个样例都是播放本地视频，要播放网络视频使用如下代码：

```swift
let videoURL =  URL(string: "http://hangge.com/demo.mp4")!
let player = AVPlayer(url: videoURL)
//.........
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1107.html