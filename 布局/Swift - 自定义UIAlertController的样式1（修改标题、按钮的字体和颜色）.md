# Swift - 自定义UIAlertController的样式1（修改标题、按钮的字体和颜色）

2017-04-21发布：hangge阅读：5947

自 **iOS8** 起，苹果把 **UIActionSheet** 和 **UIAlertView** 合并为了 **UIAlertController**。**UIAlertController** 的用法我之前也写过相关文章：[Swift - 告警提示框（UIAlertController）的用法](https://www.hangge.com/blog/cache/detail_651.html)。其默认样式如下：

[![原文:Swift - 自定义UIAlertController的样式1（修改标题、按钮的字体和颜色）](https://www.hangge.com/blog_uploads/201704/2017041611223213568.png)](https://www.hangge.com/blog/cache/detail_1658.html#)

有网友问这个 **UIAlertController** 默认的样式能不能修改。虽然 **UIAlertController** 没有直接提供相关的属性或方法来修改样式，但我们可以通过 **KVC** 机制（在运行时动态访问和修改对象的属性）来实现。

## 一、修改标题的字体和颜色

### 1，效果图

（1）标题抬头颜色改成红色，字号放大（**20**）

（2）标题内容颜色改成灰色，并使用斜体字。

[![原文:Swift - 自定义UIAlertController的样式1（修改标题、按钮的字体和颜色）](https://www.hangge.com/blog_uploads/201704/2017041611224011305.png)](https://www.hangge.com/blog/cache/detail_1658.html#)

### 2，自定义UIAlertController类

为方便使用，我们这里通过继承 **UIAlertController** 实现一个自定义的类：**MyAlertController**。并在其 **viewDidLoad** 方法中动态改变标题样式。

```
import` `UIKit` `class` `MyAlertController``: ``UIAlertController` `{``  ` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``    ` `    ``//标题字体样式（红色，字体放大）``    ``let` `titleFont = ``UIFont``.systemFont(ofSize: 20)``    ``let` `titleAttribute = ``NSMutableAttributedString``.``init``(string: ``self``.title!)``    ``titleAttribute.addAttributes([``NSFontAttributeName``:titleFont,``                   ``NSForegroundColorAttributeName``:``UIColor``.red],``                   ``range:``NSMakeRange``(0, (``self``.title?.characters.count)!))``    ``self``.setValue(titleAttribute, forKey: ``"attributedTitle"``)``    ` `    ``//消息内容样式（灰色斜体）``    ``let` `messageFontDescriptor = ``UIFontDescriptor``.``init``(fontAttributes: [``      ``UIFontDescriptorFamilyAttribute``:``"Arial"``,``      ``UIFontDescriptorNameAttribute``:``"Arial-ItalicMT"``,``      ``])``    ` `    ``let` `messageFont = ``UIFont``.``init``(descriptor: messageFontDescriptor, size: 13.0)``    ``let` `messageAttribute = ``NSMutableAttributedString``.``init``(string: ``self``.message!)``    ``messageAttribute.addAttributes([``NSFontAttributeName``:messageFont,``                    ``NSForegroundColorAttributeName``:``UIColor``.gray],``                  ``range:``NSMakeRange``(0, (``self``.message?.characters.count)!))``    ``self``.setValue(messageAttribute, forKey: ``"attributedMessage"``)``  ``}``  ` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```



### 3，使用样例

```
import` `UIKit` `class` `ViewController``: ``UIViewController` `{``  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``  ``}``  ` `  ``override` `func` `viewDidAppear(_ animated: ``Bool``){``    ``super``.viewDidAppear(animated)``    ` `    ``let` `alertController = ``MyAlertController``(title: ``"系统提示"``,``                        ``message: ``"您确定要离开 hangge.com 吗？"``,``                        ``preferredStyle: .alert)``    ``let` `cancelAction = ``UIAlertAction``(title: ``"取消"``, style: .cancel, handler: ``nil``)``    ``let` `okAction = ``UIAlertAction``(title: ``"好的"``, style: .``default``, handler: {``      ``action ``in``      ``print``(``"点击了确定"``)``    ``})``    ``alertController.addAction(cancelAction)``    ``alertController.addAction(okAction)``    ``self``.present(alertController, animated: ``true``, completion: ``nil``)``  ``}``  ` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```



## 二、修改按钮的样式

### 1，修改按钮颜色

（1）下面我们将按钮的颜色改成橙色。

[![原文:Swift - 自定义UIAlertController的样式1（修改标题、按钮的字体和颜色）](https://www.hangge.com/blog_uploads/201704/2017041611361246929.png)](https://www.hangge.com/blog/cache/detail_1658.html#)



（2）为方便使用，我们这里通过继承 **UIAlertController** 实现一个自定义的类：**MyAlertController**。并在其 **addAction** 方法中动态改变按钮样式。

```
import` `UIKit` `class` `MyAlertController``: ``UIAlertController` `{``  ` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``  ``}``  ` `  ``override` `func` `addAction(_ action: ``UIAlertAction``) {``    ``super``.addAction(action)``    ``//通过tintColor实现按钮颜色的修改。``    ``self``.view.tintColor = ``UIColor``.orange``    ``//也可以通过设置 action.setValue 来实现``    ``//action.setValue(UIColor.orange, forKey:"titleTextColor")``  ``}``  ` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```



### 2，在按钮上添加图标

（1）下面我们在确定和取消按钮上分别添加相应的图标，同时两个按钮的文字颜色也不一样。

[![原文:Swift - 自定义UIAlertController的样式1（修改标题、按钮的字体和颜色）](https://www.hangge.com/blog_uploads/201704/2017041611554646524.png)](https://www.hangge.com/blog/cache/detail_1658.html#)

（2）同样，我们在 **UIAlertController** 的 **addAction** 方法中动态改变按钮样式。

```
import` `UIKit` `class` `MyAlertController``: ``UIAlertController` `{``  ` `  ``override` `func` `viewDidLoad() {``    ``super``.viewDidLoad()``  ``}``  ` `  ``override` `func` `addAction(_ action: ``UIAlertAction``) {``    ``super``.addAction(action)``    ` `    ``//设置确定按钮图标和样式``    ``if` `action.style == .``default` `{``      ``//使用图片原来颜色``      ``let` `iconImage = ``UIImage``(named:``"tick"``)?.withRenderingMode(.alwaysOriginal)``      ``action.setValue(iconImage, forKey: ``"image"``)``      ``action.setValue(``UIColor``.green, forKey:``"titleTextColor"``)``      ` `    ``}``    ``//设置取消按钮图片和样式``    ``else` `if` `action.style == .cancel {``      ``let` `iconImage = ``UIImage``(named:``"multiply"``)?.withRenderingMode(.alwaysOriginal)``      ``action.setValue(iconImage, forKey: ``"image"``)``      ``action.setValue(``UIColor``.red, forKey:``"titleTextColor"``)``    ``}``  ``}``  ` `  ``override` `func` `didReceiveMemoryWarning() {``    ``super``.didReceiveMemoryWarning()``  ``}``}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1658.html