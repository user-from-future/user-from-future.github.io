上一篇我们说了requests的简单用法，知道了如何发送请求，今天我们更深层次的来学习requests。我们看看高级一点的操作，比如讲文件上传，cookies设置，代理设置之类的。

## 1.文件上传

我们知道requests可以模拟提交一些数据，比如讲，我们现在想上传文件，我们可以这样做。

```python
import requests

f = {'f':open('a.text','rb')}

r = requests.post('http://httpbin.org/post',files = f)

print(r.text)
```

运行一下，我们看效果。

![](https://img-blog.csdnimg.cn/ef5e6ad92af0460382b2832b5d5dc8e2.png)

我们可以看到里面包含了files的这个字段，而form这个字段是空的，这证明了文件上传部分会单独有一个files字段来标识。

## 2.cookies

我们可以用cookies来维持登录状态，在浏览器里面，在开发者工具里面，我们可以找到cookies字段，我们可以直接复制即可。

![](https://img-blog.csdnimg.cn/122b633c68f4403984bf59754f65ba88.png)

我们将cookies设置到headers里面，然后，发送请求，就可以登录了。

## 3.SSL证书验证

此外，requests还有证书验证的功能，当发送HTTP请求的时候，它会检查SSL证书，我们可以使用verify参数控制是否检查此证书。一般默认是打开的。

那我们的代码怎么写呢？

> response = requests.get\('http://www.baidu.com',verify = False\)

## 4.代理设置

对于一些网站，在测试的时候还能获取内容，一旦频繁爬取，就有可能被封IP，导致一段时间无法访问。那么，为了防止这种情况发生，我们就要设置代理来解决，这里就用到了proxies参数。我之前也有说过，我就不过多的赘述。

## 5.超时设置

在网路不好的时候，或者服务器响应太慢，甚至有时候还会报错，为了防止服务器不能及时响应，我们可以设置一个超时设置，这里就用到了timeout参数。

> response = requests.get\('http://www.baidu.com',timeout= 3\)

我们就简单介绍到这里，更多的可以关注官方的文档。