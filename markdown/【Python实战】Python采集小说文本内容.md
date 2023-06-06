# 前言

![1b83b1d3fff541e6844ba7bfc4b8f724.gif](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 今天，我们将采集某小说数据，通过这个案例，加深我们对正则表达式的理解。我们今天来通过使用正则表达式来获取我们想要的文本。

## **环境使用**

- python 3.9
- pycharm

## **模块使用**

- requests

## **模块介绍**

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

## **模块安装问题:**

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

# 代码实现

现在我们一步步实现我们的需求，首先，先举个例子，我们来看下某一个小说的 [URL](https://www.qb5.tw/book_116659/50912006.html "URL") 。

## 第一章获取

### 获取数据

我们首先获取网页数据。

```python
import requests

url = 'https://www.qb5.tw/book_116659/50912006.html

response = requests.get(url=url)

print(response.text)
```

运行之后会得到下面反馈的内容，在pycharm是一行显示，不够直观，我把获取到的网页数据单独拿出了。

![290f555f1d3e481b9b095f06603ee544.png](https://img-blog.csdnimg.cn/290f555f1d3e481b9b095f06603ee544.png)

### 解析数据

看到这很多人就会发现有好多都是不需要的，所以我们就要把其中正文部分提取出来，然后，我们使用审查元素方法，查看一下我们的目标页面，你会看到如下内容。

![95709913d0664b7fa93bc1725db0db59.png](https://img-blog.csdnimg.cn/95709913d0664b7fa93bc1725db0db59.png)

不难发现，文章的所有内容都放在了一个名为div的“东西下面”，这个"东西"就是html标签。HTML标签是HTML语言中最基本的单位，HTML标签是HTML最重要的组成部分。

细心的朋友可能就能发现，\<a> **标签里面存放的内容，是我们关心的正文部分。** 下面我们用正则表达式去提取，大家也可以用xpath去提取。

```python
text = re.findall('<div id="content">(.*?)</div>', html_data)[0].replace(
        ' 全本小说网 www.qb5.tw，最快更新<a href="https://www.qb5.tw/book_116659/">宇宙职业选手</a>最新章节！<br><br>',
        '').replace('&nbsp;', '').replace('<br />', '\n')
```

find\_all匹配的返回的结果是一个列表。提取匹配结果后，使用text属性，提取文本内容，滤除br标签。随后使用replace方法，剔除空格，替换为回车进行分段。\&nbsp在html中是用来表示空格的。

这样我们第一章的内容就获取到了。

![4d3bce05978e45cf92c298f057ae7c5d.png](https://img-blog.csdnimg.cn/4d3bce05978e45cf92c298f057ae7c5d.png)

可以看到，我们很自然的匹配到了所有正文内容，并进行了分段。我们已经顺利获得了一个章节的内容，要想下载正本小说，我们就要获取每个章节的链接。我们先分析下小说目录 [URL](https://www.qb5.tw/book_116659/ "URL") 。

## 获取小说章节链接

![6541098673014d99986fa0d870289f9f.png](https://img-blog.csdnimg.cn/6541098673014d99986fa0d870289f9f.png)

通过开发者工具我们可以看到如下：

![a75dae98aa7b40e6a1204df7f67a28f5.png](https://img-blog.csdnimg.cn/a75dae98aa7b40e6a1204df7f67a28f5.png)

我们这里还是用正则表达式获取内容，我们要其中的url和章节标题。

```python
chapter_url = 'https://www.qb5.tw/book_116659/'

chapter_html = requests.get(chapter_url).text

info_list = re.findall('<dd><a href="(.*?)">(.*?)</a></dd>', chapter_html)
```

## 整本书获取

我们只需要for循环遍历第一章的代码就可以了，不过这里需要稍微变动一下。

```python
for info in info_list:

    url = 'https://www.qb5.tw/book_116659/' + info[0]
```

## 小说保存

最后，保存我们获取到的内容。

```python
open('宇宙职业选手.txt', mode='a', encoding='utf-8').write(text)
```

![b8c650c30ff54a7c888a195128de24c1.png](https://img-blog.csdnimg.cn/b8c650c30ff54a7c888a195128de24c1.png)

到这里，我们就可以免费读自己喜欢的小说了。

_**最后嘱咐大家一句，爬虫世界确实很有意思，技术是无罪的，学习是可以的，但还是实际操作就要适可而止了，不要触碰到法律的边界线。**_

# 总结

通过本文的学习，我们学习了数据采集。我们在采集数据的时候，遇到各种问题，自己在尝试解决问题，也是在一种学习，本次实战，我们明白如何使用正则表达式解析数据。今天就到这里，有什么问题，可以在评论区留言。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)