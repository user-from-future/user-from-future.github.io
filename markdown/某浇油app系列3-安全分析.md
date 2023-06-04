## 声明

###   
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[某浇油app系列3-安全分析](https://mp.weixin.qq.com/s/WCyNeVQItC2YXAoHEEdm7w "某浇油app系列3-安全分析")

已经发公众号的为什么还发csdn  
有的圈内朋友，不经过我的允许，删减摘录我公众号的内容，这里就不提谁了，心里清楚，还能获得一些关注和流量。很无语，所以我还不如自己也发发。【猛男落泪】

## 前言

继续交友系列

是下面这个图片，评论区里的广告，不注意看还以为是用户发的

![](https://img-blog.csdnimg.cn/img_convert/1afd1da868b54000f13bedce3dd88bc0.png)

同样的，看首页的推荐接口：

![](https://img-blog.csdnimg.cn/img_convert/99952c6987dcc3566c4a00052b7571a5.png)

## 分析

copy到postman里，能直接用，去掉一些参数也是

![](https://img-blog.csdnimg.cn/img_convert/345e8958ed12b9b6c1b25a78d87009cb.png)

再试试header里的参数：就发现，这两个参数必不可少了

看到token，首先想到是不是登录后返回的

![](https://img-blog.csdnimg.cn/img_convert/dadf03f31d41534a0cff94e51643331f.png)

在app这边，账号退出再重新登录，抓包继续看：

卧槽了，这个token就是账号登录后返回的，

![](https://img-blog.csdnimg.cn/img_convert/9c5d69eaefdaf190ff8195d7ed12ca17.png)

毫无压力啊卧槽

但是仔细看，登录有两个加密参数，password和pwd

![](https://img-blog.csdnimg.cn/img_convert/79996e10506f040a7d9cc9df62e20062.png)

好的，这就来到了本篇文章的正题

### 1.加密逻辑定位

1.1. frida动态调试

首先脱个壳

先搜"password"，运气很好，一下就找到这里：

![](https://img-blog.csdnimg.cn/img_convert/6abeeeaf27ad6946d49709e3a713840d.png)

然后objection走一下呢，卧槽， 有对抗

![](https://img-blog.csdnimg.cn/img_convert/fd70f40964b8eb81ae69831ac8c28ec9.png)

用antifrida脚本看看，ok，可以

![](https://img-blog.csdnimg.cn/img_convert/aaed51b59f14060f87b2ee85c97f2abc.png)

加上自己的hook逻辑

![](https://img-blog.csdnimg.cn/img_convert/f625ab7b8dded7cb908e60a65aea485e.png)

但是感觉还是没走到，貌似，不在一个classloader\?

![](https://img-blog.csdnimg.cn/img_convert/5b88e505d9cbdf4358e76cbaedbdc4c7.png)

那就切换classloader，涉及到的代码网上一大把，这里就不说了。

根据我的操作，结果还是不行，那好吧，继续上xposed看看：

1.2. xpoed 自写hook插件分析

相关的xp配置这里就省略了，上面已经说过了。

![](https://img-blog.csdnimg.cn/img_convert/73a1664c64f82e803a53756ce9645ac1.png)

然后重新走登录逻辑，看看打印日志：

![](https://img-blog.csdnimg.cn/img_convert/e292bd72786b3e2732594a9272b67c32.png)

这个加密结果基本能跟抓包的对应上

![](https://img-blog.csdnimg.cn/img_convert/5b06f1ef626f48b750ba28433ea363be.png)

那加密逻辑就是这里了。很奇怪，frida就是不行，xp就是可以，不管了，东西太多了，不用一个一个都去纠结

1.3.  算法助手插件辅助分析

这里加个餐，用另一个方法定位这个加密参数逻辑

用军哥大佬开发的【算法助手】工具，这个也是xp的一个插件，进到xp里勾选上目标app，然后点进来，按照以下步骤操作

![](https://img-blog.csdnimg.cn/img_convert/33e9ce01b8b3f94c23ac5425fc27f754.png)

第三步点完之后，会自动启动app，手机这边正常重新登录操作，

点击登陆后，回到算法助手，返回到日志tab下，点进你的目标app

![](https://img-blog.csdnimg.cn/img_convert/c81ea01240fe84d8e40e4127698fb1d9.png)

然后就会有很多这种日志：

![](https://img-blog.csdnimg.cn/img_convert/efac85e7dc186e11fca7db0a5900535f.png)

直接点搜索按钮：

![](https://img-blog.csdnimg.cn/img_convert/704c28058b662e258f536d2cda3fc27d.png)

到抓包工具里，复制加密字段，然后拿来搜

![](https://img-blog.csdnimg.cn/img_convert/c87a0fefc6ea63ddd2341f58674a4b8c.png)

![](https://img-blog.csdnimg.cn/img_convert/da150ad0a1b83ee21d39884baeebee86.png)

点进去，对照着看，基本能确定，是同一个

![](https://img-blog.csdnimg.cn/img_convert/d8f80399f091a33b2fe7d74399b3dd63.png)

然后往下滑，有一个调用栈，这不就出来了吗，直接就找到位置了，发现这个位置基本跟刚才hook写的地方一致

![](https://img-blog.csdnimg.cn/img_convert/ba7d3f136cfe701d2bb6a0223a1a20da.png)

这个方法适用于无法直接搜到加密参数的时候，或者你毫无头绪的时候，可以辅助查看

好了回到正题，跟着刚才的定位，再往里跟逻辑，实际的加密其实就是这个：CustAmuIExt

![](https://img-blog.csdnimg.cn/img_convert/40f86907c2fa9a3082092e1b832474be.png)

仔细看就知道就是native层的加密了

### 2.native层分析

掏出ida，拖进这个ve04:

![](https://img-blog.csdnimg.cn/img_convert/b904bba2a64b1a67ad1af515aad252f7.png)

卧槽，原来就是个tea加密

![](https://img-blog.csdnimg.cn/img_convert/1bfee348e326bbb13f58ac9c0c432c77.png)

这里唯一要注意的就是，哪个key，标准的tea加密的key是数字，而这里传进去的key是 【liaoxxxlib\_xxx】

看下这个tea加密：  

![](https://img-blog.csdnimg.cn/img_convert/903a3f94e4f3d6c2b0118705d5ed8a7c.png)

![](https://img-blog.csdnimg.cn/img_convert/1ebc1d9ceb5fd3d30ee4ef97a3f20096.png)

就这么几行，量它没有魔改，看着就搞个所以，注意下key就行，整个加密就这样，感觉不难

## 验证

具体我就不测试了。可以自行分析

分析完了后，看了下消息，挺好，才几条，感觉这个app里的人要真一点

![](https://img-blog.csdnimg.cn/img_convert/cd49f39ef52613a9c0aaed42d647fb76.png)

![](https://img-blog.csdnimg.cn/img_convert/01dc0463b5c6329641b072e772ddfd7a.png)

## 结语

感觉搞了这几个浇油类的app之后，大多都很简单，有点不想继续更这个系列了。练手可以，但是如果后续都是这种程度的话，就有点不想继续写了，相信各位朋友看多了也没啥意思了，不过还是看受众效果或者后面有没有代表性的吧。over

哥哥们，关注下我好不好

![](https://img-blog.csdnimg.cn/img_convert/d16e57221a60d76db4f4777b74619cd4.png)

工作避坑\&内推（仅成都）、技术交流、商务合作、技术交流群

扫码或者搜ID： **geekbyte**