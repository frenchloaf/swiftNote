# Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）

2016-08-15发布：hangge阅读：9395

**BiliBili**（简称B站），想必大家都不陌生。**Ijkplayer** 框架是B站提供了一个开源的流媒体解决方案，集成了 **ffmpeg**，还支持硬解码(需 **iOS8** 以上版本)。使用 **Ijkplayer** 框架我们可以很方便地实现视频直播功能（**Http**/**RTMP**/**RTSP** 这几种直播源都支持）。

本文主要介绍如何使用 **Ijkplayer** 框架播放在线直播视频（当然其本地播放能力也很强大）。对于几种直播流不太清楚地，可以参考我前面写的这篇文章：[RTMP、RTSP、HTTP视频协议详解（附：直播流地址、播放软件）](https://www.hangge.com/blog/cache/detail_1325.html)

**一，环境部署**

在使用 **Ijkplayer** 前，我们需要先搭建运行环境。

**1，在“终端”中运行如下命令，安装homebrew, git, yasm等环境。**

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install git
brew install yasm
```

**2，将Ijkplayer项目下载到本地**
**GitHub**地址：https://github.com/Bilibili/ijkplayer
或者在“终端”中使用 **git** 命令下载到本地：

```bash
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
```


**3，将ffmpeg集成到ijkplayer中**
运行如下命令，进入项目文件夹并集成 **ffmpeg**。

```bash
cd ijkplayer-ios
git checkout -B latest k0.6.0
 
./init-ios.sh
 
cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```


**二、运行样例**
**Ijkplayer** 工程目录中自带了一个 **demo** 样例，我们可以先运行下看看效果。

**1，打开ios文件夹下的IJKMediaDemo工程**

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081117283895816.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

**2，运行后界面如下**

可以选择播放本地文件，或者输入视频地址来播放。同时 **demo** 中还自带了一些视频源，可以直接点击播放。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081210384997901.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

**3，在Input URL页面中输入视频地址即可播放**

不过程序默认做了限制，只能播放 **http**/**https** 视频流。

我们这里可以修改下，放开限制。让其也能支持其它协议视频播放（比如 **RTMP** 协议）。

将 **IJKDemoInputURLViewController.m** 中的相关判断给注释掉即可。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081210435476945.png)](https://www.hangge.com/blog/cache/detail_1326.html#)



```swift
//if ([scheme isEqualToString:@"http"] || [scheme isEqualToString:@"https"]) {
    [IJKVideoViewController presentFromViewController:self withTitle:[NSString stringWithFormat:@"URL: %@", url] URL:url completion:^{
//            [self.navigationController popViewControllerAnimated:NO];
    }];
//}
```

**三、将Ijkplayer集成到我们的项目中来**

下面我们通过一个样例演示在我们项目中如何通过 **ijkplayer** 来播放视频流。官方提供的 **demo** 用的是 **OC**，这里我使用 **Swift** 来实现。

**1，制作framework**

首先我们要将 **IJKMediaPlayer** 编译成 **framework**，这样不仅体积小，而且使用起来也更加方便。

（1）打开 **IJKMediaPlayer.xcodeproj**（**ios**/**IJKMediaPlayer** 文件夹下）

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213455324170.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

（2）点击 **IJKMediaFramework** 出现选择框，选择 **edit scheme**

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213473781299.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

（3）将 **build configuration** 改为 **Release**

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213495029381.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

（4）分别在模拟器和真机(**Generic iOS Device** 也可以)上编译

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213513624889.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213514438049.png)](https://www.hangge.com/blog/cache/detail_1326.html#)



（5）我们打开生成 **framework** 的目录

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213541149023.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

（6）可以看到这里生成了两个 **framework**。看名字就知道一个是给模拟器使用，一个是给真机使用。这个根据个人的运行需求选择对应的 **framework**。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081213554199381.png)](https://www.hangge.com/blog/cache/detail_1326.html#)


（7）当然如果在开发阶段，真机模拟器都需要调试这样切来切去就太麻烦了。我们将两个 **framework** 合并，这样就可以同时在真机和模拟器上调试了。

我们进入 **Products** 目录，输入如下 **lipo** 命令将二者合并输出：

```
lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
```


（8）接着将合并后的 **framework** 拷贝到 **Release-iphoneos/IJKMediaFramework.framework** 中



```
cp IJKMediaFramework Release-iphoneos/IJKMediaFramework.framework/
```




（9）我们可以通过 **lipo -info IJKMediaFramework** 命令查看 **framework** 支持情况。可看到现在这个真机、模拟器都是支持的。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081214073244678.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

**2，项目配置**

（1）我们将前面制作好的 **framework**（**Release-iphoneos/IJKMediaFramework.framework**）添加到我们项目中来。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081214132973365.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

（2）接着添加一些依赖的动态库。

AudioToolbox.framework
AVFoundation.framework
CoreGraphics.framework
CoreMedia.framework
CoreVideo.framework
libbz2.tbd
libz.tbd
MediaPlayer.framework
MobileCoreServices.framework
OpenGLES.framework
QuartzCore.framework
UIKit.framework
VideoToolbox.framework

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081214195443318.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

**四、测试样例**

项目配置完毕后，接下来写一个简单的样例演示如何使用 **ijkplayer**。

**1，样例效果图**

程序启动后会自动播放一个采用 **RTMP** 协议的直播，并全屏显示。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081214401968319.png)](https://www.hangge.com/blog/cache/detail_1326.html#)



同时播放视图会自适应手机横、竖两种状态。

[![原文:Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081214403018188.png)](https://www.hangge.com/blog/cache/detail_1326.html#)

**2，样例代码**

```swift
import UIKit
import IJKMediaFramework
 
class ViewController: UIViewController {
     
    var player:IJKFFMoviePlayerController!
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        let options = IJKFFOptions.optionsByDefault()
         
        //视频源地址
        let url = NSURL(string: "rtmp://live.hkstv.hk.lxdns.com/live/hks")
        //let url = NSURL(string: "http://live.hkstv.hk.lxdns.com/live/hks/playlist.m3u8")
 
        //初始化播放器，播放在线视频或直播（RTMP）
        let player = IJKFFMoviePlayerController(contentURL: url, withOptions: options)
        //播放页面视图宽高自适应
        let autoresize = UIViewAutoresizing.FlexibleWidth.rawValue |
            UIViewAutoresizing.FlexibleHeight.rawValue
        player.view.autoresizingMask = UIViewAutoresizing(rawValue: autoresize)
         
        player.view.frame = self.view.bounds
        player.scalingMode = .AspectFit //缩放模式
        player.shouldAutoplay = true //开启自动播放
         
        self.view.autoresizesSubviews = true
        self.view.addSubview(player.view)
        self.player = player
    }
     
    override func viewWillAppear(animated: Bool) {
        //开始播放
        self.player.prepareToPlay()
    }
     
    override func viewWillDisappear(animated: Bool) {
        //关闭播放器
        self.player.shutdown()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1326.html