
# 说明

- 本项目完全基于 [wxappUnpacker](https://github.com/qwerty472123/wxappUnpacker "wxappUnpacker") 改进的。


# 分包功能

当检测到 wxapkg 为子包时, 添加-s 参数指定主包源码路径即可自动将子包的 wxss,wxml,js 解析到主包的对应位置下. 完整流程大致如下: 
1. 获取主包和若干子包
2. 解包主包 `./bingo.sh testpkg/master-xxx.wxapkg`
3. 解包子包 `./bingo.sh testpkg/sub-1-xxx.wxapkg -s=../master-xxx`

TIP
> -s 参数可为相对路径或绝对路径, 推荐使用绝对路径, 因为相对路径的起点不是当前目录 而是子包解包后的目录

```
├── testpkg
│   ├── sub-1-xxx.wxapkg #被解析子包
│   └── sub-1-xxx               #相对路径的起点
│       ├── app-service.js
│   ├── master-xxx.wxapkg
│   └── master-xxx             # ../master-xxx 就是这个目录
│       ├── app.json
```

------------------------------------------------------------------以下是其他内容，与上面有些关联，纯属记录----------------------------

【微信小程序】反编译获取任何微信小程序源码和Charles抓包工具使用教程
原文链接：http://www.zhideqiang.com/course/28613

一、前言
最近在学习微信小程序开发，半个月学习下来，很想实战一下踩踩坑，于是就仿写了一个阿里妈妈淘宝客小程序的前端实现，过程一言难尽，差不多两周时间过去了，发现小程序的坑远比想象的要多的多！！在实际练手中，完全是黑盒的，看到人家上线的小程序的效果，纯靠推测，部分效果在绞尽脑汁后能做出大致的实现，但是有些细节，费劲全力都没能做出来。很想一窥源码，查看究竟，看看大厂的前端大神们是如何规避了小程序的各种奇葩的坑。

于是就想到获取到小程序地源文件，然后再对其进行反编译还原为源代码，来作为学习参考。我百度了各种关于小程序地反编译教程，但是感觉都不太适合像我这样地初学小白，踩了挺多坑。在这里把我重新简化好的，快速地获取一个微信小程序源码的方式记录下来。

二、简单聊一下xxxxx.wxapkg
先来想想一个很简单的问题，小程序的源文件存放在哪？
当然是在微信的服务器上。
但是在微信服务器上，普通用户想要获取到，肯定是十分困难的，有没有别的办法呢？
简单思考一下我们使用小程序的场景就会明白，当我们点开一个微信小程序的时候，其实是微信已经将它的从服务器上下载到了手机，然后再来运行的。
所以，虽然我们没能力从服务器上获取到，但是我们应该可以从手机本地找到到已经下载过的小程序源文件
那么如何才能在手机里找到小程序的源文件包呢？
具体目录位置直接给出：
/data/data/com.tencent.mm/MicroMsg//appbrand/pkg/
在这个目录下，会发现一些 xxxxxxx.wxapkg 类型的文件，这些就是微信小程序的包
微信小程序的格式就是:.wxapkg
.wxapkg是一个二进制文件，有其自己的一套结构。
关于.wxapkg的详细内容可以参考lrdcq大神的博文：微信小程序源码阅读笔记
但是这里有个坑，想要进入到上面这个目录的话，用手机自带的文件管理器肯定是不行的，安卓或者iPhone都要要用到第三方的文件管理器，比如：RE文件管理器，并且安卓需要取得root权限，而苹果手机肯定是要越狱的，且iphone的越狱难度>>安卓获取root，不管越狱还是root，这都太费劲，当然有能力的同学可以直接从手机上来操作，但是这里不推荐从真机上获取。
三、准备材料
node.js运行环境 下载地址
如果没有安装nodejs，请先安装一下
反编译的脚本。 下载地址
这里提供一个Github上qwerty472123大神写的node.js版本的，当然也有其它版本的，这里我只是简单地用node.js版本举例
安卓模拟器（要求自带root权限）下载地址自行百度
我使用的是夜神模拟器，用来获取小程序源文件
RE管理器 下载地址自行百度
到时候要拖到模拟器中的
四、详细步骤
1、查小程序的appid与当前上传的标记

手机，电脑都有软件通过抓包可以查到这些信息，我用的charles（下载地址），可以看到人人商城官方小程序的appid是wx5d7c9a63ffdf10f8，当前已经上传了30次。 这个30很重要，我们后面找小程序文件的话就是看这个。



关于charles 说下，由于小程序采用https的加密方式，所以需要模拟器上和手机上都需要安装相应的证书。具体写再文章后面，复制网络的文章，给大家参考！

2、使用安卓模拟器获取到.wxapkg文件

不用越狱，不用root，使用电脑端的安卓模拟器来获取是一个非常简单快捷且万能的获取方式，具体步骤如下：
打开安装好的安卓模拟器，并在模拟器中安装QQ、微信、RE管理器
QQ、微信在模拟器自带的应用商店里搜索下载安装即可
QQ、微信在模拟器自带的应用商店里搜索下载安装即可
RE管理器的下载地址自行百度
下载好后直接拖拽进打开的模拟器窗口就会自动安装
设置一下模拟器
以我个人认为比较好用的夜神模拟器举例
首先到模拟器内部设置超级用户权限
1
2

这些操作的目的都是为了能让RE管理器顺利的获取到ROOT权限
接下来在模拟器里打开微信，然后在微信中运行你想要获取的下程序（这其实是让微信把小程序的源文件包从服务器下载到了本地了）
就以我说的这款淘宝客的小程序举例
在模拟器微信中运行一下后，直接切回模拟器桌面运行RE浏览器 来到目录
/data/data/com.tencent.mm/MicroMsg//appbrand/pkg/
就抵达了目的文件夹
你会看到发现里面的一些.wxapkg后缀的文件，就是它们没错啦，可以根据使用的时间来判断那个是你刚才从服务器下载过来的
一般小程序的文件不会太大，可以结合时间来判断，长按压缩所选文件,然后再将压缩好的包通过QQ发送到我的电脑
如果不进行压缩的话，是无法将这个文件通过QQ来发送的
所以QQ的这个功能可以让我们很方便的拿到源文件，而不必到电脑目录去找模拟器的文件目录。
解压。这样几步简单操作，就成功拿到了小程序的源文件了。
五、使用反编译脚本解包 wxapkg
到这里你应该已经将反编译脚本从github下载 或者 clone 到本地某个目录
3
打开nodejs命令窗口，按住shift+右击，或者新建一个文件 命名为cmd.bat  ，编辑内容为cmd.exe 保存打开。
4
cd 到你clone或者下载好的反编译脚本目录下
5
在node命令窗口中依次安装如下依赖：
npm install esprima
npm install css-tree
npm install cssbeautify
npm install vm2
npm install uglify-es
npm install js-beautify
npm install escodegen
安装好依赖之后，就是最后一步了，反编译 .wxapkg 文件
这里再说明一点，小编之前反编译一直出错，看下图，就是依赖没安装完整，看报错来安装即可
比如  Error: Cannot find module ‘js-beautify’   就执行  npm install js-beautify


 

在当前目录下输入 node wuWxapkg.js [-d] //files 就是你想要反编译的文件名 例如：我有一个需要反编译的文件 _163200311_32.wxapkg 已经解压到了C盘根目录下,那么就输出命令 node .wuWxapkg.js C:\_163200311_32.wxapkg
回车运行
6
反编译脚本就能一步将.wxapkg 文件还原为微信开发者工具能够运行的源文件，目录地址和你反编译的文件地址是一样的 然后在微信开发者工具新增项目即可打开
运行成功，源码获取完成
六、结束语
至此我们就通过非常简单的方式获取到了一个想要的小程序源文件，并对齐进行了反编译还原以后想要再反编译其他的小程序，非常快速，只需要两步即可完成

使用模拟器找到小程序.wxapkg文件
使用nodejs 反编译脚本将.wxapkg文件反编译
使用此方法，绝大部分的小程序都能正常反编译出来，但是也会有一些特殊的情况，具体可以查看qwerty472123大神的readme文件

.apk 之类的文件反编译非常困难，而小程序竟可以如此轻松随意地被获取到源码，根源在于小程序的开发团队并没有对小程序的执行文件进行有效的保护，也就是加密，所以我们才能使用别人写好的脚本直接进行反编译，其过程类似于解压。

实际上，小程序只是很简单的将图片、js和json文件压在一起，而压制的过程就是Wxml -> Html、 Wxml -> JS、Wxss -> Css，转换后文件二进制格式跟后缀名为wx二进制格式完全一致。

上线的源代码能如此简单的被获取到，不得不说小程序的源码安全存在很大的隐患，这一点很多开发者应该也知道，所以发现有些小程序会将重要的js逻辑代码柔在一个js文件中，这样，即使被获取了源码，也不是很容易读懂，但是任然避免不了被窥视的问题。小程序作为微信生态内的新生力量，不仅被官方，也被很多开发者和内容创业者寄予厚望，处于对代码的安全性的考虑，这个漏洞迟早有一天会被 修复（封掉） 的。

所以这种这里介绍的获取小程序源码的方法，应该是不会太长久的。

七、一步一步教你学会抓包工具Charles的使用(下载破解+代理设置+证书配置)
以前使用抓包神器fiddler抓包还是很厉害的，听说过Charles一直没用过，只从换了mac，fiddler就没发用了，只能研究下Charles，这都不是重点，主要是现在的请求都使用了https抓包就不太好了，各种证书验证，无意中发现有人研究出来抓包https的方法，按照其步骤操作了一遍，神奇的效果发生了，https也可以咦

本文基于Windows平台（ver：4.2），侧重移动端抓包配置

Charles主要定位：抓包工具；

本机，局域网，移动端；

Windows平台启动界面：download（https://www.charlesproxy.com/download/）



启动界面：共6部分；



1、查看本机IP：

Help->Local IP Address;//查看本机IP地址；

2、手机端配置：

PC端抓包前提条件：（1、移动端和代理端处于同一局域网；2、代理服务器白名单包括移动终端IP地址）

样图（华为honour9）和PC端加入白名单配置（默认弹窗提示：需允许）



配置完成后建议重启WiFi（tip：重启后确保处于同一局域网络）

3、抓包

在手机端操作后再PC端可以具体查看session：



如果不愿意记录，可以点击红色录制按钮，请求不会被记录（同Proxy->Stop Recording == CTRL+R）

4、涉及到https请求抓取，如下步骤：

更新本地证书->配置ssl->移动终端安装证书：



Help->SSL Proxying->Install Charles Root Certificate(安装)

SSL配置：



Proxy->SSL Proxying Settings;

首先勾选：Enable SSL Proxying（域名：端口）；

所有https加入ssl：1、*:*；2、null:443;

具体域名：m.baidu.com:443;

手机端安装证书：



在手机端输入PC-IP:port;或者输入chls.pro/ssl 获取证书下载链接；下载后安装；

tips：1、Android部分下载是后缀pem文件不能直接安装，需要在手机安全中找到安装证书-从SD卡安装，找到下载的证书文件，安装完成后启动应用（测试的页面或app）

2、IOS10.3.0及以上，charles的根证书已经在安装列表中显示,但它是被关闭的。在iOS 10.3之前,当你将安装一个自定义证书,iOS会默认信任,不需要进一步的设置。而iOS 10.3之后,安装新的自定义证书默认是不受信任的。如果要信任已安装的自定义证书,需要手动打开开关以信任证书。

解决方案：设置->通用->关于本机->证书信任设置-> 找到charles proxy custom root certificate然后信任该证书



然后就可以录制https请求了；

5、开始尽情包数据分析吧

6、取消代理->关闭Charles

 

八、一步一步教你学会抓包工具Charles的使用(mac环境下)
mac操作步骤，和win操作差不多，具体步骤都差不多，可参考下：

1.下载Charles 4.0.2
http://xclient.info/s/charles.html

help–>SSLProxying–> Install Charles Root Ceriticate

android使用Charles抓包https请求

看到的界面：
android使用Charles抓包https请求

找到Charles Proxy CA(xxx)——>打开——>选择信任——>始终信任
有的看到的是Charles Proxy Custom Root Certificate 信任步骤与Charles Proxy CA一致

android使用Charles抓包https请求

3.手机安装证书
安装手机证书
help–>SSLProxying–> Install Charles Root Ceriticate on a Mobile Device or Remote Browser

android使用Charles抓包https请求

然后在手机浏览器中访问手机 http://charlesproxy.com/getssl

出现安装证书提示，随便打个名称 比如android，选择WLAN（这里Android，一定要选WLAN而不是VPNxxx），确定

到这里手机端就设置好了，下面设置过滤条件

4.设置代理https端口
Charles的工具栏上点击Proxy –》SSL Proxying Settings

android使用Charles抓包https请求

然后添加需要代理的host及其port
这里设置的是用*代表全部的host，端口号 443

android使用Charles抓包https请求

接下来就可以访问https请求测试

android使用Charles抓包https请求

以上就是android使用Charles抓包https请求的全文介绍,希望对您学习mac开发和使用有所帮助
