# 前言

![](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 我们要实现了一个最基本的爬虫，但提取页面信息时使用的是正则表达式，这还是比较烦琐，而且万一有地方写错了，可能导致匹配失败，所以使用正则表达式提取页面信息多多少少还是有些不方便。

**目录**

[前言](#%E5%89%8D%E8%A8%80)

[环境使用](#%E7%8E%AF%E5%A2%83%E4%BD%BF%E7%94%A8)

[模块使用](#%E6%A8%A1%E5%9D%97%E4%BD%BF%E7%94%A8)

[模块介绍](#%E6%A8%A1%E5%9D%97%E4%BB%8B%E7%BB%8D)

[使用XPath](#%E4%BD%BF%E7%94%A8XPath)

[XPath概览](#XPath%E6%A6%82%E8%A7%88)

[XPath常用规则](#XPath%E5%B8%B8%E7%94%A8%E8%A7%84%E5%88%99)

---

**环境使用**

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
>   
> 失败三: cmd里面显示已经安装过了, 或者安装成功了, 但是在pycharm里面还是无法导入
> 
> 解决方法: 可能安装了多个python版本 \(anaconda 或者 python 安装一个即可\) 卸载一个就好，或者你pycharm里面python解释器没有设置好。

我们要实现了一个最基本的爬虫，但提取页面信息时使用的是正则表达式，这还是比较烦琐，而且万一有地方写错了，可能导致匹配失败，所以使用正则表达式提取页面信息多多少少还是有些不方便。

我们今天说说另外一种方法。

对于网页的节点来说，它可以定义id、class或其他属性。而且节点之间还有层次关系，在网页中可以通过XPath或CSS选择器来定位一个或多个节点。那么，在页面解析时，利用XPath或CSS选择器来提取某个节点，然后再调用相应方法获取它的正文内容或者属性，不就可以提取我们想要的任意信息了吗\?

在Python中 ,怎样实现这个操作呢\?不用担心,这种解析库已经非常多,其中比较强大的库有lxml ,Beautiful Soup、pyquery 等，我们将会介绍这3个解析库的用法。有了它们，我们就不用再为正则表达式发愁,而且解析效率也会大大提高。

## 使用XPath

XPath，全称XML Path Language，即 XML路径语言，它是一门在XML文档中查找信息的语言。它最初是用来搜寻XML文档的,但是它同样适用于HTML文档的搜索。  
所以在做爬虫时，我们完全可以使用XPath来做相应的信息抽取。本节中，我们就来介绍XPath的基本用法。

### XPath概览

XPath 的选择功能十分强大，它提供了非常简洁明了的路径选择表达式。另外，它还提供了超过100个内建函数，用于字符串、数值、时间的匹配以及节点、序列的处理等。几乎所有我们想要定位的节点，都可以用XPath来选择。  
XPath于1999年11月16日成为W3C标准，它被设计为供XSLT、XPointer以及其他XML解析软件使用，更多的文档可以访问其官方网站: https://www.w3.org/TR/xpath/ 。

### XPath常用规则

这里列出了XPath的常用匹配规则,示例如下:\[/title\[\@lang=" eng'\]  
这就是一个XPath规则，它代表选择所有名称为title，同时属性lang 的值为eng 的节点。后面会通过Python的 lxml库，利用XPath进行HTML的解析。

后面就会学习到子节点，父节点，节点的相关知识，我们在使用的时候要先安装lxml模块，我们今天就说到这里。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)