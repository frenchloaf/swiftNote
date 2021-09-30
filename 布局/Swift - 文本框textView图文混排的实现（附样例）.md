# Swift - 文本框textView图文混排的实现（附样例）

2016-08-10发布：hangge阅读：4985

（本文代码已升级至Swift3）

我们使用文本框（**UITextView**）时，除了输入文字外，可能还会想在里面插入一些图片。或者有一些图文混排的内容需要展示出来。 这个只需要通过 **textView** 的属性化文本即可实现。j将图片以附件的形式插入即可。

本文通过样例演示如何实现 **textView** 的图文混排，同时还可以选择插入图片的模式，是保持原图大小，还是自适应尺寸（这些可以混合使用的。）

**1，效果图**
（1）不改变插入图片的大小

[![原文:Swift - 文本框textView图文混排的实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016080910443854802.png)](https://www.hangge.com/blog/cache/detail_1213.html#)

（2）让图片与行高保持一致。这样图片就不会撑大行高，同时会与文字的大小保持一致。适合用来插入表情图标。
   [![原文:Swift - 文本框textView图文混排的实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016080910471693444.png)](https://www.hangge.com/blog/cache/detail_1213.html#)    [![原文:Swift - 文本框textView图文混排的实现（附样例）](https://www.hangge.com/blog_uploads/201608/2016080910480278900.png)](https://www.hangge.com/blog/cache/detail_1213.html#)



（3）让图片占满一行。适合普通图片或者大图的插入。

[![原文:Swift - 文本框textView图文混排的实现（附样例）](https://www.hangge.com/blog_uploads/201608/201608091048216278.png)](https://www.hangge.com/blog/cache/detail_1213.html#)



**2，样例代码**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //图文混排显示的文本区域
    @IBOutlet weak var textView: UITextView!
     
    //文字大小
    let textViewFont = UIFont.systemFont(ofSize: 22)
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化显示默认内容
        insertString("欢迎欢迎!")
        insertPicture(UIImage(named: "icon")!, mode:.fitTextLine)
        insertString("\n欢迎访问：")
        insertPicture(UIImage(named: "logo")!)
        insertPicture(UIImage(named: "bg")!, mode:.fitTextView)
    }
     
    //插入文字
    func insertString(_ text:String) {
        //获取textView的所有文本，转成可变的文本
        let mutableStr = NSMutableAttributedString(attributedString: textView.attributedText)
        //获得目前光标的位置
        let selectedRange = textView.selectedRange
        //插入文字
        let attStr = NSAttributedString(string: text)
        mutableStr.insert(attStr, at: selectedRange.location)
         
        //设置可变文本的字体属性
        mutableStr.addAttribute(NSFontAttributeName, value: textViewFont,
                                range: NSMakeRange(0,mutableStr.length))
        //再次记住新的光标的位置
        let newSelectedRange = NSMakeRange(selectedRange.location + attStr.length, 0)
         
        //重新给文本赋值
        textView.attributedText = mutableStr
        //恢复光标的位置（上面一句代码执行之后，光标会移到最后面）
        textView.selectedRange = newSelectedRange
    }
     
    //插入图片
    func insertPicture(_ image:UIImage, mode:ImageAttachmentMode = .default){
        //获取textView的所有文本，转成可变的文本
        let mutableStr = NSMutableAttributedString(attributedString: textView.attributedText)
         
        //创建图片附件
        let imgAttachment = NSTextAttachment(data: nil, ofType: nil)
        var imgAttachmentString: NSAttributedString
        imgAttachment.image = image
         
        //设置图片显示方式
        if mode == .fitTextLine {
            //与文字一样大小
            imgAttachment.bounds = CGRect(x: 0, y: -4, width: textView.font!.lineHeight,
                                          height: textView.font!.lineHeight)
        } else if mode == .fitTextView {
            //撑满一行
            let imageWidth = textView.frame.width - 10
            let imageHeight = image.size.height/image.size.width*imageWidth
            imgAttachment.bounds = CGRect(x: 0, y: 0, width: imageWidth, height: imageHeight)
        }
         
        imgAttachmentString = NSAttributedString(attachment: imgAttachment)
         
        //获得目前光标的位置
        let selectedRange = textView.selectedRange
        //插入文字
        mutableStr.insert(imgAttachmentString, at: selectedRange.location)
        //设置可变文本的字体属性
        mutableStr.addAttribute(NSFontAttributeName, value: textViewFont,
                                range: NSMakeRange(0,mutableStr.length))
        //再次记住新的光标的位置
        let newSelectedRange = NSMakeRange(selectedRange.location+1, 0)
         
        //重新给文本赋值
        textView.attributedText = mutableStr
        //恢复光标的位置（上面一句代码执行之后，光标会移到最后面）
        textView.selectedRange = newSelectedRange
        //移动滚动条（确保光标在可视区域内）
        self.textView.scrollRangeToVisible(newSelectedRange)
    }
     
    //插入图片1：保持原始尺寸
    @IBAction func btnClick1(_ sender: AnyObject) {
        insertPicture(UIImage(named: "logo")!)
    }
     
    //插入图片2：适应行高
    @IBAction func btnClick2(_ sender: AnyObject) {
        insertPicture(UIImage(named: "icon")!, mode:.fitTextLine)
    }
     
    //插入图片3：适应textView宽度
    @IBAction func btnClick3(_ sender: AnyObject) {
        insertPicture(UIImage(named: "bg")!, mode:.fitTextView)
    }
     
    //插入文字
    @IBAction func btnClick4(_ sender: AnyObject) {
        insertString("hangge.com")
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//插入的图片附件的尺寸样式
enum ImageAttachmentMode {
    case `default`  //默认（不改变大小）
    case fitTextLine  //使尺寸适应行高
    case fitTextView  //使尺寸适应textView
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1213.zip](https://www.hangge.com/blog_uploads/201703/2017032117014784254.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1213.html