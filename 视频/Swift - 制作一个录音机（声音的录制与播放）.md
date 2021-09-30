# Swift - 制作一个录音机（声音的录制与播放）

2015-06-24发布：hangge阅读：7650

（本文代码已升级至Swift4）



### **1，技术介绍**

（1）**AVFoundation.framework** 框架提供了 **AVAudioRecorder** 类。它可以实现录音功能。

（2）而使用该框架的 **AVAudioPlayer** 类，可以实现声音的播放。

### **2，下面制作一个录音机样例**

（1）按住录音按钮则开始录音，松开则停止录音。录音文件保存在用户文件夹下。

（2）录音过程中会实时显示声音的音量大小（这个可以用来做声音脉冲图，获得更好的展示效果）

（3）点击播放录音则可播放录制的声音文件。

### **3，效果图如下：**

[![原文:Swift - 制作一个录音机（声音的录制与播放）](https://www.hangge.com/blog_uploads/201506/2015062410123814024.png)](https://www.hangge.com/blog/cache/detail_772.html#)



### **4，Info.plist配置**

由于苹果安全策略更新，在使用 **Xcode8** 开发时，需要在 **Info.plist** 配置请求麦克风相的关描述字段（**Privacy - Microphone Usage Description**）

[![原文:Swift - 制作一个录音机（声音的录制与播放）](https://www.hangge.com/blog_uploads/201705/2017051820072364621.png)](https://www.hangge.com/blog/cache/detail_772.html#)



### **5，代码如下：**

```swift
import UIKit
import AVFoundation
 
class ViewController: UIViewController {
     
    var recorder:AVAudioRecorder? //录音器
    var player:AVAudioPlayer? //播放器
    var recorderSeetingsDic:[String : Any]? //录音器设置参数数组
    var volumeTimer:Timer! //定时器线程，循环监测录音的音量大小
    var aacPath:String? //录音存储路径
     
    @IBOutlet weak var volumLab: UILabel! //显示录音音量
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化录音器
        let session:AVAudioSession = AVAudioSession.sharedInstance()
         
        //设置录音类型
        try! session.setCategory(AVAudioSessionCategoryPlayAndRecord)
        //设置支持后台
        try! session.setActive(true)
        //获取Document目录
        let docDir = NSSearchPathForDirectoriesInDomains(.documentDirectory,
                                                         .userDomainMask, true)[0]
        //组合录音文件路径
        aacPath = docDir + "/play.aac"
        //初始化字典并添加设置参数
        recorderSeetingsDic =
            [
                AVFormatIDKey: NSNumber(value: kAudioFormatMPEG4AAC),
                AVNumberOfChannelsKey: 2, //录音的声道数，立体声为双声道
                AVEncoderAudioQualityKey : AVAudioQuality.max.rawValue,
                AVEncoderBitRateKey : 320000,
                AVSampleRateKey : 44100.0 //录音器每秒采集的录音样本数
        ]
    }
     
    //按下录音
    @IBAction func downAction(_ sender: AnyObject) {
        //初始化录音器
        recorder = try! AVAudioRecorder(url: URL(string: aacPath!)!,
                                        settings: recorderSeetingsDic!)
        if recorder != nil {
            //开启仪表计数功能
            recorder!.isMeteringEnabled = true
            //准备录音
            recorder!.prepareToRecord()
            //开始录音
            recorder!.record()
            //启动定时器，定时更新录音音量
            volumeTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self,
                                selector: #selector(ViewController.levelTimer),
                                userInfo: nil, repeats: true)
        }
    }
     
    //松开按钮，结束录音
    @IBAction func upAction(_ sender: AnyObject) {
        //停止录音
        recorder?.stop()
        //录音器释放
        recorder = nil
        //暂停定时器
        volumeTimer.invalidate()
        volumeTimer = nil
        volumLab.text = "录音音量:0"
    }
     
    //播放录制的声音
    @IBAction func playAction(_ sender: AnyObject) {
        //播放
        player = try! AVAudioPlayer(contentsOf: URL(string: aacPath!)!)
        if player == nil {
            print("播放失败")
        }else{
            player?.play()
        }
    }
     
    //定时检测录音音量
    func levelTimer(){
        recorder!.updateMeters() // 刷新音量数据
        let averageV:Float = recorder!.averagePower(forChannel: 0) //获取音量的平均值
        let maxV:Float = recorder!.peakPower(forChannel: 0) //获取音量最大值
        let lowPassResult:Double = pow(Double(10), Double(0.05*maxV))
        volumLab.text = "录音音量:\(lowPassResult)"
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_772.zip](https://www.hangge.com/blog_uploads/201705/2017051817331726532.zip)



## 附：让声音从通过扬声器播放出来

运行上面的样例可以发现，当我们播放录音时声音是从听筒位置发出的。如果嫌音量小的话，可以将声音输出设置为下面的扬声器。

具体设置方法见下方的高亮部分。

```swift
//播放录制的声音
@IBAction func playAction(_ sender: AnyObject) {
    //播放
    player = try! AVAudioPlayer(contentsOf: URL(string: aacPath!)!)
    if player == nil {
        print("播放失败")
    }else{
        //让音频通过喇叭播放
        try! AVAudioSession.sharedInstance().overrideOutputAudioPort(.speaker)
        player?.play()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_772.html