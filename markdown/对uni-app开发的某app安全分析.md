## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[对uni-app开发的某app分析](https://mp.weixin.qq.com/s/naxiHAhKcB1v2Qy7hRU_-g "对uni-app开发的某app分析")

已经发公众号的为什么还发csdn  
有的圈内朋友，不经过我的允许，删减摘录我公众号的内容，这里就不提谁了，心里清楚，还能获得一些关注和流量。很无语，所以我还不如自己也发发。【猛男落泪】

## 前言

偶然发现某个app，居然用的 **uni-app** 开发的，而且两三天更新一次，关键它还是热更新的，都不用在应用市场更新，直接自更新自动重启，我觉得这个还不多见，所以记录下来

而且比较骚的是，我最后居然用苹果手机搞定它的，别慌，我不太会ios的，大佬们别激动

目标：Y29t\{deleteme\}LnRpYW\{deleteme\}5xaW9u\{deleteme\}Zy5jb20=

## 分析

### 什么是uni-app

> uni-app（uni，读you ni，是统一的意思）是一个使用Vue.js开发所有前端应用的框架，开发者编写一套代码，可发布到iOS、Android、Web（响应式）、以及各种小程序、快应用等多个平台

说人话就是，它是跨平台的一种开发框架，这点跟flutter有点像，不过不像的是它的重点是在前端部分，所以开发出来的app，本质是一个类h5的app。

而由于其不是正统的安卓开发技术栈里的东西，所以也是不太好hook某个参数的，因为它是js生成的，跟安卓逻辑没有太大联系，就跟flutter一样，只能找找有没有跟java层相关的业务代码，看能不能hook了，如果不行，就难搞了。

有老哥可能会想，能用web端的hook手段操作吗，想一下，你怎么在手机上执行你的hook代码呢，怎么打断点调试啊？是不是感觉不太好操作。所以我个人觉得这才是它的难点

### 分析目标app

在我写这篇文章的时候，该app的安卓端已经有问题了，反正就是登录不上，很奇怪，但是不影响我们分析，因为ios端也有，用苹果手机看一样的（只是针对此时此刻来说）

抓包

老套路，打开app，抓包分析，整个抓包分析操作我是在苹果手机里操作的，一打开刷新数据页面：

![](https://img-blog.csdnimg.cn/img_convert/3a526cbaadd5c6e68e3e41aa31f23bc1.jpeg)

卧槽，有某验验证，而且还是4代

没事，这个不重要，它不是我们本篇文章的重点

先手过，拿到接口，全程很轻松

![](https://img-blog.csdnimg.cn/img_convert/c10178c0a4f08eb5f656a438112cc1fd.png)

现在的重点就是这几个参数了

timestamp和page不说了，token是登录后返回的，除了sign，其他的都是一些固定的参数

好的，今天的重点就是sign了

## 安全调试

### 常规分析

打开jadx，直接搜吧

![](https://img-blog.csdnimg.cn/img_convert/3d637a5dafc10bce8229f63b5753c43b.png)

没啥有用的，再搜，这个结果，这个类名，明显是sdk包的东西，也不对

![](https://img-blog.csdnimg.cn/img_convert/3de14b9af2b414c4388dc340d9970d7f.png)

别慌，因为它这个app是热更新，那么我卸载重装或者清除一下app的数据缓存，它一定会让我重新更新，那么我拿到更新包再看看，说不定有东西

一顿操作之后，重设置抓包环境，再打开app

一打开它就请求了个接口，应该就是用这个接口检测app版本，有更新就下载，没更新就跳过，应该就是这么实现热更新的，有点骚，说实话，我还很少在ios端看到能热更新的，跳过了每次的发版审核，还是牛逼

![](https://img-blog.csdnimg.cn/img_convert/5596e3845025bcbd2b5c09245ae4fb50.png)

提示要更新，点完更新后（以上步骤也是在苹果手机上操作的）

![](https://img-blog.csdnimg.cn/img_convert/6431e9432e70143997a9cbc3072d6962.jpeg)

看尿壶抓的结果，这个20M的东西就有点可疑了

![](https://img-blog.csdnimg.cn/img_convert/779d4e55b8d1d3abf1f087a7297b9106.png)

复制这个链接，下载下来：

![](https://img-blog.csdnimg.cn/img_convert/eddf7afdd3fc5a03cba541e4428c315e.png)

![](https://img-blog.csdnimg.cn/img_convert/eacb58cc9cfa36fe8a216953646e4bed.png)

wgt后缀的文件是个什么勾8

### wgt介绍

什么是wgt

> .wgt文件扩展名是由三个用的应用程序。一个是Opera网络浏览器。这些.wgt文件称为使用这个浏览器作为一个小型的Web应用程序部件。这些文件包含有关部件配置，图像，索引，样式表，JavaScript的等信息，他们通常是在一个压缩归档格式。 .wgt文件也被称为包含了Xwindow的神经生物集群项目进来的记录文件二进制格式。应用程序的Xwindow神经生物集群被用来通过神经生物学家执行研究和该软件功能作为一个虚拟的工作站。该方案IMPS也利用了的.wgt以文件扩展软件来实现其功能，如普查和调查数据的处理。这些.wgt文件收集的人口普查局的数据，并使用该软件保存。这些文件所使用的调查和专业人士进行调查统计和普查任务。

wgt如何打开

Opera浏览器可以打开，不过这里不重要，我们不用直接打开它

说实话，我看了半天wgt介绍没看出个所以然

### 直接干吧

既然它是热更新，不管了，直接改成zip文件后缀，然后解压

![](https://img-blog.csdnimg.cn/img_convert/daf59c0696137dbb6c5a37f2c3aee345.png)

插一句，其实也可以改为apk文件（我在苹果端下载的一样可以），用jadx打开，一样可以显示，然后分析

![](https://img-blog.csdnimg.cn/img_convert/9857be344f668c8962233387abd33b61.png)

或者也可以用jadx另存为gradle，然后用android studio打开，结果也一样，就不展示了

![](https://img-blog.csdnimg.cn/img_convert/2b636fc74429f8308294d3a055d99167.png)

再看这些文件，不就是uni-app的文件吗（搞过前端开发的朋友应该比较熟悉），直接找sign在哪个文件里面就行了，肯定不能乱找，uni-app既然是个前端框架，那只找js文件吧，再看文件名，根据MVC规范，我猜要嘛在app-config，要嘛在app-server里

果不其然，就在app-server里了，打开文件发现这个js都没啥混淆，直接能看到的，这不就跟分析web网站一样了吗，只是没法动态调式

再次一顿搜索：  

![](https://img-blog.csdnimg.cn/img_convert/46c8047214cb79230d494c024171d165.png)

79个结果，感觉还是有点多，不想一个一个翻了，重新请求一下接口，再看看：

![](https://img-blog.csdnimg.cn/img_convert/870919eb57af8bb6d97f85ce66a1ae24.png)

说实话，看着真像一个md5，然后做了大写转换，那直接搜下toUpperCase

![](https://img-blog.csdnimg.cn/img_convert/b76c82e42439e7039bf414eaa923b35d.png)

很快就找到了：

![](https://img-blog.csdnimg.cn/img_convert/af95e43b047018c7a170f386f1217770.png)

就是这里了，接下来就是算法实现了

## 算法实现+验证

首先看下接口的参数：

![](https://img-blog.csdnimg.cn/img_convert/2e4965a639fc6cc21d68e313a67a2c32.png)

用python实现：

![](https://img-blog.csdnimg.cn/img_convert/919f7ac9084ce631cf1117dbf7793968.png)

能对上，搞定，接口请求

![](https://img-blog.csdnimg.cn/img_convert/c35c20b115018ec9b1394b1ad29ac048.png)

搞定！

## 结语

整个过程比较轻松加愉快，嘿嘿，本质上不算难，只是uni-app的确实不多，资料很少。主要这个app其实js没有做混淆，并不是难搞的那种，假如哪一天做了混淆或者上了jsvmp啥的，就难说了

想要技术交流的朋友们，加我wxID： **geekbyte**