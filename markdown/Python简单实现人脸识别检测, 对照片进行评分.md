大家好，今天和大家说说如何用Python简单实现人脸识别检测, 对照片进行排名，看看自己有多漂亮。  
  
\[开发环境\]:

> Python 3.8  
> Pycharm 2021.2

\[模块使用\]:

```
    requests >>> pip install requests
    tqdm >>> pip install tqdm 简单实现进度条效果
    os
    base64
```

模块安装问题:  
\- 如果安装python第三方模块:  
1\. win + R 输入 cmd 点击确定, 输入安装命令 pip install 模块名 \(pip install requests\) 回车  
2\. 在pycharm中点击Terminal\(终端\) 输入安装命令  
\- 安装失败原因:  
\- 失败一: pip 不是内部命令  
解决方法: 设置环境变量  
  
\- 失败二: 出现大量报红 \(read time out\)  
解决方法: 因为是网络链接超时,  需要切换镜像源  
清华：https://pypi.tuna.tsinghua.edu.cn/simple  
阿里云：http://mirrors.aliyun.com/pypi/simple/  
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/  
华中理工大学：http://pypi.hustunique.com/  
山东理工大学：http://pypi.sdutlinux.org/  
豆瓣：http://pypi.douban.com/simple/  
例如：pip3 install \-i https://pypi.doubanio.com/simple/ 模块名  
  
\- 失败三: cmd里面显示已经安装过了, 或者安装成功了, 但是在pycharm里面还是无法导入  
解决方法: 可能安装了多个python版本 \(anaconda 或者 python 安装一个即可\) 卸载一个就好。或者你pycharm里面python解释器没有设置好。

在弄好这些之后，我们要借用百度的一些接口来实现。

![](https://img-blog.csdnimg.cn/0cbff259b7794e118fd232a95fda9440.png)

点击控制台，然后点击产品服务，搜索人脸识别，就可以获取接口了。

```python
 host = 'https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=[官网获取的AK]&client_secret=[官网获取的SK]'
    response = requests.get(host)
    access_token = response.json()['access_token']
```

选择图片

```python
f = open(f'{ImgName}', mode='rb')
```

图片转码

```python
 img_base64 = base64.b64encode(f.read())
```

就会返回字符串，里面有个颜值评分，我们提取出来即可。

```python
params = {
            "image": img_base64,  # 需要传递 图片 base64
            "image_type": "BASE64",
            "face_field": "beauty"
        }
```

这样，我们就获取的了颜值评分。大家还可以在这个基础上，实现对多个照片评分并排名。