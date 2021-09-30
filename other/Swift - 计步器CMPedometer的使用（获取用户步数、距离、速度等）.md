# Swift - 计步器CMPedometer的使用（获取用户步数、距离、速度等）

2017-02-15发布：hangge阅读：3593

### 1，CMPedometer介绍

- **CMPedometer**（计步器）是 **CoreMotion** 框架中的一个功能类，主要是保存有关步行（或跑步、爬楼）的一些信息。
- **CMPedometer** 同时能保存近**7**天的步数记录，并提供了查询近**7**天内任意时间段的步数信息。
- 想要使用 **CMPedometer**，系统最低版本需要是 **iOS8**（部分功能需要 **iOS9**）

### 2，使用样例

（1）程序启动后先判断设备是否支持计步器。

（2）如果支持的话则实时获取当天的用户运动数据，比如：跑步+走路的步数、距离、爬的楼层数、下的楼层数、当前移动速度。并将这些数据不断更新到界面上。

[![原文:Swift - 计步器CMPedometer的使用（获取用户步数、距离、速度等）](https://www.hangge.com/blog_uploads/201702/2017020818464829490.png)](https://www.hangge.com/blog/cache/detail_1547.html#)

### 3，样例实现

（1）由于安全限制，首先我们要在 **info.plist** 文件中加入访问用户健康和运动信息的相关描述。

[![原文:Swift - 计步器CMPedometer的使用（获取用户步数、距离、速度等）](https://www.hangge.com/blog_uploads/201702/201702081616076710.png)](https://www.hangge.com/blog/cache/detail_1547.html#)

（2）样例代码

```swift
import UIKit
import CoreMotion
 
class ViewController: UIViewController {
    //用于显示实时信息
    @IBOutlet weak var textView: UITextView!
     
    //计步器对象
    let pedometer = CMPedometer()
     
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //开始计步器更新
        startPedometerUpdates()
    }
     
    // 开始获取步数计数据
    func startPedometerUpdates() {
        //判断设备支持情况
        guard CMPedometer.isStepCountingAvailable() else {
            self.textView.text = "\n当前设备不支持获取步数\n"
            return
        }
         
        //获取今天凌晨时间
        let cal = Calendar.current
        var comps = cal.dateComponents([.year, .month, .day], from: Date())
        comps.hour = 0
        comps.minute = 0
        comps.second = 0
        let midnightOfToday = cal.date(from: comps)!
         
        //初始化并开始实时获取数据
        self.pedometer.startUpdates (from: midnightOfToday, withHandler: { pedometerData, error in
            //错误处理
            guard error == nil else {
                print(error!)
                return
            }
             
            //获取各个数据
            var text = "---今日运动数据---\n"
            if let numberOfSteps = pedometerData?.numberOfSteps {
                text += "步数: \(numberOfSteps)\n"
            }
            if let distance = pedometerData?.distance {
                text += "距离: \(distance)\n"
            }
            if let floorsAscended = pedometerData?.floorsAscended {
                text += "上楼: \(floorsAscended)\n"
            }
            if let floorsDescended = pedometerData?.floorsDescended {
                text += "下楼: \(floorsDescended)\n"
            }
            if let currentPace = pedometerData?.currentPace {
                text += "速度: \(currentPace)m/s\n"
            }
            if let currentCadence = pedometerData?.currentCadence {
                text += "速度: \(currentCadence)步/秒\n"
            }
             
            //在线程中更新文本框数据
            DispatchQueue.main.async{
                self.textView.text = text
            }
        })
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1547.html