## 前言

  
最近行业市场不太景气啊，趁着有时间多学学吧，武装自己，等机会  
刚好，发现一个很6的东西。这个问题是在差不多半个月前，群友 \@十一 发现的，然后在群里跟大家讨论。

![](https://img-blog.csdnimg.cn/img_convert/8d747b7b0c6d482043fb8f75e8726dc3.png)

  
这个网站，请求的时候，requests正常：

![](https://img-blog.csdnimg.cn/img_convert/4023f39eab717e87e0f5fe67e795b600.png)

原始的curl也可以：

![](https://img-blog.csdnimg.cn/img_convert/d8c7e94e917427e71759dd0ed3170363.png)

aiohttp，直接报错

![](https://img-blog.csdnimg.cn/img_convert/44114696d182876a8b08042b98fd0196.png)

httpx，也直接报错：

![](https://img-blog.csdnimg.cn/img_convert/a0f98fad34cbf8965415d3cd2c8d01a1.png)

不过httpx的报错要明显点，这就进入了有意思的环节

## 分析

  
  
看到这里，估计就有朋友开始冷笑了：搞爬虫大部分不都用requests的吗，谁会用httpx和aiohttp这些啊，感觉这个是不是没啥用啊。  
别急，朋友，在文章的下面会说这个，当然如果你赶时间或者觉得没啥意思，那就没必要浪费时间继续看了

### 1.httpx

  
拿着上面的报错，去网上搜，找到如下issues:

> https://github.com/encode/httpx/issues/1561
> 
> https://github.com/encode/httpx/issues/1363

  
  
看完里面各位大佬的一顿分析，说是hyper库的问题：

![](https://img-blog.csdnimg.cn/img_convert/a2406feb3f5215f193bfeb0e220d5ab9.png)

![](https://img-blog.csdnimg.cn/img_convert/2cbc459cac7d00117b8a03b6a2164595.png)

然后hyper库的开发者，如下链接回复：

> [Should we support looser header name validation\? · Issue #113 · python-hyper/h11 · GitHub](https://github.com/python-hyper/h11/issues/113 "Should we support looser header name validation? · Issue #113 · python-hyper/h11 · GitHub")

大概意思是这个不是一个问题，而是http请求的严格性判断问题，请求头的协议，按国际标准，是不能出现 “\[Cache-Control\]” 这种带有特殊符号作为响应头的键名的，所以报错  
而requests却可以，或许是因为requests的校验不严格，直接就放过了：

![](https://img-blog.csdnimg.cn/img_convert/434aac3722a9556a7a72aa643c1fdeb1.png)

而，浏览器访问也是可以的：

![](https://img-blog.csdnimg.cn/img_convert/e0f9a8f8d0bfa0f1207a0266d0b1d836.png)

那么我个人就有理由认为

**这是一个bug，httpx和aiohttp都存在的bug**

httpx的作者，对这个问题那段时间确实在尝试解决，github机器人都想关闭了，httpx作者还不想放弃：

![](https://img-blog.csdnimg.cn/img_convert/dae70e87a42814843a2b6b28fed3a19a.png)

且至今没解决，遇到的人还不少，至少，上个月都还有人在说这个问题

![](https://img-blog.csdnimg.cn/img_convert/51c24901a2bf7e5a326bdbb80dde6f00.png)

卧槽，这时间，上周都还有人在问啊：

![](https://img-blog.csdnimg.cn/img_convert/c9ebcf2ca6fc8c2e91a71e4c0dc7afca.png)

其中也有说解决办法，看到有个老哥说改h11的源码，改成这样：

![](https://img-blog.csdnimg.cn/img_convert/f0f82a035b75f00ba78ffcaf8a970187.png)

但是报错依然在

![](https://img-blog.csdnimg.cn/img_convert/9d4f729758c533407bf365e01c6776e1.png)

另外有个老哥说了这个方法：

![](https://img-blog.csdnimg.cn/img_convert/01b8a451bbcf91bca8ed34f3c1e88ef8.png)

```python
h11.readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")
```

我把这个代码直接放我代码里，报错了，根本没有这个属性

![](https://img-blog.csdnimg.cn/img_convert/f7f934d7e02105daabc952a1c11ea58a.png)

经过一顿查阅，他换了个属性名，是这个：

![](https://img-blog.csdnimg.cn/img_convert/54b02479288278726624d24be42b8f03.png)

```python
import h11from h11 import _readersimport re

h11._readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")
```

但是我试了，换成了新的报错：

![](https://img-blog.csdnimg.cn/img_convert/6e170f97a883f561b5ed5049251220bd.png)

也许是正则表达式写的不完美：

![](https://img-blog.csdnimg.cn/img_convert/e88901e03eea74ac325c4307f84260c0.png)

正则表达式通用匹配一下，然后就可以了：

![](https://img-blog.csdnimg.cn/img_convert/4b4cadddf113e50e8b687d068ab8b194.png)

```python
import httpximport h11from h11 import _readersimport re

# h11._readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")

h11._readers.header_field_re = re.compile(b"(?P<field_name>.*?):[ \t](?P<field_value>.*?)")

headers = {  

'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'
}
url = 'https://scanstatus.sxxxxx'
# url = 'https://club.xxxxxx/thread-26829078-1-1.html'

resp = httpx.get(url=url, headers=headers)

print(resp.text)
```

  
但是，我换了一个网站，也就是那个issue里某华某为的社区地址，不出正常的结果：

![](https://img-blog.csdnimg.cn/img_convert/9729308451f8b43c9741c8616a5b7f89.png)

所以，修改正则表达式并不是一个通用的方案。

插一句，在请求的时候，就不要开代理或者抓包软件了，不然他报错更奇怪：

![](https://img-blog.csdnimg.cn/img_convert/97cd2a1fa11c645cd7e88b068fed1662.png)

综上所述，也就是，只要服务器的返回头埋的坑够多，这个方法根本无法完美解决

那么这个问题，httpx作者就给了个这个解释就没下文了

> `https:` `/` `/` `github.com` `/` `encode` `/` `httpx` `/` `issues` `/` `767` `#issuecomment-1367498458`

就这么不了了之了。。。

因为根据http协议的响应头原理，确实不会出现这种不标准的字段

更官方的解释可以去查查http协议原理，或者看看以下资料：

> `https:` `/` `/` `zh.wikipedia.org` `/` `zh` `/` `HTTP` `%` `E5` `%` `A4` `%` `B4` `%` `E5` `%` `AD` `%` `97` `%` `E6` `%` `AE` `%` `B5https:` `/` `/` `developer.mozilla.org` `/` `zh` `-` `CN` `/` `docs` `/` `Web` `/` `API` `/` `Headerhttps:` `/` `/` `developer.mozilla.org` `/` `zh` `-` `CN` `/` `docs` `/` `Glossary` `/` `Response_header`

  
我的理解如下：  

> 客户端需要先解析响应头，通过响应头的一些规范来解析返回的响应体的，既然你响应头都报错了，那后续的响应体解析自然也不会往下走了，直接报错退出。

### 2.aiohttp

对于aiohttp来说，像上面那么加正则表达式是无效的：

![](https://img-blog.csdnimg.cn/img_convert/a0b59618c7d3993e2d9cc7e6694ebeaa.png)

![](https://img-blog.csdnimg.cn/img_convert/41a3d8e4e30fe2f5ecbcaecd7bce6246.png)

改完发现并没有用

![](https://img-blog.csdnimg.cn/img_convert/019d7bfbe1caedb899c767942025cf7d.png)

这有说，改成1，也试了，不行

![](https://img-blog.csdnimg.cn/img_convert/c362c9ed0fbbcaf61276b746a9f884e3.png)

更多的就不演示了

反正就是一顿查找，发现aiohttp上并没有合理方案解决

## 针对的解决方案

### 1.用requests

  
上面有了，这里就不展示了  
  
但是requests我们知道，它是不支持http2.0协议的

### 2.用正则表达式替换

  
上面也有了，这里不贴了但是不能完美覆盖后续的响应头特殊参数  

### 3.用urllib3处理

  
这个库用的倒不多，可以直接请求：

![](https://img-blog.csdnimg.cn/img_convert/081c50c5591bb0d3f0258b55929d23a9.png)

但是他也是个可以直接拿来请求的库，相比requets，那很多功能还是没有。其实requests本质就是借助了urllib3库的  
所以，这个库，不出意外应该也是不支持http2.0的

## 用非常规的请求库尝试

  
既然目前的方法，除了requests/urllib3，其他都不行，那我在这个基础之上，强制http2，是不是就可以干掉requets库了，python的常规请求库直接无可用的，直接干死这些爬虫？  
  
想的挺美的

### 1.用tls-client试试

虽然他是用来对抗tls的，试试呢：  

![](https://img-blog.csdnimg.cn/img_convert/e6e51ef38a0212074548675554b7fff5.png)

可行。为啥，因为他原理就是完全模拟浏览器，大概看了它的源码，用curl\_impersonate库，打包成了一个dll（具体怎么打包的不可知，这部分没开源，插一句， **据群友反馈，也是因为这个dll，会导致内存泄漏** ），然后可以直接用，上面说过了，浏览器可以访问，那这个库肯定也可以访问了  
  
那么测试那个某为的网址：  

![](https://img-blog.csdnimg.cn/img_convert/fabd54b95ba62372f9cfeddc68725e37.png)

不行，被识别了，牛吧，还是得是某为啊

### 2.用curl\_cffi试试

![](https://img-blog.csdnimg.cn/img_convert/389a82bf8549f6cd20cdb2d836e9d411.png)

测试那个某为的网址：  

![](https://img-blog.csdnimg.cn/img_convert/9f2f8a5c125958377a3b430dabb5b26c.png)

这么对比，看来curl\_cffi才是获胜者啊，只能说，牛逼啊！！！  
  

## 实现一个anti aiohttp、httpx的服务

  
简单的用fastapi 实现一个简单的服务，设置了下响应头：  

![](https://img-blog.csdnimg.cn/img_convert/f98fa0425a76e82a781e38cb11647f5a.png)

### 1.用httpx请求

httpx代码不变，只是把url换了，果然报错  

![](https://img-blog.csdnimg.cn/img_convert/d3f7bd4b02328a3c253f60e158e309a2.png)

### 2.用aiohttp请求

再来看aiohttp，果然报错，哈哈哈哈

![](https://img-blog.csdnimg.cn/img_convert/7a0cd0add8656e7c972b9b3626c23097.png)

### 2.用http.client请求

![](https://img-blog.csdnimg.cn/img_convert/94be74f499cd1cd670d5e32d1994381b.png)

### 3.用urllib3、requests测试

![](https://img-blog.csdnimg.cn/img_convert/e95f7a6628a34fa0a6653518903ff206.png)

![](https://img-blog.csdnimg.cn/img_convert/0ecd9cf06d0b2ad9ca3300ce468770c4.png)

还得是这哥俩啊，直接能跑

### 4.用node的request请求看看

![](https://img-blog.csdnimg.cn/img_convert/20a0cf6eeb52762446b00041ea6c6178.png)

刺激，也给防住了。  

### 5.用golang 看看

  
刺激啊，也没有正常返回

![](https://img-blog.csdnimg.cn/img_convert/93ede21bd05af8edb9d2334057045279.png)

### 6.用postman看看

postman也直接无返回

![](https://img-blog.csdnimg.cn/img_convert/fc3c36377e240fde83cc653d240108d2.png)

### 7.用安卓看看

```java
package com.geek.spiderclient;



import androidx.appcompat.app.AppCompatActivity;



import android.os.Bundle;

import android.util.Log;

import android.widget.TextView;



import java.io.BufferedReader;

import java.io.IOException;

import java.io.InputStream;

import java.io.InputStreamReader;

import java.io.UnsupportedEncodingException;

import java.net.HttpURLConnection;

import java.net.URL;



public class MainActivity extends AppCompatActivity {

    private TextView textView;



    @Override

    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        textView = findViewById(R.id.hello);



        new Thread(new Runnable() {

            String result = "";



            @Override

            public void run() {

                try {

                    String url_string = "http://192.168.30.251:8000/";// 由于高版本有https的限制，需要修改targetSdk为27及一下。

                    URL url = new URL(url_string);

                    //得到connection对象。

                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();

                    //设置请求方式

                    connection.setRequestMethod("GET");

                    //连接

                    connection.connect();

                    //得到响应码

                    int responseCode = connection.getResponseCode();

                    Log.d("headers", "" + connection.getHeaderFields());

                    if (responseCode == HttpURLConnection.HTTP_OK) {

                        //得到响应流

                        InputStream inputStream = connection.getInputStream();

                        //将响应流转换成字符串

                        result = is2String(inputStream);//将流转换为字符串。

                        Log.d("result", "result=============" + result);

                    }

                } catch (Exception e) {

                    e.printStackTrace();

                }

                runOnUiThread(new Runnable() {

                    @Override

                    public void run() {

                        Log.e("result", "runOnUiThread");

                        textView.setText(result);

                    }

                });

            }

        }).start();

    }



    public String is2String(InputStream is) {

        //连接后，创建一个输入流来读取response

        BufferedReader bufferedReader = null;

        try {

            bufferedReader = new BufferedReader(new InputStreamReader(is, "utf-8"));

            String line = "";

            StringBuilder stringBuilder = new StringBuilder();

            String response = "";

            //每次读取一行，若非空则添加至 stringBuilder

            while ((line = bufferedReader.readLine()) != null) {

                stringBuilder.append(line);

            }

            //读取所有的数据后，赋值给 response

            response = stringBuilder.toString().trim();

            return response;

        } catch (IOException e) {

            e.printStackTrace();

        }

        return null;

    }

}
```

```


```

  
打包好之后：

![](https://img-blog.csdnimg.cn/img_convert/86423b647854d35b84153542e9ef74c1.png)

安卓是可以的  
  
  
再来看看服务端这边，其实是有正常请求的，但是客户端那边无法正常解而已。

![](https://img-blog.csdnimg.cn/img_convert/d2cacfa5ff76e4e6183572fab9f6536f.png)

  
那么有朋友估计有想法，那既然报错，我捕获异常强制打印结果不行吗？  
这里还是用回到python httpx库试试

![](https://img-blog.csdnimg.cn/img_convert/6fe7ac8fd532332fc0dc544deb5b6d1f.png)

不行的哈哈，原因前面说过了

## 配置将服务强制http2.0协议

### 1.配置服务端

说干就干，准备尝试把代码移植刀服务器上，直接启动这边的服务，然后nginx搭建好，配置好http2，我买的服务器它默认没给开80和443等常规的库，问了客服说要申请备案了才行，卧槽，很迷，无所谓，我把这个服务搭建到6363端口上：

![](https://img-blog.csdnimg.cn/img_convert/8e323b0ef4021ab71dc36e6cbf629e9d.png)

  
nginx如下配置：

```bash
server {

      # listen 6363;

      listen [::]:6363 ssl  http2; # managed by Certbot

      listen 6363  ssl http2; # managed by Certbot

      ssl_certificate /root/cert.pem; # managed by Certbot

      ssl_certificate_key /root/key.pem; # managed by Certbot

      if ($server_protocol !~* "HTTP/2.0") {

        return 444;

      }

      root /data/www/fast-tortoise;

      server_name 0.0.0.0;

      location / {

            proxy_set_header x-Real-IP $remote_addr;

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header Host $http_host;

            proxy_pass http://127.0.0.1:8000/; # gunicorn绑定的fastapi的端口号

        }

      # 配置static的静态文件：

      location ~ ^\/static\/.*$ {

            root /data/www/fast-tortoise/static;

         }

}
```

```
 ```

  
证书文件用以下命令生成即可：前提得安装openssl库

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

  

### 2.请求测试

  
配置好了之后，首先requests肯定是不行了，因为不支持http2.0。这里就不演示了。  
  
用httpx测试下，唉？直接就给放行了

![](https://img-blog.csdnimg.cn/img_convert/b9432823f1bd8159acd9b000ac5bdcb9.png)

  
卧槽？反爬虫路途创业未半中道崩卒....  
.....  
  
看来http2.0把这个问题修复了呀......  
  
......  
那就只能在http1.0，http1.1下防住一部分的爬虫了  
  
那有朋友估计要问了，你这个操作，好像只是有点奇怪，但实际并没有文章标题说的那么邪乎啊，说直接点，基本没啥用啊

## 解惑

  
如果坚持看到这里的朋友，还没有忘记前面说的有关requests问的话。ok，这里开始说说。  
其实我一开始发现这个所谓的bug的时候，感觉也不是太有用，requests都能跑，没把requests防住，那肯定没啥意思啊。  
那么这个所谓的bug，真的没用吗？  
只是这一个的话肯定防不住的，可以用来跟其他反爬手段组合啊，比如tls指纹，或者其他的风控检测手段等等的。  
  
其实，我在发这篇文章之前，也开发了两个爬虫练习题

![](https://img-blog.csdnimg.cn/img_convert/c619733f680f1d3deb40db0787f64acc.png)

  
第一题就是这个bug  
第二题是另外一个检测手段， **可以检测到requests和httpx，tls-client，curl\_cffi，aiohttp，还有常规的请求软件，而且没有用到tls指纹** 。

  
本来说是先让群友拿来玩，然后过几天发文章公开检测手段以及如何bypass的。

  
我没想到，我还没发多久，就有 intellectual disability 搞我服务器.....  
顿时就觉得没意思了，服务关了，第二题的检测手段和bypass的方法我也不打算公开了。由于这篇文章之前就答应过群友 \@十一，且早就写好了，那就继续发吧。

  
想知道第一种检测手段，还有哪些特殊字符不能被正常解析吗？

有想知道第二种检测手段的朋友？

  
**可以时不时的关注我的动向，也许我会在某一天用新的形式发出来**

## 结语

  
**爬虫的核心，还是指纹等静态特征完全的模拟浏览器环境，行为等动态特征完全的模拟人为操作**  
  

搜ID： **geekbyte**