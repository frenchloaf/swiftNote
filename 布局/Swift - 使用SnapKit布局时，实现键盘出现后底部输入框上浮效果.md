# Swift - 使用SnapKit布局时，实现键盘出现后底部输入框上浮效果

2016-06-14发布：hangge阅读：2620

我之前写过一篇文章：[Swift - 键盘出现后自动改变页面布局，防止下方元素被键盘遮挡](https://www.hangge.com/blog/cache/detail_911.html)。介绍了通过监听键盘通知（**UIKeyboardWillChangeFrameNotification**），并在通知响应中修改底部文本输入框的约束条件，从而实现虚拟键盘的跟随效果。即键盘出现时，页面下方的输入框也会自动上移，防止被遮挡。

**使用SnapKit布局时，如何实现键盘跟随？**

之前的文章中，我们页面使用的是 **Constraints** 约束布局。如果页面元素是使用 **SnapKit** 这个第三方布局库来布局的话，也是可以实现键盘的跟随上移效果。

实现方式同样是监听键盘通知，然后改变下约束，从而实现上移动画效果。

**效果图如下：**

​    [![原文:Swift - 使用SnapKit布局时，实现键盘出现后底部输入框上浮效果](https://www.hangge.com/blog_uploads/201606/2016061314280892462.png)](https://www.hangge.com/blog/cache/detail_1228.html#)    [![原文:Swift - 使用SnapKit布局时，实现键盘出现后底部输入框上浮效果](https://www.hangge.com/blog_uploads/201606/2016061314281486063.png)](https://www.hangge.com/blog/cache/detail_1228.html#)

**代码如下：**

```swift
import UIKit
import SnapKit
 
class ViewController: UIViewController {
     
    //底部工具栏下约束
    var bottomConstraint: Constraint?
 
    //底部工具栏视图
    var toolBar = UIView()
     
    //发送按钮
    var sendBtn = UIButton(type: .System)
     
    //文本输入框
    var textField = UITextField()
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //添加底部工具栏视图
        toolBar.backgroundColor = UIColor(red: 0, green: 0, blue: 0, alpha: 0.1)
        self.view.addSubview(toolBar)
         
        //设置底部工具栏视图约束
        toolBar.snp_makeConstraints { (make) -> Void in
            make.left.right.equalTo(self.view)
            make.height.equalTo(50)
            self.bottomConstraint = make.bottom.equalTo(self.view).constraint
        }
         
        //添加按钮
        sendBtn.setTitle("发送", forState: .Normal)
        sendBtn.setTitleColor(UIColor.whiteColor(),forState: .Normal)
        sendBtn.backgroundColor=UIColor.orangeColor()
        sendBtn.addTarget(self,action:#selector(sendMessage(_:)),forControlEvents:.TouchUpInside)
        toolBar.addSubview(sendBtn)
         
        //设置按钮约束
        sendBtn.snp_makeConstraints { (make) -> Void in
            make.width.equalTo(60)
            make.height.equalTo(30)
            make.centerY.equalTo(toolBar)
            make.right.equalTo(toolBar).offset(-10)
        }
         
        //添加输入框
        textField.borderStyle = UITextBorderStyle.RoundedRect
        toolBar.addSubview(textField)
         
        //设置输入框约束
        textField.snp_makeConstraints { (make) -> Void in
            make.left.equalTo(toolBar).offset(10)
            make.right.equalTo(sendBtn.snp_left).offset(-10)
            make.height.equalTo(30)
            make.centerY.equalTo(toolBar)
        }
         
        //监听键盘通知
        NSNotificationCenter.defaultCenter().addObserver(self,
                        selector: #selector(ViewController.keyboardWillChange(_:)),
                        name: UIKeyboardWillChangeFrameNotification, object: nil)
    }
     
    //键盘改变
    func keyboardWillChange(notification: NSNotification) {
        if let userInfo = notification.userInfo,
            value = userInfo[UIKeyboardFrameEndUserInfoKey] as? NSValue,
            duration = userInfo[UIKeyboardAnimationDurationUserInfoKey] as? Double,
            curve = userInfo[UIKeyboardAnimationCurveUserInfoKey] as? UInt {
             
            let frame = value.CGRectValue()
            let intersection = CGRectIntersection(frame, self.view.frame)
             
            //self.view.setNeedsLayout()
            //改变下约束
            self.bottomConstraint?.updateOffset(-CGRectGetHeight(intersection))
             
            UIView.animateWithDuration(duration, delay: 0.0,
                                       options: UIViewAnimationOptions(rawValue: curve),
                                       animations: { _ in
                                        self.view.layoutIfNeeded()
                }, completion: nil)
        }
    }
     
    //发送按钮点击
    func sendMessage(sender: AnyObject) {
        //关闭键盘
        textField.resignFirstResponder()
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

```



**SnapKit相关文章：**
[Swift - 自动布局库SnapKit的使用详解1（配置、使用方法、样例）](https://www.hangge.com/blog/cache/detail_1097.html)
[Swift - 自动布局库SnapKit的使用详解2（约束的更新、移除、重做）](https://www.hangge.com/blog/cache/detail_1110.html)
[Swift - 自动布局库SnapKit的使用详解3（约束优先级，约束做动画）](https://www.hangge.com/blog/cache/detail_1114.html)
[Swift - 自动布局库SnapKit的使用详解4（样例1：实现一个登录页面）](https://www.hangge.com/blog/cache/detail_1112.html)
[Swift - 自动布局库SnapKit的使用详解5（样例2：实现一个计算器界面）](https://www.hangge.com/blog/cache/detail_1113.html)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1228.html#