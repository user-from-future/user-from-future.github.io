## 项目实战目录

[python项目实战——银行取款机系统（一）](https://blog.csdn.net/BROKEN__Y/article/details/127703108 "python项目实战——银行取款机系统（一）")

[python项目实战——银行取款机系统（二）](https://blog.csdn.net/BROKEN__Y/article/details/127709126 "python项目实战——银行取款机系统（二）")

[python项目实战——银行取款机系统（三）](https://blog.csdn.net/BROKEN__Y/article/details/127711247 "python项目实战——银行取款机系统（三）")

python项目实战——银行取款机系统（四）

**前言**

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests

上一篇我们说到了，分析了大致思路，今天，我们将具体实现其中一部分的功能—— **锁定** 。

## 锁定

一般来说，我们丢失银行卡后，会去银行挂失，也就是锁定，或者，我们的密码错误超过三次，就会被锁定，那么我们怎么实现呢。

第一步，输入我们的卡号。

```python
 def searchUserInfo(self):
        cardNum = input("请输入卡号：")
```

第二步，判断卡号是否存在，若存在，继续操作，反之，结束运行。

```python
# 验证是否存在卡号
        user = self.allUsers.get(cardNum)
 
        if not user:
            print("该卡号不存在！！！锁定失败")
            return -1
```

运行一下：  
![](https://img-blog.csdnimg.cn/fabf5b171ee1473da29fb5705ffe0a07.png)

第三步，我们看看卡的状态，是不是处于未锁定状态。如果被锁定了就没有必要再锁定了。

```python

        if user.card.cardLock:
            print("该卡已经被锁定！！！请解锁后在使用其他他功能")
            return -1
```

我们来看看锁定的卡号会是什么效果。

![](https://img-blog.csdnimg.cn/000f208127a1497e8f1012a5fb476133.png)

第四步，如果这些都没有问题，我们就进行下一步，验证密码。

```python
# 验证密码

        if not self.checkPasswd(user.card.cardPasswd):
            print("密码输入错误！！！锁定失败")
            return -1

        tempIdCard = input("请输入身份证号：")
        if tempIdCard != user.idCard:
            print("身份证号码输入错误！！！锁定失败")
            return -1
```

验证密码通过后，就可以操作锁定卡号。

```python
# 锁定
        user.card.cardLock = True
        print("锁定成功")
```

在这里，我们运行一下。

![](https://img-blog.csdnimg.cn/0d41d509e5854d15be4b6eb56c637dab.png)

到这里，锁定的所有功能都可以实现了。

我们下一期讲介绍关于解锁的代码相关讲解。