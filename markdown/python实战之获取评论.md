## 前言

**目录**

[前言](#%E5%89%8D%E8%A8%80)

[开发工具](#%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7)

[环境搭建](#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)

---

利用 Python 爬取音乐评论。废话不多说。

让我们愉快地开始吧\~

## 开发工具

- Python 版本：3.6.4

**相关模块：**

1.  requests 模块；
2.  re 模块；
3.  pymysql 模块；
4.  以及一些 Python 自带的模块。

## 环境搭建

安装 Python 并添加到环境变量，pip 安装需要的相关模块即可。

通过这次爬取，学习了数据库 MySQL，因为之前都是在 windows 上操作，而这回需要在 Mac 上操作，所以就在 Mac 上安装了 MySQL 以及 MySQL 的管理工具 Sequel Pro，最后也是安装成功，数据库连接也没有问题。

![](https://img-blog.csdnimg.cn/img_convert/7bf72d58c6d177f076439a36f41fb4b2.jpeg)

接下来创建数据库，表格及主键信息。

```
import pymysql
```

```
import pymysql
```

针对音乐中去年夏天的网页进行分析，查看了所有评论的尾页，发现时间缩水了，因为热评中有一条评论的时间 7 月 12 号，而所有评论最后一页的时间却是 7 月 16 号。很明显，所有评论并不是货真价实的所有评论，不知这算不算音乐的 BUG。

还有一个就是直接点击最后一页的时候，并不能直接返回真正的信息，需要从最后一页往前翻，到了真正的信息页时，然后再往后翻，才能得到最后一页的真正信息。

![](https://img-blog.csdnimg.cn/img_convert/b53ccfad16888a818cefe9f1b396e301.jpeg)

同样是 Ajax 请求，确认网址后，分析一下请求头，发现主要是三个参数发生变化：

jsoncallback

pagenum

lasthotcommentid

pagenum 不难理解，就是页数。jsoncallback 经过实验后，发现并不会影响请求，所以设置时无需改动，lasthotcommentid 的值对应的是上一页最后一个评论者的 ID，所以需要随时改动。

即改变 pagenum，lasthotcommentid 的值，就可成功实现请求。

![](https://img-blog.csdnimg.cn/img_convert/439d71bcda00ecf51347c4511bb8f6bc.jpeg)

![](https://img-blog.csdnimg.cn/img_convert/bc1e0c01773410e71e0175fada8ceb2a.jpeg)

代码如下：

```python
import re
import json
import time
import pymysql
import requests

URL = 'https://c.y.qq.com/base/fcgi-bin/fcg_global_comment_h5.fcg?'

HEADERS = {
    'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36'
}

PARAMS = {
    'g_tk': '5381',
    'jsonpCallback': 'jsoncallback4823183319594757',
    'loginUin': '0',
    'hostUin': '0',
    'format': 'jsonp',
    'inCharset': 'utf8',
    'outCharset': 'GB2312',
    'notice': '0',
    'platform': 'yqq',
    'needNewCode': '0',
    'cid': '205360772',
    'reqtype': '2',
    'biztype': '1',
    'topid': '213910991',
    'cmd': '8',
    'needmusiccrit': '0',
    'pagenum': '0',
    'pagesize': '25',
    'lasthotcommentid': '',
    'callback': 'jsoncallback4823183319594757',
    'domain': 'qq.com',
    'ct': '24',
    'cv': '101010',
}

LAST_COMMENT_ID = ''

db = pymysql.connect(host='127.0.0.1', user='root', password='774110919', port=3306,  db='QQ_Music', charset='utf8mb4')
cursor = db.cursor()

def get_html(url, headers, params=None, tries=3):
    try:
        response = requests.get(url=url, headers=headers, params=params)
        response.raise_for_status()
        response.encoding = 'utf-8'
    except requests.HTTPError:
        print("connect failed")
        if tries > 0:
            print("reconnect...")
            last_url = url
            get_html(last_url, headers, tries-1)
        else:
            print("3 times failure")
            return None
    return response

def paras_html(html):
    data = {}
    content = json.loads(html[29:-3])
    for item in content['comment']['commentlist']:
        data["nike"] = item.get("nick")
        data["comment"] = re.sub(r"\\n", " ", item.get("rootcommentcontent"))
        data["comment"] = (re.sub(r"\n", " ", data["comment"]))[0:255]
        data["praisenum"] = item.get("praisenum")
        data["comment_id"] = item.get("commentid")
        data["time"] = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(int(item.get("time"))))
        yield data

def to_mysql(data):
    table = 'comments'
    keys = ', '.join(data.keys())
    values = ', '.join(['%s'] * len(data))
    sql = 'INSERT INTO {table}({keys}) VALUES ({values}) ON DUPLICATE KEY UPDATE'.format(table=table, keys=keys, values=values)
    update = ','.join([" {key} = %s".format(key=key) for key in data])
    sql += update
    try:
        if cursor.execute(sql, tuple(data.values())*2):
            print('Successful')
    except:
        print('Failed')
        db.rollback()
    db.commit()

if __name__ == '__main__':
    main()
```