# Swift - 使用CoreTelephony获取运营商信息、网络制式（4G、3G、2G）

2017-07-03发布：hangge阅读：2832

在项目开发中，有时需要获取当前设备的运营商信息（是电信、还是联通、移动）。又或者想知道当前设备使用的移动网络制式（**4G**、**3G**、还是 **2G**）。这个借助系统的 **CoreTelephony** 框架就能够实现。

### 1，效果图

程序启动后自动获取手机的运营商、网络制式等信息，并输出到控制台中。

[![原文:Swift - 使用CoreTelephony获取运营商信息、网络制式（4G、3G、2G）](https://www.hangge.com/blog_uploads/201706/2017063020113993686.png)](https://www.hangge.com/blog/cache/detail_1607.html#)



### 2，样例代码

```swift
import UIKit
import CoreTelephony
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取并输出运营商信息
        let info = CTTelephonyNetworkInfo()
        if let carrier = info.subscriberCellularProvider {
            let currentRadioTech = info.currentRadioAccessTechnology!
            print("数据业务信息：\(currentRadioTech)")
            print("网络制式：\(getNetworkType(currentRadioTech: currentRadioTech))")
            print("运营商名字：\(carrier.carrierName!)")
            print("移动国家码(MCC)：\(carrier.mobileCountryCode!)")
            print("移动网络码(MNC)：\(carrier.mobileNetworkCode!)")
            print("ISO国家代码：\(carrier.isoCountryCode!)")
            print("是否允许VoIP：\(carrier.allowsVOIP)")
        }
    }
     
    //根据数据业务信息获取对应的网络类型
    func getNetworkType(currentRadioTech:String) -> String {
        var networkType = ""
        switch currentRadioTech {
        case CTRadioAccessTechnologyGPRS:
            networkType = "2G"
        case CTRadioAccessTechnologyEdge:
            networkType = "2G"
        case CTRadioAccessTechnologyeHRPD:
            networkType = "3G"
        case CTRadioAccessTechnologyHSDPA:
            networkType = "3G"
        case CTRadioAccessTechnologyCDMA1x:
            networkType = "2G"
        case CTRadioAccessTechnologyLTE:
            networkType = "4G"
        case CTRadioAccessTechnologyCDMAEVDORev0:
            networkType = "3G"
        case CTRadioAccessTechnologyCDMAEVDORevA:
            networkType = "3G"
        case CTRadioAccessTechnologyCDMAEVDORevB:
            networkType = "3G"
        case CTRadioAccessTechnologyHSUPA:
            networkType = "3G"
        default:
            break
        }
        return networkType
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



### 附：手机的数据业务对应的通信技术

- **CTRadioAccessTechnologyGPRS**：2G（有时又叫2.5G，介于2G和3G之间的过度技术）
- **CTRadioAccessTechnologyEdge**：2G （有时又叫2.75G，是GPRS到第三代移动通信的过渡)
- **CTRadioAccessTechnologyWCDMA**：3G
- **CTRadioAccessTechnologyHSDPA**：3G (有时又叫 3.5G)
- **CTRadioAccessTechnologyHSUPA**：3G (有时又叫 3.75G)
- **CTRadioAccessTechnologyCDMA1x** ：2G
- **CTRadioAccessTechnologyCDMAEVDORev0**：3G
- **CTRadioAccessTechnologyCDMAEVDORevA**：3G
- **CTRadioAccessTechnologyCDMAEVDORevB**：3G
- **CTRadioAccessTechnologyeHRPD**：3G (有时又叫 3.75G，是电信使用的一种3G到4G的演进技术)
- **CTRadioAccessTechnologyLTE**：4G (或者说接近4G)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1607.html