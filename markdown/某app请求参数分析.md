## 声明

这个内容之前在我公众号发过了。来这里只是备份下

[某app请求参数分析](https://mp.weixin.qq.com/s/Jqn_n8HXMBu500d9rIWgrQ "某app请求参数分析")

## 前言

我现在成为一个开发者，是看了一篇文章决定的，这篇文章就是当时还在知道创宇的余玹大佬发的信息安全技能树，自此之后走向了一条不归路，链接：

> https://blog.knownsec.com/Knownsec\_RD\_Checklist/v3.0.html
> 
> https://evilcos.me/security\_skill\_tree\_basic/index.html

如今突然想起来这个app，当时的我还不会app，再看它，感觉整改过了，所以还得多亏有关部门这几年来的严厉打击网络犯罪啊

声明下，这个app本身不存在诈X，因为里面没有金融类功能，是通过微信转账的，只是涉及诈x的人在使用这个app

## 分析

老办法，打开charles，打开postern转发，打开app，开始抓包

1.短信验证码获取接口

![](https://img-blog.csdnimg.cn/img_convert/9b8d1c1fcab9a7e5c2505c7f986cdcf1.png)

这个请求参数的type为md5就有点意思，直接就给我们提示了，哈哈哈哈。

2.登录接口：

![](https://img-blog.csdnimg.cn/img_convert/86c1cceb79827ba60a2e09cee709dcaf.png)

同时发现这个登录接口有抓包检测，开着postern一直显示登录中

3.用户信息接口：

![](https://img-blog.csdnimg.cn/img_convert/2c150ebe5a3a6e2ebf15ad0c317d42b6.png)

4.广场列表接口：

![](https://img-blog.csdnimg.cn/img_convert/af1c50a7ba6461f4ad044fad89a31e08.png)

5.滚动头条接口：

![](https://img-blog.csdnimg.cn/img_convert/b92c3f7c1a6ad3021432a3f8453c48d1.png)

6.直播间推荐接口

![](https://img-blog.csdnimg.cn/img_convert/7c848cf04654df854a743a526622cd46.png)

7.首页banner滚动接口：

![](https://img-blog.csdnimg.cn/img_convert/b06d6932f5d955a69e98cf4cc916749f.png)

8.工会接口：

![](https://img-blog.csdnimg.cn/img_convert/3155b9e165a75f11862db134e02d7922.png)

9.关注列表:

![](https://img-blog.csdnimg.cn/img_convert/8a2815d601e83e342ad8c811d7019b19.png)

10.直播列表：

![](https://img-blog.csdnimg.cn/img_convert/b109e974115d6060cdd0e77806cc66db.png)

随便点开一个直播间，直播间请求参数类型是octet-stream，多半是protobuf了，这个直接用blackboxprotobuf库就可以了，我看到这个数据提交都没有加密参数，我就没去研究了

聊天接口也是，就是tcp协议的传输，也不去看了

细心的朋友发现了，除了聊天和直播以外的接口，其他的接口请求数据基本都有带有这个sign，这次的重点就是上面的接口里的sign了。

## 参数定位

老路子，打开jadx或者jeb分析一波再说，打开一看，卧槽，有壳

![](https://img-blog.csdnimg.cn/img_convert/2c8f08b35be93d675e99374d18a88040.png)

那得先脱壳一波，脱壳过程就略过了，你懂的。

没过多久，脱壳成功，文件如下：

![](https://img-blog.csdnimg.cn/img_convert/7afb6c61af77a4ec56ffb553e824bed0.png)

然后用jadx打开，直接搜sign，两百多个，也还好，排除掉这个app用的第三方sdk的包名的，剩下的没几个

![](https://img-blog.csdnimg.cn/img_convert/f1eef61d6ca0fa4f695ffc44b02e3de4.png)

随便找几个，指向的都是这个一段，因为这个是脱壳后编译出来的，没法编译出安卓样式的代码，不过看逻辑倒是能看懂

![](https://img-blog.csdnimg.cn/img_convert/f58f282ec10deb675054c0adcc95cca3.png)

看完逻辑，说明sign就是掉的这个xxx.getSign方法，进入这个方法看看，同样的没法正常编译，问题不大，基本还是看得懂

![](https://img-blog.csdnimg.cn/img_convert/a841bc3afb474d461080b3e44c5ec941.png)

## 调试hook

那先不废话，直接hook一下就知道了，hook代码如下，用attatch模式启动，然后这边app刷新下，结果数据如下：

![](https://img-blog.csdnimg.cn/img_convert/ff0b804a66a83908d9741d42f7cbff30.png)

为了验证确实是，再看看charles这边的结果，直接拿到最后一个打印出来的返回值搜，确实有

![](https://img-blog.csdnimg.cn/img_convert/83efc793e42b2cdeaef823fa6f35681d.png)

那就ojbk了，这么简单？当我正准备接着看逻辑的，时候，app直接闪退了：

![](https://img-blog.csdnimg.cn/img_convert/d95eea28bab9e47011df854bc18f24d8.png)

还是可以，至少还有点反调试的套路，看看进程，这就一目了然了

![](https://img-blog.csdnimg.cn/img_convert/03cf5450d03b96c595f92475e5201dbd.png)

还是有点东西，不过东西不多，我还是能hook上，这个不重要，我都已经跟到逻辑了，接着再看下最后的结果：

![](https://img-blog.csdnimg.cn/img_convert/0b3bbf59e828781810749695d931bf5a.png)

所以，实际传的确实是两个参数，但是，jadx看到的方法体里，是三个参数，同时看这个逻辑，就很奇怪，怎么直接就return r3了，都没看到进入后面的逻辑啊

![](https://img-blog.csdnimg.cn/img_convert/d0d389c11ed2e7235373127f8b0e8502.png)

按理应该0009走了之后会走0016，但是并没有，奇怪

![](https://img-blog.csdnimg.cn/img_convert/b1073c5601d0a3ee2dca277dbc307e8b.png)

这个不管了， 应该是脱壳出来的效果不太好

那就用objection看看实际的方法体吧，不是说objection 对于java层来说简直是嘎嘎乱杀吗\?

![](https://img-blog.csdnimg.cn/img_convert/0d65386e99d7e99d4890ff06870dfb16.png)

结果看到没有，卧槽，这应该是加固过的原因，这就有点意思了啊

行，因为有个sign\_type已经给了提示是md5，接着再看刚才那个方法体，其实就是加了盐的md5

![](https://img-blog.csdnimg.cn/img_convert/afa9745c96e6aed3c22e12c218fd81eb.png)

但为了验证一下，还是hook 这个xxxx.md5看看，变成了啥

![](https://img-blog.csdnimg.cn/img_convert/74f8b2cac7bf0bb1bae443f5ad050ac3.png)

果然了，那至于这个代码逻辑为啥看着有点怪，不纠结了，能搞定问题就行了。没想到这么简单，嘻嘻嘻

## python还原

还原代码我就不贴了吧，就是加盐的md5，from hashlib import md5就搞定的事，就不多逼逼了

## 结语

这个app对于很多朋友来说简直分分钟，可能直接都懒得写hook代码，看源码都能看懂了，大佬应该都不屑于看。我发出来的想法是，记录过去，憧憬未来，最近一直在研究app，突然想起了它，同时也刚好有了能拿下它的能力，嘿嘿

另外，因为某些原因，我暂时没有做安全了，所以涉及到诈X的事，哥哥们不用找我了，我也没精力管了，遇到问题可以打报警电话的，可以装个国家反诈中心app（我之前呆过的某安全公司参与的项目）

下期再见！