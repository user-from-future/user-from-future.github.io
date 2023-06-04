**目录**

[前言](#%E5%89%8D%E8%A8%80)

[分析](#%E5%88%86%E6%9E%90)

[加密参数分析](#%E5%8A%A0%E5%AF%86%E5%8F%82%E6%95%B0%E5%88%86%E6%9E%90)

[webview调试](#webview%E8%B0%83%E8%AF%95)

[1.webviewdebughook调试](#1.webviewdebughook%E8%B0%83%E8%AF%95)

[2.webviewpp调试](#2.webviewpp%E8%B0%83%E8%AF%95)

[源码静态分析](#%E6%BA%90%E7%A0%81%E9%9D%99%E6%80%81%E5%88%86%E6%9E%90)

[x-request-id](#x-request-id)

[x-token](#x-token)

[sign:](#sign%3A)

[web端调试，逆向](#web%E7%AB%AF%E8%B0%83%E8%AF%95%EF%BC%8C%E9%80%86%E5%90%91)

[uniapp调试分析](#uniapp%E8%B0%83%E8%AF%95%E5%88%86%E6%9E%90)

[流程分析](#%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90)

[尝试主动调用](#%E5%B0%9D%E8%AF%95%E4%B8%BB%E5%8A%A8%E8%B0%83%E7%94%A8)

[广告（不喜忽略）](#%E5%B9%BF%E5%91%8A%EF%BC%88%E4%B8%8D%E5%96%9C%E5%BF%BD%E7%95%A5%EF%BC%89)

[结语](#%E7%BB%93%E8%AF%AD)

---

# 前言

昨晚我群里有个老哥在问移动端滑块怎么分析

![](https://img-blog.csdnimg.cn/img_convert/88f9277ed63b27d1bc8f0460527d27a4.png)

**因为大多app端的滑块都是加载在webview上的**

我当时突然就想到才没几天搞定的一个app，也有webview部分的操作，因为是用uni打包的，所以实际走的还是webview。而uni相关的其实之前发过一篇 [对uni-app开发的某app逆向分析](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484843&idx=1&sn=f75fb361e154490419f61c6ac86ef81b&chksm=fb18f409cc6f7d1f5591c8a601a545d7de23657060429b815e3f16719aa4f8347b2974936a37&scene=21#wechat_redirect "对uni-app开发的某app逆向分析")

不过今天这个app有点不同，我感觉是值得记录的，所以准备再写一篇有关这个的

# 分析

首先，打开这个app，抓包，左边就是想要的数据，右边就是抓到的包

![](https://img-blog.csdnimg.cn/img_convert/ac84ed51553327ce58dfccf03cd0516d.png)

然后加密参数有几个

```bash

x-request-id
x-token
sign
```

然后返回有加密

![](https://img-blog.csdnimg.cn/img_convert/c8f9f0ddaeba9fdd8f4c8d03af4b454b.png)

## 加密参数分析

首先就是对app一顿分析

搜一个sign:

![](https://img-blog.csdnimg.cn/img_convert/b46d5da7a924ec1a1b1da8540f96cde5.png)

一开始我并不知道这个app是uni-app打包的，一搜，发现最后一个，这个assets/apps/xxxxx/app-services.js有点可疑，因为它也可能并没有用到这部分的代码。

为了进一步确认，用android studio自带的devices monitor工具发现果然是webview，那基本就确定是uni打包了。

![](https://img-blog.csdnimg.cn/img_convert/9625beae9ffd6d4e2efc50ab09da8724.png)

插一句，我用的android studio 版本是4.1.1，然后你需要装jdk1.8，然后到sdk tools里，把相关的安装下，然后如果你的sdk目录有这个就行了，不然启动报错。

![](https://img-blog.csdnimg.cn/img_convert/c4eb7ddc2841a0288fc927a659022c0e.png)

详细的百度即可

## webview调试

### 1.webviewdebughook调试

那么，既然是webview，那就准备用webview调试一波，也就是文章开头我群里的大佬 \@qxp 说的路子。

当然这个部分 \@孤雁逐云 也发过相关的文章，这里我就不详细解释了，就大概的展示下

用xp插件，对该app开启webviewdebugg:

```
https://github.com/feix760/WebViewDebugHook
```

然后用chrome浏览器打开这个网址（第一次打开要开下科学软件，打开了就可以把科学关了）

```
chrome://inspect/#devices
```

![](https://img-blog.csdnimg.cn/img_convert/8f10be02ed9f3675b807355c8ea85340.png)

静等一会儿，这个加载有点慢，等出现有设备就行了。然后点【inspect】，就会出来一个弹窗，类似web页面的调试工具，然后就是正常的分析了。以前用它分析过某宝 app端的滑块：

![](https://img-blog.csdnimg.cn/img_convert/46d84fd8cc9744ce1af611e7797c801a.png)

![](https://img-blog.csdnimg.cn/img_convert/8fe96183443d1b90ac736abd1fbaf1c5.png)

但是，今天很奇怪，啥都没有

![](https://img-blog.csdnimg.cn/img_convert/99c8aa6d35c36310eb0287bf6cc589dc.png)

昨天都还可以加载的，有点迷，没任何东西了。

有时候环境很容易把人搞迷，不过没事，不能在一棵树上吊死，换一个工具

### 2.webviewpp调试

之前 \@孤烟逐云 老哥给我推荐过一个新的工具：

```
https://github.com/WankkoRee/WebViewPP
```

这个也是xp插件，直接在手机端上操作，用它来看看：

首先点开

![](https://img-blog.csdnimg.cn/img_convert/5e0b3cac898562f33f445d5589c9e588.png)

然后点这个：

![](https://img-blog.csdnimg.cn/img_convert/16c27fa8988d10fd54a6f9a425b84fab.png)

然后点组件左边的下载图标，管他有没有用，全都下载了再说

![](https://img-blog.csdnimg.cn/img_convert/f249aca07120ca0282bfd6a34bad734c.png)

下载完回去，选这个hook规则管理：

![](https://img-blog.csdnimg.cn/img_convert/933a159174ddbb4056252d70e1493a5b.png)

找到你的目标app

![](https://img-blog.csdnimg.cn/img_convert/bfc77b3e4ed9807ad0d77659442856fb.png)

点进去，把下面的全都点亮

![](https://img-blog.csdnimg.cn/img_convert/b269c10b095735ac8253f7a057df33ec.png)

然后再点这个，把主流的规则全都点一遍

![](https://img-blog.csdnimg.cn/img_convert/0fcb8eef646704906e118990c51323bf.png)

![](https://img-blog.csdnimg.cn/img_convert/20d7e6a8205fddc7a302d63645dd173d.png)

点完就会自动出现这些hook代码：

![](https://img-blog.csdnimg.cn/img_convert/98f7170fb99bf6f1708aa3165da45a1e.png)

你觉得不行还可以自己写你要的

这样就配置好了，然后这个webviewpp在后台，别关了，现在启动目标app，进入你要调试的页面，这右下角就自动出来了两个浮窗按钮

![](https://img-blog.csdnimg.cn/img_convert/1acc064e66a4ae5081bec10e95cd3de2.png)

点一下，选到network：

![](https://img-blog.csdnimg.cn/img_convert/d650cc160e34f9b57af4d78cf8597cf0.png)

然后刷下页面，下拉加载数据接口，结果发现，一直是空，

![](https://img-blog.csdnimg.cn/img_convert/b9311f660747ef23e053696e613e4ca2.png)

调试工具里根本没有抓到请求包，啥都没有，这就尴尬了。查阅uni相关资料，发现应该是js层发请求，所以，这样是抓不到，那这个就没法了，抓不到网络包，那就没法调试了。尴尬。

## 源码静态分析

既然无法动态调试，那只能先静态调试了

先对该app解包，然后直接来到assets文件夹里的这个目录的这个文件里：

![](https://img-blog.csdnimg.cn/img_convert/18589a15ebc7e43c5436f4b881f9f2f0.png)

直接搜那几个加密参数：

### x-request-id

![](https://img-blog.csdnimg.cn/img_convert/cd614d06801b7dcbde3d6eccda15c55a.png)

### x-token

![](https://img-blog.csdnimg.cn/img_convert/6158a0f7008eae1624e8094c91b9733f.png)

### sign:

![](https://img-blog.csdnimg.cn/img_convert/2c8df496bfbed12064a0dabeb37ae95d.png)

![](https://img-blog.csdnimg.cn/img_convert/04f35ebe6b210795c72306ae3117c055.png)

然后这里是返回解密逻辑：

![](https://img-blog.csdnimg.cn/img_convert/c4eb2f063a0d4a19e16cd2b8a5d314c1.png)

走一遍逻辑，发现x-request-id，看着很像uuid4

![](https://img-blog.csdnimg.cn/img_convert/eafcc1bafda0f28279f49cd3a8d7a4f6.png)

x-token 到这里就g了，这个参数，我完全不知道他怎么调用的，传的这个f是干嘛的

![](https://img-blog.csdnimg.cn/img_convert/2de04f877bdadb35ca0cdbb78b8427bd.png)

我去，线索断了，这就尴尬了。

中途我试过，找到app-services.js，加入以下内容

![](https://img-blog.csdnimg.cn/img_convert/6810a5d46462789a6a28f327b1ce8bfc.png)

然后重打包app，重新安装（mt管理器操作），然后借助webview调试，发现啥都没有，这就尴尬了。

我猜测大概率是这个js把console.log方法改写了，不让我输出，之前ibox就是这样的

在我一筹莫展之际，突然想起，这apk是uni打包，而且核心逻辑还在js里，那很大概率有web端网址，一顿操作之后发现，果然有web网站

![](https://img-blog.csdnimg.cn/img_convert/0ce448852a6707a51751a6b9c08f07ee.png)

# web端调试，逆向

打开之后，直接看调用栈，这对于我放弃js大半年的人的来说，属实生疏了，唉

![](https://img-blog.csdnimg.cn/img_convert/2049454ad5cb7d28820b564a5283a5a2.png)

不需要花太多时间，就跟到了实际的逻辑：

![](https://img-blog.csdnimg.cn/img_convert/e8afc0da81020d2c1c8afd69be73f36b.png)

![](https://img-blog.csdnimg.cn/img_convert/443020a3080f70571835c589adf80415.png)

最后根据我的一顿分析：总结如下：

![](https://img-blog.csdnimg.cn/img_convert/af953d9a649702379e367334aea747db.png)

x-token是下面的接口返回的：

![](https://img-blog.csdnimg.cn/img_convert/f91fb67280b81f667a52e44df6d109ae.png)

加密逻辑也跟上面一致，然后我一运行

![](https://img-blog.csdnimg.cn/img_convert/0ae5ef36320e197fb0095769b852dcd6.png)

g了，反正就是怎么也不出正常的结果。

最后呼唤算法还原大佬 \@小小白，一顿操作帮我还原了：

![](https://img-blog.csdnimg.cn/img_convert/e4eb87ff7f8bb58a7b0e3747edcbc3e1.png)

是真的牛逼啊卧槽。

但是，话说，不是说好的app吗，咋整到js上去了。想一下，假如，它这个js，有强混淆，而且很难调试的话，怎么办？或者说，我就是要在app层面进行分析呢\?

终于到了本篇文章的重头戏。

根据我查阅相关的资料，其实uni开发的，有好几种模式，但是实际还是借助了webview，以及jsbridge，进行联调，然后实现app端的数据交互

那么思路有以下：

```
1.hook webview，找逻辑
```

我的直觉，第三种可能更好用，一顿操作后发现，根本不行

# uniapp调试分析

最后偶然看到它的文档里【org.json.JSONArray】类，果然还是要走json类啊，hook完，发现确实有，

然后再一步步跟逻辑找逻辑，跟到这个类：

【io.xxxxx.xxxxx.JSONUtil】，objection hook结果：

![](https://img-blog.csdnimg.cn/img_convert/cd54d77a0796d0e59051be6729760c8c.png)

继续hook这几个方法，然后手机翻页看看，看看它这个入参，发现就是最后的页面数据了，但是这个数据结构有点奇怪

![](https://img-blog.csdnimg.cn/img_convert/c5e03ecb7f94620bfe161186588aa694.png)

对比下手机页面的数据，和这个入参，确实对上了

![](https://img-blog.csdnimg.cn/img_convert/2068606af044b6519d666d5dc33c979a.png)

再看看调用栈：有点东西啊，还用的tb的webview组件

![](https://img-blog.csdnimg.cn/img_convert/a6e3637ad957deebbf7021f4c38512cb.png)

## 流程分析

然后根据我的分析，实际的逻辑就是（不一定对，部分方法名已脱敏）

整理下：

```python
1.先用io.xxxxx.AdaUniWebView初始化view
2.用io.xxxxx.evalJS预加载js代码
3.再用io.xxxxxx.xxx.execSync 生成view对象
4.再用io.xxxxxx.xxxx.exec 用上面生成的对象发起请求
5.用io.xxxx.xxxx.prompt 实际请求
6.js 原生执行js
7.用io.dcloud.common.util.JSONUtil.createJSONArray 接收js代码返回的对象，里面包含实际的数据（js部分已经解密好）
```

部分代码可见：

![](https://img-blog.csdnimg.cn/img_convert/2bf52fc4119bc9ac2b49810f86257212.png)

![](https://img-blog.csdnimg.cn/img_convert/17dfe77627bf8d87004226fc13e7fc5a.png)

![](https://img-blog.csdnimg.cn/img_convert/49c57976f4a360f70263ef0e6646ed45.png)

## 尝试主动调用

我用xp插件，尝试主动调用，提炼下逻辑：

![](https://img-blog.csdnimg.cn/img_convert/c6c7fb3932375e6fa654b652b6229ee7.png)

结果失败了，貌似是不认我的传入的参数，那就没法了。

那思路就只有，用xposedappium之类的操作，然后hook拿结果了。具体流程这里就不展示了。

如果要完成最后拿数据的目标的话，当然最方便的还是直接走web端拿数据。

如果没搞错的话，只要是uni开发的app，这一套应该都是通用的，所以你懂我意思吧

![](https://img-blog.csdnimg.cn/img_convert/87fc4640f0671dd1e702e0a231a32c8f.png)

# 广告（不喜忽略）

![](https://img-blog.csdnimg.cn/img_convert/ecde623d4121c6ae2bf398f40d8713f4.jpeg)

![](https://img-blog.csdnimg.cn/img_convert/5c2a73dee98ec6bd59d22c4cb391d468.jpeg)

# 结语

工作避坑\&内推（仅成都）、技术交流、商务合作、技术交流群

扫码或者搜ID： **geekbyte**

![](https://img-blog.csdnimg.cn/img_convert/dd92d8a7588a6b92874f30146a4a6f9a.jpeg)