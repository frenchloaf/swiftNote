# Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）

2016-05-17发布：hangge阅读：4152

（本文代码已升级至Swift4）

**1，技术介绍**
（1）**AVFoundation.framework** 框架提供了 **AVCaptureSession** 类。使用它可以实现视频捕获功能。

（2）使用 **AVCaptureVideoPreviewLayer** 可以将摄像头拍摄的画面实时显示在 **ViewController** 上。
（3）对于捕获到的视频，我们使用 **AVCaptureMovieFileOutput** 将其输出到文件中。

**2，下面实现一个录像功能**

（1）点击“开始”按钮，开始录制视频。默认先保存到 **Documents** 目录下，命名为 **temp.mp4**。

（2）点击“停止”按钮，停止视频录制。将录制好的录像再转存到系统照片库中。

**3，效果图如下：**

 [![原文:Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）](https://www.hangge.com/blog_uploads/201605/2016051521394828989.png)](https://www.hangge.com/blog/cache/detail_1184.html#) [![原文:Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）](https://www.hangge.com/blog_uploads/201605/2016051521395455845.png)](https://www.hangge.com/blog/cache/detail_1184.html#) [![原文:Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）](https://www.hangge.com/blog_uploads/201605/2016051521400516302.png)](https://www.hangge.com/blog/cache/detail_1184.html#)


**4，Info.plist 配置** 

由于苹果安全策略更新，在使用 **Xcode8** 开发时，需要在 **Info.plist** 配置请求摄像头、麦克风、相册权限的描述字段： 

- **Privacy - Camera Usage Description**
- **Privacy - Microphone Usage Description**
- **Privacy - Photo Library Usage Description**

[![原文:Swift - 视频录制教程1（调用摄像头录像，并保存到系统相册）](https://www.hangge.com/blog_uploads/201801/2018010215422922710.png)](https://www.hangge.com/blog/cache/detail_1184.html#)



**5，样例代码**

```swift
import UIKit
import AVFoundation
import Photos
 
class ViewController: UIViewController , AVCaptureFileOutputRecordingDelegate {
     
    //视频捕获会话。它是input和output的桥梁。它协调着intput到output的数据传输
    let captureSession = AVCaptureSession()
    //视频输入设备
    let videoDevice = AVCaptureDevice.default(for: AVMediaType.video)
    //音频输入设备
    let audioDevice = AVCaptureDevice.default(for: AVMediaType.audio)
    //将捕获到的视频输出到文件
    let fileOutput = AVCaptureMovieFileOutput()
     
    //开始、停止按钮
    var startButton, stopButton : UIButton!
    //表示当时是否在录像中
    var isRecording = false
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //添加视频、音频输入设备
        let videoInput = try! AVCaptureDeviceInput(device: self.videoDevice!)
        self.captureSession.addInput(videoInput)
        let audioInput = try! AVCaptureDeviceInput(device: self.audioDevice!)
        self.captureSession.addInput(audioInput);
         
        //添加视频捕获输出
        self.captureSession.addOutput(self.fileOutput)
         
        //使用AVCaptureVideoPreviewLayer可以将摄像头的拍摄的实时画面显示在ViewController上
        let videoLayer = AVCaptureVideoPreviewLayer(session: self.captureSession)
        videoLayer.frame = self.view.bounds
        videoLayer.videoGravity = AVLayerVideoGravity.resizeAspectFill
        self.view.layer.addSublayer(videoLayer)
         
        //创建按钮
        self.setupButton()
        //启动session会话
        self.captureSession.startRunning()
    }
     
    //创建按钮
    func setupButton(){
        //创建开始按钮
        self.startButton = UIButton(frame: CGRect(x:0,y:0,width:120,height:50))
        self.startButton.backgroundColor = UIColor.red
        self.startButton.layer.masksToBounds = true
        self.startButton.setTitle("开始", for: .normal)
        self.startButton.layer.cornerRadius = 20.0
        self.startButton.layer.position = CGPoint(x:self.view.bounds.width/2 - 70,
                                                  y:self.view.bounds.height-50)
        self.startButton.addTarget(self, action: #selector(onClickStartButton(_:)),
                                   for: .touchUpInside)
         
        //创建停止按钮
        self.stopButton = UIButton(frame: CGRect(x:0,y:0,width:120,height:50))
        self.stopButton.backgroundColor = UIColor.gray
        self.stopButton.layer.masksToBounds = true
        self.stopButton.setTitle("停止", for: .normal)
        self.stopButton.layer.cornerRadius = 20.0
         
        self.stopButton.layer.position = CGPoint(x: self.view.bounds.width/2 + 70,
                                                 y:self.view.bounds.height-50)
        self.stopButton.addTarget(self, action: #selector(onClickStopButton(_:)),
                                  for: .touchUpInside)
         
        //添加按钮到视图上
        self.view.addSubview(self.startButton)
        self.view.addSubview(self.stopButton)
    }
     
    //开始按钮点击，开始录像
    @objc func onClickStartButton(_ sender: UIButton){
        if !self.isRecording {
            //设置录像的保存地址（在Documents目录下，名为temp.mp4）
            let paths = NSSearchPathForDirectoriesInDomains(.documentDirectory,
                                                            .userDomainMask, true)
            let documentsDirectory = paths[0] as String
            let filePath = "\(documentsDirectory)/temp.mp4"
            let fileURL = URL(fileURLWithPath: filePath)
            //启动视频编码输出
            fileOutput.startRecording(to: fileURL, recordingDelegate: self)
             
            //记录状态：录像中...
            self.isRecording = true
            //开始、结束按钮颜色改变
            self.changeButtonColor(target: self.startButton, color: .gray)
            self.changeButtonColor(target: self.stopButton, color: .red)
        }
    }
     
    //停止按钮点击，停止录像
    @objc func onClickStopButton(_ sender: UIButton){
        if self.isRecording {
            //停止视频编码输出
            fileOutput.stopRecording()
             
            //记录状态：录像结束
            self.isRecording = false
            //开始、结束按钮颜色改变
            self.changeButtonColor(target: self.startButton, color: .red)
            self.changeButtonColor(target: self.stopButton, color: .gray)
        }
    }
     
    //修改按钮的颜色
    func changeButtonColor(target: UIButton, color: UIColor){
        target.backgroundColor = color
    }
     
    //录像开始的代理方法
    func fileOutput(_ output: AVCaptureFileOutput,
                    didStartRecordingTo fileURL: URL,
                    from connections: [AVCaptureConnection]) {
    }
     
    //录像结束的代理方法
    func fileOutput(_ output: AVCaptureFileOutput,
                    didFinishRecordingTo outputFileURL: URL,
                    from connections: [AVCaptureConnection], error: Error?) {
        var message:String!
        //将录制好的录像保存到照片库中
        PHPhotoLibrary.shared().performChanges({
            PHAssetChangeRequest.creationRequestForAssetFromVideo(atFileURL: outputFileURL)
        }, completionHandler: { (isSuccess: Bool, error: Error?) in
            if isSuccess {
                message = "保存成功!"
            } else{
                message = "保存失败：\(error!.localizedDescription)"
            }
             
            DispatchQueue.main.async {
                //弹出提示框
                let alertController = UIAlertController(title: message, message: nil,
                                                        preferredStyle: .alert)
                let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
                alertController.addAction(cancelAction)
                self.present(alertController, animated: true, completion: nil)
            }
        })
    }
}
```


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1184.html