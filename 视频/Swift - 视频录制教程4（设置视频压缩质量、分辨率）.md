# Swift - 视频录制教程4（设置视频压缩质量、分辨率）

2016-05-30发布：hangge阅读：2580

**视频录制相关文章：**
[Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）](https://www.hangge.com/blog/cache/detail_1184.html)
[Swift - 视频录制教程2（小视频拍摄，将多段视频进行合并）](https://www.hangge.com/blog/cache/detail_1185.html)
[Swift - 视频录制教程3（设置拍摄窗口大小，录制正方形视频）](https://www.hangge.com/blog/cache/detail_1204.html)

在之前的小视频录制文章中，我们使用 **AVAssetExportSession** 将合并后的视频压缩输出成一个最终的视频文件。当时使用的是高品质的压缩（**AVAssetExportPresetHighestQuality**）。

```swift
let exporter = AVAssetExportSession(asset: composition,
                                    presetName:AVAssetExportPresetHighestQuality)!
```

当然除了**AVAssetExportPresetHighestQuality**，还有许多其它的设置视频分辨率（**Export preset**）供我们选择使用。比如为了方便传输，节约带宽，可以将视频转成低分辨率。

**1，固定分辨率预设属性**
（1）**AVAssetExportPreset640x480**：设置视频分辨率640x480
（2）**AVAssetExportPreset960x540**：设置视频分辨率960x540
（3）**AVAssetExportPreset1280x720**：设置视频分辨率1280x720
（4）**AVAssetExportPreset1920x1080**：设置视频分辨率1920x1080
（5）**AVAssetExportPreset3840x2160**：设置视频分辨率3840x2160

**2，相对质量预设属性**
（1）**AVAssetExportPresetLowQuality**：低质量
（2）**AVAssetExportPresetMediumQuality**：中等质量
（3）**AVAssetExportPresetHighestQuality**：高质量
这种设置方式，最终生成的视频分辨率与具体的拍摄设备有关。比如 **iPhone6** 拍摄的视频：

使用AVAssetExportPresetHighestQuality则视频分辨率是1920x1080（不压缩）。
AVAssetExportPresetMediumQuality视频分辨率是568x320 

AVAssetExportPresetLowQuality视频分辨率是224x128


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1190.html