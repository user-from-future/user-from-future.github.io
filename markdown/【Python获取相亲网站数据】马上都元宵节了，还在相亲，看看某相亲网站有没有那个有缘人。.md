# 前言

![1b83b1d3fff541e6844ba7bfc4b8f724.gif](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 马上都元宵节了，还在相亲，看看某相亲网站有没有那个有缘人。今天我们来爬取某相亲网站获取我们想要的数据，比如说，对方的姓名，年龄，身高，体重等等。今天我们主要使用CSS选择的方法来匹配我们想要的数据，通过这篇的学习，可以加深大家对CSS的用法的了解，以及明白不同于正则匹配的地方。话不多说，让我手把手教你，如何获取吧。

### **环境使用**

- python 3.9
- pycharm

### **模块使用**

- requests
- re
- csv
- os
- parsel

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

### **模块安装问题:**

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

## 代码实现

### 发送请求

首先我们需要确定我们的 [目标网址](https://love.19lou.com/recommend "目标网址") 。

![a56cafba3bc942cd998a25c4f5986dcf.png](https://img-blog.csdnimg.cn/a56cafba3bc942cd998a25c4f5986dcf.png)

### 获取数据

我们通过开发者工具会发现，每一个女嘉宾都是有自己单独的网页，而且其网页构成规律明显，都是由固定网页＋uid构成，所以我们只需要找到每一个女嘉宾对应的uid即可。

> <https://love.19lou.com/detail/51639237>
> 
> <https://love.19lou.com/detail/51404458>
> 
> <https://love.19lou.com/detail/51371926>

接下来我们目标明确，获取女嘉宾的uid。

第一种方法，直接在网页源代码中获取uid。

```
respnse = requests.get(url, headers=headers)

    # print(respnse.text)
    uids = re.findall('uid:(\d+)', respnse.text, re.S)
    print(uids)
```

第二种方法，通过找到对应的数据包，里面就有女嘉宾的uid。

```python
# 第二种方法获取uid，即数据包,xhr刷新。得到下面链接
# https://love.19lou.com/valueApp/api/love/searchLoveUser?page=2&perPage=12&sex=0
    #https://love.19lou.com/valueApp/api/love/searchLoveUserpage=1&perPage=12&sex=0&minWeight=31&maxAge=59
    url1 = 'https://love.19lou.com/valueApp/api/love/searchLoveUser?page=2&perPage=12&sex=0'
    respnse1 = requests.get(url1,headers=headers)
    #第1种写法，有局限性
    uids=respnse1.json()['data']['items']
    print(uids)
    for i in range(0,12):
         uid = uids[i]['uid']
         print(uid)
    #第2种写法
    for index in respnse1.json()['data']['items']:
         print(index['uid'])
```

构成我们想要的女嘉宾的网页。

```python
    for uid in uids:
        html_url = 'https://love.19lou.com/detail/' + uid
        print(html_url)
```

### 解析数据

我们请求我们刚刚得到的html\_url，并做数据转换。

```python
html_response = requests.get(html_url, headers=headers)
        # print(html_response)
        html_data = html_response.text
        selector = parsel.Selector(html_data)
        # css选择器，根据标签属性内容提取数据
        # 会找标签
```

今天我们就不用正则去匹配我们想要的数据，用CSS去匹配我们想要的数据。

![9901ef60d37644ed861e850871f5bb7f.jpg](https://img-blog.csdnimg.cn/9901ef60d37644ed861e850871f5bb7f.jpg)

其他的数据内容获取都是类似的，这里我们就不做过多的赘述。直接上代码。

```python
        name = selector.css('.username::text').get()
     
        # print(name)
        info_list = selector.css('.info-tag::text').getall()
        
        sex = info_list[0].split('：')[1]
        age = info_list[1].split('：')[1]
        height = info_list[2].split('：')[1]

        birth = info_list[-1].split('：')[1]
```

在这里我和大家简单解释一下里面部分代码（info\_list\[0\].split\('：'\)\[1\]）的含义。

> \# 提取性别”女“字  
> \# print\(info\_list\)#\['性别：女', '年龄：29岁', '身高：164cm', '体重：45kg', '出生年份：1994'\]  
> \# print\(info\_list\[0\]\)#性别：女  
> \# print\(info\_list\[0\].split\('：'\)\)#\['性别', '女'\]  
> \# print\(info\_list\[0\].split\('：'\)\[1\]\)#女

接下来我们获取其个人基本信息。

```python
        basic_list = selector.css('.basic-item span::text').getall()
        # basic_list = selector.css('.basic-item span::text').getall()[2:]#切片
        # print(basic_list)
        shuxiang = basic_list[2].split('：')[1]
        xingzuo = basic_list[3].split('：')[1]
        nativePlace = basic_list[4].split('：')[1]
        location = basic_list[5].split('：')[1]
        edu = basic_list[6].split('：')[1]
        marriage = basic_list[7].split('：')[1]
        job = basic_list[8].split('：')[1]
        money = basic_list[9].split('：')[1]
        house = basic_list[10].split('：')[1]
        car = basic_list[11].split('：')[1]
```

### 输出数据

```python
print(name, sex, age, height, weight, birth, shuxiang, xingzuo, nativePlace, location, edu, marriage, job, money,
              house, car)
```

### 保存数据

我们把这些数据保存的csv文件里面，并把女嘉宾的图片保存下来。

```python
        dit = {
            '昵称': name,
            '性别': sex,
            '身高': height,
            '体重': weight,
            '出生年份': birth,
            '生肖': shuxiang,
            '星座': xingzuo,
            '籍贯': nativePlace,
            '所在地': location,
            '学历': edu,
            '婚姻状况': marriage,
            '工作': job,
            '年收入': money,
            '房产': house,
            '车辆': car,
            '详情页': img_url,
        }
        csv_writer.writerow(dit)
        
        
```

保存图片就比较简单了，和之前一样，requests，然后二进制保存数据。

```python
        img_url = selector.css('.page .left-detail .abstract .avatar img::attr(src)').get()
        # img_url = selector.css('.page .left-detail .abstract .avatar img')
        content = requests.get(img_url, headers=headers).content
        with open('图片\\' + str(name) + '.jpg', mode='wb') as f:
            f.write(content)
```

## 运行结果展示

![8e3adb08710e4fefa83cc1f5aca485f6.png](https://img-blog.csdnimg.cn/8e3adb08710e4fefa83cc1f5aca485f6.png)

![0f02de804a9840f6b7051bff4e3030a2.png](https://img-blog.csdnimg.cn/0f02de804a9840f6b7051bff4e3030a2.png)

![285a3f92182846339efceb5b5e005c1c.png](https://img-blog.csdnimg.cn/285a3f92182846339efceb5b5e005c1c.png)

相信大家跟着我一步一步操作，也做出了和我一样的效果，是不是很简单。

如果有不会的可以在评论区留言。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg) ​