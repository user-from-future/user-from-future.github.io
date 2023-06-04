## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[某浇油app的请求参数分析](https://mp.weixin.qq.com/s/hhBpUBDSCM4noeMLxiFo0A "某浇油app的请求参数分析")

## 前言

最近在研究app，总还是想有点输出不是？所以这次的案例就是某社-交app，

包名：Y29\{deleteme\}tL\{deleteme\}mJha\{deleteme\}Whl

难度程度我觉得算简单，不过还是有点意思，值得记录一下

## 环境准备

- 一台已root的真机，我选用的n6p，最好选谷歌亲儿子，我自己有台红米k20，虽然已经解锁+root，但是容易出现各种问题，浪费很多时间去调试

- postern抓包工具，详细的配置就不说了，网上搜

- 一台装有frida，jadx，尿壶的电脑

- 上述说的app，版本号，11.4

由于我的手机系统是安卓8，所以我就下载的frida-server 12.8.0，对应的python库也是12.8.0，安装：

> pip3 install frida==12.8.0   frida-tools==5.3.0

## 分析

事先说明下，这个app需要登录，所以如果想跟着搞一波的，你可能得注册一个账号。我是很早就因为某原因就注册了的（真不是各位哥哥姐姐想的那啥，当时就是在做某某项目才注册的，至今记得同事小姐姐看到我手机上装了有这种软件后那种嫌弃的表情，真不是为了那啥的.......）

电脑打开尿壶，手机配置好postern开始抓包

发现，有多个接口

![](https://img-blog.csdnimg.cn/img_convert/cd956f7523d28677e08a8648f860cfbc.png)

只有这个接口才是真的数据接口，然后看到是post请求，带了4个请求参数

![](https://img-blog.csdnimg.cn/img_convert/269c2b2ec4a9386a952fda68332c6d0f.png)

返回数据就是我们要的小姐姐了，呸，（手动撤回），返回数据就是我们要的就是数据了。可以用 https://spidertools.cn/#/formatJSON 这个网站做json数据处理

![](https://img-blog.csdnimg.cn/img_convert/afa82544ab1bc8082fc06e78281ed86e.png)

然后多次翻页，多次刷新，发现这四个参数基本固定存在，有部分接口的4个参数对应的值不太一样而已。

先看第一个，userID不说了，就是我的用户id，后面就是些参数，然后有个接口会带有经纬度的参数，当然这里的是没有的

![](https://img-blog.csdnimg.cn/img_convert/6f4af94b1fd9dc830aceba324725814f.png)

再看第二个，感觉【\_】左边的就是时间戳了，右边的估计就是加密字段，而且相同的接口每次请求都不一样，那盲猜是带了有时间戳作为加密。

![](https://img-blog.csdnimg.cn/img_convert/97ed8135528cfe875f4fb3500c3973d4.png)

说实话哈，可能有经验的老哥哥看到【\_】右边的字符串，应该会很敏感地拿去算下长度：

![](https://img-blog.csdnimg.cn/img_convert/30df2e48a68c84c20edc0e3c4264de23.png)

32位，盲猜一波这个算法很可能是md5，这个搞得多了的老哥可能会有这种意识

再看第三个，这个参数就有点长了，但是仔细看，就是带了很多的设备参数，有些敏感的我就打码了，这个参数基本不变，每个请求接口都是固定的（那肯定啊，你设备参数还能变不成对吧？）

![](https://img-blog.csdnimg.cn/img_convert/eab1d94e5c1adc2becf60f4bfb76c4fe.png)

再看最后一个，我打码的地方是固定的，因为带有了这个app的特征。前面那个依旧是时间戳字段，后面的就是一些加密字段了。

![](https://img-blog.csdnimg.cn/img_convert/dd0956d8e69a651056aeef148523aab7.png)

一般这种参数，按常规的app防护手段，都会用设备参数在本地加密后去服务端注册，等待风控检测，风控过了之后会返回一个值，说实话我看着这个stk就很像这种东西，这里先不表，后面再说

OK，总结下，四个参数里，要搞定的其实就是第二个\_t和第四个stk，下面开始分析参数生成

## 参数定位

打开jadx，先搜一波：

![](https://img-blog.csdnimg.cn/img_convert/099190634ae6116c6fb0e047a06954a4.png)

直接点第一个进去，卧槽？这么简单，4个参数都在

![](https://img-blog.csdnimg.cn/img_convert/cf2783ab9edfb15974202d09030ba63c.png)

再看后面的：

![](https://img-blog.csdnimg.cn/img_convert/4373b37219e3a739ea1840dbdef9773f.png)

根据我的分析发现，基本确定就是我刚才进的第一个了，那就先看\_t吧，先看上面的，这不就是第三个参数里的字段吗哈哈哈哈，这里可以不用管，反正基本是固定的，再看我已经标注出来的，1和2的位置

![](https://img-blog.csdnimg.cn/img_convert/e6da2bd936e179a50a4308718c0d7477.png)

反正意思就是\_t有两种，如果获取到了第一个参数，就加进去加密，否则则直接加密。

然后再看最后一个参数stk，基本是一个写法。

![](https://img-blog.csdnimg.cn/img_convert/e3966eed2d01a0fa6c542acaf02dd191.png)

好的，位置是找到了，开始写hook代码

## hook调试

手机连上电脑，adb shell进入手机终端，然后启动frida，怎么选用手机对应的frida版本在之前的文章有： [app安全 \- 环境准备](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484633&idx=1&sn=303ea868f73fa0f309724379b65051a2&chksm=fb18f57bcc6f7c6db0dcb75461969472c03923fe72460472980779c35cbf6a7cc6a5e9678af5&scene=21#wechat_redirect "app安全 \- 环境准备") ，这里就不细说了。

好执行后，只要不报错就是启动成了

![](https://img-blog.csdnimg.cn/img_convert/4bd0a87e87720eeeca2fcef7032bcafc.png)

### 先看看\_t参数怎么来的

继续看看jadx的逻辑，读逻辑知道，是用的j.a做的加密

![](https://img-blog.csdnimg.cn/img_convert/d553c14d6299aeaebb56296a250ad62d.png)

点进j.a看看，卧槽，这不就是md5吗，但是感觉是他自己稍微做了魔改的，那无所谓的，反正能用就行了

![](https://img-blog.csdnimg.cn/img_convert/fd30d3b5a2f81e720d7db6ad1c2e5c40.png)

开始hook 这个j.a之前，hook脚本编写，这里我用的pycharm，说实话个人还是用不惯vscode，还是习惯pycharm

![](https://img-blog.csdnimg.cn/img_convert/609d95177be4a1cff7e26548e616690c.png)

终端执行，选用的attach模式进入，好的，没报错，说明附加上了

![](https://img-blog.csdnimg.cn/img_convert/bab52d03b64eee3453d5a8df522d195a.png)

现在app还是打开状态，此时先把尿壶里的抓包记录清理一下：

![](https://img-blog.csdnimg.cn/img_convert/92056179752cde40ce95ec51c827d9d0.png)

接着就是做各种的刷新和翻页的操作，看终端会不会有输出，感觉这个函数是被掉了两次的

![](https://img-blog.csdnimg.cn/img_convert/af6297edc4ae1e269859db6c21101c51.png)

再看尿壶这边的抓包记录：

这是第一个

![](https://img-blog.csdnimg.cn/img_convert/82731a6c4ab51c11c4e0bfe58d6aeafd.png)

明显感觉这个返回值跟实际请求的\_t后面部分对上了

![](https://img-blog.csdnimg.cn/img_convert/74c346bc766759770b6ac194d7c270f5.png)

再看第二个：

![](https://img-blog.csdnimg.cn/img_convert/d5c7f4fc8d34afc631b0b0f25d9443d0.png)

完毕，既然这里就是加密的，那看看加密前后到底发生了啥，仔细看可以发现，其实，每请求一个借口的时候，其实调了两次j.a加密，

![](https://img-blog.csdnimg.cn/img_convert/cf85ca75e32a5715970a5bf1931729ac.png)

那基本确定他其实走的上面这个逻辑，那肯定啊，因为能获取到p，

![](https://img-blog.csdnimg.cn/img_convert/b02e618781ae8f1c37950199d51d7b49.png)

整理一下，也就是这样的逻辑：

![](https://img-blog.csdnimg.cn/img_convert/9aaa86a2e3ee0a5ee3f4bd175dabf2ec.png)

但还差这最后一段，乍一看不知道怎么拆

![](https://img-blog.csdnimg.cn/img_convert/f97142dc6283de53e40161aeabe6da66.png)

先看源码，

![](https://img-blog.csdnimg.cn/img_convert/10d9623f90c4c93fa9b4e75f4fd4255b.png)

因为a2就是p的值加密后的，所以直接用加密后的p做切割：

![](https://img-blog.csdnimg.cn/img_convert/24e2ec76e232d3ffb716af5ac66b5f23.png)

![](https://img-blog.csdnimg.cn/img_convert/5cf2de2767abaef49e0b0b67fc70827b.png)

再用1645630067确认是不是时间戳，果然是

![](https://img-blog.csdnimg.cn/img_convert/04acb8c9b09e3e44aba5ab49c21f60d9.png)

ok，\_t参数搞定。

### 再看stk参数怎么来的

![](https://img-blog.csdnimg.cn/img_convert/c2fae5ed0e6be980dd90788502b9a469.png)

c.a\(\)点进去，这里synchronized就是做线程保护的，防止数据被影响，

![](https://img-blog.csdnimg.cn/img_convert/1d991332cc5185bc14f5198d35055c0c.png)

再看.b

![](https://img-blog.csdnimg.cn/img_convert/63f68852c7ecc6fca24149cca3ca0739.png)

接着跟

![](https://img-blog.csdnimg.cn/img_convert/8a215b2c4202b2aedbfff853bdcdcb53.png)

![](https://img-blog.csdnimg.cn/img_convert/8e9ee3600a566252db5766a1ab390652.png)

跟到这里的时候，继续点：

![](https://img-blog.csdnimg.cn/img_convert/b9f819e395e9e91e3c88ab2401cb027c.png)

继续跟进去，看到这个，卧槽，完蛋，native

![](https://img-blog.csdnimg.cn/img_convert/64c1904f70f265ae5d5676632764f826.png)

看来得打开ida搞so层了，不过在打开so之前，我还是做了java层hook验证，有overload

![](https://img-blog.csdnimg.cn/img_convert/ccf134fd8fb59b45e51677afdc2bc8dc.png)

这里，我有种感觉哈，我感觉得都hook下

![](https://img-blog.csdnimg.cn/img_convert/312c6a9390ec4a4e9724492aca480882.png)

接着手机拿着各个地方都去点了一下，终端有结果了：发现并不是我们要的数据

![](https://img-blog.csdnimg.cn/img_convert/6b2d1b467d3acd485c26eb187abd71c9.png)

那就spawn模式吧

![](https://img-blog.csdnimg.cn/img_convert/8451830a36133231d05580079b162992.png)

值是有了，结果发现app启动之后，居然需要重新登陆了，我启动多次都这样，每次启动全都需要重新登录，而且登录的时候开始有了某验的验证。。。。卧槽，有风控检测？？？？

头皮发麻呀，

然后，我试着去hook这个方法：

![](https://img-blog.csdnimg.cn/img_convert/63f32b1be527a81fdd2fa82b2ef6f350.png)

反正就是hook不到，提示不存在这个类

![](https://img-blog.csdnimg.cn/img_convert/20b71acc976311869ceeb361a884ee96.png)

有点迷茫了

我看到这个报错，突然想起了frida hook的另一种Java.choose，继续刚才那个类的hook，就不用java.use\('xxx'\)了

![](https://img-blog.csdnimg.cn/img_convert/14e8e1c3dd0fb21614c407988e76b611.png)

重新换回attach模式，执行，一下就有了，卧槽，可以

![](https://img-blog.csdnimg.cn/img_convert/7a04278c92432e454be2ced3779b6383.png)

我继续看抓包工具的结果，然后，特么的无意间看到这个接口：这个stk是登录之后返回的.......

![](https://img-blog.csdnimg.cn/img_convert/f7c0290970192ac5b1bac26d200abd09.png)

作为验证，我拿着这个时间戳去解析，这时间还真的是我刚才登录的时间。。。。

![](https://img-blog.csdnimg.cn/img_convert/0ffc3c0b41ea5f5ea94e63d413aa0619.png)

说实话我真没想到直接就是接口就返回了，而且他的登录接口也没啥东西，是我多虑了，哈哈哈哈哈哈。那行，over了。

唉，虚惊一场，是我想多了，还得是那句，先走基本流程在抓包工具里搜一下看是不是直接返回的再说，其实我也有搜的，搜的时候看到结果太多了，随便展开几个都是在request里，我就以为是在本地生成的我就没细看了

![](https://img-blog.csdnimg.cn/img_convert/cc1f62100114022086610d75bbe2f11a.png)

这个app很简单，按正常流程，包括写加密算法还原，应该一二十分钟就能搞定的，只是我想多了，还好刚才没有打开ida，要不然错得老远了，哈哈哈哈哈哈

## 算法还原

按照正常流程，就开始用python仿照下面这个写个算法，然后代码执行，能拿数据就完了

![](https://img-blog.csdnimg.cn/img_convert/f78c20b7d1d72933dccac5d88e8cb294.png)

但是这里，我就暂时不给还原的代码，都很简单，就照着写就完了（其实我根本没去还原，嘻嘻嘻）

## 结语

ok，本次分享完毕，下一篇不出意外应该也是个app的案例

要加群或者需要问题探讨或者其他问题，都可以加我微信： **geekbye**