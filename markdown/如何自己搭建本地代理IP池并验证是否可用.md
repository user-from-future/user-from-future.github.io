大家好，我们上一篇讲到关于反爬的相关知识。其中有一种就是检测IP访问次数的，大家都知道用IP代理池。网络上有付费的，也有免费的，这些东西容易失效，需要的自行获取，这里就不多赘述。

代理IP在网络爬虫、网络营销、电商等互联网领域得到了广泛的应用，通过代理IP能够轻松解决IP限制的问题。很多有技术的同学搭建了属于自己的代理IP池，当然购买付费代理IP的占大多数。

不过付费代理IP也是从代理IP服务商的代理IP池中提取出来的。那么代理IP池是什么样子的呢\?

所谓代理IP池，其实就是将大量可用的代理IP汇聚到一起，通过技术手段实现管理和提取。代理IP池中的代理IP都是有生命周期的，一旦失效或者临近失效就会被清理出代理IP池。

当然，我们不用担心代理IP池中的代理IP都被提取或者失效的问题，因为代理IP池是有相关技术人员维护的，会有源源不断的代理IP补充进来。我们在提取代理IP时，是根据代理IP平台的规则来提取的，在规则的允许范围内可以随意提取。

很多小伙伴自建的代理IP池时通过工具收集的免费代理IP，其实用免费代理IP搭建代理IP池是一件比较费心费力还费时的事情，当然对于想去研究代理IP池搭建技术小伙伴，还是非常赞同通过免费代理IP来搭建代理IP池的。

大家获取之后，可以保存一个txt文件。或者放在程序里面，然后直接import调用即可，大家可以自己根据习惯学择。

```python
PROXIES = [
    {'http':'47.101.44.122:80'},
    {'http':'116.63.93.172:8081'},
    {'http':'42.180.208.43:8070'},
    {'http':'60.255.151.82:80'},
    ]
]
```

有些IP具有失效性，大家需要自行更新这些，也可以用代码是实现，关于验证IP是否可用的部分代码如下。

## 补充学习：

### 什么是代理ip池？

通俗地比喻一下，它就是一个池子，里面装了很多代理ip。它有如下的行为特征：

1.  池子里的ip是有生命周期的，它们将被定期验证，其中失效的将被从池子里面剔除。
2.  池子里的ip是有补充渠道的，会有新的代理ip不断被加入池子中。
3.  池子中的代理ip是可以被随机取出的。

这样，代理池中始终有多个不断更换的、有效的代理ip，且我们可以随机从池子中取出代理ip，然后让爬虫程序使用代理ip访问目标网站，就可以避免爬虫被ban的情况。

今天，我们就来说一下如何构建自己的代理ip池。而且，我们要做一个比较灵活的代理池，它提供两种代理方式：

```python
use = []

for proxies in proxies_list:

    try:

        response = requests.get('https://www.baidu.com',proxies=proxies,timeout=0.1)

        if response.status_code == 200:

            use.append(proxies)

    except Exception as e:

        print(e)

return use
```

那我们做好了这些，如何调用呢。

这里就需要我们从文件中随机取一个IP访问网址，这里用到了random库，有不会安装的，自行百度。

```python
import requests

f = open('IP.txt',"r")

file = f.readline

item = []

for proxies in file:

    proxies =eval(proxies.replace('\',''))

    item.append(proxies)

proxies = random.choice(item)

response = requests.get(url=url,headers=headers,proxies=proxies)

print(response)
```

那我们在添加了代理IP之后怎么验证使用的ip是否可用？这里我们可以通过访问IP检测网址验证：http://current.ip.16yun.cn:802，只要返回的是代理IP那么就证明代理使用成功了，我们可以直接去访问需要获取的数据网站了。

今天就分享到这里。