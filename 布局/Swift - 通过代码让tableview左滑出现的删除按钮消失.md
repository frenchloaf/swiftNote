# Swift - 通过代码让tableview左滑出现的删除按钮消失

2017-02-10发布：hangge阅读：2571

我们知道通过实现 **editingStyleForRowAt** 这个协议方法可以使表格（**tableView**）具有滑动删除功能。即在任一单元上向左滑动，右侧便会出现删除按钮，点击则会触发相关的方法让我们执行相应的业务逻辑。

[![原文:Swift - 通过代码让tableview左滑出现的删除按钮消失](https://www.hangge.com/blog_uploads/201702/2017020814511883320.png)](https://www.hangge.com/blog/cache/detail_1546.html#)

这时如果点击表格上任意位置，那么删除按钮便会消失，表格重新还原到初始状态（有动画效果）。

之前有网友问除了通过点击表格让删除按钮消失，还有没有其它办法通过程序调用来实现，比如我们想点击右上角的完成按钮后将出现的删除按钮给隐藏。这里有两种实现办法。

###  1，调用表格的reloadData()方法

这种方式直接重新加载表格数据，当然滚动条位置不会变。不过删除按钮是直接消失的，没有动画效果。

```swift
self.tableView?.reloadData()
```

当然调用 **reloadRows** 方法单独重新加载出现删除按钮的单元格也是可以的。

```swift
self.tableView?.reloadRows(at: [indexPath], with: .none)
```

###  2，设置表格的isEditing属性

每次点击“完成”按钮时，我将表格的 **isEditing** 设置为 **false**。删除按钮便会消失，并且有动画效果。

```swift
self.tableView?.isEditing = false
```

当然调用 **setEditing** 方法也是可以的：

```swift
self.tableView?.setEditing(false, animated: true)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1546.html