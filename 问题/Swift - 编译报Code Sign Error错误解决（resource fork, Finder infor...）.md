# Swift - 编译报Code Sign Error错误解决（resource fork, Finder infor...）

2017-05-01发布：hangge阅读：698

### 1，问题描述

今天进项目文件夹修改了几张 **LaunchImage** 图片，接着编译就不成功，一直报 **Code Sign Error** 错误。（只有虚拟机编译运行会这样，真机调试没问题。）
**
**

**具体的报错信息如下**：

CodeSign /Users/hangge/Library/Developer/Xcode/DerivedData/信息移动作业平台-casodjeqrpelytgibjbhvzdxrnnv/Build/Products/Debug-iphonesimulator/信息移动作业平台.app
cd /Users/hangge/Documents/ntyd/platforms/ios
export CODESIGN_ALLOCATE=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/codesign_allocate
export PATH="/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/usr/bin:/Applications/Xcode.app/Contents/Developer/usr/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
Signing Identity: "-"
/usr/bin/codesign --force --sign - --timestamp=none /Users/hangge/Library/Developer/Xcode/DerivedData/信息移动作业平台-casodjeqrpelytgibjbhvzdxrnnv/Build/Products/Debug-iphonesimulator/信息移动作业平台.app
/Users/hangge/Library/Developer/Xcode/DerivedData/信息移动作业平台-casodjeqrpelytgibjbhvzdxrnnv/Build/Products/Debug-iphonesimulator/信息移动作业平台.app: resource fork, Finder information, or similar detritus not allowed
Command /usr/bin/codesign failed with exit code 1

[![原文:Swift - 编译报Code Sign Error错误解决（resource fork, Finder infor...）](https://www.hangge.com/blog_uploads/201703/2017030910101234385.png)](https://www.hangge.com/blog/cache/detail_1587.html#)



### 2，解决办法

（1）使用终端进入到项目文件夹。 

（2）执行如下命令：

```swift
find . -type f -name '*.jpeg' -exec xattr -c {} \;
find . -type f -name '*.png' -exec xattr -c {} \;
find . -type f -name '*.tif' -exec xattr -c {} \;
```

（3）重新清理并编译程序即可。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1587.html