# 前言

![](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 大家好，我们今天来爬取某站的高校名单，把其高校名单，成员和内容数获取下来，不过，我们发现这个网站比我们平时多了一个验证，下面看看我是怎么解决的。

![](https://img-blog.csdnimg.cn/img_convert/dd1231593f71ef2113352a571b87d17b.webp?x-oss-process=image/format,png)

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

话不多说，我们和平时一样，发送我们的请求，按照平时，我们看看代码怎么写。

```python
url = 'https://bizapi.csdn.net/community-cloud/v1/homepage/community/by/tag?deviceType=PC&tagId=37'
headers = {  
'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36'}  
  
response = requests.get(url=url,headers=headers)

```

我们在这里使用 `requests` 库发送 GET 请求，并将 URL 和请求头作为参数传递给 `get` 方法。请求的 URL 是 `https://bizapi.csdn.net/community-cloud/v1/homepage/community/by/tag?deviceType=PC&tagId=37` ，表示查询社区根据标签分类的数据。请求头包含了 `User-Agent` 和 `Accept` 字段，分别表示客户端的 User-Agent 和 Accept 协议类型。

不过我们会发现，我们得不到数据，就说明我们被反爬了，我尝试了很多次，我们发现它做了一个验证。

```python

headers = {  
'accept': 'application/json, text/plain, */*',  
'origin': 'https://bbs.csdn.net',  
'referer': 'https://bbs.csdn.net/college?utm_source=csdn_bbs_toolbar&spm=1035.2022.3001.8850&category=37',  
'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36',  
'x-ca-key': '203899271',  
'x-ca-nonce': '13b10c23-6a9b-423e-92a7-b114bc2c7f48',  
'x-ca-signature': 'Hhnf/RUARDM2jddNAkl2tJ6hpXfweWbY1U4/yh6FCZM=',  
'x-ca-signature-headers': 'x-ca-key,x-ca-nonce',  
}

```

我们这里科普一下，x-ca-signature 是对请求内容的签名，用于验证请求的完整性和可信性。 签名通常是通过使用私钥和一种哈希算法（如 SHA256）对请求内容进行计算得到的。 如果请求头中出现这三个参数，放心，是为了反爬用的，当然也可以用于限制请求频率，防止恶意攻击。

在解决该反爬问题时，第一步就是要找到他们的加密点。寻找 x-ca-key、x-ca-nonce、x-ca-signature 加密位置这一步主要看你对开发者工具的使用熟练程度了，寻找任意一个携带该请求头参数的请求，然后添加相应断点。通过请求地址中的部分关键字，即可添加 XHR 断点。再次刷新页面，可进入断点中，一般会停留在send\(\)函数位置。 下面的步骤就是比较枯燥的了，需要一点点的解密，例如在本函数头部找到headers，发现其参数 x-ca-key、x-ca-nonce、x-ca-signature 已经被赋值。

这里我们没有做多页爬虫，就没有去解密了，感兴趣的朋友自己去尝试。

## 内容获取

我们拿到了数据，接下来就可以提取内容了，我们看看代码怎么写，这里就很简单了。

```python
data =responses.json()['data']  
for list in data:   
    tagName = list['tagName']  
    list_url= list['url']  
    res = requests.get(list_url)  
    num = re.findall('<div title="(\d+)"',res.text)  
    print(tagName,list_url,num)

```

我们这里使用 `responses.json()['data']` 读取 API 响应 JSON 数据，并在一个数组中提取数据。然后，它使用一个 for 循环遍历数组中的每个元素，提取 `tagName` 和 `url` 两个字段，并使用 `requests.get()` 发送 GET 请求获取数据。最后，它使用正则表达式从响应文本中提取 `num` 数据，并将其打印到控制台上。

## 全部代码

```python
import re

import requests

url = 'https://bizapi.csdn.net/community-cloud/v1/homepage/community/by/tag?deviceType=PC&tagId=37'
# url = https://bbs.csdn.net/college?category=37 # 主页

headers = {
    'accept': 'application/json, text/plain, */*',
    'origin': 'https://bbs.csdn.net',
    'referer': 'https://bbs.csdn.net/college?utm_source=csdn_bbs_toolbar&spm=1035.2022.3001.8850&category=37',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36',
    'x-ca-key': '203899271',
    'x-ca-nonce': '13b10c23-6a9b-423e-92a7-b114bc2c7f48',
    'x-ca-signature': 'Hhnf/RUARDM2jddNAkl2tJ6hpXfweWbY1U4/yh6FCZM=',
    'x-ca-signature-headers': 'x-ca-key,x-ca-nonce',
}
responses = requests.get(url,headers=headers)
data =responses.json()['data']
# print(data)
for list in data:
    # print(list)
    tagName = list['tagName']
    list_url= list['url']
    res = requests.get(list_url)
    # print(res.text)
    num = re.findall('<div title="(\d+)"',res.text)
    print(tagName,list_url,num)
```

# 总结

我们这样就获取到了内容，本文仅供学习。

![](https://img-blog.csdnimg.cn/img_convert/9e7d5f1f72aca0f21252de78cc872146.webp?x-oss-process=image/format,png)

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)