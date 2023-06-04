## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[某浇油app系列2-安全分析](https://mp.weixin.qq.com/s/f9QwItIPSRVg0A38NzzLiw "某浇油app系列2-安全分析")

## 前言

废话不多说，继续浇油app系列分析，也是某短视频app上的

图片发不出来

## 分析

下面这个是注册包，这注册登录界面咋看着像上一篇的app呀

![](https://img-blog.csdnimg.cn/img_convert/b2ba5414e27d50795dc072ef29878948.png)

登录包：

![](https://img-blog.csdnimg.cn/img_convert/9fba40ae5781c8b92825904725d963b9.png)

注册完了进入主页面，界面确实感觉跟系列1很像

![](https://img-blog.csdnimg.cn/img_convert/1b93b097bae3c7d80d10a11716bd423c.png)

返回没有加密，请求看着感觉挺简单的，就url里的几个加密参数了

### 1.开始分析url的加密参数

分析发现，请求头里我去掉任何一个参数，都会提示是签名验证失败

![](https://img-blog.csdnimg.cn/img_convert/0a52fea03ea53cff3e46c437ed3c4e53.png)

这种搞多了你就知道，就是用的整个请求头的hashmap作为原始字符加密，然后带上加密参数一起请求的，某宝就是这样的，你懂的，不展开说了

ok，开始代码分析

发现有数字壳，一顿操作后，一搜索其中一个关键词，我擦，这么简单？

![](https://img-blog.csdnimg.cn/img_convert/3b96b0b951cd726c5e992f78568ca7f0.png)

准备用frida 跟下：

![](https://img-blog.csdnimg.cn/img_convert/a08705a766c18a8266356004481fabde.png)

![](https://img-blog.csdnimg.cn/img_convert/ea2215fd63a3e3a5de6e6a426aca362e.png)

![](https://img-blog.csdnimg.cn/img_convert/2fd1dd5eb857830bfceefa4dfa1cd154.png)

有对抗，刺激，终于不是那种很简单的了，突然有点兴趣，顺便说一句，我这个frida已经是魔改过的版本了，还能检测到，还是有点东西

（有关这个反调试对抗的，可以私信我，给你推荐一个大佬，是这方面的专家）

现在去做反调试分析已经来不及了，换用xposed，这里我选用lsposed

### 1.搭建xsposed环境

怎么配置lsposed到手机上我就不说了，网上资源太多了，之前也有大概的提过

打开as后，新建一个项目，选择empty activity之后，开始如下配置：

1.1. 配置xml

```
<?xml version="1.0" encoding="utf-8"?>
```

![](https://img-blog.csdnimg.cn/img_convert/e936776267adc04f2970c6de15f5f7fd.png)

1.2. 在gradle依赖里添加如下

![](https://img-blog.csdnimg.cn/img_convert/627a98701b43eb7e766cf9df2c40fa70.png)

 -    
 -    

```
compileOnly 'de.robv.android.xposed:api:82'
```

添加完，点右上角的sync now，等待一会儿

![](https://img-blog.csdnimg.cn/img_convert/b5e54b080a017f8889a9689e10beb0a1.png)

1.3. 邮件app项目，创建一个assets

![](https://img-blog.csdnimg.cn/img_convert/48640889f48d12f7b817c938eabb8d70.png)

1.4. assets文件夹，添加文件xposed\_init

![](https://img-blog.csdnimg.cn/img_convert/48249460b0bfff9b90dddd8635c3d876.png)

这个文件名固定为【xposed\_init】，表示是xposed的入口文件，内容写上你的自己的类名（这个类名就是自己编写的hook类，必须保证类名一致）

也可以编写完hook类后再执行此步，我习惯先执行这一步，免得后面开发完忘记这一步

![](https://img-blog.csdnimg.cn/img_convert/3cc21662fa8c17c13fbb699aa32abb06.png)

### 2.编写hook逻辑

在java文件夹，右键，创建 【Java class】，名字随意，但要跟xposed\_init的一致，不然无法进入hook逻辑

![](https://img-blog.csdnimg.cn/img_convert/a77f079b2323bfa5b6b25d28cfb3dde5.png)

![](https://img-blog.csdnimg.cn/img_convert/9e9d3cf51a81bac28863f1bed9f0cffa.png)

创建完就如上，后面写上implements IX  tab键补充选如下

![](https://img-blog.csdnimg.cn/img_convert/664dccbcc56628b2d467b220b786191d.png)

然后放到标红的地方，alt+enter键，选择补齐方法：

![](https://img-blog.csdnimg.cn/img_convert/0da90b9693d15174af05386827e199ba.png)

![](https://img-blog.csdnimg.cn/img_convert/8aaeccd923b2cfaff878cc09ed6cb26b.png)

然后就会自动出现如下

![](https://img-blog.csdnimg.cn/img_convert/f32f3d23fbc6fe8d37a4623a33e0c14c.png)

再写个if逻辑，只对目标app执行hook操作：

![](https://img-blog.csdnimg.cn/img_convert/eca12fb892b66c0655ddb4ac02943b22.png)

里面的就是你的逻辑代码了

因为这个app是有壳的，所以要先用下面的代码，找到实际的classloader

![](https://img-blog.csdnimg.cn/img_convert/95f4a47782fc18b82e64d59c7629c0c3.png)

```
Class<?> ActivityThread = XposedHelpers.findClass("android.app.ActivityThread", lpparam.classLoader);
```

才能进行hook逻辑

好的，环境整好了

### 3.在搭建环境的同时

unpack完看就是这些，卧槽，这个混淆有点东西啊

![](https://img-blog.csdnimg.cn/img_convert/b5b278ffd64fcfc5135cb62d122f6e97.png)

### 4.开始分析

还是刚才搜到的这里 ，

![](https://img-blog.csdnimg.cn/img_convert/5f42c03576e8ad92094d8ca09d0edd38.png)

根据调试，发现并没有走这个逻辑。有点某宝的意思啊，直接能搜到，静态看代码逻辑感觉也很像，一hook发现并没有走到这里

还是看下抓包工具的结果

![](https://img-blog.csdnimg.cn/img_convert/5e9b34a27967292fa2e56ff80742d1e4.png)

翻了几页，发现那几个加密参数就没变过，但也不敢托大，我又搜了下url的，因为这几个参数是跟url拼接的，搜【user\_search】

最后跟逻辑跟到这里：

![](https://img-blog.csdnimg.cn/img_convert/e201d58314b289afa05e5f1885ab49bb.png)

### 5.写hook代码

先找到这个两个类：

![](https://img-blog.csdnimg.cn/img_convert/ba0747348c0e7b4e27ee9cfe460a89a6.png)

hook那两个类：

![](https://img-blog.csdnimg.cn/img_convert/cc57506a3f17880051357b011ba02fbf.png)

![](https://img-blog.csdnimg.cn/img_convert/0848b24e61bfbe9c7387eb88a291e036.png)

安装到手机上，打开lsposed，点下这个

![](https://img-blog.csdnimg.cn/img_convert/517c5d85ce72b8eebd525ad45cc2ae98.png)

![](https://img-blog.csdnimg.cn/img_convert/ae3a15ff187c63fb53ea5224b8e64a01.png)

然后杀死目标app，再启动目标app，

as里的日志，选到你安装的手机，然后我习惯这么配置：

![](https://img-blog.csdnimg.cn/img_convert/17fd215e83d0f55b2f54231f308d3406.png)

app往下拉，翻个页，日志就打印出来了，下面这个url就是我们的目标url

![](https://img-blog.csdnimg.cn/img_convert/daf5f21e7e8fbbcded70d9c89d9c174d.png)

但是他还没拼接好，说明那几个加密参数还在后面的逻辑，继续看：

跳了好几层get，然后到这里

![](https://img-blog.csdnimg.cn/img_convert/0802257d7567089c0920c4829dcf6d9c.png)

再借助lsposed，发现，实际的拼接在这里，

![](https://img-blog.csdnimg.cn/img_convert/c4456ec32d31cb4aa9548533676b3e15.png)

![](https://img-blog.csdnimg.cn/img_convert/2a237f6ce9e65d2b3064eaad5b670455.png)

hook的日志显示，就是这个checkrequest方法做了url拼接，所以逻辑就是这里了

![](https://img-blog.csdnimg.cn/img_convert/c8df68efc95023e281640faaf382dc3f.png)

刺激

再看这个方法的参数：

![](https://img-blog.csdnimg.cn/img_convert/433ed3e5eff8f959b48809ee3ff147f0.png)

我感觉这个caller的对象嫌疑很大，光标放上去，看到他就调用了两次自己的方法，直接对这两个方法hook下：

![](https://img-blog.csdnimg.cn/img_convert/4f4bdb66cf00c1e544c1a3a5f780d193.png)

![](https://img-blog.csdnimg.cn/img_convert/5ba4fc1ed160c7027200a158e32ca96c.png)

对比下，这个不太好看

![](https://img-blog.csdnimg.cn/img_convert/a5b2bb02b26dd3de245a10e6003ab846.png)

放到文本编辑器里，对比，

![](https://img-blog.csdnimg.cn/img_convert/3d8167353dcdd3a5a9fa620eb5bd5553.png)

其他都有了，就最后4个没有，那这加密还在后面的逻辑

打下调用栈看看：

```
Log.e(TAG, "Stack:", new Throwable("Stack dump"));
```

也没发现啥有用的

![](https://img-blog.csdnimg.cn/img_convert/2ab33170789121a63d81107025636d7c.png)

这就尴尬了，继续看逻辑，光标点中【url】，看跟url有关的操作，发现了如下这个：

![](https://img-blog.csdnimg.cn/img_convert/e479ad427dbcb905d71f5b34a6e49ed5.png)

感觉有点可疑，追进去：

![](https://img-blog.csdnimg.cn/img_convert/ca385864065ed91d4824d2033578625c.png)

卧槽，进到了native，hook验证下，果然是了：

![](https://img-blog.csdnimg.cn/img_convert/e7d81546cdd251184b58289523635605.png)

把它拿出来看看，果然是，传的第二个参数是没有加密的url，返回就是有加密的url了

![](https://img-blog.csdnimg.cn/img_convert/94704580900d0c85e92ece7f80a45e67.png)

终于遇到个加密在native层的浇油app了，

### 6.native层分析

ok，进入native层分析了，根据app展示的

![](https://img-blog.csdnimg.cn/img_convert/ea3bd3c9d4127565f4e0f08cf9a4d93c.png)

打开ida小姐姐，拖进libutil.so，很快就找到这个方法，静态注册的，挺好

![](https://img-blog.csdnimg.cn/img_convert/f03045f1d5616e4d5f825ffc0f88736a.png)

这个是ts:

![](https://img-blog.csdnimg.cn/img_convert/6cef95b89261cb93fe41a5b5eeebb6ab.png)

这个是ckey，dno

![](https://img-blog.csdnimg.cn/img_convert/737ac177e93f6c023646657f1599eb9f.png)

这里是h:

![](https://img-blog.csdnimg.cn/img_convert/c65e860acac298c5b59924e118cef8c3.png)

先按下键盘的反斜杠【\\】，看着好点了：

![](https://img-blog.csdnimg.cn/img_convert/31c434be548176ec59cff4296a278c0e.png)

再把参数改下，光标选中a1，按键盘【y】,改为【JNIEnv\*】

![](https://img-blog.csdnimg.cn/img_convert/506afe94632f8fa7d99dbb21b33c89d9.png)

选择ok之后，直接就看到ts的生成了，虽然猜到了是时间戳，但是有这个就100\%确定了：

![](https://img-blog.csdnimg.cn/img_convert/0749d189f1ba60e3714ef468b327957d.png)

ckey也基本肉眼可见：

![](https://img-blog.csdnimg.cn/img_convert/189cccfe9e503393601ba4e19f5087e8.png)

后面的几个也差不多，就省略了。

刺激！！！

我觉得挺简单的，为什么这么说，因为每次我打开，生成的加密参数都是固定的，也就是我如果为了拿数据的话，完全可以写死。

具体的算法还原细节，这里就不展开了，感兴趣的可以去看看白龙，龙哥的文章的算法还原部分。

## 验证

这都不用怀疑，肯定是可以的

![](https://img-blog.csdnimg.cn/img_convert/3a1c28551d86e7464085479558020677.png)

## 结语

好啦，今天的系列完成，我截图的相册里还有好几张，换句话，后面还有好几个浇油app案例

**geekbyte**