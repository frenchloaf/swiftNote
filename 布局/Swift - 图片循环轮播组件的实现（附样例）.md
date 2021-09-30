

# Swift - 图片循环轮播组件的实现（附样例）

2016-09-26发布：hangge阅读：5907

（本文代码已升级至Swift4）

图片的无限循环轮播功能常常用在广告或者新闻展示上面，本文演示如何实现一个 **iOS** 系统下的图片轮播组件。

**1，组件功能介绍**

（1）每隔一段时间，轮播器就会自动滚动到下一张图片。如果当前是最后一张图片的话，则又滚动回第一张图片。这样无限循环下去。

（2）在组件下方位置有页控制器（小圆点），显示图片数量和当前的位置。

（3）除了自动轮播，我们还可以通过手动滑动组件来显示上一张，下一张图片。手动控制滚动的时候自动轮播功能停止，手动滚动结束后自动轮播功能又自动启用。

（4）点击图片我们可以获取到当前图片的位置索引，便于我们后续操作。这里直接将索引值显示出来。

（5）点击“刷新数据”按钮后，组件会自动重新加载数据源数据，并刷新显示。

**2，组件效果图**

   [![原文:Swift - 图片循环轮播组件的实现（附样例）](https://www.hangge.com/blog_uploads/201701/201701011931087435.png)](https://www.hangge.com/blog/cache/detail_1314.html#)   [![原文:Swift - 图片循环轮播组件的实现（附样例）](https://www.hangge.com/blog_uploads/201701/201701011931201157.png)](https://www.hangge.com/blog/cache/detail_1314.html#)

**3，组件实现原理**

（1）我们在一个 **scrollerView** 中横向放置三个并排的 **imageView**。管多少张图片都是交替使用这三个 **imageView** 来显示。

（2）每次图片滚动结束的时候，计算当前显示的图片索引值。并重置三个 **imageView** 里的图片和位置。

（3）由于图片可以通过网络加载，这里我用了一个第三方的图片库：**ImageHelper**。可以自动帮我们实现图片缓存，避免重复加载。（关于 **ImageHelper**，可以参考我之前写过的这篇文章：[Swift - 图片处理库ImageHelper详解](https://www.hangge.com/blog/cache/detail_975.html)）

（4）在图片未加载出来的时候，其位置上会先显示一张加载提示图片。这里我同样借助 **ImageHelper**，将一段提示文字转成 **Image**，用来作为加载完毕前的提示。

[![原文:Swift - 图片循环轮播组件的实现（附样例）](https://www.hangge.com/blog_uploads/201609/2016092521321523115.png)](https://www.hangge.com/blog/cache/detail_1314.html#)

**4，组件代码**

整个组件只有一个 **swift** 文件（**SliderGalleryController.swift**），里面包含一个组件实现类：**SliderGalleryController**。以及相关协议：**SliderGalleryControllerDelegate**。

```swift
import UIKit
 
//图片轮播组件代理协议
protocol SliderGalleryControllerDelegate{
    //获取数据源
    func galleryDataSource()->[String]
    //获取内部scrollerView的宽高尺寸
    func galleryScrollerViewSize()->CGSize
}
 
//图片轮播组件控制器
class SliderGalleryController: UIViewController,UIScrollViewDelegate{
    //代理对象
    var delegate : SliderGalleryControllerDelegate!
     
    //屏幕宽度
    let kScreenWidth = UIScreen.main.bounds.size.width
     
    //当前展示的图片索引
    var currentIndex : Int = 0
     
    //数据源
    var dataSource : [String]?
     
    //用于轮播的左中右三个image（不管几张图片都是这三个imageView交替使用）
    var leftImageView , middleImageView , rightImageView : UIImageView?
     
    //放置imageView的滚动视图
    var scrollerView : UIScrollView?
     
    //scrollView的宽和高
    var scrollerViewWidth : CGFloat?
    var scrollerViewHeight : CGFloat?
     
    //页控制器（小圆点）
    var pageControl : UIPageControl?
     
    //加载指示符（用来当iamgeView还没将图片显示出来时，显示的图片）
    var placeholderImage:UIImage!
     
    //自动滚动计时器
    var autoScrollTimer:Timer?
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //获取并设置scrollerView尺寸
        let size : CGSize = self.delegate.galleryScrollerViewSize()
        self.scrollerViewWidth = size.width
        self.scrollerViewHeight = size.height
         
        //获取数据
        self.dataSource =  self.delegate.galleryDataSource()
        //设置scrollerView
        self.configureScrollerView()
        //设置加载指示图片
        self.configurePlaceholder()
        //设置imageView
        self.configureImageView()
        //设置页控制器
        self.configurePageController()
        //设置自动滚动计时器
        self.configureAutoScrollTimer()
         
        self.view.backgroundColor = UIColor.black
    }
     
    //设置scrollerView
    func configureScrollerView(){
        self.scrollerView = UIScrollView(frame: CGRect(x: 0,y: 0,
                                    width: self.scrollerViewWidth!,
                                    height: self.scrollerViewHeight!))
        self.scrollerView?.backgroundColor = UIColor.red
        self.scrollerView?.delegate = self
        self.scrollerView?.contentSize = CGSize(width: self.scrollerViewWidth! * 3,
                                                height: self.scrollerViewHeight!)
        //滚动视图内容区域向左偏移一个view的宽度
        self.scrollerView?.contentOffset = CGPoint(x: self.scrollerViewWidth!, y: 0)
        self.scrollerView?.isPagingEnabled = true
        self.scrollerView?.bounces = false
        self.view.addSubview(self.scrollerView!)
         
    }
     
    //设置加载指示图片
    func configurePlaceholder(){
        //这里我使用ImageHelper将文字转换成图片，作为加载指示符
        let font = UIFont.systemFont(ofSize: 17.0, weight: UIFont.Weight.medium)
        let size = CGSize(width: self.scrollerViewWidth!, height: self.scrollerViewHeight!)
        placeholderImage = UIImage(text: "图片加载中...", font:font,
                                   color:UIColor.white, size:size)!
    }
     
    //设置imageView
    func configureImageView(){
        self.leftImageView = UIImageView(frame: CGRect(x: 0, y: 0,
                                                       width: self.scrollerViewWidth!,
                                                       height: self.scrollerViewHeight!))
        self.middleImageView = UIImageView(frame: CGRect(x: self.scrollerViewWidth!, y: 0,
                                                         width: self.scrollerViewWidth!,
                                                         height: self.scrollerViewHeight! ))
        self.rightImageView = UIImageView(frame: CGRect(x: 2*self.scrollerViewWidth!, y: 0,
                                                        width: self.scrollerViewWidth!,
                                                        height: self.scrollerViewHeight!))
        self.scrollerView?.showsHorizontalScrollIndicator = false
         
        //设置初始时左中右三个imageView的图片（分别时数据源中最后一张，第一张，第二张图片）
        if(self.dataSource?.count != 0){
            resetImageViewSource()
        }
         
        self.scrollerView?.addSubview(self.leftImageView!)
        self.scrollerView?.addSubview(self.middleImageView!)
        self.scrollerView?.addSubview(self.rightImageView!)
    }
     
    //设置页控制器
    func configurePageController() {
        self.pageControl = UIPageControl(frame: CGRect(x: kScreenWidth/2-60,
                        y: self.scrollerViewHeight! - 20, width: 120, height: 20))
        self.pageControl?.numberOfPages = (self.dataSource?.count)!
        self.pageControl?.isUserInteractionEnabled = false
        self.view.addSubview(self.pageControl!)
    }
     
    //设置自动滚动计时器
    func configureAutoScrollTimer() {
        //设置一个定时器，每三秒钟滚动一次
        autoScrollTimer = Timer.scheduledTimer(timeInterval: 3, target: self,
                selector: #selector(SliderGalleryController.letItScroll),
                userInfo: nil, repeats: true)
    }
     
    //计时器时间一到，滚动一张图片
    @objc func letItScroll(){
        let offset = CGPoint(x: 2*scrollerViewWidth!, y: 0)
        self.scrollerView?.setContentOffset(offset, animated: true)
    }
     
    //每当滚动后重新设置各个imageView的图片
    func resetImageViewSource() {
        //当前显示的是第一张图片
        if self.currentIndex == 0 {
            self.leftImageView?.imageFromURL(self.dataSource!.last!,
                                             placeholder: placeholderImage)
            self.middleImageView?.imageFromURL(self.dataSource!.first!,
                                               placeholder: placeholderImage)
            let rightImageIndex = (self.dataSource?.count)!>1 ? 1 : 0 //保护
            self.rightImageView?.imageFromURL(self.dataSource![rightImageIndex],
                                              placeholder: placeholderImage)
        }
            //当前显示的是最好一张图片
        else if self.currentIndex == (self.dataSource?.count)! - 1 {
            self.leftImageView?.imageFromURL(self.dataSource![self.currentIndex-1],
                                             placeholder: placeholderImage)
            self.middleImageView?.imageFromURL(self.dataSource!.last!,
                                               placeholder: placeholderImage)
            self.rightImageView?.imageFromURL(self.dataSource!.first!,
                                              placeholder: placeholderImage)
        }
            //其他情况
        else{
            self.leftImageView?.imageFromURL(self.dataSource![self.currentIndex-1],
                                             placeholder: placeholderImage)
            self.middleImageView?.imageFromURL(self.dataSource![self.currentIndex],
                                               placeholder: placeholderImage)
            self.rightImageView?.imageFromURL(self.dataSource![self.currentIndex+1],
                                              placeholder: placeholderImage)
        }
    }
     
    //scrollView滚动完毕后触发
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        //获取当前偏移量
        let offset = scrollView.contentOffset.x
         
        if(self.dataSource?.count != 0){
             
            //如果向左滑动（显示下一张）
            if(offset >= self.scrollerViewWidth!*2){
                //还原偏移量
                scrollView.contentOffset = CGPoint(x: self.scrollerViewWidth!, y: 0)
                //视图索引+1
                self.currentIndex = self.currentIndex + 1
                 
                if self.currentIndex == self.dataSource?.count {
                    self.currentIndex = 0
                }
            }
             
            //如果向右滑动（显示上一张）
            if(offset <= 0){
                //还原偏移量
                scrollView.contentOffset = CGPoint(x: self.scrollerViewWidth!, y: 0)
                //视图索引-1
                self.currentIndex = self.currentIndex - 1
                 
                if self.currentIndex == -1 {
                    self.currentIndex = (self.dataSource?.count)! - 1
                }
            }
             
            //重新设置各个imageView的图片
            resetImageViewSource()
            //设置页控制器当前页码
            self.pageControl?.currentPage = self.currentIndex
        }
    }
     
    //手动拖拽滚动开始
    func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {
        //使自动滚动计时器失效（防止用户手动移动图片的时候这边也在自动滚动）
        autoScrollTimer?.invalidate()
    }
     
    //手动拖拽滚动结束
    func scrollViewDidEndDragging(_ scrollView: UIScrollView,
                                  willDecelerate decelerate: Bool) {
        //重新启动自动滚动计时器
        configureAutoScrollTimer()
    }
     
    //重新加载数据
    func reloadData() {
        //索引重置
        self.currentIndex = 0
        //重新获取数据
        self.dataSource =  self.delegate.galleryDataSource()
        //页控制器更新
        self.pageControl?.numberOfPages = (self.dataSource?.count)!
        self.pageControl?.currentPage = 0
        //重新设置各个imageView的图片
        resetImageViewSource()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**5，使用样例**



```swift
import UIKit
 
class ViewController: UIViewController, SliderGalleryControllerDelegate {
     
    //获取屏幕宽度
    let screenWidth =  UIScreen.main.bounds.size.width
     
    //图片轮播组件
    var sliderGallery : SliderGalleryController!
     
    //图片集合
    var images = ["http://img4q.duitang.com/uploads/item/201503/18/20150318230437_Pxnk3.jpeg",
                  "http://img4.duitang.com/uploads/item/201501/31/20150131234424_WRJGa.jpeg",
                  "http://img5.duitang.com/uploads/item/201502/11/20150211095858_nmRV8.jpeg",
                  "http://cdnq.duitang.com/uploads/item/201506/11/20150611213132_HPecm.jpeg"]
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化图片轮播组件
        sliderGallery = SliderGalleryController()
        sliderGallery.delegate = self
        sliderGallery.view.frame = CGRect(x: 10, y: 40, width: screenWidth-20,
                                          height: (screenWidth-20)/4*3);
         
        //将图片轮播组件添加到当前视图
        self.addChildViewController(sliderGallery)
        self.view.addSubview(sliderGallery.view)
         
        //添加组件的点击事件
        let tap = UITapGestureRecognizer(target: self,
                                         action: #selector(ViewController.handleTapAction(_:)))
        sliderGallery.view.addGestureRecognizer(tap)
    }
     
    //图片轮播组件协议方法：获取内部scrollView尺寸
    func galleryScrollerViewSize() -> CGSize {
        return CGSize(width: screenWidth-20, height: (screenWidth-20)/4*3)
    }
     
    //图片轮播组件协议方法：获取数据集合
    func galleryDataSource() -> [String] {
        return images
    }
     
    //点击事件响应
    @objc func handleTapAction(_ tap:UITapGestureRecognizer)->Void{
        //获取图片索引值
        let index = sliderGallery.currentIndex
        //弹出索引信息
        let alertController = UIAlertController(title: "您点击的图片索引是：",
                                                message: "\(index)", preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "确定", style: .cancel, handler: nil)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }
     
    //“刷新数据”按钮点击
    @IBAction func reloadBtnTap(_ sender: AnyObject) {
        images = ["http://img3.duitang.com/uploads/item/201607/15/20160715160527_rSBfF.jpeg",
                  "http://d1.17xgame.com/d/wallpaper/2013/01/17/52549.jpg",
                  "http://p4.image.hiapk.com/uploads/allimg/140805/7730-140P5115S4.jpg"]
        sliderGallery.reloadData()
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1314.zip](https://www.hangge.com/blog_uploads/201802/2018020517310664691.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1314.html

