# 前言

**目录**

[前言](#%E5%89%8D%E8%A8%80%C2%A0)

[环境使用](#%E7%8E%AF%E5%A2%83%E4%BD%BF%E7%94%A8)

[模块使用](#%E6%A8%A1%E5%9D%97%E4%BD%BF%E7%94%A8)

[模块介绍](#%E6%A8%A1%E5%9D%97%E4%BB%8B%E7%BB%8D%C2%A0)

[代码实现](#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)

[1.确定我们的目标网址](#1.%E7%A1%AE%E5%AE%9A%E6%88%91%E4%BB%AC%E7%9A%84%E7%9B%AE%E6%A0%87%E7%BD%91%E5%9D%80)

[2.通过谷歌确定去访问](#2.%E9%80%9A%E8%BF%87%E8%B0%B7%E6%AD%8C%E7%A1%AE%E5%AE%9A%E5%8E%BB%E8%AE%BF%E9%97%AE)

[3.定位目标元素](#%C2%A03.%E5%AE%9A%E4%BD%8D%E7%9B%AE%E6%A0%87%E5%85%83%E7%B4%A0)

---

> 2022年已经过去，各大厂商都在做年度总结。某站在1月13日中午19点30分公布了2022百大UP主名单，那么今年的某站年度UP主都是谁呢？接下来就让我们一起了解一下吧。不过，我们去用python获取名单，是非常有成就的。

## **环境使用**

- python 3.9
- pycharm

## **模块使用**

 -    selenium
 -    谷歌驱动

```python
from selenium import webdriver
```

### 模块介绍

> - selenium
> 
> 之前，我们爬虫是模拟浏览器，但始终不是用的浏览器，但今天我们要说的是另一种爬虫方式，这次不是模拟浏览器，而是用程序去控制浏览器进行一些列操作，也就是selenium。selenium是python的一个第三方库，对外提供的接口可以操控浏览器，比如说输入、点击，跳转，下拉等动作。
> 
> 在使用selenium模块之前要做两件事，一是安装selenium模块，可以用终端用pip，也可以在pycharm里的setting安装；二是我们需要下载一款浏览器驱动程序，下载的驱动程序要和浏览器的版本一致。
> 
> - 谷歌驱动
> 
> 1.下载网址
> 
> CNPM Binaries Mirror
> 
> 2.文件安装（放置）位置  
> 可以把这个文件理解成一个脚本入口。说它是安装，其实就是把下载的 chromedriver.exe 文件复制到相应的位置。
> 
> 将文件复制到两个位置：1...\\python\\Scripts复制一份到安装Python的文件夹中的Scripts文件夹中；2.如果用的是Pycharm，再复制一份到..\\python\\site-packages\\selenium\\webdriver\\chrome文件中。这个地址可以将鼠标放在Pycharm里面安装库的地方的相应库上就能看到。

# 代码实现

今天这个目标要求特别简单，只要几行代码就能实现。话不多说，直接进入正题。

### 1.确定我们的目标网址

```python
目标网址：https://www.bilibili.com/blackboard/BPU2022-poweruplist.html
```

### 2.通过谷歌确定去访问

```python
driver = webdriver.Chrome()

driver.get('https://www.bilibili.com/blackboard/BPU2022-poweruplist.html')
```

效果如下：

![548f2eb04b75447b93ac97b4343a0e9b.png](https://img-blog.csdnimg.cn/548f2eb04b75447b93ac97b4343a0e9b.png)

### 3.定位目标元素

![9ce55a25548f4bf38172ad340b08ea90.png](https://img-blog.csdnimg.cn/9ce55a25548f4bf38172ad340b08ea90.png)

使用浏览器的开发者工具，我们找到了目标所在的位置，我们直接右击复制得到我们的selector。

```css
selector = #app > div > div.main-content > div.list > ul > li > p.name
```

我们用一行代码得到我们所需要的数据。

```python
names = driver.find_elements(By.CSS_SELECTOR,'#app > div > div.main-content > div.list > ul > li > p.name')
```

这是一个列表，我们都知道用for循环遍历。

```python
for name in names:
    print(name.text)
```

到这里，我们就用了6行代码就获取到了我们想要的数据，为了看到我们是不是获得了100位up主的数据，我们加一个计数，效果如下：

![ce3c11ae740641d194a5af76c6b7080a.png](https://img-blog.csdnimg.cn/ce3c11ae740641d194a5af76c6b7080a.png)

大家感兴趣的还可以获取up主的个人简介，什么照片的。下面这种效果也可以做，按字母排序。

> 2022百大up主:
> 
> A——矮乐多Aliga、AHALOLO、阿萨Aza
> 
> B——本喵叫兔兔、不刷题的吴姥姥
> 
> C——CSGO久菜合子、翠花不太脆
> 
> D——大狸子切切里、盗月社食遇记、电影最TOP
> 
> E——尔东和小明、二喵的饭
> 
> F——FoFTG、非非宇Fay、泛式、芳斯塔芙、范李猿
> 
> G——怪力老陈、尴尬的铁根er
> 
> H——HOPICO、侯翠翠、浑元Rysn、画渣不渣的三查、红警HBK08、HOLA小测佬
> 
> J——剑客范十三、极客湾Geekerwan、九三的耳朵不是特别好、嘉然今天吃什么
> 
> L——light是光华、-LKs-、老番茄、拉宏桑、利利那TD25、老实憨厚的笑笑、利维坦mY、老饭骨、凉风Kaze、绫人太太啊、罗翔说刑法、刘庸干净又卫生
> 
> M——Milk缪客、Mr迷瞪、猫不理咖啡、魔法Zc目录、某幻君、绵羊料理、木鱼水心
> 
> O——哦呼w、欧阳春晓Aurora
> 
> P——培根悖论唠唠嗑、怕上火暴王老菊、碰碰彭碰彭
> 
> Q——祁么么mo
> 
> S——STN工作室、Super也好君、山城小栗旬的理发日记、苏打baka、深海色带鱼、手工耿、设计师深海、酸梨大王、司墨尧smile、帅农鸟哥、森纳映画、 碎嘴企鹅、帅soserious、柿子菌meow
> 
> T——天才女友GG、谭乔、天真的和感伤的小说家
> 
> U——Upspeed盛嘉成
> 
> W——莴苣某人、王师傅和小毛毛、汪苏泷、网不红萌叔Joey、无穷小亮的科普日常
> 
> X——小潮院长、徐大虾咯、晓观队长、小Lin说、小片片说大片、吸奇侠、小王Albert、逍遥散人、星有野、咻咻满、小约翰可汗
> 
> Y——衣戈猜想、伊丽莎白鼠、影视飓风、雨说体育徐静雨、硬核的半佛仙人
> 
> Z——终极小腾、知了解压萌物、籽岷、真探唐仁杰、正直讲史-李正Str

![7be687bf9af64e0ca5cba789680e34ab.jpeg](https://img-blog.csdnimg.cn/7be687bf9af64e0ca5cba789680e34ab.jpeg)

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)