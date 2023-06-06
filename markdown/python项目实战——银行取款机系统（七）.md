## 项目实战目录

[python项目实战——银行取款机系统（一）](https://blog.csdn.net/BROKEN__Y/article/details/127703108 "python项目实战——银行取款机系统（一）")

[python项目实战——银行取款机系统（二）](https://blog.csdn.net/BROKEN__Y/article/details/127709126 "python项目实战——银行取款机系统（二）")

[python项目实战——银行取款机系统（三）](https://blog.csdn.net/BROKEN__Y/article/details/127711247 "python项目实战——银行取款机系统（三）")

[python项目实战——银行取款机系统（四）](https://blog.csdn.net/BROKEN__Y/article/details/127722125 "python项目实战——银行取款机系统（四）")

[python项目实战——银行取款机系统（五）](https://blog.csdn.net/BROKEN__Y/article/details/127722405 "python项目实战——银行取款机系统（五）")

[python项目实战——银行取款机系统（六）](https://blog.csdn.net/BROKEN__Y/article/details/127738787 "python项目实战——银行取款机系统（六）")

python项目实战——银行取款机系统（七）

**前言**

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests

上一篇我们说到了，分析了大致思路，今天，我们将具体实现其中一部分的功能——存款。

## **存款**

当工资发下来后，我们会第一时间去ATM机存款，那么，我们在python怎么实现呢。

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
            print("该卡号不存在！！！存款失败")
            return -1
```

![](https://img-blog.csdnimg.cn/f5c239e9aeea44178c52a78c737fa5d2.png)

第三步，我们看看卡的状态，是不是处于未锁定状态。如果没有被锁定了就肯定存不了钱了啊。

```python
        if user.card.cardLock:
            print("该卡已锁定！请解锁后再使用其功能！")
            return -1
```

![](https://img-blog.csdnimg.cn/93b6f2614e3341efbfa0a881f68a7ee6.png)

第四步，如果这些都没有问题，我们就进行下一步，验证密码。

```python
# 验证密码
        if not self.checkPasswd(user.card.cardPasswd):
            # print("密码输入错误！！！查询失败")
            print("密码输入错误次数超过三次,该卡已被锁定,请解锁后操作")
            user.card.cardLock = True
            return -1
```

密码正确后，我们就可以存钱了。（不过，我们这里存在一个逻辑错误，存款好像是不需要密码的，不过，不影响我们的大体程序）

```python
 # 开始存款
        amount = int(input("验证成功！请输入存款金额："))
        if amount < 0:
            print("存款金额有误，存款失败！")
            return -1
        user.card.cardMoney += amount
        print("您存款%d元，最新余额为%d元！" % (amount, user.card.cardMoney))
```

在这里，我们运行一下。

![](https://img-blog.csdnimg.cn/7edc7be20b6a42e1b6af003edfe2a4c5.png)

到这里，存款的所有功能都可以实现了。

我们下一期讲介绍关于的转账代码相关讲解。