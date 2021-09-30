# Swift - 实现一个圆形数字标签组件（数值变化时有动画效果）

2017-01-13发布：hangge阅读：1856

本文实现一个简单的圆形数组标签组件，平时可以作为按钮角标、或者作为表格中的计数器token等使用。

### 1，效果图

（1）组件的尺寸、背景颜色、文字颜色、字体大小都可以设置。

（2）组件只有在数字大于0的时候才会显示，其它情况下隐藏起来。

（3）数字改变时，组件还会有动画效果。（先缩小再放大，有弹性效果。）

（4）为方便演示，我里在界面上添加了个 **Stepper**，点击时数字标签组件会自动加**1**或减**1**。

[![原文:Swift - 实现一个圆形数字标签组件（数值变化时有动画效果）](https://www.hangge.com/blog_uploads/201701/2017010813120883347.png)](https://www.hangge.com/blog/cache/detail_1516.html#)



### 2，组件代码

- 整个组件通过继承 **UILabel** 来实现。
- 通过设置圆角半径和遮罩实现圆形背景。
- 文字大小自适应标签宽度，并设置中线对其。防止数字过多时显示不全。

```swift
import UIKit
 
//圆形背景的计数器组件
class CounterLabel: UILabel {
     
    //设置数值
    var num:Int = 0{
        didSet{
            //如果num小于等于0则不显示
            if num <= 0{
                self.isHidden = true
            }else{
                self.isHidden = false
                self.text = "\(num)"
                playAnimate()
            }
        }
    }
     
    //init方法
    override init(frame: CGRect) {
        super.init(frame: frame)
         
        initialSetup()
    }
     
    //init方法
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
         
        initialSetup()
    }
     
    //页面初始化相关设置
    private func initialSetup() -> Void {
        self.layer.masksToBounds = true
        self.textAlignment = .center
        //默认字体
        self.font = UIFont.systemFont(ofSize: 14)
        //背景默认为绿色
        self.backgroundColor = UIColor(red: 0x09/255, green: 0xbb/255, blue: 0x07/255,
                                       alpha: 1)
        //文字默认为白色
        self.textColor = .white
        //文字大小自适应标签宽度
        self.adjustsFontSizeToFitWidth = true
        //文本中线于label中线对齐
        self.baselineAdjustment = UIBaselineAdjustment.alignCenters
        //默认不显示，当设置了num且大于0才显示
        self.isHidden = true
    }
     
    //布局相关设置
    override func layoutSubviews() {
        super.layoutSubviews()
        //修改圆角半径
        self.layer.cornerRadius = self.bounds.width/2
    }
     
    //数字改变时播放的动画
    func playAnimate() {
        //从小变大，且有弹性效果
        self.transform = CGAffineTransform(scaleX: 0.1, y: 0.1)
        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.5,
                       initialSpringVelocity: 0.5, options: UIViewAnimationOptions(),
                       animations: {
                        self.transform = CGAffineTransform.identity
            }, completion: nil)
    }
}
```



### 3，使用样例

```swift
import UIKit
 
class ViewController: UIViewController {
     
    var counter:CounterLabel!
     
    var stepper:UIStepper!
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //计数器组件初始化
        counter = CounterLabel(frame:CGRect(x:50, y:60, width:20, height:20))
        counter.num = 5
        self.view.addSubview(counter)
         
        //步进器初始化
        stepper = UIStepper()
        stepper.center = CGPoint(x:60, y:120)
        stepper.value = Double(counter.num)
        stepper.stepValue = 1
        stepper.isContinuous = true
        stepper.addTarget(self, action:#selector(stepperValueChanged),
                          for: .valueChanged)
        self.view.addSubview(stepper)
    }
     
    //步进器值改变
    func stepperValueChanged(){
        counter.num = Int(stepper.value)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1516.html