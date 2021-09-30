# Swift - 设置UILabel、UITextView的文字行技巧

2017-03-01发布：hangge阅读：14367

有时我们需要调整**标签**或**TextView的**的文本行间距大小，但这两个组件都没有相关属性可以直接设置。这个就需要借助富文本（**NSAttributedString**）来实现。

## 一、设置UILabel的行技巧

### 1，效果图

偏是默认的行修改，是将行基本修改成**20**。

   [![原文:Swift - 设置UILabel、UITextView的文字行间距](https://www.hangge.com/blog_uploads/201702/2017022710033845035.png)](https://www.hangge.com/blog/cache/detail_1570.html#)   [![原文:Swift - 设置UILabel、UITextView的文字行间距](https://www.hangge.com/blog_uploads/201702/2017022710034336218.png)](https://www.hangge.com/blog/cache/detail_1570.html#)

### 2，样例代码

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let label = UILabel(frame:CGRect(x:10, y:20, width:300, height:100))
        //设置允许换行
        label.numberOfLines = 0
        //要显示的文字
        let str = "欢迎访问hangge.com 欢迎访问hangge.com 欢迎访问hangge.com"
        //通过富文本来设置行间距
        let paraph = NSMutableParagraphStyle()
        //将行间距设置为28
        paraph.lineSpacing = 20
        //样式属性集合
        let attributes = [NSFontAttributeName:UIFont.systemFont(ofSize: 15),
                          NSParagraphStyleAttributeName: paraph]
        label.attributedText = NSAttributedString(string: str, attributes: attributes)
        self.view.addSubview(label)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```



## 二、设置UITextView的行诀

### 1，UITextView是只读的情况

也就是说**TextView的**只用来显示一段文字，不输入。那么设置行间距的方法和**标签**是一样的。下面代码将行间距修改成**20**。

[![原文:Swift - 设置UILabel、UITextView的文字行间距](https://www.hangge.com/blog_uploads/201702/2017022710293554246.png)](https://www.hangge.com/blog/cache/detail_1570.html#)



```swift
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let textView = UITextView(frame:CGRect(x:10, y:20, width:300, height:100))
        //要显示的文字
        let str = "欢迎访问hangge.com 欢迎访问hangge.com 欢迎访问hangge.com"
        //通过富文本来设置行间距
        let paraph = NSMutableParagraphStyle()
        //将行间距设置为28
        paraph.lineSpacing = 20
        //样式属性集合
        let attributes = [NSFontAttributeName:UIFont.systemFont(ofSize: 15),
                          NSParagraphStyleAttributeName: paraph]
        textView.attributedText = NSAttributedString(string: str, attributes: attributes)
        self.view.addSubview(textView)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

###  2，UITextView是可编辑的情况

如果是想在输入内容的时候就按照设置的行间距进行动态改变，那就需要将相关代码放到**TextView中**的**委托**方法里。

[![原文:Swift - 设置UILabel、UITextView的文字行间距](https://www.hangge.com/blog_uploads/201702/2017022710242570250.png)](https://www.hangge.com/blog/cache/detail_1570.html#)

```swift
import UIKit
 
class ViewController: UIViewController, UITextViewDelegate {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let textView = UITextView(frame:CGRect(x:10, y:20, width:300, height:100))
        //设置代理
        textView.delegate = self
        //添加到视图中
        self.view.addSubview(textView)
    }
     
    func textViewDidChange(_ textView: UITextView) {
        //获取文本内容
        let str = textView.text!
        //通过富文本来设置行间距
        let paraph = NSMutableParagraphStyle()
        //将行间距设置为28
        paraph.lineSpacing = 20
        //样式属性集合
        let attributes = [NSFontAttributeName:UIFont.systemFont(ofSize: 15),
                          NSParagraphStyleAttributeName: paraph]
        textView.attributedText = NSAttributedString(string: str, attributes: attributes)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1570.html