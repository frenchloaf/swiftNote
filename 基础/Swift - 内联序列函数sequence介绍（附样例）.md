# Swift - 内联序列函数sequence介绍（附样例）

2016-10-08发布：hangge阅读：1799

**Swift 3** 新增了两个全局函数：**sequence(first: next:)** 和 **sequence(state: next:)**。使用它们可以返回一个无限序列。我们可以给他们一个初始值，或者初始状态，然后他们便会以懒加载的方式应用到一个闭包。下面通过样例分别演示这两个函数如何使用。

**1，sequence(first: next:)介绍**

```
//函数声明
public func sequence<t>(first: T, next: @escaping (T) -> T?) -> UnfoldSequence<T, (T?, Bool)>
```

其作用是通过设置的初始值 **first**，循环得到 **next(first)**、**next(next(first))**、**next(next(next(first)))**......

假设我们打印出从1到100范围内所有的2的n次方数，过去我们使用 **repeat...while** 可以这么写：

```
var i = 1
repeat {
    print(i)
    i = i * 2
} while i <= 100
```

输出结果：

[![原文:Swift - 内联序列函数sequence介绍（附样例）](https://www.hangge.com/blog_uploads/201609/2016092914055857706.png)](https://www.hangge.com/blog/cache/detail_1377.html#)



可以看到 **repeat...while** 循环依赖我外面声明的一个变量 **i**。

（1）而改用 **sequence(first: next:)** 实现如下，可以看到省去了外部变量的定义。

```swift
for i in sequence(first: 1, next: { $0 * 2 }) {
    if i > 100 { break }
    print(i)
}
```


（2）我们还可以将所有的处理逻辑，状态判断都放在 **next** 闭包里。所以上面样例又可以这样写。

```swift
for _ in sequence(first: 1, next: {
    print($0)
    let value = $0 * 2
    return value <= 100 ? value : nil  //next中返回nil表示sequence结束
}) {}
```


（3）从某一个树节点一直向上遍历到根节点

```swift
for node in sequence(first: leaf, next: { $0.parent }) {
    // node is leaf, then leaf.parent, then leaf.parent.parent, etc.
}
```

**2，sequence(state: next:)**

```swift
//函数声明
public func sequence<T, State>(state: State, next: @escaping (inout State) -> T?)
    -> UnfoldSequence<T, State>
```

这个方法是设置个初始状态（可变的），后面将其传 入**next** 闭包改变状态，并获取下一个序列，依次类推。



（1）下面样例通过指定最大的x轴，y轴坐标值，列出其内部所有的整点。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    //命名一个坐标点类型
    typealias PointType = (x: Int, y: Int)
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        //遍历序列并打印出所有坐标
        for point in cartesianSequence(xCount: 5, yCount: 3) {
            print("(x: \(point.x), y: \(point.y))")
        }
    }
     
    //返回所有符合条件的坐标点序列
    func cartesianSequence(xCount: Int, yCount: Int) -> UnfoldSequence<PointType, Int> {
        assert(xCount > 0  && yCount > 0,
               "必须使用正整数创建序列。")
        return sequence(state: 0, next: {
            (index: inout Int) -> PointType? in
            guard index < xCount * yCount else { return nil }
            defer { index += 1 }
            return (x: index % xCount, y: index / xCount)
        })
    }
 
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

运行结果如下：

[![原文:Swift - 内联序列函数sequence介绍（附样例）](https://www.hangge.com/blog_uploads/201610/2016100120581880042.png)](https://www.hangge.com/blog/cache/detail_1377.html#)



（2）假设我们有两个书架，一个里面是中文书、另一个里面是外文书。现在要返回一个交叉序列，即一本中文书、一本外文书这样交替显示。直到某个书柜的书显示完毕即结束。

```swift
import UIKit
 
class ViewController: UIViewController {
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //中文书架
        let bookShelf1 = BookShelf()
        bookShelf1.append(book: Book(name: "平凡的世界"))
        bookShelf1.append(book: Book(name: "活着"))
        bookShelf1.append(book: Book(name: "围城"))
        bookShelf1.append(book: Book(name: "三国演义"))
         
        //外文书架
        let bookShelf2 = BookShelf()
        bookShelf2.append(book: Book(name: "The Kite Runner"))
        bookShelf2.append(book: Book(name: "Cien anos de soledad"))
        bookShelf2.append(book: Book(name: "Harry Potter"))
         
        //创建两个书架图书的交替序列
        for book in sequence(state: (false, bookShelf1.makeIterator(), bookShelf2.makeIterator()),
                    next: { (state:inout (Bool,AnyIterator<Book>,AnyIterator<Book>)) -> Book? in
            state.0 = !state.0
            return state.0 ? state.1.next() : state.2.next()
        }) {
            print(book.name)
        }
         
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
// 图书
struct Book {
    var name: String
}
 
// 书架
class BookShelf {
    //图书集合
    private var books: [Book] = []
     
    //添加新书
    func append(book: Book) {
        self.books.append(book)
    }
     
    //创建Iterator
    func makeIterator() -> AnyIterator<Book> {
        var index : Int = 0
        return AnyIterator<Book> {
            defer {
                index = index + 1
            }
            return index < self.books.count ? self.books[index] : nil
        }
    }
}
```

运行结果如下：

[![原文:Swift - 内联序列函数sequence介绍（附样例）](https://www.hangge.com/blog_uploads/201610/2016100211192367171.png)](https://www.hangge.com/blog/cache/detail_1377.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1377.html