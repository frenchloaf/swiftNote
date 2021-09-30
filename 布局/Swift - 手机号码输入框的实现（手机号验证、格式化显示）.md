# Swift - 手机号码输入框的实现（手机号验证、格式化显示）

2017-07-10发布：hangge阅读：5666

在开发中经常会碰到要实现手机号输入功能，通常做法都是使用 **UITextField** 来输入手机号码，然后提交的时候再用正则表单式去验证下填写内容是否合格。
本文演示如何实现 **UITextField** 中手机号码的实时验证，同时会自动地在号码中插入横杠使其显示为 **344** 格式。

### 1，效果图 

（1）输入号码的时候，会自动在相应位置插入分隔符（**-**），显示格式为：**xxx-xxxx-xxxx**。同时删除数字的时候也会自动将相应的分隔符给删除。

（2）手机号的验证是实时的，就是说每次输入文字（或者粘贴进来），都会先判断是否符合规则，如果不符合就直接丢弃。具体规则如下：

- 号码必需以 **1** 开头
- 号码最多可以输入 **11** 位
- 号码只能为数字

  [![原文:Swift - 手机号码输入框的实现（手机号验证、格式化显示）](https://www.hangge.com/blog_uploads/201707/2017070719082660736.png)](https://www.hangge.com/blog/cache/detail_1610.html#)   [![原文:Swift - 手机号码输入框的实现（手机号验证、格式化显示）](https://www.hangge.com/blog_uploads/201707/201707071908338115.png)](https://www.hangge.com/blog/cache/detail_1610.html#)

### 2，样例代码

```
import UIKit
 
class ViewController: UIViewController, UITextFieldDelegate {
         
    //保存上一次的文本内容
    var _previousText:String!
     
    //保持上一次的文本范围
    var _previousRange:UITextRange!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建手机号输入框
        let phoneField = UITextField(frame: CGRect(x:20, y:80, width:200, height:30))
 
        //默认边框样式为圆角矩形
        phoneField.borderStyle = UITextBorderStyle.roundedRect
        //使用数字键盘
        phoneField.keyboardType = UIKeyboardType.numberPad
         
        //设置代理
        phoneField.delegate = self
        //设置编辑事件响应
        phoneField.addTarget(self, action: #selector(phoneNumberFormat(_:)),
                             for: .editingChanged)
         
        //添加到页面时图中
        self.view.addSubview(phoneField)
    }
     
    //输入框内容改变时对其内容做格式化处理
    func phoneNumberFormat(_ textField: UITextField) {
        //当前光标的位置（后面会对其做修改）
        var cursorPostion = textField.offset(from: textField.beginningOfDocument,
                                             to: textField.selectedTextRange!.start)
         
        //过滤掉非数字字符，只保留数字
        var digitsText = getDigitsText(string: textField.text!,
                                       cursorPosition: &cursorPostion)
         
        //避免超过11位的输入
        if digitsText.characters.count > 11 {
            textField.text = _previousText
            textField.selectedTextRange = _previousRange
            return
        }
         
        //得到带有分隔符的字符串
        let hyphenText = getHyphenText(string: digitsText, cursorPosition: &cursorPostion)
         
        //将最终带有分隔符的字符串显示到textField上
        textField.text = hyphenText
         
        //让光标停留在正确位置
        let targetPostion = textField.position(from: textField.beginningOfDocument,
                                               offset: cursorPostion)!
        textField.selectedTextRange = textField.textRange(from: targetPostion,
                                                          to: targetPostion)
    }
   
    //除去非数字字符，同时确定光标正确位置
    func getDigitsText(string:String, cursorPosition:inout Int) -> String{
        //保存开始时光标的位置
        let originalCursorPosition = cursorPosition
        //处理后的结果字符串
        var result = ""
         
        var i = 0
        //遍历每一个字符
        for uni in string.unicodeScalars {
            //如果是数字则添加到返回结果中
            if CharacterSet.decimalDigits.contains(uni) {
                result.append(string[i])
            }
            //非数字则跳过，如果这个非法字符在光标位置之前，则光标需要向前移动
            else{
                if i < originalCursorPosition {
                    cursorPosition = cursorPosition - 1
                }
            }
            i = i + 1
        }
  
        return result
    }
     
     
    //将分隔符插入现在的string中，同时确定光标正确位置
    func getHyphenText(string:String, cursorPosition:inout Int) -> String {
        //保存开始时光标的位置
        let originalCursorPosition = cursorPosition
        //处理后的结果字符串
        var result = ""
         
        //遍历每一个字符
        for i in 0  ..< string.characters.count  {
            //如果当前到了第4个、第8个数字，则先添加个分隔符
            if i == 3 || i == 7 {
                result.append("-")
                //如果添加分隔符位置在光标前面，光标则需向后移动一位
                if i < originalCursorPosition {
                    cursorPosition = cursorPosition + 1
                }
            }
            result.append(string[i])
        }
         
        return result
    }
     
    //该方法就能在文本框将要变化的时候执行一些代码
    func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange,
                   replacementString string: String) -> Bool {
        //先保存输入框原先的值和选中范围
        _previousText = textField.text!
        _previousRange = textField.selectedTextRange!
         
        //输入的第一个数字必需为1
        if range.location == 0 {
            if (string as NSString).intValue != 1 {
                return false
            }
        }
        return true
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//通过对String扩展，字符串增加下表索引功能
extension String
{
    subscript(index:Int) -> String
    {
        get{
            return String(self[self.index(self.startIndex, offsetBy: index)])
        }
        set{
            let tmp = self
            self = ""
            for (idx, item) in tmp.characters.enumerated() {
                if idx == index {
                    self += "\(newValue)"
                }else{
                    self += "\(item)"
                }
            }
        }
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1610.zip](https://www.hangge.com/blog_uploads/201707/2017070719141371401.zip)

## 功能改进：将号码输入框封装成组件

从上面样例可以看到，为了实现号码的验证以及格式化显示需要写一堆代码，这些代码都放在主视图控制器中会显得很杂乱。而且如果一个页面上同时要输入多个手机号，那处理起来就会比较麻烦了。

为方便使用，下面我们通过继承 **UITextField** 实现一个自定义的电话号码输入框组件，将这些验证和格式化显示的代码都封装在内部。

### 1，手机号输入组件（PhoneField.swift）

```
import UIKit
 
class PhoneField: UITextField {
     
    //保存上一次的文本内容
    var _previousText:String!
     
    //保持上一次的文本范围
    var _previousRange:UITextRange!
     
    override init(frame: CGRect) {
        super.init(frame: frame)
         
        //默认边框样式为圆角矩形
        self.borderStyle = UITextBorderStyle.roundedRect
        //使用数字键盘
        self.keyboardType = UIKeyboardType.numberPad
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    //当本视图的父类视图改变的时候
    override func willMove(toSuperview newSuperview: UIView?) {
        //监听值改变通知事件
        if newSuperview != nil {
            NotificationCenter.default.addObserver(self,
                                   selector: #selector(phoneNumberFormat(_:)),
                                   name: NSNotification.Name.UITextFieldTextDidChange,
                                   object: nil)
        }else{
            NotificationCenter.default.removeObserver(self,
                                      name: Notification.Name.UITextFieldTextDidChange,
                                      object: nil)
        }
    }
     
    //输入框内容改变时对其内容做格式化处理
    func phoneNumberFormat(_ notification: Notification) {
        let textField = notification.object as! UITextField
         
        if(!textField.isEqual(self)){
            return
        }
         
        //输入的第一个数字必需为1
        if textField.text != "" && (textField.text![0] as NSString).intValue != 1 {
            //第1位输入非1数则使用原来值，且关闭停留在开始位置
            textField.text = _previousText
            let start = textField.beginningOfDocument
            textField.selectedTextRange = textField.textRange(from: start, to: start)
            return
        }
         
        //当前光标的位置（后面会对其做修改）
        var cursorPostion = textField.offset(from: textField.beginningOfDocument,
                                             to: textField.selectedTextRange!.start)
         
        //过滤掉非数字字符，只保留数字
        var digitsText = getDigitsText(string: textField.text!,
                                       cursorPosition: &cursorPostion)
         
        //避免超过11位的输入
        if digitsText.characters.count > 11 {
            textField.text = _previousText
            textField.selectedTextRange = _previousRange
            return
        }
         
        //得到带有分隔符的字符串
        let hyphenText = getHyphenText(string: digitsText, cursorPosition: &cursorPostion)
         
        //将最终带有分隔符的字符串显示到textField上
        textField.text = hyphenText
         
        //让光标停留在正确位置
        let targetPostion = textField.position(from: textField.beginningOfDocument,
                                               offset: cursorPostion)!
        textField.selectedTextRange = textField.textRange(from: targetPostion,
                                                          to: targetPostion)
         
        //现在的值和选中范围，供下一次输入使用
        _previousText = self.text!
        _previousRange = self.selectedTextRange!
    }
     
    //除去非数字字符，同时确定光标正确位置
    func getDigitsText(string:String, cursorPosition:inout Int) -> String{
        //保存开始时光标的位置
        let originalCursorPosition = cursorPosition
        //处理后的结果字符串
        var result = ""
         
        var i = 0
        //遍历每一个字符
        for uni in string.unicodeScalars {
            //如果是数字则添加到返回结果中
            if CharacterSet.decimalDigits.contains(uni) {
                result.append(string[i])
            }
                //非数字则跳过，如果这个非法字符在光标位置之前，则光标需要向前移动
            else{
                if i < originalCursorPosition {
                    cursorPosition = cursorPosition - 1
                }
            }
            i = i + 1
        }
         
        return result
    }
     
    //将分隔符插入现在的string中，同时确定光标正确位置
    func getHyphenText(string:String, cursorPosition:inout Int) -> String {
        //保存开始时光标的位置
        let originalCursorPosition = cursorPosition
        //处理后的结果字符串
        var result = ""
         
        //遍历每一个字符
        for i in 0  ..< string.characters.count  {
            //如果当前到了第4个、第8个数字，则先添加个分隔符
            if i == 3 || i == 7 {
                result.append("-")
                //如果添加分隔符位置在光标前面，光标则需向后移动一位
                if i < originalCursorPosition {
                    cursorPosition = cursorPosition + 1
                }
            }
            result.append(string[i])
        }
         
        return result
    }
}
 
//通过对String扩展，字符串增加下表索引功能
extension String
{
    subscript(index:Int) -> String
    {
        get{
            return String(self[self.index(self.startIndex, offsetBy: index)])
        }
        set{
            let tmp = self
            self = ""
            for (idx, item) in tmp.characters.enumerated() {
                if idx == index {
                    self += "\(newValue)"
                }else{
                    self += "\(item)"
                }
            }
        }
    }
}
```



### 2，使用样例

```
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //创建并添加手机号输入框
        let phoneField = PhoneField(frame: CGRect(x:20, y:80, width:200, height:30))
        self.view.addSubview(phoneField)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1610_2.zip](https://www.hangge.com/blog_uploads/201709/2017091219324377713.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1610.html