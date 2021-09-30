# Swift - 将Data数据转换为[UInt8]（bytes字节数组）

2017-06-21发布：hangge阅读：13408

有时上传或者发送图片、文字时，需要将数据转换为 **bytes** 字节数组。下面介绍两种将 **Data** 转换为 **[UInt8]** 的方法。 

假设我们有如下 **Data** 数据要转换：

```swift
let data = "航歌".data(using: .utf8)!
```

### 方法一：使用 [UInt8] 新的构造函数

```swift
let bytes = [UInt8](data)
print(bytes)
```

[![原文:Swift - 将Data数据转换为[UInt8\]（bytes字节数组）](https://www.hangge.com/blog_uploads/201705/2017052911283282695.png)](https://www.hangge.com/blog/cache/detail_1698.html#)

### 方法二：通过 Pointer 指针获取

```swift
let bytes = data.withUnsafeBytes {
    [UInt8](UnsafeBufferPointer(start: $0, count: data.count))
}
print(bytes)
```