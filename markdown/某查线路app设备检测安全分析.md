## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[某查线路app设备检测分析](https://mp.weixin.qq.com/s/HMrhWmB4k_c90WWPOyBDkA "某查线路app设备检测分析")

已经发公众号的为什么还发csdn  
有的圈内朋友，不经过我的允许，删减摘录我公众号的内容，这里就不提谁了，心里清楚，还能获得一些关注和流量。很无语，所以我还不如自己也发发。【猛男落泪】

## 前言

又好久没更新了。最近一直都在知识的海洋里苦苦挣扎，也算是学了一些东西，自以为技术还可以了，结果遇到几个难搞的app，又把我干废了。

还是得多学多练啊

这个app是好友\@Artio 给我的，我本着好奇想玩一下，结果还挺有意思。

## 分析

打开app，wifi访问，习惯性的开postern+charles，准备抓个包看看：

![](https://img-blog.csdnimg.cn/img_convert/f6a7cd09edb5120281c1d7f67d24c393.png)

然后就有这个提示，我开流量访问，就很正常的访问数据，我就觉得很奇怪了，难道这个app禁止wifi访问？如果禁止wifi访问的话，是不是也算一种另类的反抓包办法啊哈哈

搞这个的兄弟们应该都知道，像这种提示，是toast实现的（有关toast文档在这里：https://developer.android.google.cn/guide/topics/ui/notifiers/toasts\?hl=en#java），那就跟下逻辑，找堆栈，看他是怎么识别我用的wifi的

objection 启动，内存漫游分析

![](https://img-blog.csdnimg.cn/img_convert/ae0d509d57c99b03c930731af349e69f.png)

卧槽，一启动就挂了，看来有frida检测，用脚本过掉frida检测，这里我用的脚本是针对两个的，一个字符串，一个pthread的

![](https://img-blog.csdnimg.cn/img_convert/30fee8b6a592567bd7f88b80d94a9f28.png)

先只开针对str的：

![](https://img-blog.csdnimg.cn/img_convert/8b69bf9498b48dd3e82137377ba1a265.png)

ok，app正常打开，frida也没掉，那就算过了（后面我才知道并没有这么简单）

那么要objection 分析的话，就开两个终端

先执行 objection \-g xxx explore

另一个终端马上执行  frida \-U \-l hook.js   xxx

好了，现在就可以在objection里分析了，用search  wifi，找到了3000多个

![](https://img-blog.csdnimg.cn/img_convert/149521f71c767f245d4a3dd45ebca492.png)

我人麻了，哪个才是啊，后面一步步跟逻辑，发现了toast的类，

![](https://img-blog.csdnimg.cn/img_convert/ba16959f71e3ae4edce0ab861f5223dc.png)

然后在toast基础之上他自己封装了一个类：

![](https://img-blog.csdnimg.cn/img_convert/ad36cd4b7604df79ebae924f40075ac0.png)

然后用show调的

![](https://img-blog.csdnimg.cn/img_convert/717d9a13c553e864eb9786b31ca597f0.png)

那就一步步回溯堆栈看他是怎么判断的了。思路是没问题的对吧。我调试了老半天，愣是没找到怎么判断的，找到下面几个比较可疑的地方，结果hook发现根本没走到这里

![](https://img-blog.csdnimg.cn/img_convert/1f1e15da70c8b5d26cf1b415885fc693.png)

迷茫了

好了，今天的技术分享到此结束，遇到问题睡大觉，要学会放弃，菜狗就得认命。。。。

![](https://img-blog.csdnimg.cn/img_convert/b6fb808b1ea45d92dcafd2d7d3cb95ad.jpeg)

好了，不扯了。后面冷静之后再分析，发现其实是frida检测到了，只是对strhook是不够的，还得有pthread的，所以我把上面frida对抗的脚本两个都开上了，用spwan模式再次启动

然后就有了，wifi可以使用了：

![](https://img-blog.csdnimg.cn/img_convert/5008e0cf190c7be97e6f638d329b737f.png)

为了验证这个，我把frida-server停了再重启app，wifi可以正常打开，还真的是被检测了

好的，现在再打开postern看看，发现还是提示网络错误那个，那就是有vpn检测了

对于vpn检测的怎么搞呢，肉师傅星球里有相关的，同时里面也有个练习app，Ipconfig，可以安装打开看看：

这是啥都不开的状态 ：

![](https://img-blog.csdnimg.cn/img_convert/0dcf69d3598457239f1c828798284cab.png)

这是开了流量的状态：

![](https://img-blog.csdnimg.cn/img_convert/c0de0696f07ce20b3a969543989a2f62.png)

这是开了wifi的状态（开不开流量都是如下）：

![](https://img-blog.csdnimg.cn/img_convert/25441fa4d0bbd961e187f53461850b01.png)

这是开了postern vpn的状态：

![](https://img-blog.csdnimg.cn/img_convert/5866e715835797526693782e76d6716d.png)

这区别是不是很大啊，所以，其实，大部分的vpn检测就是对tun0的检测，那我们再搞个hook vpn检测的frida脚本是不是就搞定了

好像说得通，那么，还愣着干嘛，搞起来啊，这下面的就是主要的逻辑

![](https://img-blog.csdnimg.cn/img_convert/8435766cb3f72e3ca72f9628c4985744.png)

现在跑一下这个过vpn的脚本，然后看看Ipconfig是不是被修改了就行了：

![](https://img-blog.csdnimg.cn/img_convert/acb5ba40b85795959904fdf3e48267bc.png)

Ipconfig显示：

![](https://img-blog.csdnimg.cn/img_convert/623ca87038d174007545d1ccae3ef58c.png)

两个图片对比下：

![](https://img-blog.csdnimg.cn/img_convert/36b48b1ce0854dcb8e45ff202c621663.png)

好的，至少是过了，注意ccmni2的值wlan0，ccmni1的值还是tun0，如果你想改的更彻底一点的话，可以再把这个值也改了，或者把值置空，这里我就不去操作了，自己研究了。

好的，把这段过vpn的代码封装成函数，跟上面过frida检测的放在一个js文件里，spwan模式启动：

![](https://img-blog.csdnimg.cn/img_convert/d2b9cf3f8ac119413af2551b9f4e99d9.png)

![](https://img-blog.csdnimg.cn/img_convert/446204835d98ac43f618e7756c3526db.png)

选择两个地方

![](https://img-blog.csdnimg.cn/img_convert/c3f335017b0af31cac56ad7703e1f3a3.png)

然后charles看结果，ok，完美抓到包，返回的结果刚好就是刚才就是手机页面上显示的推荐路线

![](https://img-blog.csdnimg.cn/img_convert/b0bcc12b6bcef7a5a9c9b5bf54eca70e.png)

接下来就是处理加密参数了，卧槽，等会儿，还有wtoken（嘻嘻，有点意思）

除了wtoken以外，其他就这几个参数需要传

![](https://img-blog.csdnimg.cn/img_convert/065fce331265be5276b6c8f6034898e2.png)

发现，time-stamp是啥就不说了，request-id是time-stamp加了几位数

![](https://img-blog.csdnimg.cn/img_convert/304a90a33b5af430468df2b0ed496492.png)

![](https://img-blog.csdnimg.cn/img_convert/13243a062a4d9c342a47b899c78a89c8.png)

hmac逻辑在这里：

![](https://img-blog.csdnimg.cn/img_convert/76f29f3320f9aacc0409e79617a211d8.png)

![](https://img-blog.csdnimg.cn/img_convert/38d94fb549fcbae8f2e73a16c0dc1e2d.png)

ok，都是在java层里直接可以看到的参数 ，这里就不再继续了，over

## 结语

搞这种还是得沉住气啊，慢慢来，别慌，不然思维容易卡住，一点点来，抽丝剥茧

当我hook掉之后，再次打开这个app，就不再限制wifi访问了，好神奇，具体细节我就没去研究了。

至于他怎么判断wifi的，我也没去抠细节了，可以看看相关的文章：

https://blog.csdn.net/qiqigeermumu/article/details/110429819

https://blog.csdn.net/cn\_yaojin/article/details/76854159

Ipconfig想玩一下的，可以在这里下载：

> 链接：https://pan.baidu.com/s/1K22pbglrt4Fawt\_T\_F8O1g
> 
> 提取码：ipjc