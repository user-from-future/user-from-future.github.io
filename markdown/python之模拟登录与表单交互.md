无论是简单网页还是采用异步加载技术的网页，都是通过GET方法请求网址来获取网页信息的。但如何通过获取登录表单后的信息的？本节将讲解Reqquests库的Post方法，通过观测表单代码和逆向工程来填写表单以获取网页信息，以及通过提交Cookie信息来模拟登录网站。

本文的主要知识点如下：

表单交互：利用Requests库的POST方法进行表单交互

Cookie：了解Cookie的基本概念

模拟登录：学会利用Cookie信息模拟登录网站

**模拟登录**  
有时，表单字段可能通过加密或者其他形式进行包装。这就增大了构造表单的难度，这是可选择提交Cookie信息进行模拟登录。

**Cookie概述**  
Cookie，指某些网站为了辨别用户身份、进行session跟踪而存储在本地终端上的数据。互联网购公司通过追踪用户的Cookie信息，给用户提供相关兴趣的商品。同样，因为Cookie保存了用户的信息，我们便可通过提交Cookie来模拟登陆网站。

**提交Cookie模拟登录**  
下面以某网为例，查找Cookie信息并提交来模拟登录药智网。

（1）进入某网，打开Chrome浏览器的开发者工具，选择Network选项。

（2）手工输入账号和密码进行登录，此时会发现Network中会加载许多文件。

（3）这时并不需要查看登录网页的文件信息，而是直接查看登陆后的文件信息

\(1\)将Cookie添加到headers中进行传参

```python
import requests
 
member_url = 'https://www.yaozh.com/member/'
headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36',
    'Cookie': '_ga=GA1.2.1508206450.1629445703; UtzD_f52b_saltkey=NzQM8wJu;UtzD_f52b_lastvisit=1629442261; yaozh_uidhas=1; UtzD_f52b_ulastactivity=1629445859%7C0; _gid=GA1.2.468845100.1630220751; yaozh_userId=828458; PHPSESSID=q3p104a4m5oidc0ftanp7n4rk7; Hm_lvt_65968db3ac154c3089d7f9a4cbb98c94=1629705497,1629792799,1630220751,1630292865; yaozh_mylogin=1630304933; UtzD_f52b_creditnotice=0D0D2D0D0D0D0D0D0D721338;
UtzD_f52b_creditbase=0D0D6D0D0D0D0D0D0;UtzD_f52b_creditrule=%E6%AF%8F%E5%A4%A9%E7%99%BB%E5%BD%95;acw_tc=707c9f9816303118236322722e19d2bb3bf070bb34e00eed1b4400b5b6ab67;UtzD_f52b_lastact=1630311824%09uc.php%09; Hm_lpvt_65968db3ac154c3089d7f9a4cbb98c94=1630311838'
}
 
response = requests.get(member_url,headers=headers)
 
print(response.text)
```

  
大家要注意一点，有时候你直接复制的Cookie值是不正确的，如果这时候出现错误就需要你去网站进行核对。

（2）Cookie直接作为参数传参

```python
member_url = 'https://www.yaozh.com/member/'
 
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.67 Safari/537.36'
}
 
Cookies = '_ga=GA1.2.1508206450.1629445703; UtzD_f52b_saltkey=NzQM8wJu; UtzD_f52b_lastvisit=1629442261; yaozh_uidhas=1; UtzD_f52b_ulastactivity=1629445859|0; _gid=GA1.2.468845100.1630220751; yaozh_userId=828458; PHPSESSID=q3p104a4m5oidc0ftanp7n4rk7; Hm_lvt_65968db3ac154c3089d7f9a4cbb98c94=1629705497,1629792799,1630220751,1630292865; yaozh_mylogin=1630304933; UtzD_f52b_creditnotice=0D0D2D0D0D0D0D0D0D721338; UtzD_f52b_creditbase=0D0D6D0D0D0D0D0D0; UtzD_f52b_creditrule=每天登录; acw_tc=707c9f9816303118236322722e19d2bb3bf070bb34e00eed1b4400b5b6ab67; UtzD_f52b_lastact=1630311824    uc.php    ; Hm_lpvt_65968db3ac154c3089d7f9a4cbb98c94=1630311838'
 
"""
#需要的是字典
cook_dict = {}
cookies_list = cookies.split('; ')
for cookie in cookies_list:
    cook_dict[cookie.split('=')[0]] = cookie.split('=')[1]
"""
 
# 字典推导式
cook_dict = {cookie.split('=')[0]:cookie.split('=')[1] for cookie in cookies.split('; ')}
 
response = requests.get(member_url, headers=headers, cookies=cook_dict)
 
data = response.content.decode()
```

**表单交互**  
本文将讲解Requests库的POST使用方法，通过观测表单的网页源代码进行表单的提交，最后通过逆向工程的方法获取表单提交的字段，进而进行表单交互。

**POST方法**  
Requests库的POST使用方法简单，只需简单传递一个字典结构的数据给data参数。这样，在发起请求时会自动编码为表单形式，以此来完成表单的填写。

```python
import requests
 
params = {
    'key':'value1',
    'key':'value2',
    'key':'value3'
}
 
html = requests.post(url,data=params)      #post方法
 
print(html.text)      
 
```

  
查看网页源代码提交表单  
（1）打开某网，定位到登陆位置，利用Chrome浏览器进行“检查”，找到登录元素所在的位置，如下图：

这里要注意几个参数，第一就是咱们的username、pwd，第二就是最后两行的formhash、backurl，因为四个参数是From-Data里面的参数是非常重要的.

（2）构造代码

```python
import requests
 
url = 'https://www.yaozh.com/login'   #登录的网址
headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36'
}      
 
data = {     #传递的参数
    'username':'18303056364',
    'pwd':'qfkjr8yn',
    'formhash':'5AE08D06CB',
    'backurl':'https%3A%2F%2Fwww.yaozh.com%2F'
}
 
session = requests.session()   #建立session会话
r = session.post(url,data=params,headers=headers)  #带着参数去网址进行登录
 
member_url= 'https://www.yaozh.com/member/'    #登录以后要访问的页面
r2 = session.get(member_url,headers=headers)   #带着cookie访问个人中心
print(r2.text) 
```

这里面有一个点需要提示，就是formhash和backurl这两个元素的值需要我们找登陆前的。在未登录的网页源代码中可找到。formhash是会根据时间而改变的。