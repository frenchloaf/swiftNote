# Swift - 时间轴效果的实现（附样例）

2016-11-04发布：hangge阅读：3840

时间轴展示也是一种比较常见的展现形式。一般用于展示以时间为主线的一连串事件，比如日常活动、企业发展历程、历史事件等等。时间轴可以运用于不同领域，最大的作用就是把过去的事物系统化、完整化、精确化。本文演示如何使用 **Swift** 实现一个垂直时间轴。

### 1，效果图

（1）这里做一个日常消费记录的时间轴列表。按日期从远到近纵向排列。

（2）时间线上每一个小图标代表一条消费记录，图标旁会显示具体的消费详情。

（3）消费详情包括：消费日期、金额、内容、以及备注信息（灰色区域）。其中备注信息不是必须的。

（4）由于备注信息不是必填项，且内容长度不固定。所以每个消费记录条目高度都是自适应的。

[![原文:Swift - 时间轴效果的实现（附样例）](https://www.hangge.com/blog_uploads/201610/2016103009232718118.png)](https://www.hangge.com/blog/cache/detail_1383.html#)

### 2，实现原理

（1）由于页面需要进行大量的约束设置以及修改操作，所以我这里使用了一个第三方的约束库 **SnapKit**。**SnapKit** 详细使用方法，可以参考我过去写的文章（[点此查看](https://www.hangge.com/blog/cache/detail_1097.html)）。

（2）整个时间轴展示页面使用的仍是 **tableView**，不过隐藏单元格分隔线，并设置行高自适应。

（3）通过自定义单元格来实现每条消费记录。其中单元格内时间线是使用背景色为灰色，宽度为1的 **UIView** 实现。

（4）时间轴上图标图片原来是矩形的，这里通过设置 **UIImageView** 圆角半径，将其显示成圆形。

### 3，样例代码

（1）自定义单元格（**TimeLineCell.swift**）

```swift
import UIKit
import SnapKit
 
class TimeLineCell: UITableViewCell {
    //时间轴线上的图标
    var timeLineIcon: UIImageView!
    //费用显示文本标签
    var costLabel: UILabel!
    //消费时间文本标签
    var dateTimeLabel: UILabel!
    //消费条目文本标签
    var titleLabel: UILabel!
    //备注标签容器
    var containView: UIView!
    //备注显示文本标签
    var appendixLabel: UILabel!
    //图标上半部分的时间线
    var forepartTimeLineLabel: UIView!
    //图标下半部分的时间线
    var backpartTimeLineLabel: UIView!
    //备注标签容器的高度约束（将高度设为0）
    var heightContraint: Constraint?
    //时间线离左右的横向间距
    let horizontalGap: CGFloat = 25
     
    //是否有备注
    var hasAppendix:Bool = false {
        didSet
        {
            if hasAppendix {
                //有备注则高度约束实效，备注容器高度更具内容自适应
                self.heightContraint?.deactivate()
            }else{
                //没有备注高度约束生效，高度变成0
                self.heightContraint?.activate()
            }
        }
    }
     
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
         
        setSubviews()
    }
     
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
     
    func setSubviews(){
        //图标上半部分的时间线初始化。并设置相关约束。
        forepartTimeLineLabel = UIView()
        forepartTimeLineLabel.backgroundColor = UIColor.lightGray
        contentView.addSubview(forepartTimeLineLabel)
        forepartTimeLineLabel.snp.makeConstraints { (make) -> Void in
            make.left.equalTo(contentView).offset(horizontalGap)
            make.height.equalTo(25)
            make.width.equalTo(1)
            make.top.equalTo(contentView)
        }
         
        //时间线上图标初始化。并设置相关约束。
        timeLineIcon = UIImageView()
        timeLineIcon.layer.masksToBounds = true
        //设置圆角半径，显示成圆形。
        timeLineIcon.layer.cornerRadius = 12
        contentView.addSubview(timeLineIcon)
        timeLineIcon.snp.makeConstraints { (make) -> Void in
            make.top.equalTo(forepartTimeLineLabel.snp.bottom)
            make.centerX.equalTo(forepartTimeLineLabel.snp.centerX)
            make.width.height.equalTo(25)
        }
         
        //费用显示文本标签初始化。并设置相关约束。
        costLabel = UILabel()
        costLabel.font = UIFont.systemFont(ofSize: 16)
        costLabel.sizeToFit()
        contentView.addSubview(costLabel)
        costLabel.snp.makeConstraints { (make) -> Void in
            make.top.equalTo(timeLineIcon)
            make.left.equalTo(forepartTimeLineLabel.snp.right).offset(horizontalGap)
        }
         
        //消费时间文本标签初始化。并设置相关约束。
        dateTimeLabel = UILabel()
        dateTimeLabel.font = UIFont.systemFont(ofSize: 12)
        dateTimeLabel.textColor = UIColor.lightGray
        dateTimeLabel.sizeToFit()
        contentView.addSubview(dateTimeLabel)
        dateTimeLabel.snp.makeConstraints { (make) -> Void in
            make.left.equalTo(costLabel.snp.right).offset(5)
            make.centerY.equalTo(costLabel)
        }
         
        //消费条目文本标签初始化。并设置相关约束。
        titleLabel = UILabel()
        titleLabel.font = UIFont.systemFont(ofSize: 16)
        titleLabel.sizeToFit()
        contentView.addSubview(titleLabel)
        titleLabel.snp.makeConstraints { (make) -> Void in
            make.left.equalTo(costLabel)
            make.top.equalTo(costLabel.snp.bottom).offset(10)
        }
         
        //备注便签容器初始化。并设置相关约束。
        containView = UIView()
        containView.backgroundColor = UIColor(red: 200/255, green: 200/255,
                                              blue: 200/255, alpha: 0.3)
        containView.layer.cornerRadius = 3
        contentView.addSubview(containView)
        containView.snp.makeConstraints { (make) -> Void in
            make.top.equalTo(titleLabel.snp.bottom).offset(10)
            make.left.equalTo(forepartTimeLineLabel.snp.right).offset(10)
            make.right.equalTo(contentView).offset(-10)
        }
         
        //备注文本标签初始化。并设置相关约束。
        appendixLabel = UILabel()
        appendixLabel.lineBreakMode = NSLineBreakMode.byWordWrapping
        appendixLabel.font = UIFont.systemFont(ofSize: 16)
        appendixLabel.numberOfLines = 4
        appendixLabel.sizeToFit()
        containView.addSubview(appendixLabel)
        appendixLabel.snp.makeConstraints { (make) -> Void in
            make.edges.equalTo(containView).inset(UIEdgeInsetsMake(10, 10, 10, 10))
        }
         
        containView.snp.makeConstraints { (make) -> Void in
            self.heightContraint = make.height.equalTo(0).constraint
            make.bottom.equalTo(contentView.snp.bottom).offset(-10)
        }
         
        //图标下半部分的时间线初始化。并设置相关约束。
        backpartTimeLineLabel = UIView()
        backpartTimeLineLabel.backgroundColor = UIColor.lightGray
        contentView.addSubview(backpartTimeLineLabel)
        backpartTimeLineLabel.snp.makeConstraints { (make) -> Void in
            make.left.width.equalTo(forepartTimeLineLabel)
            make.top.equalTo(timeLineIcon.snp.bottom)
            make.bottom.equalTo(contentView)
        }
    }
}
```

（2）主页面（**ViewController.swift**）

```swift
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    //所有消费记录
    var consumptions:[Consumption]?
     
    //使用时间轴形式的表格
    var tableView:UITableView?
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //初始化数据，这一次数据，我们放在属性列表文件里
        consumptions = [
            Consumption(title:"充了手机话费", cost:100.0, datetime:"2016.10.10 12:10",
                        appendix:""),
            Consumption(title:"超市购物", cost:810.0, datetime:"2016.10.11 12:10",
                        appendix:"买了台豆浆机，一袋大米，一桶油，两斤苹果，一包饼干，两只牙刷。"),
            Consumption(title:"同事结婚随礼", cost:500, datetime:"2016.10.11 17:10",
                        appendix:""),
            Consumption(title:"办健身卡", cost:1000, datetime:"2016.10.15 11:00",
                        appendix:"有效期至2016年10月")]
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:UITableViewStyle.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(TimeLineCell.self,
                                      forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //让单元格自适应
        tableView!.rowHeight = UITableViewAutomaticDimension
        tableView!.estimatedRowHeight = 100
        tableView!.separatorStyle = UITableViewCellSeparatorStyle.none
        tableView!.tableFooterView = UIView()
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1;
    }
 
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.consumptions!.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
        let identifier = "SwiftCell"
        var cell = tableView.dequeueReusableCell(withIdentifier: identifier)
            as? TimeLineCell
        if cell == nil{
            cell = TimeLineCell(style: .default, reuseIdentifier: identifier)
            cell?.selectionStyle = .none
        }
         
        //获取记录
        let consumption = self.consumptions![indexPath.row]
        //设置时间轴上的图标
        cell!.timeLineIcon.image = UIImage(named: "money")
        //设置消费金额
        cell!.costLabel.text = "\(consumption.cost) 元"
        //设置消费时间
        cell?.dateTimeLabel.text = consumption.datetime
        //设置消费内容
        cell?.titleLabel.text = consumption.title
        //设置备注信息
        cell?.appendixLabel.text = consumption.appendix
        //设置是否有备注（自动更新备注标签容器相关约束）
        cell?.hasAppendix = (cell?.appendixLabel.text != "")
        return cell!
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
 
//消费记录
struct Consumption {
    var title:String //消费条目
    var cost:Double //费用
    var datetime:String //时间
    var appendix:String //备注
}
```

**源码下载**：![img](https://www.hangge.com/blog/admin/include/edit/sysimage/icon16/zip.gif)[hangge_1383.zip](https://www.hangge.com/blog_uploads/201610/2016103009303513218.zip)


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1383.html