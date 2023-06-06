# 前言

![](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 今天，我们将采集某二手车数据，通过这个案例，加深我们对xpath的理解。通过爬取数据后数据分析能够直观的看到二手车市场中某一品牌的相对数据，能够了解到现在的二手车市场情况，通过分析数据看到二手车的走势，车商就可以利用这些数据进行定价，让想买二手车却不了解市场的人了解到大概的价格走势，到了店里不会被骗。

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

# 数据采集

## 发送请求

首先，我们要进行数据来源分析，知道我们的需求是什么？

### 明确需求:

- 明确采集网站是什么\?
- 明确采集数据是什么\?

车辆基本信息

然后，我们分析车辆基本信息数据, 具体是请求那个网址可以得到我们想要的数据。

> 通过开发者工具, 进行抓包分析:
> 
> 打开开发者工具: F12 / 鼠标右键点击检查选择network
> 
> 刷新网页: 让本网页数据内容重新加载一遍 \<方便分析数据出处>
> 
> 搜索数据来源: 复制你想要的内容, 进行搜索即可

```python
    import requests
    url = 'https://www.che168.com/china/a0_0msdgscncgpi1ltocsp1exx0/'
    header = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'}

    res = requests.get(url,headers=headers)
```

我们和之前一样，获取数据，我们会发现，车辆的基本信息就在网页源代码中，我们今天就用xpath的方法来解析数据。  

## 解析数据

接下来，我们用xpath解析数据。

![](https://img-blog.csdnimg.cn/d4827d84234c45c4bd2de8364be58328.png)

我们通过网页源代码，我们可以获取到每一个网页的url。

```python
    selector=parsel.Selector(res.text)

    detail_url_list = selector.xpath('//ul[@class="viewlist_ul"]/li/a[@class="carinfo"]/@href').getall()
```

我们可以看到，得到下面数据。

![](https://img-blog.csdnimg.cn/aa61e6ccde1247c39834383eaf260b11.png)

我们会发现，我们得到了两种网页，所以，在这里我们拼接网页就需要注意，这里，我不多说，直接看我是怎么写的。

```python
        if detail_url.split('/') == '':
            detail_url = 'https:'+detail_url
        else:
            detail_url = 'https://www.che168.com'+detail_url
```

这样，我们就得到了每一个车辆信息的数据网页，看看运行之后的效果吧。

![](https://img-blog.csdnimg.cn/afadd29b3c574975bb7d23f5d3de78d8.png)

接下来，我们就依次访问某个链接，获取我们想要的数据。

```python
    responses = requests.get(detail_url,headers=headers)
    detail_selector = parsel.Selector(responses.text)
```

![](https://img-blog.csdnimg.cn/848d63ff70dd4d289f218f3d24f73145.png)

我用不同颜色标注的，就是我们这次想要获取的数据，我们这里以车辆名称为例，讲解下path如何写。

```python
title = detail_selector.xpath('string(//h3[@class="car-brand-name"])').get("").strip()
```

我们看看网页源代码是如何得到的xpath。

![](https://img-blog.csdnimg.cn/bc462aed96b54ac99ede0eecfacb355a.png)

> 可能有人就要问了，这个
> 
> \(""\).strip\(\)
> 
> 是什么意思？这个就是去除空格的，只是为了后期数据的美观。

后面的我就不一一展示了，我直接放代码了，不懂的在评论区交流。

```python
tableShowMileage = detail_selector.xpath('//ul[@class="brand-unit-item fn-clear"]/li[1]/h4/text()').get("").strip()
theRegistrationTime = detail_selector.xpath('//ul[@class="brand-unit-item fn-clear"]/li[2]/h4/text()').get("").strip()
blockADisplacement = detail_selector.xpath('//ul[@class="brand-unit-item fn-clear"]/li[3]/h4/text()').get("").strip()
addr = detail_selector.xpath('//ul[@class="brand-unit-item fn-clear"]/li[4]/h4/text()').get("").strip()
guobiao = detail_selector.xpath('//ul[@class="brand-unit-item fn-clear"]/li[5]/h4/text()').get("").strip()
price = detail_selector.xpath('string(//span[@id="overlayPrice"])').get()
```

我们打印这些数据，看看效果吧。

![](https://img-blog.csdnimg.cn/de0e9c6993d9466ca4194fbfd0587186.png)

可能大家注意到了，有返回空值的，这个可能就是被反爬，大家感兴趣可以用代理IP试试。

## 保存数据

和我们上一篇一样，我们先写入字典，然后在写入csv文件里面。

```python
        dit ={
            '车辆':title,
            '表显里程':tableShowMileage,
            '上牌时间':theRegistrationTime,
            '挡位/排量':blockADisplacement,
            '车辆所在地':addr,
            '查看限迁地':guobiao,
            '价格':price,
        }
        
        csv_writer.writerow(dit)
```

大家感兴趣还可以获取车辆信息更详细的数据，其实原理都是一样的。

# 总结

通过本文的学习，我们学习了数据采集。我们在采集数据的时候，遇到各种问题，自己在尝试解决问题，也是在一种学习，本次实战，我们明白如何使用xpath解析数据。今天就到这里，有什么问题，可以在评论区留言。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)