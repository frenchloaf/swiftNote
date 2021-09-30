# Swift - 省市县三级联动功能的实现（使用UIPickerView选择框）

2016-06-30发布：hangge阅读：6582

（本文代码已升级至Swift4）

省、市、县（地区）的联动选择功能在实际项目中很常见。在之前的文章（[Swift - 选择框（UIPickerView）的用法](https://www.hangge.com/blog/cache/detail_541.html)）中，我介绍了 **UIPickerView** 的基本用法，本文演示如何实现联动功能。

**1，三级联动效果图**
（1）切换省时，后面市县两级数据会动态改变。
（2）切换市时，后面的县级数据会动态改变。
（3）点击“获取信息”，取得最终选择的省、市、县索引及其名字，并打印出来。
[![原文:Swift - 省市县三级联动功能的实现（使用UIPickerView选择框）](https://www.hangge.com/blog_uploads/201606/2016063011072211988.png)](https://www.hangge.com/blog/cache/detail_1169.html#)[![原文:Swift - 省市县三级联动功能的实现（使用UIPickerView选择框）](https://www.hangge.com/blog_uploads/201606/2016063011072880275.png)](https://www.hangge.com/blog/cache/detail_1169.html#)[![原文:Swift - 省市县三级联动功能的实现（使用UIPickerView选择框）](https://www.hangge.com/blog_uploads/201606/2016063011073447140.png)](https://www.hangge.com/blog/cache/detail_1169.html#)

**2，地址信息文件**

首先我们要准备一个包含所有地址信息的plist数据文件（**address.plist**），省市县的数据是一级级嵌套的树形结构，如下图：

[![原文:Swift - 省市县三级联动功能的实现（使用UIPickerView选择框）](https://www.hangge.com/blog_uploads/201606/2016063011082487834.png)](https://www.hangge.com/blog/cache/detail_1169.html#)


部分数据节选如下（完整数据见我上传的源码包）：

```xml
<plist version="1.0">
    <array>
        <dict>
            <key>cities</key>
            <array>
                <dict>
                    <key>areas</key>
                    <array>
                        <string>普陀区</string>
                        <string>定海区</string>
                        <string>嵊泗县</string>
                        <string>岱山县</string>
                    </array>
                    <key>city</key>
                    <string>舟山</string>
                </dict>
                <dict>
                    <key>areas</key>
                    <array>
                        <string>萧山区</string>
                        <string>余杭区</string>
                        <string>桐庐县</string>
                        <string>淳安县</string>
                        <string>临安市</string>
                        <string>滨江区</string>
                        <string>富阳市</string>
                    </array>
                    <key>city</key>
                    <string>杭州</string>
                </dict>
            </array>
            <key>state</key>
            <string>浙江</string>
        </dict>
        <dict>
            <key>cities</key>
            <array>
                <dict>
                    <key>areas</key>
                    <array>
                        <string>连云区</string>
                        <string>新浦区</string>
                        <string>灌云县</string>
                        <string>东海县</string>
                    </array>
                    <key>city</key>
                    <string>连云港</string>
                </dict>
            </array>
            <key>state</key>
            <string>江苏</string>
        </dict> 
    </array>
</plist>

```


**3，功能实现**
（1）程序启动后，首先从 **address.plist** 中将所有的地址数据读取出来，存储到本地变量（数组形式）。
（2）**pickerView** 默认加载所有省，以及第一个省的所有市数据，以及第一个市的所有县数据（如果有的话）。
（3）通过 **pickerView** 选中项的改变方法 **didSelectRow**，取得选中的列和行索引。随后调用 **reloadComponent()** 方法，根据索引刷新显示对应的内容。

```swift
import UIKit
 
class ViewController:UIViewController, UIPickerViewDelegate, UIPickerViewDataSource{
     
    //选择器
    var pickerView:UIPickerView!
     
    //所以地址数据集合
    var addressArray = [[String: AnyObject]]()
     
    //选择的省索引
    var provinceIndex = 0
    //选择的市索引
    var cityIndex = 0
    //选择的县索引
    var areaIndex = 0
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化数据
        let path = Bundle.main.path(forResource: "address", ofType:"plist")
        self.addressArray = NSArray(contentsOfFile: path!) as! Array
         
        //创建选择器
        pickerView=UIPickerView()
        //将dataSource设置成自己
        pickerView.dataSource=self
        //将delegate设置成自己
        pickerView.delegate=self
        self.view.addSubview(pickerView)
         
        //建立一个按钮，触摸按钮时获得选择框被选择的索引
        let button = UIButton(frame:CGRect(x:0, y:0, width:100, height:30))
        button.center = self.view.center
        button.backgroundColor = UIColor.blue
        button.setTitle("获取信息",for:.normal)
        button.addTarget(self, action:#selector(ViewController.getPickerViewValue),
                         for: .touchUpInside)
        self.view.addSubview(button)
    }
     
     
    //设置选择框的列数为3列,继承于UIPickerViewDataSource协议
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        return 3
    }
     
    //设置选择框的行数，继承于UIPickerViewDataSource协议
    func pickerView(_ pickerView: UIPickerView,
                    numberOfRowsInComponent component: Int) -> Int {
        if component == 0 {
            return self.addressArray.count
        } else if component == 1 {
            let province = self.addressArray[provinceIndex]
            return province["cities"]!.count
        } else {
            let province = self.addressArray[provinceIndex]
            if let city = (province["cities"] as! NSArray)[cityIndex]
                as? [String: AnyObject] {
                return city["areas"]!.count
            } else {
                return 0
            }
        }
    }
     
    //设置选择框各选项的内容，继承于UIPickerViewDelegate协议
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int,
                    forComponent component: Int) -> String? {
            if component == 0 {
                return self.addressArray[row]["state"] as? String
            }else if component == 1 {
                let province = self.addressArray[provinceIndex]
                let city = (province["cities"] as! NSArray)[row]
                    as! [String: AnyObject]
                return city["city"] as? String
            }else {
                let province = self.addressArray[provinceIndex]
                let city = (province["cities"] as! NSArray)[cityIndex]
                    as! [String: AnyObject]
                return (city["areas"] as! NSArray)[row] as? String
            }
    }
     
    //选中项改变事件（将在滑动停止后触发）
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int,
                    inComponent component: Int) {
        //根据列、行索引判断需要改变数据的区域
        switch (component) {
        case 0:
            provinceIndex = row;
            cityIndex = 0;
            areaIndex = 0;
            pickerView.reloadComponent(1);
            pickerView.reloadComponent(2);
            pickerView.selectRow(0, inComponent: 1, animated: false)
            pickerView.selectRow(0, inComponent: 2, animated: false)
        case 1:
            cityIndex = row;
            areaIndex = 0;
            pickerView.reloadComponent(2);
            pickerView.selectRow(0, inComponent: 2, animated: false)
        case 2:
            areaIndex = row;
        default:
            break;
        }
    }
     
    //触摸按钮时，获得被选中的索引
    @objc func getPickerViewValue(){
        //获取选中的省
        let p = self.addressArray[provinceIndex]
        let province = p["state"]!
         
        //获取选中的市
        let c = (p["cities"] as! NSArray)[cityIndex] as! [String: AnyObject]
        let city = c["city"] as! String
         
        //获取选中的县（地区）
        var area = ""
        if (c["areas"] as! [String]).count > 0 {
            area = (c["areas"] as! [String])[areaIndex]
        }
         
        //拼接输出消息
        let message = "索引：\(provinceIndex)-\(cityIndex)-\(areaIndex)\n"
            + "值：\(province) - \(city) - \(area)"
         
        //消息显示
        let alertController = UIAlertController(title: "您选择了",
                                    message: message, preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }
}
```

源码下载：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1169.zip](https://www.hangge.com/blog_uploads/201709/2017092616283382244.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1169.html