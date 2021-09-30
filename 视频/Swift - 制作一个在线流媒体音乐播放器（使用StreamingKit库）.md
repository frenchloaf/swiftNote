# Swift - 制作一个在线流媒体音乐播放器（使用StreamingKit库）

2017-06-12发布：hangge阅读：3972

在之前的文章中，我介绍了如何使用 **AVPlayer** 制作一个简单的音乐播放器（[点击查看1](https://www.hangge.com/blog/cache/detail_1668.html)、[点击查看2](https://www.hangge.com/blog/cache/detail_1669.html)）。虽然这个播放器也可以播放网络音频，但其实际上是将音频文件下载到本地后再播放的。

本文演示如何使用第三方的 **StreamingKit** 库，来实现网络流音频的播放。

## 一、StreamingKit介绍和配置

### 1，基本介绍

（1）**StreamingKit** 是一个适用于 **iOS** 和 **Mac OSX** 的音频播放流媒体库。**StreamingKit** 提供了一个简洁的面向对象 **API**，用于在 **CoreAudio** 框架下进行音频的解压和播放（采用硬件或软件编解码器）处理。

（2）**StreamingKit** 的主要机制是对从播放器输入的数据源进行解耦，从而使高级定制的数据源可以进行诸如基于流媒体的渐进式下载、编码解码、自动恢复、动态缓冲之类的处理。**StreamingKit** 是唯一支持不同格式音频文件无缝播放的音频播放流媒体库。

（3）**Github** 主页：https://github.com/tumtumtum/StreamingKit

### 2，主要特点

- 免费开源
- 简洁的 **API**
- 可读性很强的源代码
- 精心使用多线程提供了一个快速响应的 **API**，既能防止线程阻塞，又能保证缓冲流畅
- 缓冲并无缝播放所有不同格式的音频文件
- 容易实现的音频数据源（支持本地、**HTTP**、**AutoRecovering HTTP** 作为数据源）
- 容易 **kuo** 扩展数据源以支持自动缓冲、编码等
- 低耗电和低 **CPU** 使用率（**CPU** 使用率 **0％**，流式处理时使用率为 **1％**）
- 优化线性数据源，仅随机访问数据源需要搜索
- **StreamingKit0.2.0** 使用 **AudioUnit API** 而不是速度较慢的音频队列 **API**，允许对原始 **PCM** 数据进行实时截取以获得并行测量、**EQ** 等特征
- 电能计量
- 内置的均衡器（**iOS5.0** 及以上版本、**OSX10.9** 及以上版本）支持音频播放的同时动态改变、启用、禁用均衡器
- 提供了 **iOS** 和 **Mac OSX** 应用实例

### 3，安装配置

（1）将源码包下载下来后，将其中的 **StreamingKit/StreamingKit** 文件夹复制到项目中来。

[![原文:Swift - 制作一个在线流媒体音乐播放器（使用StreamingKit库）](https://www.hangge.com/blog_uploads/201705/2017050216332011344.png)](https://www.hangge.com/blog/cache/detail_1667.html#)

（2）创建桥接头文件，内容如下：

```swift
#import "STKAudioPlayer.h"
```



## 二、制作一个网络音频播放器

### 1，效果图

（1）程序运行后自动开始播放音乐（整个队列一个有 **3** 首歌曲，默认先播放第一首）

（2）点击“**上一曲**”“**下一曲**”按钮可以切换当前播放歌曲。

（3）歌曲播放过程中进度条会随之变化，进度条右侧会显示出当前歌曲播放时间。

（4）进度条可以拖动，拖动结束后自动播放该时间点的音乐。

（5）点击“**暂停**”按钮可以交替切换播放器暂停、继续状态。

（6）点击“**结束**”按钮，结束整个播放器的音乐播放。

[![原文:Swift - 制作一个在线流媒体音乐播放器（使用StreamingKit库）](https://www.hangge.com/blog_uploads/201705/201705041425478513.png)](https://www.hangge.com/blog/cache/detail_1667.html#)

### 2，实现步骤

（1）在 **info.plist** 中添加如下配置以支持 **http** 传输。

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```


（2）为了让播放器能在后台持续播放，我们需要将 **Targets** -> **Capabilities** -> **BackgroundModes** 设为 **ON**，同时勾选“**Audio, AirPlay, and Picture in Picture**”。

[![原文:Swift - 制作一个在线流媒体音乐播放器（使用StreamingKit库）](https://www.hangge.com/blog_uploads/201705/2017050216433197354.png)](https://www.hangge.com/blog/cache/detail_1667.html#)

同时还要在 **AppDelegate.swift** 中注册后台播放。

```swift
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


（3）主视图代码（**ViewController.swift**）

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //显示歌曲标题
    @IBOutlet weak var titleLabel: UILabel!
     
    //暂停按钮
    @IBOutlet weak var pauseBtn: UIButton!
     
    //可拖动的进度条
    @IBOutlet weak var playbackSlider: UISlider!
     
    //当前播放时间标签
    @IBOutlet weak var playTime: UILabel!
     
    //更新进度条定时器
    var timer:Timer!
     
    //音频播放器
    var audioPlayer: STKAudioPlayer!
     
    //播放列表
    var queue = [Music(name: "歌曲1",
                       url: URL(string: "http://mxd.766.com/sdo/music/data/3/m10.mp3")!),
                 Music(name: "歌曲2",
                       url: URL(string: "http://mxd.766.com/sdo/music/data/3/m12.mp3")!),
                 Music(name: "歌曲3",
                       url: URL(string: "http://mxd.766.com/sdo/music/data/3/m13.mp3")!)]
     
    //当前播放音乐索引
    var currentIndex:Int = -1
     
    //是否循环播放
    var loop:Bool = false
     
    //当前播放状态
    var state:STKAudioPlayerState = []
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //设置进度条相关属性
        playbackSlider!.minimumValue = 0
        playbackSlider!.isContinuous = false
         
        //重置播放器
        resetAudioPlayer()
         
        //开始播放歌曲列表
        playWithQueue(queue: queue)
         
        //设置一个定时器，每三秒钟滚动一次
        timer = Timer.scheduledTimer(timeInterval: 0.1, target: self,
                    selector: #selector(tick), userInfo: nil, repeats: true)
    }
     
    //重置播放器
    func resetAudioPlayer() {
        var options = STKAudioPlayerOptions()
        options.flushQueueOnSeek = true
        options.enableVolumeMixer = true
        audioPlayer = STKAudioPlayer(options: options)
         
        audioPlayer.meteringEnabled = true
        audioPlayer.volume = 1
        audioPlayer.delegate = self
    }
     
    //开始播放歌曲列表（默认从第一首歌曲开始播放）
    func playWithQueue(queue: [Music], index: Int = 0) {
        guard index >= 0 && index < queue.count else {
            return
        }
        self.queue = queue
        audioPlayer.clearQueue()
        let url = queue[index].url
        audioPlayer.play(url)
         
        for i in 1 ..< queue.count {
            audioPlayer.queue(queue[Int((index + i) % queue.count)].url)
        }
        currentIndex = index
        loop = false
    }
     
    //停止播放
    func stop() {
        audioPlayer.stop()
        queue = []
        currentIndex = -1
    }
     
    //单独播放某个歌曲
    func play(file: Music) {
        audioPlayer.play(file.url)
    }
     
    //下一曲
    func next() {
        guard queue.count > 0 else {
            return
        }
        currentIndex = (currentIndex + 1) % queue.count
        playWithQueue(queue: queue, index: currentIndex)
    }
     
    //上一曲
    func prev() {
        currentIndex = max(0, currentIndex - 1)
        playWithQueue(queue: queue, index: currentIndex)
    }
     
    //下一曲按钮点击
    @IBAction func nextBtnTapped(_ sender: Any) {
        next()
    }
     
    //上一曲按钮点击
    @IBAction func prevBtnTapped(_ sender: Any) {
        prev()
    }
     
    //暂停继续按钮点击
    @IBAction func pauseBtnTapped(_ sender: Any) {
        //在暂停和继续两个状态间切换
        if self.state == .paused {
            audioPlayer.resume()
        }else{
            audioPlayer.pause()
        }
    }
 
    //结束按钮点击
    @IBAction func stopBtnTapped(_ sender: Any) {
        stop()
    }
     
    //定时器响应，更新进度条和时间
    func tick() {
        if state == .playing {
            //更新进度条进度值
            self.playbackSlider!.value = Float(audioPlayer.progress)
             
            //一个小算法，来实现00：00这种格式的播放时间
            let all:Int=Int(audioPlayer.progress)
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
        }
    }
     
    //拖动进度条改变值时触发
    @IBAction func playbackSliderValueChanged(_ sender: Any) {
        //播放器定位到对应的位置
        audioPlayer.seek(toTime: Double(playbackSlider.value))
        //如果当前时暂停状态，则继续播放
        if state == .paused
        {
            audioPlayer.resume()
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//Audio Player相关代理方法
extension ViewController: STKAudioPlayerDelegate {
 
    //开始播放歌曲
    func audioPlayer(_ audioPlayer: STKAudioPlayer,
                     didStartPlayingQueueItemId queueItemId: NSObject) {
        if let index = (queue.index { $0.url == queueItemId as! URL }) {
            currentIndex = index
        }
    }
     
    //缓冲完毕
    func audioPlayer(_ audioPlayer: STKAudioPlayer,
        didFinishBufferingSourceWithQueueItemId queueItemId: NSObject) {
        updateNowPlayingInfoCenter()
    }
     
    //播放状态变化
    func audioPlayer(_ audioPlayer: STKAudioPlayer,
                     stateChanged state: STKAudioPlayerState,
                     previousState: STKAudioPlayerState) {
        self.state = state
        if state != .stopped && state != .error && state != .disposed {
        }
        updateNowPlayingInfoCenter()
    }
     
    //播放结束
    func audioPlayer(_ audioPlayer: STKAudioPlayer,
                     didFinishPlayingQueueItemId queueItemId: NSObject,
                     with stopReason: STKAudioPlayerStopReason,
                     andProgress progress: Double, andDuration duration: Double) {
        if let index = (queue.index {
            $0.url == audioPlayer.currentlyPlayingQueueItemId() as! URL
        }) {
            currentIndex = index
        }
         
        //自动播放下一曲
        if stopReason == .eof {
            next()
        } else if stopReason == .error {
            stop()
            resetAudioPlayer()
        }
    }
     
    //发生错误
    func audioPlayer(_ audioPlayer: STKAudioPlayer,
                     unexpectedError errorCode: STKAudioPlayerErrorCode) {
        print("Error when playing music \(errorCode)")
        resetAudioPlayer()
        playWithQueue(queue: queue, index: currentIndex)
    }
     
    //更新当前播放信息
    func updateNowPlayingInfoCenter() {
        if currentIndex >= 0 {
            let music = queue[currentIndex]
            //更新标题
            titleLabel.text = "当前播放：\(music.name)"
             
            //更新暂停按钮名字
            let pauseBtnTitle = self.state == .playing ? "暂停" : "继续"
            pauseBtn.setTitle(pauseBtnTitle, for: .normal)
             
            //设置进度条相关属性
            playbackSlider!.maximumValue = Float(audioPlayer.duration)
        }else{
            //停止播放
            titleLabel.text = "播放停止!"
            //更新进度条和时间标签
            playbackSlider.value = 0
            playTime.text = "--:--"
        }
    }
}
 
//歌曲类
class Music {
    var name:String
    var url:URL
     
    //类构造函数
    init(name:String, url:URL){
        self.name = name
        self.url = url
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1667.zip](https://www.hangge.com/blog_uploads/201705/2017050414455611767.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1667.html