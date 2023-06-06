# 前言

![1b83b1d3fff541e6844ba7bfc4b8f724.gif](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 大家好，我们今天来爬取热搜榜，把其文章名称，链接和作者获取下来，我们保存到本地，我们通过测试，发现其实很简单，我们只要简单获取数据就可以。没有加密的东西。

效果如下： ![a34f11a5373b465588f0ef9579007eb7.png](https://img-blog.csdnimg.cn/a34f11a5373b465588f0ef9579007eb7.png)

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

我们话不多说，我们先找到url，也就是请求地址。我们代码如下：

```python

url = 'https://blog.csdn.net/phoenix/web/blog/hot-rank?page=0&pageSize=25&type=' 
headers = {  
'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'}  
  
res = requests.get(url, headers=headers)

```

我们这里首先定义了一个 `url` 变量，它表示要访问的 URL。然后，它定义了一个 `headers` 变量，其中包含了一些 HTTP 请求头信息，如 `User-Agent` 表示 HTTP 请求的 User-Agent。最后，它使用 `requests.get()` 函数发送 HTTP GET 请求，并将 `headers` 变量作为参数传递给该函数。

# 解析数据

我们获取到了内容，接下来就是解析数据，我们不难发现这个是一个json数据，我们直接取值就好了，我们来看看代码怎么写。

```python
datas = res.json()['data']   
for data in datas:    
    period = data['period']  
    nickName = data['nickName']  
    articleTitle = data['articleTitle']  
    articleDetailUrl = data['articleDetailUrl']  
    viewCount = data['viewCount']  
    commentCount = data['commentCount']  
    favorCount = data['favorCount']  
    hotRankScore = data['hotRankScore']
    print(period,nickName,articleTitle,avatarUrl,viewCount,favorCount,commentCount,hotRankScore)
```

我们将从 `res.json()` 中获取 `data` 数据，并将其存储在 `datas` 变量中。 `res.json()` 返回的是一个包含多个字典的对象，每个字典代表一个数据。

在这个例子中， `res.json()` 返回的字典中的 `data` 字段的值为 ：

```python
[{'period': '1', 'nickName': '', 'articleTitle': '', 'articleDetailUrl': '', 'viewCount': '', 'commentCount': '', 'favorCount': '', 'hotRankScore': '0.08536632385314886', 'avatarUrl': 'null', 'viewCount': '0', 'favorCount': '0', 'commentCount': '0', 'hotRankScore': '0.08536633735229816'}]
```

我们使用这个数据来遍历 `datas` 变量中的每个字典。在每个字典中，我们使用 `data` 字段的值来获取期数、昵称、标题、详细URL、访问次数、评论次数、喜欢次数、热门排名分数。

# 保存数据

```python
now_time =time.strftime('%Y-%m-%d-%H-%M',time.localtime(time.time()))  
  
f = open(f'{now_time}热榜数据.csv', mode='a', encoding='utf-8', newline='')  
csv_writer = csv.DictWriter(f, fieldnames=['日期', '姓名', '文章标题', '文章链接', '浏览量',  
'评论量', '收藏量', '热榜值'])  
csv_writer.writeheader()

```

我们首先打开一个名为 `data.csv` 的文件，并指定使用 `a` 模式打开文件。然后，使用 `csv.DictWriter()` 函数创建一个 CSV 写入器，并指定要写入的列名。在这个例子中，我们指定了 `fieldnames` 参数，它包含了我们要写入的列名。

接下来，我们使用 `csv_writer.writeheader()` 方法写入列名。这个方法会将列名写入文件的第一行。

最后，我们使用 `csv_writer.writerow()` 方法写入数据。

我们先写入字典。

```python

dit = {'日期': period, '姓名': nickName, '文章标题': articleTitle, '文章链接': articleDetailUrl, '浏览量': viewCount,  
'评论量': commentCount, '收藏量': favorCount, '热榜值': hotRankScore}  
print(dit)  
csv_writer.writerow(dit)

```

这段代码创建了一个字典 `dit` ，其中包含了每个元素的值。然后，它使用 `csv_writer.writerow()` 方法将字典写入CSV文件中。

全部代码：

```python
import requests
import csv
import time

now_time =time.strftime('%Y-%m-%d-%H-%M',time.localtime(time.time()))

f = open(f'{now_time}热榜数据.csv', mode='w', encoding='utf-8', newline='')
csv_writer = csv.DictWriter(f, fieldnames=['日期', '姓名', '文章标题', '文章链接', '浏览量',
                                           '评论量', '收藏量', '热榜值'])
csv_writer.writeheader()



for page in range(0, 4):
    url = f'https://blog.csdn.net/phoenix/web/blog/hot-rank?page={page}&pageSize=25&type='
    # 'https://blog.csdn.net/phoenix/web/blog/hot-rank?page=0&pageSize=25&type='

    headers = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'}

    res = requests.get(url, headers=headers)

    # print(res.text)

    datas = res.json()['data']
    # print(datas)
    for data in datas:
        # print(data)
        # break
        period = data['period']
        nickName = data['nickName']
        articleTitle = data['articleTitle']
        articleDetailUrl = data['articleDetailUrl']
        viewCount = data['viewCount']
        commentCount = data['commentCount']
        favorCount = data['favorCount']
        hotRankScore = data['hotRankScore']

        # print(period,nickName,articleTitle,avatarUrl,viewCount,favorCount,commentCount,hotRankScore)

        dit = {'日期': period, '姓名': nickName, '文章标题': articleTitle, '文章链接': articleDetailUrl, '浏览量': viewCount,
               '评论量': commentCount, '收藏量': favorCount, '热榜值': hotRankScore}
        print(dit)
        csv_writer.writerow(dit)

```

# 总结

我们这样就获取到了内容，本文仅供学习。效果如下：

![f9715dedc16e4545a2542f0cdffc0eb0.png](https://img-blog.csdnimg.cn/f9715dedc16e4545a2542f0cdffc0eb0.png)

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)