# Swift - 通过UIView对象找到其所在的UIViewController

2017-01-26发布：hangge阅读：3097

有时我们需要通过 **UIView** 对象获取该对象所属的 **UIViewController**。比如我们在自定义单元格（**tableViewCell**）中需要对导航控制器（**navigationController**）进行一些操作，就需要先获取到其所在的 **UIViewController**。

### 1，实现原理

我们可以通过消息响应者链找到 **UIView** 所在的 **UIViewController**。

[![原文:Swift - 通过UIView对象找到其所在的UIViewController](https://www.hangge.com/blog_uploads/201701/2017011411435558881.jpg)](https://www.hangge.com/blog/cache/detail_1521.html#)



- **UIView** 类继承于 **UIResponder**，通过 **UIResponder** 的**next** 方法来获取 **UIViewController**。
- 如果 **next** 返回是空，则继续向上遍历 **superview** 并再次使用 **next** 方法获取。这样一直找下去，直到找到或抛出异常。

### 2，实现代码

通过扩展 **UIView**，给其添加个 **firstViewController** 方法，从而让我们可以获得任意视图对象（**View**）所属视图控制器。

```swift
extension UIView {
    //返回该view所在VC
    func firstViewController() -> UIViewController? {
        for view in sequence(first: self.superview, next: { $0?.superview }) {
            if let responder = view?.next {
                if responder.isKind(of: UIViewController.self){
                    return responder as? UIViewController
                }
            }
        }
        return nil
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1521.html