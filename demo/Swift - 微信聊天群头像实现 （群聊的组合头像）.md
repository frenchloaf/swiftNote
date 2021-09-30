# Swift - 微信聊天群头像实现 （群聊的组合头像）

2016-12-12发布：hangge阅读：2213

我前面写过一篇如何实现**QQ**讨论组头像的文章：[Swift - QQ讨论组头像的实现 （多人聊天的组合头像）](https://www.hangge.com/blog/cache/detail_1462.html)。这次演示如何实现微信中群聊头像。同**QQ**相比，微信里的组合头像实现起来会更简单些。

### 1，样例效果图

（1）组合图片的功能我使用扩展 **UIImage** 的方式实现。

（2）根据初始化传入的图片数组中图片数量的不同（超过**9**张图片的话也只显示前**9**个），组件会自动设置内部图片的尺寸和位置。

（3）除了只有一张图片的情况外。内部小图片尺寸实际上只有两种。即小于等于**4**张，或者大于**4**张这两种情况。

（4）生成返回的是一个 **UIImage** 对象，为了让头像图标有圆角效果。这里给 **imageView** 设置了相关圆角属性。

[![原文:Swift - 微信聊天群头像实现 （群聊的组合头像）](https://www.hangge.com/blog_uploads/201612/2016120411433559425.png)](https://www.hangge.com/blog/cache/detail_1463.html#)

### 2，样例代码

（1）**UIImageExGroupIcon.swift**（扩展 **UImage** 实现具体功能）

- 首先根据图片数量的不同，我们先初步生成田字格或9宫格这两种单元格布局。
- 再根据实际数量，删除多余单元格，并调整单元格位置。
- 最后将各个 **image** 绘制到对应的单元格雨区上。

```swift
import UIKit
 
extension UIImage {
     
    //生成群聊图标
    class func groupIcon(wh:CGFloat, images:[UIImage], bgColor:UIColor?) -> UIImage {
        let finalSize = CGSize(width:wh, height:wh)
        var rect: CGRect = CGRect.zero
        rect.size = finalSize
         
        //开始图片处理上下文（由于输出的图不会进行缩放，所以缩放因子等于屏幕的scale即可
        UIGraphicsBeginImageContextWithOptions(finalSize, false, 0)
         
        //绘制背景
        if (bgColor != nil) {
            let context: CGContext = UIGraphicsGetCurrentContext()!
            //添加矩形背景区域
            context.addRect(rect)
            //设置填充颜色
            context.setFillColor(bgColor!.cgColor)
            context.drawPath(using: .fill)
        }
         
        //绘制图片
        if images.count >= 1 {
            //获取群聊图标中每个小图片的位置尺寸
            var rects = self.getRectsInGroupIcon(wh:wh, count:images.count)
            var count = 0
            //将每张图片绘制到对应的区域上
            for image in images {
                if count > rects.count-1 {
                    break
                }
                 
                let rect = rects[count]
                image.draw(in: rect)
                count = count + 1
            }
        }
         
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return newImage!
    }
     
    //获取群聊图标中每个小图片的位置尺寸
    class func getRectsInGroupIcon(wh:CGFloat, count:Int) -> [CGRect] {
        //如果只有1张图片就直接占全部位置
        if count == 1 {
            return [CGRect(x:0, y:0, width:wh, height:wh)]
        }
         
        //下面是图片数量大于1张的情况
        var array = [CGRect]()
        //图片间距
        var padding: CGFloat = 10
        //小图片尺寸
        var cellWH: CGFloat
        //用于后面计算的单元格数量（小于等于4张图片算4格单元格，大于4张算9格单元格）
        var cellCount:Int
         
        if count <= 4 {
            cellWH = (wh-padding*3)/2
            cellCount = 4
        } else {
            padding = padding/2
            cellWH = (wh-padding*4)/3
            cellCount = 9
        }
         
        //总行数
        let rowCount = Int(sqrt(Double(cellCount)))
        //根据单元格长宽，间距，数量返回所有单元格初步对应的位置尺寸
        for i in 0..<cellCount {
            //当前行
            let row = i/rowCount
            //当前列
            let column = i%rowCount
            let rect = CGRect(x:padding*CGFloat(column+1)+cellWH*CGFloat(column),
                              y:padding*CGFloat(row+1)+cellWH*CGFloat(row),
                              width:cellWH, height:cellWH)
            array.append(rect)
        }
 
        //根据实际图片的数量再调整单元格的数量和位置
        if count == 2 {
            array.removeSubrange(0...1)
            for i in 0..<array.count {
                array[i].origin.y = array[i].origin.y - (padding+cellWH)/2
            }
        }else if count == 3 {
            array.remove(at: 0)
            array[0].origin.x = (wh-cellWH)/2
        }else if count == 5 {
            array.removeSubrange(0...3)
            for i in 0..<array.count {
                if i<2 {
                    array[i].origin.x = array[i].origin.x - (padding+cellWH)/2
                }
                array[i].origin.y = array[i].origin.y - (padding+cellWH)/2
            }
        }else if count == 6 {
            array.removeSubrange(0...2)
            for i in 0..<array.count {
                array[i].origin.y = array[i].origin.y - (padding+cellWH)/2
            }
        }else if count == 7 {
            array.removeSubrange(0...1)
            array[0].origin.x = (wh-cellWH)/2
        }
        else if count == 8 {
            array.remove(at: 0)
            for i in 0..<2 {
                array[i].origin.x = array[i].origin.x - (padding+cellWH)/2
            }
        }
        return array
    }
}
```

（2）**ViewController.swift**（测试代码）

```swift
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //图标初始化
        let image0 = UIImage(named:"0")!
        let image1 = UIImage(named:"1")!
        let image2 = UIImage(named:"2")!
        let image3 = UIImage(named:"3")!
        let image4 = UIImage(named:"4")!
        let image5 = UIImage(named:"5")!
        let image6 = UIImage(named:"6")!
        let image7 = UIImage(named:"7")!
        let image8 = UIImage(named:"8")!
 
        //聊天群图标尺寸（长宽一样）
        let viewWH:CGFloat = 100
         
        //聊天群图标背景色（这里使用灰色，不设置的话则是透明的）
        let viewBgColor = UIColor(red: 0, green:0, blue: 0, alpha: 0.1)
         
        //imageView的圆角半径
        let corner = viewWH/20
       
        //创建生成各种情况的聊天群图标
        let imageView0 = UIImageView(frame: CGRect(x:5,y:20,width:viewWH,height:viewWH))
        imageView0.image = UIImage.groupIcon(wh:viewWH, images:[image0],
                                             bgColor:viewBgColor)
        imageView0.layer.masksToBounds = true
        imageView0.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView0)
         
        let imageView1 = UIImageView(frame: CGRect(x:110,y:20,width:viewWH,height:viewWH))
        imageView1.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1],
                                             bgColor:viewBgColor)
        imageView1.layer.masksToBounds = true
        imageView1.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView1)
         
        let imageView2 = UIImageView(frame: CGRect(x:215,y:20,width:viewWH,height:viewWH))
        imageView2.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2],
                                             bgColor:viewBgColor)
        imageView2.layer.masksToBounds = true
        imageView2.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView2)
         
        let imageView3 = UIImageView(frame: CGRect(x:5,y:125,width:viewWH,height:viewWH))
        imageView3.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3],
                                             bgColor:viewBgColor)
        imageView3.layer.masksToBounds = true
        imageView3.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView3)
         
        let imageView4 = UIImageView(frame: CGRect(x:110,y:125,width:viewWH,height:viewWH))
        imageView4.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3,image4],
                                             bgColor:viewBgColor)
        imageView4.layer.masksToBounds = true
        imageView4.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView4)
         
        let imageView5 = UIImageView(frame: CGRect(x:215,y:125,width:viewWH,height:viewWH))
        imageView5.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3,image4,image5],
                                             bgColor:viewBgColor)
        imageView5.layer.masksToBounds = true
        imageView5.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView5)
         
        let imageView6 = UIImageView(frame: CGRect(x:5,y:230,width:viewWH,height:viewWH))
        imageView6.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3,image4,image5,
                                                                image6],
                                             bgColor:viewBgColor)
        imageView6.layer.masksToBounds = true
        imageView6.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView6)
         
        let imageView7 = UIImageView(frame: CGRect(x:110,y:230,width:viewWH,height:viewWH))
        imageView7.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3,image4,image5,
                                                                image6,image7],
                                             bgColor:viewBgColor)
        imageView7.layer.masksToBounds = true
        imageView7.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView7)
         
        let imageView8 = UIImageView(frame: CGRect(x:215,y:230,width:viewWH,height:viewWH))
        imageView8.image = UIImage.groupIcon(wh:viewWH, images:[image0,image1,image2,
                                                                image3,image4,image5,
                                                                image6,image7,image8],
                                             bgColor:viewBgColor)
        imageView8.layer.masksToBounds = true
        imageView8.layer.cornerRadius = corner  //圆角
        self.view.addSubview(imageView8)
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1463.zip](https://www.hangge.com/blog_uploads/201612/2016120412323541826.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1463.html