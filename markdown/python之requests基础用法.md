大家好，今天就来说说requests的基础用法。

## 1.准备工作

首先呢，我们要确保我们已经之前安装requests库，如果没有安装，可以自行搜索。

## 2.实例引入

requests库请求网页用到的是get\(\)方法，下面通过实例来看一下。

```python
import requests

res = requests.get('https://www.baidu.com/')

print(type(res))

print(res)

print(res.text)

print(res.cookies)

```

这里我们调用get\(\)方法实现，得到一个response对象，然后分别输出response的类型，状态码，内容以及cookies。

使用get\(\)方法成功实现一个get\(\)请求这不算什么，更方便的请求还有其他的。比如post\(\),put\(\)等等。

## 3.get\(\)请求

HTTP最常见的请求之一就是GET请求，下面我们首先先来了解一下利用requests构建GET的方法

### 基本实例

首先，我们构建一个最简单的get请求，请求的链接如下，该网站会判断如果用户发起的的是get请求的话，它就会返回响应的请求信息。

```python
import requests

res = requests.get('http://httpbin.org/get')

print(res.text)
```

运行的结果如下：

```python
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.27.1", 
    "X-Amzn-Trace-Id": "Root=1-637ae5d7-35da1bf57b139d152585d12a"
  }, 
  "origin": "223.215.67.113", 
  "url": "http://httpbin.org/get"
}
```

可以发现，我们成功发起了get请求，返回结果中包含请求头，url，IP等信息。

那么，对于GET请求，如果我们想要附加额外信息，一般怎么添加呢？比如讲，现在想添加两个参数，其中name是Tina，age是18。要构造这个请求链接，是不是可以直接写成：

> r = requests.get\('http://httpbin.org/get\?name=Tina\&age=18'\)

这样也是可以的，我们还可以通过字典来构造。利用params这个参数就好了。

```python
import requests

data = {

    'name':'Tina'，
    
    'age':'18'
    }

res = requests.get('http://httpbin.org/get',params = data)

print(res.text)
```

运行结果如下：

```python
{
  "args": {
    "age": "18", 
    "name": "Tina"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.27.1", 
    "X-Amzn-Trace-Id": "Root=1-637ae902-695483e87b26b3ad49d15df7"
  }, 
  "origin": "223.215.67.113", 
  "url": "http://httpbin.org/get?name=Tina&age=18"
}
```

通过运行结果判断，请求的链接自动变成了带有后缀的链接。

另外，网页返回的类型实际上是str，但是它是json\(\)格式的，我们可以用json返回一个字典。如果不是json格式，使用json就会报错，抛出json.decoder.JSONDecodeError异常。

## 4.post\(\)请求

上面我们了解了最基本的get请求，另外一种比较常见的请求方式就是post（）。使用requests实现post请求也是非常简单，示例如下。

```python
import requests

res = requests.post('http://httpbin.org/post')

print(res.text)
```

运行之后就会发现得到了结果，就说明我们post请求成功。

## 5.响应

发送请求，之后得到的肯定就是响应。除了text，还有状态码，响应头，cookies等等。