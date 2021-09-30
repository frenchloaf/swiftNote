# Swift - 将图片、文件转成Base64编码字符串（Base64加密、解密）

2017-06-14发布：hangge阅读：4736

有时上传或者发送图片、文件时，需要将 **Data** 数据转换为 **Base64** 编码的字符串。下面通过样例演示如何实现 **Base64** 的编码与解码（或者说是 **Base64** 的加密、解密）。

### 1，Base64编码

（1）这里我们将一个名为 **icon.png** 的图标转换成 **Base64 String**，并打印到控制台上。

```swift
//获取图片数据
let file = Bundle.main.path(forResource: "icon", ofType: "png")!
let fileUrl = URL(fileURLWithPath: file)
let fileData = try! Data(contentsOf: fileUrl)
 
//将图片转为base64编码
let base64 = fileData.base64EncodedString(options: .endLineWithLineFeed)
print(base64)
```

运行结果如下：

[![原文:Swift - 将图片、文件转成Base64编码字符串（Base64加密、解密）](https://www.hangge.com/blog_uploads/201706/201706121909094619.png)](https://www.hangge.com/blog/cache/detail_1711.html#)

（2）从上面的截图可以看到，转换后的字符串中有斜杠、加号这些特殊符号。如果我们还需要将这个字符串与其它参数拼接起来一起传输的话，可以在 **Base64** 编码后再进行一次 **URL** 字符串的编码（**urlEncoded**）

```swift
//获取图片数据
let file = Bundle.main.path(forResource: "icon", ofType: "png")!
let fileUrl = URL(fileURLWithPath: file)
let fileData = try! Data(contentsOf: fileUrl)
 
//将图片转为base64编码
let base64 = fileData.base64EncodedString(options: .endLineWithLineFeed)
            .addingPercentEncoding(withAllowedCharacters: .alphanumerics)
print(base64!)
```

可以看到除了数字英文外其它字符集都使用“**%**”代替了：

[![原文:Swift - 将图片、文件转成Base64编码字符串（Base64加密、解密）](https://www.hangge.com/blog_uploads/201706/2017061219181217308.png)](https://www.hangge.com/blog/cache/detail_1711.html#)

### 2，Base64解码

（1）下面我们将 **Base64** 字符串还原成图片，并显示在 **UIImageView** 上。

```swift
//Base64字符串
let base64 = "iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAQAAAAAYLlVAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAAmJLR0QA/4ePzL8AAAAHdElNRQffDAEWGTmOXnDsAAAAHGlET1QAAAACAAAAAAAAACAAAAAoAAAAIAAAACAAAAK51hVB4gAAAoVJREFUaN60l71OKlEQx38mhsTCoITYgEDhxwt4I6VSaCHPgVZIxwNgbNxQaqfJdr4AnVbKpSPaEChIWBMKrgmNAYx83EKJ7Nlz2A9gpjt7Zv7DnJn/DOBO1olzyjUP1Hinx5ABXf5R5YFrTomzzoIkwBFXPNPii5FCv2jxzBVHBOYLvk2WEh0lsKgdSmTZng/4FhfUHUNPap0LtmYDXyNN1RP4WKukWfMKv0+B/kzwI0b0KbDvHtxHireZwcf6RgqfG3g/Gl0bp5+0aWJg0KTNp83tLhp+p/Ab6AyVrj4oc0eGJHvsECPGDnskyXBHmQ+l3RCdDWfw90oXFTQSBFmSWi4RJIFGRRn+vX0IfnSF8SvnRBxlMMI5rwov+vSH8KFJo2+RY9NVEW+SoyXNojatHFPS0nvi0FMbH/IkLceUyiAuabwBOmHPRBZGZyBpyric9QqS4ZJndSYuXSUvGV4FGTumLaw3IM+KwvEyIaJEfjRKiGXFzRXyliz0SVtHTlVSsepfH6KIQeNHDYqEpmRBl8wIYUxdSkpv2ttHMUy3DaJTa8FajpfmeV+3NN6BTa83TPcbNhxxYGnK+uS+kLXEl7MlG3cBQM6Ckf1dtkrCpxfb1nMfQJgXAaU0XtyOhWVraK3ROQQAaYFlOxx/f9CEyCoOaNddEY7puSIgad8PUJQd24ibNvwV8af+JQBxoT4/SDhw5ZyIJiUh7Ast4nAmUGWZ4KL+WhCkLFD9GdwIablVrBvzkCVuBbQbeBSOMo5ceXsCyAhoj1ATVs2kI0feihBOhPW1Bu+mgzZ/HDny0oYAe7RNdu/QMx002XW497knIoBdmia7HgI7GcQWGkBMyNzwPwAAAP//K85OWAAAAoRJREFUtZa/TmJREMZ/JMbEwmgIoUGBApcXYCPlSqHF+hxoBXQ8ALs23lBqR3E7X8CttBPpiDYEChMuiSbChoYoBuVugQbPn3u5styZbs58Z75zzpyZgTfsT2oRx4tEaQu4NlFPuDiWgBvDs2C4J+krgSQPAm4IPcHQ57uvBFL0BVwPWoLhhX1fCfzkRcC14FIw2BR8JVCQol3CqWSqEPCNQICKFO0UDhkJpjoh3wiEqAuoEYeQ5lEwDsj4RiDDQEB1SUOQa+laDN8IGFKka4I6c4NNXwhs0tAfdY8nqTrlfSGQZyxgntibLASpScxu2Fg4gQ1upCi1yQMAFKUlm9LCCZSUGMXp4hZ30uIjOwsl8EP6azZ3bH12OFL4Xbk+Q0zqahYx1+u/UvY/El0SNBUXk1XHLSNUsWi/q0WViKPvKqayd5OE7JbjVXJ6o8yKw6ZLRIgRfdcYEZYcPFcoSzOHzSs51XGdc4XniLLLLXiRVcpSqbex+cO6zjlNR3F9w5z5Jd3e3lROb9Mh7QTISvPRRzruzBV+R5N6Ns9knSHLGFK1+viUJQ/lWSy7JeXjTaqswbIbcE2TsRO9Je9x6IiS59ZhF5O1WfAwZw7gMQ0MMoQcRpYAITIYNLS3aGNzRtjLCcKYjlvYDKhTocA+Kb4RJ06SFPsUqFCX+r1I3/QWfvIQhjYdxfG1zz0WFvf0pVFTl3rG7MsX0zGr+ZTzaoese+rpZZtzpTp+XV85Z3veQrJOTtMjvqJNcvqq510S/FKatTe947facuaTLYrUpMHNTZ+oURT7/f9LkF2OqdLVNJdp8+pS5Zjd6bC1aAmS5oATLmjxlyFjxgzp0eKCEw5IfzX0P7jvXGFgk1T+AAAAAElFTkSuQmCC"
 
//获取图片数据
let imageData = Data(base64Encoded: base64)
 
//显示图片
imageView.image = UIImage(data: imageData!)
```

运行结果如下：  

[![原文:Swift - 将图片、文件转成Base64编码字符串（Base64加密、解密）](https://www.hangge.com/blog_uploads/201706/2017061220154473788.jpg)](https://www.hangge.com/blog/cache/detail_1711.html#)

（2）如果 **Base64** 字符串之前有经过 **URL** 编码转换，记得要先进行 **URL** 解码（**urlDecoded**）

```swift
//Base64字符串（urlEncoded处理过）
let base64 = "iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAQAAAAAYLlVAAAABGdBTUEAALGPC%2FxhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAAmJLR0QA%2F4ePzL8AAAAHdElNRQffDAEWGTmOXnDsAAAAHGlET1QAAAACAAAAAAAAACAAAAAoAAAAIAAAACAAAAK51hVB4gAAAoVJREFUaN60l71OKlEQx38mhsTCoITYgEDhxwt4I6VSaCHPgVZIxwNgbNxQaqfJdr4AnVbKpSPaEChIWBMKrgmNAYx83EKJ7Nlz2A9gpjt7Zv7DnJn%2FDOBO1olzyjUP1Hinx5ABXf5R5YFrTomzzoIkwBFXPNPii5FCv2jxzBVHBOYLvk2WEh0lsKgdSmTZng%2F4FhfUHUNPap0LtmYDXyNN1RP4WKukWfMKv0%2BB%2FkzwI0b0KbDvHtxHireZwcf6RgqfG3g%2FGl0bp5%2B0aWJg0KTNp83tLhp%2Bp%2FAb6AyVrj4oc0eGJHvsECPGDnskyXBHmQ%2Bl3RCdDWfw90oXFTQSBFmSWi4RJIFGRRn%2BvX0IfnSF8SvnRBxlMMI5rwov%2BvSH8KFJo2%2BRY9NVEW%2BSoyXNojatHFPS0nvi0FMbH%2FIkLceUyiAuabwBOmHPRBZGZyBpyric9QqS4ZJndSYuXSUvGV4FGTumLaw3IM%2BKwvEyIaJEfjRKiGXFzRXyliz0SVtHTlVSsepfH6KIQeNHDYqEpmRBl8wIYUxdSkpv2ttHMUy3DaJTa8FajpfmeV%2B3NN6BTa83TPcbNhxxYGnK%2BuS%2BkLXEl7MlG3cBQM6Ckf1dtkrCpxfb1nMfQJgXAaU0XtyOhWVraK3ROQQAaYFlOxx%2Ff9CEyCoOaNddEY7puSIgad8PUJQd24ibNvwV8af%2BJQBxoT4%2FSDhw5ZyIJiUh7Ast4nAmUGWZ4KL%2BWhCkLFD9GdwIablVrBvzkCVuBbQbeBSOMo5ceXsCyAhoj1ATVs2kI0feihBOhPW1Bu%2BmgzZ%2FHDny0oYAe7RNdu%2FQMx002XW497knIoBdmia7HgI7GcQWGkBMyNzwPwAAAP%2F%2FK85OWAAAAoRJREFUtZa%2FTmJREMZ%2FJMbEwmgIoUGBApcXYCPlSqHF%2BhxoBXQ8ALs23lBqR3E7X8CttBPpiDYEChMuiSbChoYoBuVugQbPn3u5styZbs58Z75zzpyZgTfsT2oRx4tEaQu4NlFPuDiWgBvDs2C4J%2BkrgSQPAm4IPcHQ57uvBFL0BVwPWoLhhX1fCfzkRcC14FIw2BR8JVCQol3CqWSqEPCNQICKFO0UDhkJpjoh3wiEqAuoEYeQ5lEwDsj4RiDDQEB1SUOQa%2BlaDN8IGFKka4I6c4NNXwhs0tAfdY8nqTrlfSGQZyxgntibLASpScxu2Fg4gQ1upCi1yQMAFKUlm9LCCZSUGMXp4hZ30uIjOwsl8EP6azZ3bH12OFL4Xbk%2BQ0zqahYx1%2Bu%2FUvY%2FEl0SNBUXk1XHLSNUsWi%2Fq0WViKPvKqayd5OE7JbjVXJ6o8yKw6ZLRIgRfdcYEZYcPFcoSzOHzSs51XGdc4XniLLLLXiRVcpSqbex%2BcO6zjlNR3F9w5z5Jd3e3lROb9Mh7QTISvPRRzruzBV%2BR5N6Ns9knSHLGFK1%2BviUJQ%2FlWSy7JeXjTaqswbIbcE2TsRO9Je9x6IiS59ZhF5O1WfAwZw7gMQ0MMoQcRpYAITIYNLS3aGNzRtjLCcKYjlvYDKhTocA%2BKb4RJ06SFPsUqFCX%2Br1I3%2FQWfvIQhjYdxfG1zz0WFvf0pVFTl3rG7MsX0zGr%2BZTzaoese%2BrpZZtzpTp%2BXV85Z3veQrJOTtMjvqJNcvqq510S%2FFKatTe947facuaTLYrUpMHNTZ%2BoURT7%2Ff9LkF2OqdLVNJdp8%2BpS5Zjd6bC1aAmS5oATLmjxlyFjxgzp0eKCEw5IfzX0P7jvXGFgk1T%2BAAAAAElFTkSuQmCC"
 
 
//获取图片数据
let imageData = Data(base64Encoded: base64.removingPercentEncoding!)
 
//显示图片
imageView.image = UIImage(data: imageData!)
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1711.html