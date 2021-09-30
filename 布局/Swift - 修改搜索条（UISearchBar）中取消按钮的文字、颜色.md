# Swift - 修改搜索条（UISearchBar）中取消按钮的文字、颜色

2016-06-01发布：hangge阅读：4856

我们使用搜索栏（**UISearchBar**）时，如果将其设置为“**Shows Cancel Button**”后，输入框右侧会有个取消按钮（标题文字是**Cancel**，而且不会根据当前系统的语言环境切换）。

[![原文:Swift - 修改搜索条（UISearchBar）中取消按钮的文字、颜色](https://www.hangge.com/blog_uploads/201605/2016053118182875208.png)](https://www.hangge.com/blog/cache/detail_1126.html#)


有时我们想要将这个按钮修改成其他文字，比如“搜索”。虽然 **searchBar** 不提供直接修改这个按钮文字的方法或属性，但我们可以换种方式来实现。

[![原文:Swift - 修改搜索条（UISearchBar）中取消按钮的文字、颜色](https://www.hangge.com/blog_uploads/201605/2016053118184399822.png)](https://www.hangge.com/blog/cache/detail_1126.html#)

（本文代码已升级至Swift3）


**1，修改searchBar取消按钮的文字**
先获取到 **searchBar** 中的取消按钮，再设置这个按钮的 **title** 即可。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var searchBar: UISearchBar!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let uiButton = searchBar.value(forKey: "cancelButton") as! UIButton
        uiButton.setTitle("搜索", for: .normal)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


**2，修改searchBar取消按钮的文字颜色**
同样先获取到 **searchBar** 中的取消按钮，再设置这个按钮的 **titleColor** 即可。

[![原文:Swift - 修改搜索条（UISearchBar）中取消按钮的文字、颜色](https://www.hangge.com/blog_uploads/201606/2016060614450332306.png)](https://www.hangge.com/blog/cache/detail_1126.html#)

```swift
import UIKit
 
class ViewController: UIViewController {
     
    @IBOutlet weak var searchBar: UISearchBar!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let uiButton = searchBar.value(forKey: "cancelButton") as! UIButton
        uiButton.setTitle("搜索", for: .normal)
        uiButton.setTitleColor(UIColor.orange,for: .normal)
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1126.html