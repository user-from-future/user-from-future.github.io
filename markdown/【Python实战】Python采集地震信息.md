# 前言

![](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 昨天，我们这里发生了地震，不过，没有太大的问题，我就想着能不能把近几年发生地震的信息，收集下来，我们发现中国地震台网的官方微博会分布近几年发生地震的信息。我们可以直接在这里获取。

### **环境使用**

- python 3.9
- pycharm

### **模块使用**

- requests

### **模块介绍**

> - **requests**
> 
> requests是一个很实用的Python HTTP客户端库，爬虫和测试服务器响应数据时经常会用到，requests是Python语言的第三方的库，专门用于发送HTTP请求，使用起来比urllib简洁很多。
> 
> - **parsel**
> 
> parsel是一个python的第三方库，相当于css选择器+xpath+re。
> 
> parsel由scrapy团队开发，是将scrapy中的parsel独立抽取出来的，可以轻松解析html，xml内容，获取需要的数据。
> 
> 相比于BeautifulSoup，xpath，parsel效率更高，使用更简单。
> 
> - **re**
> 
> re模块是python独有的匹配字符串的模块，该模块中提供的很多功能是基于正则表达式实现的，而正则表达式是对字符串进行模糊匹配，提取自己需要的字符串部分，他对所有的语言都通用。
> 
> - **os**
> 
> os 就是 “operating system” 的缩写，顾名思义， [os模块](https://so.csdn.net/so/search?q=os%E6%A8%A1%E5%9D%97&spm=1001.2101.3001.7020 "os模块") 提供的就是各种 Python 程序与操作系统进行交互的接口。通过使用 os 模块，一方面可以方便地与操作系统进行交互，另一方面也可以极大增强代码的可移植性。
> 
> - **csv**
> 
> 它是一种文件格式，一般也被叫做逗号分隔值文件，可以使用 Excel 软件或者文本文档打开 。其中数据字段用半角逗号间隔（也可以使用其它字符），使用 Excel 打开时，逗号会被转换为分隔符。csv 文件是以纯文本形式存储了表格数据，并且在兼容各个操作系统。

**模块安装问题:**

- 如果安装python第三方模块:

> win + R 输入 cmd 点击确定, 输入安装命令 pip install 模块名 \(pip install requests\) 回车
> 
> 在pycharm中点击Terminal\(终端\) 输入安装命令

- 安装失败原因:

> - 失败一: pip 不是内部命令
> 
> 解决方法: 设置环境变量
> 
> - 失败二: 出现大量报红 \(read time out\)
> 
> 解决方法: 因为是网络链接超时, 需要切换镜像源
> 
> ```python
>     清华：https://pypi.tuna.tsinghua.edu.cn/simple
>     阿里云：https://mirrors.aliyun.com/pypi/simple/
>     中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
>     华中理工大学：https://pypi.hustunique.com/
>     山东理工大学：https://pypi.sdutlinux.org/
>     豆瓣：https://pypi.douban.com/simple/
>     例如：pip3 install -i https://pypi.doubanio.com/simple/ 模块名
> ```
> 
> - 失败三: cmd里面显示已经安装过了, 或者安装成功了, 但是在pycharm里面还是无法导入
> 
> 解决方法: 可能安装了多个python版本 \(anaconda 或者 python 安装一个即可\) 卸载一个就好，或者你pycharm里面python解释器没有设置好。

## 代码实现

我们这里直接请求网页，发送请求，获取内容，我们代码内容如下：

```python

url = 'https://weibo.com/ajax/statuses/mymblog?uid=2817059020&page=2&feature=0'  
res=requests.get(url)  
  
  
res.encoding = res.apparent_encoding  
  
print(res.text)

```

我们这里虽然返回了内容，但是，不是我们想要的内容，说明，我们被反爬了，我们看看这里返回了什么内容吧。

```javascript
wload(function () {

    try {

        var need_restore = "1" == "1"; // 是否走恢复身份流程。

        // 如果需要走恢复身份流程，尝试从 cookie 获取用户身份。
        if (!need_restore || !Store.CookieHelper.get("SRF")) {

            // 若获取失败走创建访客流程。
            // 流程执行时间过长（超过 3s），则认为出错。
            var error_timeout = window.setTimeout("error_back()", 5000);

            tid.get(function (tid, where, confidence) {
                // 取指纹顺利完成，清除出错 timeout 。
                window.clearTimeout(error_timeout);
                incarnate(tid, where, confidence);
            });
        } else {
            // 用户身份存在，尝试恢复用户身份。
            restore();
        }
    } catch (e) {
        // 出错。
        error_back();
    }
});

```

我们发现这里，我们少了一些参数，说明了这个网站对cookies做了验证，我们加入cookies试试，在我们把cookies传进去之后，就拿到了我们想要的内容了。

![](https://img-blog.csdnimg.cn/img_convert/17106cf872fb3d1627df04bf84f9b1ae.png)

我们会发现，这里是json格式的数据，接下来，就是字典取值就可以了，我们得到的内容如下。

```python

lists = res.json()['data']['list']  
for list in lists:  
text_raw = list['text_raw']  
print(text_raw)

```

![](https://img-blog.csdnimg.cn/img_convert/24112aebf7fa518bd4d42cd172cb75a3.webp?x-oss-process=image/format,png)

### 全部代码

```python
import requests
headers = {
    'cookie': 'XSRF-TOKEN=i_jwhItBV-N25hG4ByCztsDp; SUBP=0033WrSXqPxfM72-Ws9jqgMF55529P9D9WW5I6LOLDiQVAchoLQaqJ2V; SUB=_2AkMTCu21f8NxqwJRmf0RxWLqbIxyzwjEieKlVhxuJRMxHRl-yj9vqhAFtRB6OIrDWiYjOrzbl8TwUL-lp2Vruuysazih; WBPSESS=CcKh_7VyRZckvS8BGV3cswGEcJVP8L6xFhCv31UrVROFUvdKWlhGzVyOvd-MNsbpR3rSt54J6liS1X2a7xld9BZTMPyzWXwxaVu4ecgBw8SewBe6cQIVoaI6RoXqejD9UZWmFqAmHcgQXNw8LNFvDnoIk4zE6uUcAhWxq666R2o=',
    'referer': 'https://weibo.com/n/%E4%B8%AD%E5%9B%BD%E5%9C%B0%E9%9C%87%E5%8F%B0%E7%BD%91',
    'traceparent': '00-86b5603650ee569365b690bd914780a8-3e286bc027650efc-00',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36',
    'x-requested-with': 'XMLHttpRequest',
    'x-xsrf-token': 'i_jwhItBV-N25hG4ByCztsDp',
}
url = 'https://weibo.com/ajax/statuses/mymblog?uid=2817059020&page=1&feature=0'
res=requests.get(url,headers=headers)
res.encoding = res.apparent_encoding
lists = res.json()['data']['list']
for list in lists:
    # print(list)
    created_at = list['created_at']
    text_raw = list['text_raw']
    print(created_at,text_raw)
```

# 知识拓展

cookies是一种能够让网站服务器把少量数据储存到客户端的硬盘或内存，或是从客户端的硬盘读取数据的一种技术。

Cookies是当你浏览某网站时，由Web服务器置于你硬盘上的一个非常小的文本文件，它可以记录你的用户ID、密码、浏览过的网页、停留的时间等信息。 当你再次来到该网站时，网站通过读取Cookies，得知你的相关信息，就可以做出相应的动作，如在页面显示欢迎你的标语，或者让你不用输入ID、密码就直接登录等等。

从本质上讲，它可以看作是你的身份证。 但Cookies不能作为代码执行，也不会传送病毒，且为你所专有，并只能由提供它的服务器来读龋保存的信息片断以“名/值”对\(name-value pairs\)的形式储存，一个“名/值”对仅仅是一条命名的数据。 一个网站只能取得它放在你的电脑中的信息，它无法从其它的Cookies文件中取得信息，也无法得到你的电脑上的其它任何东西。

# 总结

我们发现，我们没有登录，获取的内容很少，我们可以登录之后，把我们的cookies传进去，就可以获取到更多的内容了。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)