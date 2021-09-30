# Swift3新特性汇总（Swift版本的新变化）

2016-09-28发布：hangge阅读：3414

之前**苹果**在**WWDC**上已经将**Swift 3**整合进了**Xcode 8 beta**中，而本月苹果发布了**Swift 3**的正式版。这也是自**2015**年底苹果开源 Swift 之后，首个发布的主要版本（**Swift 3.0**），该版本实现了**Swift**进化过程中所讨论的并通过了 90 多个提议。这里我对**Swift 3**的新特性、新变化进行了一个总结。

[![译文: Swift - Swift3 新特性汇总（一系列版本的新变化）](https://www.hangge.com/blog_uploads/201609/2016092809503689523.png)](https://www.hangge.com/blog/cache/detail_1370.html#)



**一、彻底移除在 Swift 2.2 就已经弃用的特性**

这本在我们使用 **Xcode 7**的时候就已经有刺激提示，在**Swift 3**中已经将其彻底的移除出。

**1，弃用++与--操作符**

过去我们可以使用**++**与**——**来符来实现自增自减，避免操作。

```
var i = 0
i++
++i
i--
--i
```

可以使用复合加法智力（**+=**）与减法智力（**-=**），或者使用普通的加法智力（**+**）与减法操作（**-**）实现同样的功能。

```
//使用复合加法运算（+=）与减法运算（-=）
var i = 0
i += 1
i -= 1
 
//使用普通的加法运算（+）与减法运算（-）
i = i + 1
i = i - 1
```

**2，C**
钱**语言风格的循环**我们过去可能的生活方式是**为了**循环，现在也已放弃。

```
for var i=1; i<100; i++ {
    print("\(i)")
}
```

现在可以使用**for-in**循环，或者使用**for-each**加闭包的写法同样实现的功能。

```
//for-in循环
for i in 1...10 {
    print(i)
}
 
//for-each循环
(1...10).forEach {
    print($0)
}
```


**3，移除函数参数的变种标记**
在**夫特**函数中，参数默认是常量。过去可以在参数前加关键字**变种**将其定义为变量，这样函数内部就可以对该参数进行修改（外部的参数任然不会被修改）。

```
var age = 22
add(age)
 
func add(var age:Int) {
    age += 1
}
```

现在这种做法已经被废弃，**夫特3**不再允许开发者这样来将参数标记为变量了。
**
4，函数所有都参数必须带上标签**
过去如果一个函数有多个参数，调用的时候第一个参数不用带标签，而从第二个参数开始，必须要带标签。

```
let number = additive(8, b: 12)
 
func additive(a:Int, b:Int) -> Int{
    return a + b
}
```

现在是为了确保函数参数标签的标签，所有参数都必须带上。

```
let number = additive(a: 8, b: 12)
 
func additive(a:Int, b:Int) -> Int{
    return a + b
}
```

这个变化可能会导致我们的项目代码要进行可能的改变，所以苹果又提出了一个没有给第一个参数标签的解决方案。上一个下划线。




（不过这个只是可以让我们代码从**迅捷折变2**迁移到**迅捷3**的一个中途，可以的话还是建议将所有的参数都带上标签。）



```
let number = additive(8, b: 12)
 
func additive(_ a:Int, b:Int) -> Int{
    return a + b
}
```


**5，函数声明和函数调用都需要其他**
类型作为参数，对于一个参数是函数、返回值也是函数的函数。原来我们可能会这么写：

```
func g(a: Int -> Int) -> Int->Int { ... }
```

当这样非常难以阅读，很难看出参数在哪里结束，返回值又从哪里开始在。**夫特3**中变成这么定义这个函数：

```
func g(a:(Int) -> Int) -> (Int) -> Int { ... }
```


**6，选择器不再允许使用字符串**
假想我们给按钮添加一个点击事件响应，点击后执行**tapped**函数。以前可以这样写：

```
button.addTarget(responder, action: "tapped", forControlEvents: .TouchUpInside)
```

但是因为按钮的**选择器**写的是字符串。如果字符串拼写错了，那程序会在运行时因找不到相关方法而崩溃。所以**Swift 3**将写法转子，改成**#selecor()**。这样就将允许增加器提前检查方法拼写问题，而不是用再名的运行时才发现问题。

```
button.addTarget(self, action:#selector(tapped), for:.touchUpInside)
```


**
**

**二、Swift 3 的新特性**

**1，内联序列函数sequence**
**Swift 3**新增了两个函数：**sequence(first: next:)**和**sequence(state: next:)**。使用它们可以返回一个无限序列。下面是一个简单的使用示例，更详细的介绍可以我的另一篇文章：[Swift - 内联序列函数sequence介绍（附样例）](https://www.hangge.com/blog/cache/detail_1377.html)

```
// 从某一个树节点一直向上遍历到根节点
for node in sequence(first: leaf, next: { $0.parent }) {
    // node is leaf, then leaf.parent, then leaf.parent.parent, etc.
}
 
// 遍历出所有的2的n次方数（不考虑溢出）
for value in sequence(first: 1, next: { $0 * 2 }) {
    // value is 1, then 2, then 4, then 8, etc.
}
```



**2、key-path不再只能使用String**

这个是用在键值编码（**KVC**）与键值观察（**KVO**）上的，具体**KVC**、**KVO**相关内容可以参考我自己写的文章：[Swift - 反射（Reflection）的介绍与使用样例（附） KVC介绍）](https://www.hangge.com/blog/cache/detail_976.html)

我们还是可以继续使用**String**类型的**key-Path**：

```
//用户类
class User: NSObject{
    var name:String = ""  //姓名
    var age:Int = 0  //年龄
}
 
//创建一个User实例对象
let user1 = User()
user1.name = "hangge"
user1.age = 100
 
//使用KVC取值
let name = user1.value(forKey: "name")
print(name)
 
//使用KVC赋值
user1.setValue("hangge.com", forKey: "name")
```

但使用添加的**#keyPath()**写法，这样可以避免因为我们拼写错误而引发问题。

```
//使用KVC取值
let name = user1.value(forKeyPath: #keyPath(User.name))
print(name)
 
//使用KVC赋值
user1.setValue("hangge.com", forKeyPath: #keyPath(User.name))
```


**3，Foundation去掉NS之前的样子，比如**
过去我们使用**Foundation**相关类来对文件中的**JSON**数据进行解析，这么写：

```
let file = NSBundle.mainBundle().pathForResource("tutorials", ofType: "json")
let url = NSURL(fileURLWithPath: file!)
let data = NSData(contentsOfURL: url)
let json = try! NSJSONSerialization.JSONObjectWithData(data!, options: [])
print(json)
```

在**Swift 3**中，将移除**NS**前缀，就变成了：

```
let file = Bundle.main.path(forResource: "tutorials", ofType: "json")
let url = URL(fileURLWithPath: file!)
let data = try! Data(contentsOf: url)
let json = try! JSONSerialization.jsonObject(with: data)
print(json)
```


**4，除了M_PI还有.pi**
在过去，我们使用**M_PI**恒来表示**π**。所以根据半径求周长代码如下：

```
let r =  3.0
let circumference = 2 * M_PI * r
```

在**Swift 3**，**π**提供**Float**，**Double**与**CGFloat****三种**形式所以（**Float.pi**、**Double.pi**、**CGFloat.pi**），求周长还可以这么写：

```
let r = 3.0
let circumference = 2 * Double.pi * r
 
//我们还可以将前缀省略，让其通过类型自动推断
let r = 3.0
let circumference = 2 * .pi * r
```


**5，简化GCD的写法**
关于**GCD**，我自己写过一篇相关文章：[Swift - 多线程实现方式（3） - Grand Central Dispatch（GCD）](https://www.hangge.com/blog/cache/detail_745.html)

过去写法采用**C**语言的风格，早期可能不会适应。比如创建一个简单的异步线程：

```
let queue = dispatch_queue_create("Swift 2.2", nil)
dispatch_async(queue) {
    print("Swift 2.2 queue")
}
```

**Swift 3**取消了一种类型的写法，而采用了更详细的一种对象的：

```
let queue = DispatchQueue(label: "Swift 3")
queue.async {
    print("Swift 3 queue")
}
```


**如图6所示，核心图形的写法也更加面向对象化**
**核心图形**是一个相当强大的绘图框架，但是和**GCD**一样，它原来的**API**也是**Ç**语言风格的。
比如我们要创建一个**视图**，其内部背景使用**核心图形**进行绘画（蓝色的边界，蓝色的背景）。过去我们写：

```
class View: UIView {
    override func drawRect(rect: CGRect) {
        let context = UIGraphicsGetCurrentContext()
        let blue = UIColor.blueColor().CGColor
        CGContextSetFillColorWithColor(context, blue)
        let red = UIColor.redColor().CGColor
        CGContextSetStrokeColorWithColor(context, red)
        CGContextSetLineWidth(context, 10)
        CGContextAddRect(context, frame)
        CGContextDrawPath(context, .FillStroke)
    }
}
 
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
let aView = View(frame: frame)
```

在**Swift 3**中引用了写法，只要对当前画布背景解包，之后的所有剧情操作就都是基于解对象。

```
class View: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
         
        let blue = UIColor.blue.cgColor
        context.setFillColor(blue)
        let red = UIColor.red.cgColor
        context.setStrokeColor(red)
        context.setLineWidth(10)
        context.addRect(frame)
        context.drawPath(using: .fillStroke)
    }
}
 
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
let aView = View(frame: frame)
```


**7，新增的访问控制关键字：fileprivate、open**
在**Swift 3**中在原有的**3**个访问控制关键字**private**、**public**、**internal**外。又添加了2个新关键字**fileprivate**、**open**。它们可以看成是对原来的**私有**和**公共**的进一步浏览。具体使用方法和介绍可以我写的另一篇文章：[Swift - Swift - 三个新增的两个访问关键字介绍（fileprivate、open）](https://www.hangge.com/blog/cache/detail_1376.html)

**
**

**三、一些语法的修改**
**1，数组排序：sort()和sorted()**
过去的一组排序的两种方法：**sortInPlace()**和**sort()**，现在分别改名成**sort()**和**sorted()**
**sort()**是直接的对目标数组进行排序。**排序（）**是返回一个排序后的数组，原数组不变。

```
var array1 = [1, 5, 3, 2, 4]
array1.sort()
print(array1)  //[1, 2, 3, 4, 5]
 
var array2 = [1, 5, 3, 2, 4]
let sortedArray = array2.sorted()
print(array2)  //[1, 5, 3, 2, 4]
print(sortedArray)  //[1, 2, 3, 4, 5]
```


**2，反转（）与枚举（）**
过去**的反向（）**方法实现数组反转，**枚举（）**方法实现遍历。现这两个方法都加上**编**后缀（**反转**，**列举**）

```
for i in (1...10).reversed() {
    print(i)
}
 
let array = [1, 5, 3, 2, 4]
for (index, value) in array.enumerated() {
    print("\(index + 1) \(value)")
}
```


**3，CGRect、CGPoint、CGSize**
过去的**CGRectMake**、**CGPointMake**、**CGSizeMake**已废弃。现改用**CGRect**、**CGPoint**、**CGSize代替**。

```
//Swift 2
let frame = CGRectMake(0, 0, 20, 20)
let point = CGPointMake(0, 0)
let size = CGSizeMake(20, 20)
 
//Swift 3
let frame = CGRect(x: 0, y: 0, width: 20, height: 20)
let point = CGPoint(x: 0, y: 0)
let size = CGSize(width: 20, height: 20)
```


**4，移除了API中多余的单词**
**XCPlaygroundPage.currentPage**改为**PlaygroundPage.current**
**button.setTitle(forState)**改为**button.setTitle(for)**
**button.addTarget(action,** **forControlEvents** **)**改为**button.addTarget(action, for)**

**
****arr.minElement()**改为**arr.min()**
**arr.maxElement()**改为**arr.max()**
**attributedString.appendAttributedString(anotherString)**改为**attributedString.append(anotherString)**
**names.insert("Jane", atIndex: 0)**改为**names.insert("Jane", at: 0)**

**
****NSBundle.mainBundle()**改为**Bundle.main**
**UIDevice.currentDevice()**改为**UIDevice.current**
**NSData(contentsOfURL)**改为**Data(contentsOf)**
**NSJSONSerialization.JSONObjectWithData()**改为**JSONSerialization.jsonObject(with)**
**UIColor.blueColor()**改为**UIColor.blue**

**5，枚举成员变成了小写字母开头**
**Swift**将枚举成员当做属性看3，所以现在使用小写字母开头而不是以前的大写字母。

```
.system //过去是：.System
.touchUpInside //过去是：.TouchUpInside
.fillStroke //过去是：.FillStroke
.cgColor //过去是：.CGColor
```


**6，@discardableResult**
在**Swift 3**中，如果有一个方法有返回值。而调用的没有解决该方法的返回值，**Xcode**会报出警告的时候，你可能会出现潜在问题。

[![译文: Swift - Swift3 新特性汇总（一系列版本的新变化）](https://www.hangge.com/blog_uploads/201609/2016092716530979317.png)](https://www.hangge.com/blog/cache/detail_1370.html#)


还可以通过接收返回值**删除警告**。还可以通过给方法**声明@discardableResult**来达到**删除**目的。

```
import UIKit
 
class ViewController: UIViewController {
 
    override func viewDidLoad() {
        super.viewDidLoad()
         
        printMessage(message: "Hello Swift 3!")
    }
     
    @discardableResult
    func printMessage(message: String) -> String {
        let outputMessage = "Output : \(message)"
        return outputMessage
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1370.html