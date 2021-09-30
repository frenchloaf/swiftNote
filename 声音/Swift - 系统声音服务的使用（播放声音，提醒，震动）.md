# Swift - 系统声音服务的使用（播放声音，提醒，震动）

2015-06-23发布：hangge阅读：12786

（本文代码已升级至Swift4）


**1，系统声音服务介绍：**

系统声音服务提供了一个　**Api**，用于播放不超过　**30**　秒的声音。它支持的文件格式有限，具体的说只有　**CAF**、**AIF**　和使用　**PCM**　或　**IMA**/**ADPCM**　数据的　**WAV**　文件。

但此函数没有提供操作声音和控制音量的功能，因此如果是要为多媒体或游戏创建专门声音，就不要使用系统声音服务。

[![原文:Swift - 系统声音服务的使用（播放声音，提醒，震动）](https://www.hangge.com/blog_uploads/201712/2017122811355037548.jpg)](https://www.hangge.com/blog/cache/detail_771.html#)



**2，系统声音服务支持如下三种类型**：

（1）声音：立刻播放一个简单的声音文件。如果手机静音，则用户什么也听不见。

（2）提醒：播放一个声音文件，如果手机设为静音或震动，则通过震动提醒用户。

（3）震动：震动手机，而不考虑其他设置。

**3，使用样例（首先类中要引入AudioToolbox）**

```swift
import AudioToolbox
```

**（1）声音播放**

```swift
@IBAction func systemSound(_ sender: Any) {
    //建立的SystemSoundID对象
    var soundID:SystemSoundID = 0
    //获取声音地址
    let path = Bundle.main.path(forResource: "msg", ofType: "wav")
    //地址转换
    let baseURL = NSURL(fileURLWithPath: path!)
    //赋值
    AudioServicesCreateSystemSoundID(baseURL, &soundID)
    //播放声音
    AudioServicesPlaySystemSound(soundID)
}
```


**（2）提醒**

```swift
@IBAction func systemAlert(_ sender: Any) {
    //建立的SystemSoundID对象
    var soundID:SystemSoundID = 0
    //获取声音地址
    let path = Bundle.main.path(forResource: "msg", ofType: "wav")
    //地址转换
    let baseURL = NSURL(fileURLWithPath: path!)
    //赋值
    AudioServicesCreateSystemSoundID(baseURL, &soundID)
    //提醒（同上面唯一的一个区别）
    AudioServicesPlayAlertSound(soundID)
}
```


**（3）振动**

```swift
@IBAction func systemVibration(sender: AnyObject) {
    //建立的SystemSoundID对象
    let soundID = SystemSoundID(kSystemSoundID_Vibrate)
    //振动
    AudioServicesPlaySystemSound(soundID)
}
```


**4，声音或提醒播放完毕后的回调函数**
默认情况下每触发一次声音提醒，系统就会执行一次。不管当前是否有其他的声音提醒未播放完毕。这样如果提醒声音时间比较长，在短时间内多次触发，那么就会造成重音（多个声音叠加在一起）。
我们可以设置个状态变量，播放前先根据它来判断是否要播放。同时使用 **AudioServicesAddSystemSoundCompletion()** 函数添加个声音播放完毕的回调。在开始播放、播放完毕中修改这个状态变量即可。

```swift
import UIKit
import AudioToolbox
 
class ViewController: UIViewController {
     
    //表示当前是否在播放
    var isPlaying = false
 
    override func viewDidLoad() {
        super.viewDidLoad()
    }
     
    @IBAction func btn(_ sender: Any) {
        if !isPlaying {
            //建立的SystemSoundID对象
            var soundID:SystemSoundID = 0
            //获取声音地址
            let path = Bundle.main.path(forResource: "msg", ofType: "wav")
            //地址转换
            let baseURL = NSURL(fileURLWithPath: path!)
            //赋值
            AudioServicesCreateSystemSoundID(baseURL, &soundID)
             
            //添加音频结束时的回调
            let observer = UnsafeMutableRawPointer(Unmanaged.passUnretained(self).toOpaque())
            AudioServicesAddSystemSoundCompletion(soundID, nil, nil, {
                (soundID, inClientData) -> Void in
                let mySelf = Unmanaged<ViewController>.fromOpaque(inClientData!)
                    .takeUnretainedValue()
                mySelf.audioServicesPlaySystemSoundCompleted(soundID: soundID)
            }, observer)
             
            //播放声音
            AudioServicesPlaySystemSound(soundID)
            isPlaying = true
        }
    }
     
    //音频结束时的回调
    func audioServicesPlaySystemSoundCompleted(soundID: SystemSoundID) {
        print("Completion")
        isPlaying = false
        AudioServicesRemoveSystemSoundCompletion(soundID)
        AudioServicesDisposeSystemSoundID(soundID)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_771.html