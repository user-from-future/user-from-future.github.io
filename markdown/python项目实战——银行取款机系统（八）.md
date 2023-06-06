## 项目实战目录

[python项目实战——银行取款机系统（一）](https://blog.csdn.net/BROKEN__Y/article/details/127703108 "python项目实战——银行取款机系统（一）")

[python项目实战——银行取款机系统（二）](https://blog.csdn.net/BROKEN__Y/article/details/127709126 "python项目实战——银行取款机系统（二）")

[python项目实战——银行取款机系统（三）](https://blog.csdn.net/BROKEN__Y/article/details/127711247 "python项目实战——银行取款机系统（三）")

[python项目实战——银行取款机系统（四）](https://blog.csdn.net/BROKEN__Y/article/details/127722125 "python项目实战——银行取款机系统（四）")

[python项目实战——银行取款机系统（五）](https://blog.csdn.net/BROKEN__Y/article/details/127722405 "python项目实战——银行取款机系统（五）")

[python项目实战——银行取款机系统（六）](https://blog.csdn.net/BROKEN__Y/article/details/127738787 "python项目实战——银行取款机系统（六）")

[python项目实战——银行取款机系统（七）](https://blog.csdn.net/BROKEN__Y/article/details/127739549 "python项目实战——银行取款机系统（七）")

python项目实战——银行取款机系统（八）

**前言**

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests

上一篇我们说到了，分析了大致思路，今天，我们将具体实现剩下的所有的功能。基本上程序都差不多，我这里直接放代码了

## 转账

```python
# 转账

    def transferMoney(self):
        cardNum = input("请输入您的卡号：")
        # 验证是否存在该卡号
        user = self.allUsers.get(cardNum)
        if not user:
            print("该卡号不存在，转账失败！")
            return -1
        # 判断是否锁定
        if user.card.cardLock:
            print("该卡已锁定！请解锁后再使用其功能！")
            return -1

        # 验证密码
        if not self.checkPasswd(user.card.cardPasswd):
            print("密码输入有误，该卡已锁定！请解锁后再使用其功能！")
            user.card.cardLock = True
            return -1

        # 开始转账
        amount = int(input("验证成功！请输入转账金额："))
        if amount > user.card.cardMoney or amount < 0:
            print("金额有误，转账失败！")
            return -1

        newcard = input("请输入转入账户：")
        newuser = self.allUsers.get(newcard)
        if not newuser:
            print("该卡号不存在，转账失败！")
            return -1
        # 判断是否锁定
        if newuser.card.cardLock:
            print("该卡已锁定！请解锁后再使用其功能！")
            return -1
        user.card.cardMoney -= amount
        newuser.card.cardMoney += amount
        time.sleep(1)
        print("转账成功，请稍后···")
        time.sleep(1)
        print("转账金额%d元，余额为%d元！" % (amount, user.card.cardMoney))
```

## 改密

```python
    # 改密
    def changePasswd(self):
        cardNum = input("请输入您的卡号：")
        # 验证是否存在该卡号
        user = self.allUsers.get(cardNum)
        if not user:
            print("该卡号不存在，改密失败！")
            return -1
        # 判断是否锁定
        if user.card.cardLock:
            print("该卡已锁定！请解锁后再使用其功能！")
            return -1

        # 验证密码
        if not self.checkPasswd(user.card.cardPasswd):
            print("密码输入有误，该卡已锁定！请解锁后再使用其功能！")
            user.card.cardLock = True
            return -1
        print("正在验证，请稍等···")
        time.sleep(1)
        print("验证成功！")
        time.sleep(1)

        # 开始改密
        newPasswd = input("请输入新密码：")
        if not self.checkPasswd(newPasswd):
            print("密码错误，改密失败！")
            return -1
        user.card.cardPasswd = newPasswd
        print("改密成功！请稍后！")
```

## 补卡

```python
# 补卡
    def newCard(self):
        cardNum = input("请输入您的卡号：")
        # 验证是否存在该卡号
        user = self.allUsers.get(cardNum)
        if not user:
            print("该卡号不存在！")
            return -1
        tempname = input("请输入您的姓名：")
        tempidcard = input("请输入您的身份证号码：")
        tempphone = input("请输入您的手机号码：")
        if tempname != self.allUsers[cardNum].name \
                or tempidcard != self.allUsers.idCard \
                or tempphone != self.allUsers.phone:
            print("信息有误，补卡失败！")
            return -1
        newPasswd = input("请输入您的新密码：")
        if not self.checkPasswd(newPasswd):
            print("密码错误，补卡失败！")
            return -1
        self.allUsers.card.cardPasswd = newPasswd
        time.sleep(1)
        print("补卡成功，请牢记您的新密码！")
```

## 销户

```python
    def killUser(self):
        cardNum = input("请输入您的卡号：")
        # 验证是否存在该卡号
        user = self.allUsers.get(cardNum)
        if not user:
            print("该卡号不存在，转账失败！")
            return -1
        # 判断是否锁定
        if user.card.cardLock:
            print("该卡已锁定！请解锁后再使用其功能！")
            return -1

        # 验证密码
        if not self.checkPasswd(user.card.cardPasswd):
            print("密码输入有误，该卡已锁定！请解锁后再使用其功能！")
            user.card.cardLock = True
            return -1

        del self.allUsers[cardNum]
        time.sleep(1)
        print("销户成功，请稍后！")


   
```

到这里，所有的功能都实现了。