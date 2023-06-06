写程序的时候，别人写好的工具、库、框架被称作轮子。有现成的，写好的东西不用，自己又去写一遍，这叫做重复造轮子。今天来认识爬虫的常见轮子Scrapsdy。

我们在学习Scrapy的时候要先做好准备工作，安装相关库文件。

Scrapy安装前需要先安装依赖包，不然会因缺少依赖包导致安装失败，如下表

<table border="1" cellpadding="1" cellspacing="1"><tbody><tr><td>lxml</td><td>parsel</td><td>w3lib</td><td>twisted</td><td>cryptography</td><td>pyOenSSL</td></tr><tr><td>解析XML和HTML非常高校的工具</td><td>HTML.XML数据提取</td><td>网页解码</td><td>异步网络编程框架</td><td>用于加密</td><td>进行一些加解密操作</td></tr></tbody></table>

这些包可以通过pip单独安装。也可以先创建一个requirements.txt文件，然后把包名写入并保存即可

> lxml  
> parsel  
> w3lib  
> twisted  
> crptography  
> pyOpenSSL

然后终端输入

```python
 pip install -r requirements.txt
```

**什么是scrapy框架？**

Scrapy是python开发的一个快速、高层次、轻量级的屏幕抓取和web抓取的python爬虫框架，主要用于抓取特定web站点的信息并从中提取特定结构的数据。

**各组件作用**

**ScrapyEngine \(引擎\)**

负责Scheduler、Downloader、Spiders、Item Pipeline中间的通讯信号和数据的传递，此组件相当于爬虫的“大脑”，是整个爬虫的调度中心。

**Scheduler\(调度器\)**

简单地说就是一个队列，负责接收引擎发送过来的 request请求，然后将请求排队，当引擎需要请求数据的时候，就将请求队列中的数据交给引擎。

初始的爬取URL和后续在页面中获取的待爬取的URL将放入调度器中，等待爬取，同时调度器会自动去除重复的URL（如果特定的URL不需要去重也可以通过设置实现，如post请求的URL）。

**下载器\(Downloader\)**

下载引擎发送过来的所有 request请求，并将获得的 response交还给引擎，再由引擎将 response交管给 Spiders来进行解析。

**Spiders\(爬虫\)**

它负责处理所有responses，从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler\(调度器\)。

**Item Pipeline\(项目管道\)**

就是我们封装去重类、存储类的地方，负责处理 Spiders中获取到的数据并且进行后期的处理，过滤或者存储等等。

当页面被爬虫解析所需的数据存入Item后，将被发送到项目管道\(Pipeline\)，并经过几个特定的次序处理数据，最后存入本地文件或存入数据库。

**Downloader Middlewares\(下载中间件\)**

可以当做是一个可自定义扩展下载功能的组件，是在引擎及下载器之间的特定钩子\(specific hook\)，处理Downloader传递给引擎的response。

通过设置下载器中间件可以实现爬虫自动更换user-agent、IP等功能。

**Spider Middlewares\(爬虫中间件\)**

Spider中间件是在引擎及Spider之间的特定钩子\(specific hook\)，处理spider的输入\(response\)和输出\(items及requests\)。

自定义扩展、引擎和Spider之间通信功能的组件，通过插入自定义代码来扩展Scrapy功能。

下面通过具体的实例说说spider

```python
import scrapy
from myproject.items import MyprojectItem
 
class MySpider(scrapy.Spider):
#通过xpath提取内容
    name = "qiushibaike"
 
    start_urls = ["https://www.qiushibaike.com/text"]
 
    def parse(self, response):
        contents = response.selector.xpath('//div[@class="content"]/span/text()').extract()
        #定义items作为数据暂存容器
        items = MyprojectItem()
        for i in contents:
            items['content'] = i.strip()
            yield items
           #通过生成器yield将数据传送到Pipeline进一步处理
 
        self.log('A response from %s just arrived!' % response.url)
```

今天就说到这里。

j