# Swift - Swift3新增的两个访问控制关键字介绍（fileprivate、open）

2016-09-30发布：hangge阅读：1504

原来 **Swift** 中有3种访问控制关键字（访问控制修饰符），分别为 **private**，**internal** 和 **public**。而在 **Swift 3**，又在原来的基础上新增了两种：**fileprivate**、**open**。它们可以看成是对 **private** 和 **public** 的进一步细分。本文介绍下这两新添加的关键字的作用以及与之前原有关键字的区别。

**1，fileprivate**

**fileprivate** 其实就是过去的 **private**。其修饰的属性或者方法只能在当前的 **Swift** 源文件里可以访问。即在同一个文件中，所有的 **fileprivate** 方法属性都是可以访问到的。

```
class A {
    fileprivate func test(){
        print("this is fileprivate func!")
    }
     
}
 
class B:A {
    func show(){
        test()
    }
}
```

而 **private** 现在变为了真正的私有访问控制。就是说不管在不在同一个文件中，用 **private** 修饰的方法也不可以被代码域之外的地方访问。

[![原文:Swift - Swift3新增的两个访问控制关键字介绍（fileprivate、open）](https://www.hangge.com/blog_uploads/201609/2016092910133060724.png)](https://www.hangge.com/blog/cache/detail_1376.html#)

**2，open**

**open** 其实就是过去的 **public**，过去 **public** 有两个作用：

（1）修饰的属性或者方法可以在其他作用域被访问 

（2）修饰的属性或者方法可以在其他作用域被继承或重载 **override**

但这样就会有问题，为了安全，我们可能希望某个类或属性能够被外部访问，但又不想其被继承或修改。如果将其标记成 **final** 后又会造成任何地方都不能被 **override**。
比如对 **lib** 设计者来说，他希望的结果是在 **module** 内可以被 **override**，而被 **import** 到外部后不能被 **override**。

现在新添加的 **open** 起的就是原来 **public** 的作用。而现在的 **public** 表示在其他 **module** 中不可以被 **override** 和继承，而在 **module** 内可以被 **override** 和继承。

**3，5种修饰符访问权限排序**

从高到低排序如下：

```
open > public > interal > fileprivate > private
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1376.html