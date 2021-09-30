# Swift - 调整UIScrollView的滑动速度（适用于tableView、collectionView）

2016-11-25发布：hangge阅读：6894

当我们使用手指滑动滚动视图时（**UIScrollView**、**UITableView**、**UICollectionView**），抬起手指后，会发现视图还会保持惯性继续滚动一段距离。然后逐渐减速停下。

[![原文:Swift - 调整UIScrollView的滑动速度（适用于tableView、collectionView）](https://www.hangge.com/blog_uploads/201611/2016112508595065785.png)](https://www.hangge.com/blog/cache/detail_1432.html#)


如果觉得快速滑动过程中，滚动速度过快，一滑就溜出去很远。我们可以通过修改 **decelerationRate** 属性，来控制减速的速度。有两种方式来设置 **decelerationRate** 属性。



### 1，使用系统定义的常量值

- **UIScrollViewDecelerationRateNormal**：正常减速（默认值）
- **UIScrollViewDecelerationRateFast**：快速减速

默认情况下 **scrollView** 使用的是 **UIScrollViewDecelerationRateNormal**，我们将其改成 **UIScrollViewDecelerationRateFast** 会发现视图滚动的速度明显降下来了。

```swift
self.tableView!.decelerationRate = UIScrollViewDecelerationRateFast
```



### 2，设置自定义值

**decelerationRate** 类型为 **CGFloat**，其范围是**（0.0，1.0）**。上面两个常量对应的值分别是：

- **UIScrollViewDecelerationRateNormal**：0.998（默认值）
- **UIScrollViewDecelerationRateFast**：0.99

如果系统这两个常量还不能满足我们的需求话，我们也可以设置范围的任意值。比如将其设置成 **0.1**，会发现滑动后就会很快地停下来。

```swift
self.tableView!.decelerationRate = 0.1
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1432.html