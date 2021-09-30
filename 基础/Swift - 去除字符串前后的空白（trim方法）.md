# Swift - 去除字符串前后的空白（trim方法）

2017-04-14发布：hangge阅读：7596

大多数编程语言都提供了 **trim** 方法来除去字符串前后多余的空白，**Swift** 中也提供了类似的方法：**trimmingCharacters**，这个方法除了可以清除前端或后端多余的空白，还可以清除其他指定的字符。 

### 1，删除前后多余的空格

（1）样例代码

```swift
//原始字符串
let str1 = "   欢迎访问 hangge.com   "
//除去前后空格
let str2 = str1.trimmingCharacters(in: .whitespaces)
 
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```

（2）运行结果

可以看到字符串前面和后面的空格被删除了（内部的空格没有影响）。

[![原文:Swift - 去除字符串前后的空白（trim方法）](https://www.hangge.com/blog_uploads/201704/2017041015072871972.png)](https://www.hangge.com/blog/cache/detail_1649.html#)

**CharacterSet** **里各个枚举类型的含义如下：**

- **controlCharacters**：控制符
- **whitespaces**：空格
- **newlines**：换行符
- **whitespacesAndNewlines**：空格换行
- **decimalDigits**：小数
- **letters**：文字
- **lowercaseLetters**：小写字母
- **uppercaseLetters**：大写字母
- **nonBaseCharacters**：非基础
- **alphanumerics**：字母数字
- **decomposables**：可分解
- **illegalCharacters**：非法
- **punctuationCharacters**：标点
- **capitalizedLetters**：大写
- **symbols**：符号

### 2，删除前后指定的字符

（1）下面代码将 **String** 字符串前后的尖括号给去除掉

```swift
//原始字符串
let str1 = "<<hangge.com>>"
//删除前后<>
let characterSet = CharacterSet(charactersIn: "<>")
let str2 = str1.trimmingCharacters(in: characterSet)
 
//打印结果
print("原字符串：\(str1)")
print("新字符串：\(str2)")
```

（2）运行结果

[![原文:Swift - 去除字符串前后的空白（trim方法）](https://www.hangge.com/blog_uploads/201704/2017041409223143823.png)](https://www.hangge.com/blog/cache/detail_1649.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1649.html