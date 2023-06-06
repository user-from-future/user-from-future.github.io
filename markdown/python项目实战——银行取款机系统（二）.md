## 项目实战目录

[python项目实战——银行取款机系统（一）](https://blog.csdn.net/BROKEN__Y/article/details/127703108 "python项目实战——银行取款机系统（一）")

python项目实战——银行取款机系统（二）

**前言**

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests
- random

上一篇我们说到了，分析了大致思路，今天，我们将具体实现其中一部分的功能—— 开户 。

## 开户

首先，是我们的开户功能，开户首先需要用户的姓名，身份证号码，电话号码之类，卡里面一般有卡号，余额，卡的密码，卡的状态。

### 用户

```python
class User(object):
    def __init__(self,name,idCard,phone,card):
        self.name=name
        self.idCard=idCard
        self.phone=phone
        self.card=card


```

### 卡

```python
class Card(object):
    def __init__(self,cardId,cardPasswd,cardMoney):
        self.cardId=cardId
        self.cardPasswd=cardPasswd
        self.cardMoney=cardMoney
        self.cardLock = False
```

在写完什么的代码后，我们就可以写开户的代码了。通过字典的方式把用户信息存储。

```python
 #开户
    def createUser(self):
        #目标：向用户字典中添加一对键值对（卡号-用户）
        name=input("请输入姓名：")
        idCard=input("请输入身份证号码：")
        phone=input("请输入电话号码：")

        prestoreMoney=int(input("请输入存款金额："))
        if prestoreMoney<0:
            print("预存款输入失误！！开户失败")
            return -1
```

我们来运行一下。

![](https://img-blog.csdnimg.cn/4e40c8f2345b4e4489820ed567534f25.png)

这里，存款金额我们设置了是正数，负数是会报错的。

接下来，根据我们的生活经验，接下来就是要我们设置密码

```python
onePasswd=input("请设置密码：")

        #验证密码
        if not self.checkPasswd(onePasswd):
            print("密码输入错误！！！开户失败")
            return -1
```

密码错误三次后会提示。

![](https://img-blog.csdnimg.cn/114e435ac87b4ec5be5f98a12f4206e7.png)

密码输入成功后，银行就会给你一个卡号。

```python
cardStr = self.randomCardID()
        #print(cardId)


        card=Card(cardStr,onePasswd,prestoreMoney)
        user=User(name,idCard,phone,card)#创建用户

        #存到字典
        self.allUsers[cardStr]=user
        print(f"开户成功！！！请牢记卡号{cardStr}")
```

我们运行一下。

![](https://img-blog.csdnimg.cn/0734ca66c536492e88fc499f1c3baeca.png)

一般卡号都是随机生成的，代码实现如下

```python
 #生成卡号
    def randomCardID(self):

        while True:
            str = ""
            for i in range(6):
                ch = chr(random.randrange(ord('0'), ord('9') + 1))
                str += ch
            #判断是否重复
            if not self.allUsers.get(str):
                return str
```

在这里，我们不管进行什么操作，都是需要输入密码，一般密码错误三次，就会被锁定，这些功能python也可以实现，我们会在后期会讲到，这里，就主要说说密码验证的问题。

```python
# 验证密码
    def checkPasswd(self, readPasswd):
        for i in range(3):
            tempPasswd = input("请输入密码：")
            if tempPasswd == readPasswd:
                return True
        return False
```

下一期我们将会讲解如何写查询的相关代码。