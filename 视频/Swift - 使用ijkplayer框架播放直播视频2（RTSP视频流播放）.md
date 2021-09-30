# Swift - 使用ijkplayer框架播放直播视频2（RTSP视频流播放）

2016-08-18发布：hangge阅读：3841

在前一篇文章中：[Swift - 使用ijkplayer框架播放直播视频1（RTMP/HTTP视频流播放）](https://www.hangge.com/blog/cache/detail_1326.html)。我介绍了 **ijkplayer** 框架的配置和使用。当时使用的是 **ijkplayer** 默认的编译配置，也就是精简配置。这种编译出来的包比较小，也支持大多数的视频格式。比如前文的 **RTMP** 或 **HTTP** 协议的直播视频都是可以播放的。
但有时我们需要支持更多的视频类型（比如做 **RTSP** 协议的视频直播），那么就需要修改默认的编译配置。

**1，将Ijkplayer项目下载到本地，这个就不多说了**

```bash
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
```


**2，在集成ffmpeg之前，需要先修改默认的module.sh**

原来使用的是 **module-lite.sh**，这里改成 **module-default.sh**

```bash
cd ijkplayer-ios
cd config
rm module.sh
ln -s module-default.sh module.sh
cd ..
cd ios
sh compile-ffmpeg.sh clean
```


**3，接下来的集成ffmpeg同之前的一样**

```bash
cd ijkplayer-ios
git checkout -B latest k0.6.0
 
./init-ios.sh
 
cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```


**4，framework编译、项目配置这些也是和前文一样。这里就不多说了**

**5，这里使用前文的样例做测试** 

把地址改成 **RTSP** 视频地址（rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp）

可以看到视频播放成功：

[![原文:Swift - 使用ijkplayer框架播放直播视频2（RTSP视频流播放）](https://www.hangge.com/blog_uploads/201608/2016081215200249503.png)](https://www.hangge.com/blog/cache/detail_1327.html#)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1327.html