# Swift - 位拖拖回弹效果的实现（放后自动归）

2017-02-13发布：hangge阅读：3344

下面通过一个**UIView**来演示如何实现渲染回弹的效果，是为了让效果看得更明显，这里将**UIView**的设置背景设为橙色。

### 1，效果图

边框可以向上或触发效果，触发后会自动到原位置，同时具有动画效果。

[![译文:Swift - 拖放回弹效果的实现（拖放后自动归位）](https://www.hangge.com/blog_uploads/201702/2017020117300240261.gif)](https://www.hangge.com/blog/cache/detail_1540.html#)



### 2，实现步骤

（1）在**StoryBoard**中添加一个**查看**，设置橙色背景，并添加好相关约束。

[![译文:Swift - 拖放回弹效果的实现（拖放后自动归位）](https://www.hangge.com/blog_uploads/201702/2017020117372749077.png)](https://www.hangge.com/blog/cache/detail_1540.html#)



（2）模糊一个 **UIPanGestureRecognizer**到这个**View**上。

[![译文:Swift - 拖放回弹效果的实现（拖放后自动归位）](https://www.hangge.com/blog_uploads/201702/201702011739436081.png)](https://www.hangge.com/blog/cache/detail_1540.html#)



（3）切换到**视图视图**，将这个**UIPGestureRecognizer**、橙色的**视图**、以及该**视图**的**布局**约束拉近到**ViewController**中的绑定。

（4）下面是完整代码：

```swift
import UIKit
 
class ViewController: UIViewController {
    //拖动的view
    @IBOutlet weak var myView: UIView!
    //拖动view的上约束
    @IBOutlet weak var myViewTopLayoutConstraint: NSLayoutConstraint!
    //拖动手势
    @IBOutlet var panGesture: UIPanGestureRecognizer!
     
    //保存初始时拖动view上约束的值
    var myViewTopLayoutConstant: CGFloat!
    //保存初始时拖动view的y坐标值
    var myViewOriginY: CGFloat!
     
    override func viewDidLoad() {
        super.viewDidLoad()
        //添加拖动手势响应
        panGesture.addTarget(self, action: #selector(ViewController.pan))
        //保存初始时拖动view上约束的值（用于后面还原）
        myViewTopLayoutConstant = myViewTopLayoutConstraint.constant
    }
     
    //拖动手势
    func pan() {
        //拖动开始
        if panGesture.state == .began {
            myViewOriginY = myView.frame.origin.y
        }
             
        //拖动过程
        else if panGesture.state == .changed {
            let y = panGesture.translation(in: self.view).y
            myViewTopLayoutConstraint.constant = myViewTopLayoutConstant + y
        }
           
        //拖动结束
        else if panGesture.state == .ended {
            //回弹
            UIView.animate(withDuration: 0.4, delay: 0, options: .curveEaseInOut, animations: {
                () -> Void in
                self.myView.frame.origin.y = self.myViewOriginY
                }, completion: { (success) -> Void in
                    if success {
                        //回弹动画结束后恢复默认约束值
                        self.myViewTopLayoutConstraint.constant = self.myViewTopLayoutConstant
                    }
            })
            return
        }
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1540.html