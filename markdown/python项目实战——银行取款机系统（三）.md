## 项目实战目录

[python项目实战——银行取款机系统（一）](https://blog.csdn.net/BROKEN__Y/article/details/127703108 "python项目实战——银行取款机系统（一）")

[python项目实战——银行取款机系统（二）](https://blog.csdn.net/BROKEN__Y/article/details/127709126 "python项目实战——银行取款机系统（二）")

python项目实战——银行取款机系统（三）

**前言**

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests

上一篇我们说到了，分析了大致思路，今天，我们将具体实现其中一部分的功能——查询。

## 查询

查询，我们一般是通过卡号查询自己卡里面还有多少钱。那么在python里面是如何判断的呢？

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
            print("该卡号不存在！！！查询失败")
            return -1
```

运行一下：

![](https://img-blog.csdnimg.cn/583133bb620640f1860d79b69ec6dbd7.png)

第三步，我们看看卡的状态，是不是处于未锁定状态。

```python
# 判断是否锁定
        if user.card.cardLock:
            print("该卡已被锁定,请解锁后操作")
            return -1
```

这个是卡被锁定是的输出。

![](https://img-blog.csdnimg.cn/0adf0dda77b04e39af2099d1fc669e5e.png)

第四步，如果这些都没有问题，我们就进行下一步，验证密码。

```python
# 验证密码

        if not self.checkPasswd(user.card.cardPasswd):
            # print("密码输入错误！！！查询失败")
            print("密码输入错误次数超过三次,该卡已被锁定,请解锁后操作")
            user.card.cardLock = True
            return -1
```

我们试一下密码错误三次，就会提示已被锁定。（后面会讲解关于锁定代码的文章）

![](https://img-blog.csdnimg.cn/e982ddb8a23b46dfbf4f56008681f8b5.png)

第五步，密码正确，系统就会返回卡里面的余额。

```python
print("账号：%s  余额：%d" % (user.card.cardId, user.card.cardMoney))
```

![](https://img-blog.csdnimg.cn/4fc5099b0f014f9bbdb5238b373a2412.png)

到这里，查询的所有功能都可以实现了。

我们下一期讲介绍关于锁定的代码相关讲解。