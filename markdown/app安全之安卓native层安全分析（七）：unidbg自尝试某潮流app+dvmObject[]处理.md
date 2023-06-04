## 前言

跟着龙哥搞了几次unidbg了，这次也自己尝试用来分析下某潮流app了。

## 分析

### 1.抓包

先抓个包

![](https://img-blog.csdnimg.cn/img_convert/092c5b7f6036800f48f9260261f87dfd.png)

我们要搞的就是这个sign-v1了。

### 2.调试找参数

jadx一顿分析，一搜：

![](https://img-blog.csdnimg.cn/img_convert/20a9fdbe57435aff480efa16cd66d279.png)

搜出来还不少，往下翻，找找一些特征，很快找到这里

![](https://img-blog.csdnimg.cn/img_convert/43575bdc2e75c2ea82bf336a58820d8d.png)

点进去

![](https://img-blog.csdnimg.cn/img_convert/5efb47b06c51a36c5de1c019f50bfef9.png)

![](https://img-blog.csdnimg.cn/img_convert/5094d1fc5c4b8ea12db9f4de00d7ca68.png)

ok，用objection hook之后，发现不是这个方法，但是确实是这个方法所在的类的b方法：

![](https://img-blog.csdnimg.cn/img_convert/8f2b7f7f62473010509dc1111a9f4870.png)

然后调用了这个getsign，getsign就在下面

![](https://img-blog.csdnimg.cn/img_convert/9ee3502919924cb50c65e8b42a8dff37.png)

再来hook下这两个方法：

![](https://img-blog.csdnimg.cn/img_convert/506a4bec1639363fd4785ef45529617f.png)

![](https://img-blog.csdnimg.cn/img_convert/22fa8ca085ff624065398a0a3e9fe5bc.png)

基本确定，就是这里了，抓包工具也对比就是这里：

![](https://img-blog.csdnimg.cn/img_convert/76bcf2263a4a9f6b13c26c257b9b6806.png)

### 3.ida查看：

打开ida看看，发现是静态绑定的

![](https://img-blog.csdnimg.cn/img_convert/742886925602a7a6103e08440d645941.png)

可以的，感觉很简单

## 调试

### 1.搭架子

首先搭一下架子，然后既然他不是动态注册的，那就可以不用设置调用jni\_load了：

![](https://img-blog.csdnimg.cn/img_convert/ef57305623c475b94ba2582d68a1a5d6.png)

看着没毛病，直接调用吧

### 2.调用\&补环境

废话不多说，直接拿着hook到的参数拿来调用：

![](https://img-blog.csdnimg.cn/img_convert/7dd8d9ccd56c8977a8fdc5f51dcbe378.png)

看下，这个ach是啥，ok，看样子就是随机生成一个16位的字符串

![](https://img-blog.csdnimg.cn/img_convert/9ab65c6f6ee1e94c52d70b01e05d01f9.png)

补一下环境

![](https://img-blog.csdnimg.cn/img_convert/d41b59ce3188addab0204d595844aec3.png)

继续看

![](https://img-blog.csdnimg.cn/img_convert/b37ac83b563d14b282e3a94d0a6f37bf.png)

原来就是把这几个加起来

![](https://img-blog.csdnimg.cn/img_convert/87fa88b2209866ff68632057daec4801.png)

补一下：

![](https://img-blog.csdnimg.cn/img_convert/dcfd37e84ca284d38dcfca070bdc476b.png)

结果已经出来了。但是打印的是一个dvmobject对象。很奇怪了。再看看hook的结果：

![](https://img-blog.csdnimg.cn/img_convert/17d166030a7ad95d0d6857a3d5aca001.png)

其实是个有三个元素的字符串数组。所以他应该是对象

断点调试一下

![](https://img-blog.csdnimg.cn/img_convert/eb9c7fcaec14956f626dcb27e757e7bb.png)

![](https://img-blog.csdnimg.cn/img_convert/9f133994be7bdeaea3814d0d9bd3cb2a.png)

结果这么写会报错：

![](https://img-blog.csdnimg.cn/img_convert/37d14d19da8cd89cf318595d3ce9a6b1.png)

强转string\[\]也不行，就很尴尬

![](https://img-blog.csdnimg.cn/img_convert/13ce4938dcc230879310bc7f90a55723.png)

### 3.结果修复

问了一下unidbg的大佬，应该这么写：

![](https://img-blog.csdnimg.cn/img_convert/68a52a61d7a10ef609f54585c8b04429.png)

这么写就可以了。

```java
DvmObject[] result = (DvmObject[]) vm.getObject(number.intValue()).getValue();

String sign = result[2].toString();
```

### 4.另一种调用

首先，我们都知道有，地址调用，符号调用，如下：

![](https://img-blog.csdnimg.cn/img_convert/7feb857dfa105a041005b617b46060ca.png)

其实，根据我问的unidbg玩的6的大佬，还有另一种调用

这么写，代码量减少了很多。

![](https://img-blog.csdnimg.cn/img_convert/e93fc3e0126625901b387b944d4fcf9b.png)

但是有个问题就是，那个network类，需要在构造方法里定义一下：

![](https://img-blog.csdnimg.cn/img_convert/0e764d29016379d8bac297a73ab22359.png)

另外，这个方法的签名，怎么拿到，用jadx的smali代码查看：

![](https://img-blog.csdnimg.cn/img_convert/1e66ee8d17baf15426dea372bd4e9f8c.png)

这里就直接有了，复制过去就可以用，注意最后的【；】也要带上。

### 5.验证是否可用

试试刚才的两种调用的结果，能不能拿来请求，然后正常返回呢？

![](https://img-blog.csdnimg.cn/img_convert/d3627edc828062be79c8496bc24072f6.png)

![](https://img-blog.csdnimg.cn/img_convert/4cad50a75fd26f43b3989d0744a515ad.png)

ok，两个都可以 ，说明主动调用，整个过程，很成功

## 结语

整个过程很轻松加愉快。除了最后的dvmObject数组的转换，其他没啥需要记录的。