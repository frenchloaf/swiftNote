# Swift - 活动器CMMotionActivityManager的使用（获取当前的运动状态）

2017-02-20发布：hangge阅读：1423

### 1，CMMotionActivityManager介绍

活动器 **CMMotionActivityManager** 是 **CoreMotion** 框架中的一个功能类，主要用于检查用户当前的活动状态。

总共可以检测到 **5** 种状态：**静止**、**步行**、**跑步**、**自行车**、**驾车**。

同时每次返回结果时，都会附带本次结果准确程度的描述，共有：**低** 、**中** 、**高**三个等级。

### 2，使用样例

（1）程序启动后先判断设备是否支持活动器（需要5S或以后的设备）。

（2）如果支持的话则实时获取当前的运动状态以及精确度。并将这些数据不断更新到界面上。

[![原文:Swift - 活动器CMMotionActivityManager的使用（获取当前的运动状态）](https://www.hangge.com/blog_uploads/201702/2017020915492857422.png)](https://www.hangge.com/blog/cache/detail_1552.html#)

### 3，样例实现

（1）由于安全限制，首先我们要在 **info.plist** 文件中加入访问用户健康和运动信息的相关描述。

[![原文:Swift - 活动器CMMotionActivityManager的使用（获取当前的运动状态）](https://www.hangge.com/blog_uploads/201702/201702081616076710.png)](https://www.hangge.com/blog/cache/detail_1552.html#)

（2）样例代码

```swift
import UIKit
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示实时信息
    @IBOutlet weak var textView: UITextView!
     
    //活动器对象
    let motionActivityManager = CMMotionActivityManager()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //开始获取活动状态
        startActivityUpdates()
    }
     
    // 开始获取活动器数据
    func startActivityUpdates() {
        //判断设备支持情况
        guard CMMotionActivityManager.isActivityAvailable() else {
            self.textView.text = "\n当前设备不支持获取当前运动状态\n"
            return
        }
         
        //初始化并开始实时获取数据
        let queue = OperationQueue.current
        self.motionActivityManager.startActivityUpdates(to: queue!, withHandler: {
            activity in
            //获取各个数据
            var text = "---活动器数据---\n"
            text += "当前状态: \(activity!.getDescription())\n"
            if (activity!.confidence == .low) {
                text += "准确度: 低\n"
            } else if (activity!.confidence == .medium) {
                text += "准确度: 低\n"
            } else if (activity!.confidence == .high) {
                text += "准确度: 高\n"
            }
            self.textView.text = text
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
extension CMMotionActivity {
    /// 获取用户设备当前所处环境的描述
    func getDescription() -> String {
        if self.stationary {
            return "静止"
        } else if self.walking {
            return "步行"
        } else if self.running {
            return "跑步"
        } else if self.automotive {
            return "驾车"
        }else if self.cycling {
            return "自行车"
        }
        return "未知"
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1552.html