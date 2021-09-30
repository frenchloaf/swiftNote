# Swift - The certificate used to sign "XXX" has either expired...错误解决

2016-12-09发布：hangge阅读：739

### 1，问题描述   

最近将程序发布到真机上进行调试，点击运行后无法启动程序，报如下错误。

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201611/2016112010580243409.png)](https://www.hangge.com/blog/cache/detail_1448.html#)

**Unable to install "hangge_1221"**
The certificate used to sign "hangge_1221" has either expired or has been revoked. An Updated certificate is required to sign and install the application.



### 2，问题解决

将本机相关证书更新下就可以了。
（1）点击 **Xcode** 菜单中的“**Preferences...**”

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201611/2016112011085494662.png)](https://www.hangge.com/blog/cache/detail_1448.html#)



（2）在弹出的窗口中选择“**Accounts**”->“**Team**”->“**View Details...**”

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201611/2016112011121261446.png)](https://www.hangge.com/blog/cache/detail_1448.html#)



（3）右击“**Provisiong Profiles**”列表中的任意一个 **profile** 文件，选择“**Show in Finder**” 

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201612/2016120909063694807.png)](https://www.hangge.com/blog/cache/detail_1448.html#)



（4）这时可看到本地所有的 **profile** 文件。我们可以只把需要重新下载的证书删除，或者干脆把所有的都给删除。

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201611/2016112011184231167.png)](https://www.hangge.com/blog/cache/detail_1448.html#)



（5）回到之前页面，点击“**Download All Profiles**”下载所有的 **Provisiong Profiles**。

[![原文:Swift - The certificate used to sign "XXX" has either expired...错误解决](https://www.hangge.com/blog_uploads/201611/2016112011231953476.png)](https://www.hangge.com/blog/cache/detail_1448.html#)



（6）下载完毕后，再次运行程序，发现现在可以成功启动程序了。


原文出自：[www.hangge.com](https://www.hangge.com/) 转载请保留原文链接：https://www.hangge.com/blog/cache/detail_1448.html