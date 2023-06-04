## 声明

  
以下只是搬运下我公众号的东西。很早就发过了。原帖地址：

[某禁止截屏的学穿搭app安全分析](https://mp.weixin.qq.com/s/el5LEOKR88sNa3bgUQ2jEQ "某禁止截屏的学穿搭app安全分析")

已经发公众号的为什么还发csdn  
有的圈内朋友，不经过我的允许，删减摘录我公众号的内容，这里就不提谁了，心里清楚，还能获得一些关注和流量。很无语，所以我还不如自己也发发。【猛男落泪】

## 前言

兴趣使然，发现了某app，有点意思，所以记录下来

## 分析

### 1.抓包

首先就是打开这个学习穿搭的app，页面展示如下，为了防止不必要的事呢，我就把脸打码了

![](https://img-blog.csdnimg.cn/img_convert/062fad21b892cfd7b5127bb7d32c213b.png)

里面有很多知识点，感兴趣、喜欢的朋友可以自己去学习穿搭

![](https://img-blog.csdnimg.cn/img_convert/4382e11ec8b0d44018602ee9de1c8b14.png)

回到正题，刷下页面，抓个包，看看加密参数，

  

![](https://img-blog.csdnimg.cn/img_convert/598ebf24c8a3e6facdf9862d40fbf4b6.png)

加密的参数就是标注的几个了，token是登录返回的，就不用去研究了

### 2.防截屏/录屏

有朋友可能要问了，你这是手机拍的照片啊，怎么不截屏？

因为这个app有防截屏，就很骚了，录制也不行，我已经试过了：

![](https://img-blog.csdnimg.cn/img_convert/643cdfd8b9f8e73ef79b7d28051352ff.png)

qtscrcpy投屏也不行

![](https://img-blog.csdnimg.cn/img_convert/3f16d12d0c93009846a7c96809a57c1f.png)

好的，本篇文章的目的就是解决这两个问题，一个加密参数，一个防录屏

## 调试

### 1.加密参数算法

先看加密参数，用jadx查看，拖进去，卧槽，有壳，gg

![](https://img-blog.csdnimg.cn/img_convert/c5a859b500c980378aa977b06978154f.png)

问题不大，使用点手段，脱壳之后，继续分析，怎么定位加密逻辑和怎么分析一个app，我之前总结过，不清楚的朋友可以移步： [移动安全分析流程](http://mp.weixin.qq.com/s?__biz=MzU0MjUwMTA2OQ==&mid=2247484968&idx=1&sn=5dfc98c11270394b85e085cfc94590ff&chksm=fb18f78acc6f7e9cb23c077424e009b52738ebf4beb9123f5fdfec9ba812e5487b2ecfad5873&scene=21#wechat_redirect "移动安全分析流程")

用jadx 打开脱壳后的文件，按照之前的路子，主要找sign的位置：

找的顺序，如下图片顺序

![](https://img-blog.csdnimg.cn/img_convert/b91db64f3111b701187c2b22c5ac5b30.png)

那几个加密参数都在这里了：

![](https://img-blog.csdnimg.cn/img_convert/0818ca29c904a605f7c8f578f44d536c.png)

![](https://img-blog.csdnimg.cn/img_convert/77a35b8f58c0debaca1da0376b2101cb.png)

![](https://img-blog.csdnimg.cn/img_convert/962bcc174bf9bff6e7f3bdd246eda920.png)

就是这么快速，定位到位置，一看这个算法，盲猜是没有魔改的，看着偶读没咋改动，就是标准的sha256（当然也不一定）

### 2.防录屏逻辑

关于这个呢，其实查资料就可以知道了，原理主要是对当前启动的activity，设置一段代码：

```java
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 主要就是下面这一行代码
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_SECURE, WindowManager.LayoutParams.FLAG_SECURE);
        initView();
    }
```

那么原理都知道，用frida hook 看看

代码：

```javascript
function main() {
    Java.perform(function () {
        var v = Java.use("android.view.Window")
        v.setFlags.overload('int', 'int').implementation = function (arg1, arg2) {
            console.log('arg1', arg1)
            console.log('arg2', arg2)
            this.setFlags(1, 1)
        }
        // 有的会用这个
        // var s = Java.use("android.view.SurfaceView")
        // s.setSecure.overload('java.lang.Boolean').implementation = function (arg) {
        //     this.setSecure(false)
        // }
    })
}

setTimeout(main, 2000)
```

  

![](https://img-blog.csdnimg.cn/img_convert/36a091fd98be38a6b85f982879ba3c96.png)

标注区域就是他传递的本来的参数，它这有一堆的这个参数设置，我猜应该每个activity他都设置了这个，然后根据我上面的代码，把参数修改了，改成两个【1】，就可以截屏了：

![](https://img-blog.csdnimg.cn/img_convert/ef1c0d46cecf91b8a2ad50dcc355d51e.png)

qtscrcpy也好使了：

![](https://img-blog.csdnimg.cn/img_convert/de493272cfbcd8e9a5e1919d3646050a.png)

其实有关防录屏解决方案，早就有个大佬写好了一个xp插件，我试了下，我不知道为啥没生效，具体我就没深究了，frida hook是可以的

```
https://github.com/ilanyu/BlockSecureFlag
```

## 算法还原

过程就不展示了，直接看结果吧：

![](https://img-blog.csdnimg.cn/img_convert/41eb09b872d7c937d7df4d2bb2e28d30.png)

完美！！！

## 结语

这个app加密算法还是在java层，所以很简单的，防止截屏也是java层，所以轻松搞定

技术交流、商务合作、工作避坑\&内推\(仅成都\)、爬虫课优惠、技术交流群

扫码或者搜ID： **geekbyte**