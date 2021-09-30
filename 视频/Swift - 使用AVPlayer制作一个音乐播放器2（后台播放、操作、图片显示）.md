# Swift - 使用AVPlayer制作一个音乐播放器2（后台播放、操作、图片显示）

2017-06-09发布：hangge阅读：4134

在前文中，我介绍了如何使用 **AVFoundation** 框架来制作一个简单的音频播放器（[点击查看](https://www.hangge.com/blog/cache/detail_1668.html)）。但这个播放器不支持后台播放，程序退到后台时音乐就会停止播放。

本文接着介绍如何实现后台播放功能。

### 1，效果图

（1）运行程序并播放音乐。这时我们返回桌面或者关闭屏幕，会发现音乐仍然在播放。

（2）在锁屏界面上，会显示当前的歌曲信息、专辑图片、当前进度等。同时还提供相关的控制按钮供我们使用。

（3）同样的，在上拉的音乐控制面板中，也会显示相关信息，并允许我们进行相关操作。

  [![原文:Swift - 使用AVPlayer制作一个音乐播放器2（后台播放、操作、图片显示）](https://www.hangge.com/blog_uploads/201705/201705041630334870.png)](https://www.hangge.com/blog/cache/detail_1669.html#)   [![原文:Swift - 使用AVPlayer制作一个音乐播放器2（后台播放、操作、图片显示）](https://www.hangge.com/blog_uploads/201705/2017050416304344805.png)](https://www.hangge.com/blog/cache/detail_1669.html#)   [![原文:Swift - 使用AVPlayer制作一个音乐播放器2（后台播放、操作、图片显示）](https://www.hangge.com/blog_uploads/201705/2017050416305131952.png)](https://www.hangge.com/blog/cache/detail_1669.html#)

### 2，实现步骤

（1）为了让播放器能在后台持续播放，我们需要将 **Targets** -> **Capabilities** ->**BackgroundModes** 设为 **ON**，同时勾选“**Audio, AirPlay, and Picture in Picture**”。

[![原文:Swift - 使用AVPlayer制作一个音乐播放器2（后台播放、操作、图片显示）](https://www.hangge.com/blog_uploads/201705/2017050216433197354.png)](https://www.hangge.com/blog/cache/detail_1669.html#)



（2）同时还要在 **AppDelegate.swift** 中注册后台播放。

```
import UIKit
import AVFoundation
 
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
 
    func application(_ application: UIApplication, didFinishLaunchingWithOptions
        launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
         
        // 注册后台播放
        let session = AVAudioSession.sharedInstance()
        do {
            try session.setActive(true)
            try session.setCategory(AVAudioSessionCategoryPlayback)
        } catch {
            print(error)
        }
         
        return true
    }
 
    func applicationWillResignActive(_ application: UIApplication) {
    }
 
    func applicationDidEnterBackground(_ application: UIApplication) {
    }
 
    func applicationWillEnterForeground(_ application: UIApplication) {
    }
 
    func applicationDidBecomeActive(_ application: UIApplication) {
    }
 
    func applicationWillTerminate(_ application: UIApplication) {
    }
}
```


（3）**ViewController.swift**（主视图代码，黄色部分为新增的代码）

```
import UIKit
import AVFoundation
import MediaPlayer
 
class ViewController: UIViewController {
     
    //播放按钮
    @IBOutlet weak var playButton: UIButton!
     
    //可拖动的进度条
    @IBOutlet weak var playbackSlider: UISlider!
     
    //当前播放时间标签
    @IBOutlet weak var playTime: UILabel!
     
    //播放器相关
    var playerItem:AVPlayerItem?
    var player:AVPlayer?
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化播放器
        let url = URL(string: "http://mxd.766.com/sdo/music/data/3/m10.mp3")
        playerItem = AVPlayerItem(url: url!)
        player = AVPlayer(playerItem: playerItem!)
         
        //设置进度条相关属性
        let duration : CMTime = playerItem!.asset.duration
        let seconds : Float64 = CMTimeGetSeconds(duration)
        playbackSlider!.minimumValue = 0
        playbackSlider!.maximumValue = Float(seconds)
        playbackSlider!.isContinuous = false
         
        //播放过程中动态改变进度条值和时间标签
        player!.addPeriodicTimeObserver(forInterval: CMTimeMakeWithSeconds(1, 1),
                                        queue: DispatchQueue.main) { (CMTime) -> Void in
            if self.player!.currentItem?.status == .readyToPlay && self.player?.rate != 0{
                //更新进度条进度值
                let currentTime = CMTimeGetSeconds(self.player!.currentTime())
                self.playbackSlider!.value = Float(currentTime)
                 
                //一个小算法，来实现00：00这种格式的播放时间
                let all:Int=Int(currentTime)
                let m:Int=all % 60
                let f:Int=Int(all/60)
                var time:String=""
                if f<10{
                    time="0\(f):"
                }else {
                    time="\(f)"
                }
                if m<10{
                    time+="0\(m)"
                }else {
                    time+="\(m)"
                }
                //更新播放时间
                self.playTime!.text=time
                 
                //设置后台播放显示信息
                self.setInfoCenterCredentials(playbackState: 1)
            }
        }
    }
     
    //播放按钮点击
    @IBAction func playButtonTapped(_ sender: Any) {
        //根据rate属性判断当天是否在播放
        if player?.rate == 0 {
            player!.play()
            playButton.setTitle("暂停", for: .normal)
        } else {
            player!.pause()
            playButton.setTitle("播放", for: .normal)
            //后台播放显示信息进度停止
            setInfoCenterCredentials(playbackState: 0)
        }
    }
     
    //拖动进度条改变值时触发
    @IBAction func playbackSliderValueChanged(_ sender: Any) {
        let seconds : Int64 = Int64(playbackSlider.value)
        let targetTime:CMTime = CMTimeMake(seconds, 1)
        //播放器定位到对应的位置
        player!.seek(to: targetTime)
        //如果当前时暂停状态，则自动播放
        if player!.rate == 0
        {
            player?.play()
            playButton.setTitle("暂停", for: .normal)
        }
    }
     
    //页面显示时添加相关通知监听
    override func viewWillAppear(_ animated: Bool) {
        //播放完毕
        NotificationCenter.default.addObserver(self, selector: #selector(finishedPlaying),
            name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: playerItem)
         
        //告诉系统接受远程响应事件，并注册成为第一响应者
        UIApplication.shared.beginReceivingRemoteControlEvents()
        self.becomeFirstResponder()
    }
     
    //页面消失时取消歌曲播放结束通知监听
    override func viewWillDisappear(_ animated: Bool) {
        NotificationCenter.default.removeObserver(self)
         
        //停止接受远程响应事件
        UIApplication.shared.endReceivingRemoteControlEvents()
        self.resignFirstResponder()
    }
     
    //歌曲播放完毕
    func finishedPlaying(myNotification:NSNotification) {
        print("播放完毕!")
        let stopedPlayerItem: AVPlayerItem = myNotification.object as! AVPlayerItem
        stopedPlayerItem.seek(to: kCMTimeZero)
    }
     
    //是否能成为第一响应对象
    override var canBecomeFirstResponder: Bool {
        return true
    }
     
    // 设置后台播放显示信息
    func setInfoCenterCredentials(playbackState: Int) {
        let mpic = MPNowPlayingInfoCenter.default()
         
        //专辑封面
        let mySize = CGSize(width: 400, height: 400)
        let albumArt = MPMediaItemArtwork(boundsSize:mySize) { sz in
            return UIImage(named: "cover")!
        }
         
        //获取进度
        let postion = Double(self.playbackSlider!.value)
        let duration = Double(self.playbackSlider!.maximumValue)
         
        mpic.nowPlayingInfo = [MPMediaItemPropertyTitle: "我是歌曲标题",
                               MPMediaItemPropertyArtist: "hangge.com",
                               MPMediaItemPropertyArtwork: albumArt,
                               MPNowPlayingInfoPropertyElapsedPlaybackTime: postion,
                               MPMediaItemPropertyPlaybackDuration: duration,
                               MPNowPlayingInfoPropertyPlaybackRate: playbackState]
    }
     
    //后台操作
    override func remoteControlReceived(with event: UIEvent?) {
        guard let event = event else {
            print("no event\n")
            return
        }
         
        if event.type == UIEventType.remoteControl {
            switch event.subtype {
            case .remoteControlTogglePlayPause:
                print("暂停/播放")
            case .remoteControlPreviousTrack:
                print("上一首")
            case .remoteControlNextTrack:
                print("下一首")
            case .remoteControlPlay:
                print("播放")
                player!.play()
            case .remoteControlPause:
                print("暂停")
                player!.pause()
                //后台播放显示信息进度停止
                setInfoCenterCredentials(playbackState: 0)
            default:
                break
            }
        }
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1668.zip](https://www.hangge.com/blog_uploads/201705/2017050417004261053.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1669.html