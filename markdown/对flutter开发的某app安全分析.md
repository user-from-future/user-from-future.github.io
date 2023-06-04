## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[对flutter开发的某app分析](https://mp.weixin.qq.com/s/pXpfXK-Ez0n70f3bqFuuFg "对flutter开发的某app分析")

## 前言

最近总感觉时间不够用，很多东西都堆着，答应给朋友看的某app，也没来得及看。所以也就一直没有空发文章，当然我也承诺过，尽量保证文章的质量，所以不能随便水文章，要嘛就不发，要嘛就发质量我觉得过得去的。

这不刚好有个app，可以记录一下

app名：

```
dGVj{deleteme}aC5lY{deleteme}2hvaW5nLm{deleteme}t1cmls
```

## 开始抓包

一开始用charles链接wifi代理去抓包这个app，发现只能抓到一点点无关紧要的包，需要的数据接口是抓不到的：

抓包工具显示的是connect:

![](https://img-blog.csdnimg.cn/img_convert/514538d051b5995208810c055267f062.png)

app显示界面也是空的：

![](https://img-blog.csdnimg.cn/img_convert/f3cb829d6c802cf38ce5911fda9dc455.png)

所以以为APP可能设置了no\_proxy，使用postern开启VPN，因为VPN抓包的方式可以解决no\_proxy的问题，但结果toast提示【null】，以为APP开启了双向验证，逐一去尝试解决，情况跟上面一样，都没有抓到包。

那么用r0capture抓，也没有，很骚啊

抓不到包，感觉直接一头雾水啊  

尝试hook打印堆栈，突然看到了flutter的字样，才意识到这有可能是用flutter开发的APP。

## 什么是flutter

Flutter是Google使用Dart语言开发的移动应用开发框架，使用一套Dart代码就能快速构建高性能、高保真的iOS和Android应用程序。

由于Dart使用Mozilla的NSS库生成并编译自己的Keystore，导致我们就不能通过将代理CA添加到系统CA存储来绕过SSL验证。

flutter打包的apk，会把核心逻辑放在so层，且SSL Pining也在Native层，这就导致没法抓包

## 怎么判定app是flutter开发的

把apk包复制一份，后缀改为zip，然后解压，进入lib目录，如果看到有libflutter.so，那就是flutter开发的了，否则则不是

而今天这个app，查看，确实是flutter了

![](https://img-blog.csdnimg.cn/img_convert/0fccd2072d123a2caf9d8f39b265d26a.png)

  

## 反抓包分析

为了解决这个问题，就必须要研究libflutter.so了

其实有关flutter抓包的问题，我2020年就遇到了，下面是肉师傅的知识星球，这个提问的不语就是我，（通过我发的文字，就可以看出我当时稚嫩的样子啊，真的是个没啥安全能力的菜逼，虽然现在也菜的抠脚。而且当时还以为肉师傅是女的，看好多人都这么称呼他）

![](https://img-blog.csdnimg.cn/img_convert/654e50e6d8d71ac8cfdfb320a942ca8f.png)

所以，时隔一年多，终于又遇到flutter了，出来混，迟早要还的，所以这里正好记录一下，算是对之前还债了。

**当然我搞得这个app，不是很难那种**

一开始是在看雪里找到这个帖子：

https://bbs.pediy.com/thread-261941.htm

然后跟着操作了一下，在打开app的包文件，用IDA去打开libflutter.so（注意32位与64位）

![](https://img-blog.csdnimg.cn/img_convert/1c9520703f29069b5e8ef256cda553b9.png)

然后点search->text

![](https://img-blog.csdnimg.cn/img_convert/0725c906b680635b74ddf212500346b8.png)

输入ssl\_client，

![](https://img-blog.csdnimg.cn/img_convert/94f9949ec8856c9eedb9e32c1dc207c0.png)

点ok，等一会儿，找到有【Rx,PC ："ssl\_client"】之类的字眼

![](https://img-blog.csdnimg.cn/img_convert/addceb8452d4fd77994599e3957ae27d.png)

  

点进去，进入这里：

![](https://img-blog.csdnimg.cn/img_convert/1dca1b89849eb97a17636c9ef1e405b0.png)

按下空格键

![](https://img-blog.csdnimg.cn/img_convert/ed26da4f273fc4d108c78629c791101f.png)

这时候，点ida里的菜单栏，options->General，把这个由0改成4（这个步骤我找了很久，请教了奋飞大佬找到的，ida操作不熟没办法）

![](https://img-blog.csdnimg.cn/img_convert/9b96f055d07ed1111a2a48f899bc6667.png)

Number of opcode bytes设置为4

![](https://img-blog.csdnimg.cn/img_convert/e82c3601d0632e63ab8ebdf42daa2780.png)

点ok，现在再看，已经多了点东西了

![](https://img-blog.csdnimg.cn/img_convert/d90f9ec43dc732ad6a0ef86a822cdc8b.png)

然后当前位置往上找，找到有\_unwind开头的，停下来，从\_unwind开始，拿到字符串前面10个字符（也可以大于10），照这个前10个字符，我尝试好久才知道是要从这个unwind开始拿字符串，而不是在找到ssl\_client位置开始拿前10个字符串，上面看雪那个帖子也没说清楚（也可能是我菜，没读懂）。

![](https://img-blog.csdnimg.cn/img_convert/542ff1649c92de0d3202422270110e7d.png)

  

用那几个字符串，加上flutter的hook脚本，配合drony或者postern就可以抓包了（亲测过，postern一样可以抓，配置起来没有drony繁琐）：

![](https://img-blog.csdnimg.cn/img_convert/0f3f2ff0baa2db7f58bbefbdcbcb461a.png)

用frida hook attach模式启动，同时重新刷新页面

![](https://img-blog.csdnimg.cn/img_convert/1b9d7c18fb43b6814130dd06965b9e0a.png)

  

果然抓到了包，而且app页面也正常显示数据，不好意思打的码有点厚，不过不重要，这种结果就是有数据的意思

![](https://img-blog.csdnimg.cn/img_convert/1e1105f84048a21c3157cbae8101a1f3.png)

ok，抓包搞定了，现在就看看有没有啥加密参数了

## 加密参数分析

再看刚才的接口，就一个加密字段sign:

![](https://img-blog.csdnimg.cn/img_convert/856d9b462337d441bef3a5639220ec41.png)

所以，找找这个怎么生成的即可，打开jadx，搜索：

![](https://img-blog.csdnimg.cn/img_convert/9d41013e3d8570085b68047dfbafd34d.png)

就两个地方可疑，点进去：

第一个

![](https://img-blog.csdnimg.cn/img_convert/35b2adc920af981028b04f3b8171f25e.png)

第二个：

![](https://img-blog.csdnimg.cn/img_convert/b2abc39b3078b6782b6592b55b1326ed.png)

而实际上，第一个里返回的最后也调用了第二个里的，所以实际就是第二个了，写个脚本对它进行hook操作：

![](https://img-blog.csdnimg.cn/img_convert/eb10629a9c17cac2b3dfc3a3ee7bc78b.png)

看结果，对照下抓包工具里的结果：

![](https://img-blog.csdnimg.cn/img_convert/129fb3c336918e1b5354b344cb9ee9a0.png)

有点长，我复制出来对比下，一模一样，那就是这里了

![](https://img-blog.csdnimg.cn/img_convert/8deeba00ec24a95e72b6d278117651d3.png)

相信你发现了，这后续流程跟常规的app分析没有任何区别，那是因为这个app并不难，如果是那种所有逻辑都在so层，就难搞了，这里的加密逻辑还是在java层，只是一开始的抓包就把部分朋友难住了

不废话，继续看，这个看来就是个rsa加密了，当我正要分析的时候，程序意外崩溃了，这个不要紧，估计没有返回正常的值导致的，那行，既然找到就这个位置了

打印堆栈跟下加密逻辑，顺便看到了加密的privateKey:

![](https://img-blog.csdnimg.cn/img_convert/15850a99f7f862a4605667dd6dc4c247.png)

而其实刚才那段源码里，这个key其实就在了  

![](https://img-blog.csdnimg.cn/img_convert/429f318b47c8ff29bc58ec11b2c87dd6.png)

接下来就找传进去的第一个参数了，回到刚才的逻辑，第一个参数是这个，

![](https://img-blog.csdnimg.cn/img_convert/ae9066012bd808b97af33c16382150a0.png)

直接对这个方法进行hook:

![](https://img-blog.csdnimg.cn/img_convert/3d1f8cf0d0cab83438bba224ff506fc8.png)

这str不是很眼熟吗，就是一个url啊

![](https://img-blog.csdnimg.cn/img_convert/36ac3316730644b9c505fc851ce9efe4.png)

那这个基本也稳了，不过注意的是，有的url里面的参数里就多了个开始时间，结束时间，和当前时间

## python代码还原+验证

![](https://img-blog.csdnimg.cn/img_convert/86ebbc90093717abf4798e416601144a.png)

hook的结果：

![](https://img-blog.csdnimg.cn/img_convert/f4a1b7a01c29bfe02dc1a5be9fc45de7.png)

对比下，完美，一模一样

![](https://img-blog.csdnimg.cn/img_convert/fdc279f1a7c4bea7b5fc8965faed5505.png)

然后这个接口有的是带了时间的，有时间的加密结果也一样，我就不贴图了

python代码，（感谢成都天团里颜值最高的徐少给的代码），我就不自己重写了

```python
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Hash import SHA256
import base64
import warnings
warnings.filterwarnings("ignore")

def RSA_sign(data, privateKey):
    private_keyBytes = base64.b64decode(privateKey)
    priKey = RSA.importKey(private_keyBytes)
    signer = PKCS1_v1_5.new(priKey,)
    hash_obj = SHA256.new(data.encode('utf-8'))
    signature = base64.b64encode(signer.sign(hash_obj))
    return signature


def main(startTime=None, endTime=None):
    data = '/cactus-api/posts/by-tagtagIds%5B%5D=4&orderBy=updatedAt&offset=10&limit=101648403473200'
    privateKey = '''自己用jadx打开放到这里吧，太长了，占篇幅'''
    res_sign1 = RSA_sign(data, privateKey)
    signature = res_sign1.decode('utf8')
    print(signature)

if __name__ == '__main__':
    startTime = 1648051200 # 有时间的接口这么用
    endTime = 1648137599 # 有时间的接口这么用
    main(startTime, endTime)
```

## 结语

终于算是对flutter有个大概的安全分析操作流程了，其实本次目标并不难，如果是那种很难的就难搞了。这个app其实还有某盾加固，但由于不是一线大厂加固，我直接忽略加固一样操作

那遇到那种难的，把大部分的逻辑都直接编译在so层的，怎么办？dart反编译吧，但是反编译的效果不是很好，目前没法通杀，只能多方尝试了

更多的flutter资料，以下资料链接源于肉师傅和葫芦娃的知识星球，还有微信好友\@奋飞、\@岂可修

## 引用

我原文引用的内容太多了，容易什么版权不明，这里就不发了（尴尬）