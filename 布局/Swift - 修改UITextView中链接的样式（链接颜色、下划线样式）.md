# Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）

2017-05-08发布：hangge阅读：4000

我们知道多行文本框（**UITextView**）具有 **URL** 检测功能，将其开启后，它会高亮显示内容中的 **url** 链接文字，点击后便会使用 **safari** 打开这个链接。

   [![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050513492571017.png)](https://www.hangge.com/blog/cache/detail_1671.html#)   [![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050513493275250.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

之前有网友问这个链接的样式能否修改，比如加个下划线什么的。这个通过设置 **textView** 的 **linkTextAttributes** 属性就可以实现。

### 1，修改链接颜色

下面将链接颜色修改成橙色。

[![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050513560841318.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

```
import UIKit
 
class ViewController: UIViewController  {
     
    @IBOutlet weak var textView: UITextView!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        textView.text = "欢迎访问 http://www.hangge.com"
        textView.linkTextAttributes = [NSForegroundColorAttributeName : UIColor.orange]
    }
}
```



### 2，给链接增加下划线

（1）细线

[![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050514031815499.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

```swift
textView.linkTextAttributes = [NSForegroundColorAttributeName: UIColor.orange,
                               NSUnderlineStyleAttributeName: NSUnderlineStyle.styleSingle.rawValue]
```


（2）粗线

[![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050514060478625.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

```swift
textView.linkTextAttributes = [NSForegroundColorAttributeName: UIColor.orange,
                               NSUnderlineStyleAttributeName: NSUnderlineStyle.styleThick.rawValue]
```


（3）双线

[![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050514070450615.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

```swift
textView.linkTextAttributes = [NSForegroundColorAttributeName: UIColor.orange,
                               NSUnderlineStyleAttributeName: NSUnderlineStyle.styleDouble.rawValue]
```



### 3，修改下划线颜色

默认情况下下划线的颜色和链接文字颜色是一样的，我们也可将下划线修改成其他颜色。

[![原文:Swift - 修改UITextView中链接的样式（链接颜色、下划线样式）](https://www.hangge.com/blog_uploads/201705/2017050514094125886.png)](https://www.hangge.com/blog/cache/detail_1671.html#)

```
textView.linkTextAttributes = [NSForegroundColorAttributeName: UIColor.orange,
                               NSUnderlineStyleAttributeName: NSUnderlineStyle.styleSingle.rawValue,
                               NSUnderlineColorAttributeName: UIColor.green]
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1671.html