# Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）

2016-10-12发布：hangge阅读：13885

**本文相关：**
[Swift - 继承UIView实现自定义可视化组件（附记分牌样例）](https://www.hangge.com/blog/cache/detail_654.html)
[Swift - 在StoryBoard中添加使用自定义组件（自定义进度条组件为例）](https://www.hangge.com/blog/cache/detail_1016.html)

我原来写过几篇文章，介绍如何通过继承 **UIView** 来实现自定义组件（见上方列表）。当时自定义组件是使用纯代码实现的。但是如果组件里的元素比较多，布局比较复杂。那用纯代码写就比较麻烦了。对于这种复杂的自定义组件，我们可以结合 **XIB** 文件来实现。

**一、自定义组件的创建**

这里还是同前文一样，实现一个简单的进度条组件。用户可以自由设置进度条的进度、尺寸、文字颜色、进度条颜色、背景颜色。不同的是，我们这里创建的时候引入 **xib** 文件来实现布局。

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201601/2016011210220187169.png)](https://www.hangge.com/blog/cache/detail_1394.html#)

（1）新建一个自定义组件实现类 **MyProgressBar**（ 继承 **UIView**）。由于是继承 **UIView**，“**Also create XIB file**”无法勾选。这个没关系，后面我们会手动创建一个 **xib** 文件并关联。

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201610/2016101110224181861.png)](https://www.hangge.com/blog/cache/detail_1394.html#)



（2）再新建一个同名的 **XIB** 文件（带有 **View**）

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201610/201610111030196229.png)](https://www.hangge.com/blog/cache/detail_1394.html#)



（3）打开刚创建的 **xib** 文件，将 **File's Owner** 的 **Custom Class** 设置为我们自定义组件类的名字。

注意：修改的是 **MyProgressBar.xib** 中 **File’s Owner** 的 **Custom Class**，不要修改成 **MyProgressBar.xib** 里 **View** 的 **Custom Class** 了。

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201610/2016101110384546035.png)](https://www.hangge.com/blog/cache/detail_1394.html#)



（4）将 **xib** 文件中 **View** 的 **Size** 修改为 **Freedom**，**Status bar** 修改为 **None**。并将其调整成合适的尺寸。

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201610/201610111042144465.png)](https://www.hangge.com/blog/cache/detail_1394.html#)



（5）在 **View** 中添加一个 **UIView** 和一个 **UILabel**，并设置好约束和相关属性。分别作为当前进度区块，以及当前进度标签。

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201610/2016101111150527556.png)](https://www.hangge.com/blog/cache/detail_1394.html#)



（6）将上面添加的两个组件拖拽关联到 **MyProgressBar.swift**。并补充相关的组件实现代码，以及 **xib** 视图加载的相关代码。

**MyProgressBar.swift** 完整代码如下：

```swift
import UIKit
 
@IBDesignable class MyProgressBar: UIView {
     
    //显示进度的文本标签
    @IBOutlet weak var textLabel: UILabel!
 
    //显示当前进度区域
    @IBOutlet weak var bar: UIView!
     
    //进度
    @IBInspectable var percent: Int = 0 {
        didSet {
            if percent > 100 {
                percent = 100
            }else if percent < 0 {
                percent = 0
            }
            textLabel.text =  "\(percent)%"
            setNeedsLayout()
        }
    }
     
    //文本颜色
    @IBInspectable var color: UIColor = .white {
        didSet {
            textLabel.textColor = color
        }
    }
     
    //进度条颜色
    @IBInspectable var barColor: UIColor = UIColor.orange {
        didSet {
            bar.backgroundColor = barColor
        }
    }
     
    //进度条背景颜色
    @IBInspectable var barBgColor: UIColor = UIColor.lightGray {
        didSet {
            layer.backgroundColor = barBgColor.cgColor
        }
    }
     
    //初始化默认属性配置
    func initialSetup(){
        textLabel.text =  "\(self.percent)%"
        textLabel.textColor = self.color
        bar.backgroundColor = self.barColor
        layer.backgroundColor = self.barBgColor.cgColor
    }
     
    //布局相关设置
    override func layoutSubviews() {
        super.layoutSubviews()
 
        var barFrame = bounds
        barFrame.size.width *= (CGFloat(self.percent) / 100)
        bar.frame = barFrame
    }
     
    /*** 下面的几个方法都是为了让这个自定义类能将xib里的view加载进来。这个是通用的，我们不需修改。 ****/
    var contentView:UIView!
     
    //初始化时将xib中的view添加进来
    override init(frame: CGRect) {
        super.init(frame: frame)
        contentView = loadViewFromNib()
        addSubview(contentView)
        addConstraints()
        //初始化属性配置
        initialSetup()
    }
     
     //初始化时将xib中的view添加进来
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        contentView = loadViewFromNib()
        addSubview(contentView)
        addConstraints()
        //初始化属性配置
        initialSetup()
    }
    //加载xib
    func loadViewFromNib() -> UIView {
        let className = type(of: self)
        let bundle = Bundle(for: className)
        let name = NSStringFromClass(className).components(separatedBy: ".").last
        let nib = UINib(nibName: name!, bundle: bundle)
        let view = nib.instantiate(withOwner: self, options: nil).first as! UIView
        return view
    }
    //设置好xib视图约束
    func addConstraints() {
        contentView.translatesAutoresizingMaskIntoConstraints = false
        var constraint = NSLayoutConstraint(item: contentView, attribute: .leading,
                                    relatedBy: .equal, toItem: self, attribute: .leading,
                                    multiplier: 1, constant: 0)
        addConstraint(constraint)
        constraint = NSLayoutConstraint(item: contentView, attribute: .trailing,
                                        relatedBy: .equal, toItem: self, attribute: .trailing,
                                        multiplier: 1, constant: 0)
        addConstraint(constraint)
        constraint = NSLayoutConstraint(item: contentView, attribute: .top, relatedBy: .equal,
                                        toItem: self, attribute: .top, multiplier: 1, constant: 0)
        addConstraint(constraint)
        constraint = NSLayoutConstraint(item: contentView, attribute: .bottom,
                                    relatedBy: .equal, toItem: self, attribute: .bottom,
                                    multiplier: 1, constant: 0)
        addConstraint(constraint)
    }
}
```


**二、自定义组件的使用**
通过 **xib** 创建的自定义组件与之前使用纯代码实现的自定义组件相比，在使用的时候是没什么区别的。
**1，在代码中使用自定义组件**

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let myProgressBar = MyProgressBar(frame: CGRect(x:50, y:50, width:200, height:20))
        myProgressBar.percent = 50
        self.view.addSubview(myProgressBar)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，在在StoryBoard中使用自定义组件**
（1）打开**Main.storyboard**，从组件库里添加一个视图（**View**）

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201601/2016011211221513758.png)](https://www.hangge.com/blog/cache/detail_1394.html#)


（2）在**Identity Inspector**里把视图类改成**MyProgressBar**

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201601/201601121122262462.png)](https://www.hangge.com/blog/cache/detail_1394.html#)


（3）可以看到自定义组件已经渲染显示出来了

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201601/201601121122441794.png)](https://www.hangge.com/blog/cache/detail_1394.html#)


（4）在**Attributes inspector**面板中可以调整组件的各个自定义属性

[![原文:Swift - XIB结合UIView制作自定义组件（xib加载绘制UIView）](https://www.hangge.com/blog_uploads/201601/2016011211225623493.png)](https://www.hangge.com/blog/cache/detail_1394.html#)

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1394.zip](https://www.hangge.com/blog_uploads/201610/2016101114191394166.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1394.html