# Swift - 圆形图片的生成及显示（两种办法）

2017-04-10发布：hangge阅读：6996

有时我们需要将一些矩形图片处理成圆形并显示，比如：功能图标、用户头像等。下面介绍两种显示圆形图片的方法。 

[![原文:Swift - 圆形图片的生成及显示（两种办法）](https://www.hangge.com/blog_uploads/201701/2017013013563334300.png)](https://www.hangge.com/blog/cache/detail_1535.html#)



### 1，通过设置imageView圆角来实现

这种方法实际上没有对原始图片进行处理。只不过在展示的时候，通过设置 **UIImageView** 圆角半径，从而显示成圆形图片。

```swift
//获取图片
let image = UIImage(named: "image1")
//创建imageView
let imageView = UIImageView(image: image)
imageView.frame = CGRect(x:40, y:40, width:100, height:100)
imageView.contentMode = .scaleAspectFill
//设置遮罩
imageView.layer.masksToBounds = true
//设置圆角半径(宽度的一半)，显示成圆形。
imageView.layer.cornerRadius = imageView.frame.width/2
self.view.addSubview(imageView)
```

### 2，通过图片裁剪来实现

这种方式是通过遮罩剪切的方法，重新生成一张圆形的图片。

（1）扩展 **UIImage**，增加圆形裁剪方法

```swift
extension UIImage {
    //生成圆形图片
    func toCircle() -> UIImage {
        //取最短边长
        let shotest = min(self.size.width, self.size.height)
        //输出尺寸
        let outputRect = CGRect(x: 0, y: 0, width: shotest, height: shotest)
        //开始图片处理上下文（由于输出的图不会进行缩放，所以缩放因子等于屏幕的scale即可）
        UIGraphicsBeginImageContextWithOptions(outputRect.size, false, 0)
        let context = UIGraphicsGetCurrentContext()!
        //添加圆形裁剪区域
        context.addEllipse(in: outputRect)
        context.clip()
        //绘制图片
        self.draw(in: CGRect(x: (shotest-self.size.width)/2,
                              y: (shotest-self.size.height)/2,
                              width: self.size.width,
                              height: self.size.height))
        //获得处理后的图片
        let maskedImage = UIGraphicsGetImageFromCurrentImageContext()!
        UIGraphicsEndImageContext()
        return maskedImage
    }
}
```


（2）使用样例

```swift
//获取图片
let image = UIImage(named: "image1")?.toCircle()
//创建imageView
let imageView = UIImageView(image: image)
imageView.frame = CGRect(x:40, y:40, width:100, height:100)
self.view.addSubview(imageView)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1535.html