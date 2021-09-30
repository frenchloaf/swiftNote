# Swift - 使用AVPlayer制作一个音乐播放器1（带播放时间和播放进度）

2017-06-07发布：hangge阅读：6106

（本文代码已升级至Swift4）

过去我们可以使用 **Media Player** 框架 **MPMoviePlayerController** 来播放视频、音频。但自 **iOS9.0** 起，这个便被废除。取而代之的便是 **AVFoundation** 框架的 **AVPlayer**。

### 1，AVPlayer介绍

（1）**AVPlayer** 可以用来播放视频，也可以播放任何 **iOS** 支持的音频。

（2）**AVPlayer** 既可以播放本地音频，可以播放网络音频（在线音频）。

（3）要注意的是，如果播放远程音频，**AVPlayer** 同样是全部加载到本地后才开始播放，而不是以流媒体的形式播放。

### 2，效果图

（1）下面使用 **AVPlayer** 制作一个音乐播放器。

（2）点击按钮可以时音乐在“**播放**”和“**暂停**”两个状态间切换。

（3）播放过程中进度条和旁边的标签会实时显示当前的进度。

（4）进度条滑块可以自由拖动，并播放对应时间点的音乐。

[![原文:Swift - 使用AVPlayer制作一个音乐播放器1（带播放时间和播放进度）](https://www.hangge.com/blog_uploads/201704/2017042815090466014.png)](https://www.hangge.com/blog/cache/detail_1668.html#)



### 3，样例代码

```swift
import UIKit
import AVFoundation
 
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
        let url = URL(string: "http://www.hangge.com/music.mp3")
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
            if self.player!.currentItem?.status == .readyToPlay {
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
     
    //页面显示时添加歌曲播放结束通知监听
    override func viewWillAppear(_ animated: Bool) {
        NotificationCenter.default.addObserver(self, selector: #selector(finishedPlaying),
               name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: playerItem)
    }
     
    //页面消失时取消歌曲播放结束通知监听
    override func viewWillDisappear(_ animated: Bool) {
        NotificationCenter.default.removeObserver(self)
    }
     
    //歌曲播放完毕
    @objc func finishedPlaying(myNotification:NSNotification) {
        print("播放完毕!")
        let stopedPlayerItem = myNotification.object as! AVPlayerItem
        stopedPlayerItem.seek(to: kCMTimeZero, completionHandler: nil)
        playButton.setTitle("播放", for: .normal)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1668.zip](https://www.hangge.com/blog_uploads/201711/201711301713286740.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1668.html