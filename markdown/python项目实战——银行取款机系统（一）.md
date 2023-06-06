## 项目实战目录

python项目实战——银行取款机系统（一）

**前言**

今天我们将通过python完成简易银行提款机系统的实战，我们一步步实现我们的要求。话不多说，看操作。

**环境使用**

- python 3.9
- pycharm

**模块使用**

- requests
- time

要想做银行简易取款系统，首先，我们要知道我们要哪些数据，我们先来分析一下。

## **主要有五大类：**

- 人
- 卡
- 银行
- 提款机
- 界面

### 人

> 类名：Person
> 
> 属性：姓名 身份证号码 电话 卡
> 
> 行为：开户 查询 取款 存款 转账 改密 锁定 解锁 补卡 销户

### 卡

> 类名：Card
> 
> 属性：卡号 密码 余额

### 银行

> 类名：bank
> 
> 属性：用户列表 提款机

### 提款机

> 类名：ATM
> 
> 行为：开户 查询 取款 存款 转账 改密 锁定 解锁 补卡 销户 退出

### 界面

> 类名：View
> 
> 行为：管理员界面 管理员登录 系统功能界面

通过上面的分析，我们知道了每一块的大体内容是什么，下面我们先写界面模块。

这里我们定义它的名称是view，首先是我们管理员登录的界面

```python
    def printAdminView(self):
        print("****************************************")
        print("*                                      *")
        print("*                                      *")
        print("*              欢迎登录                *")
        print("*                                      *")
        print("*                                      *")
        print("****************************************")
        inputAdmin=input("请输入管理员账号：")
        if self.admin !=inputAdmin:
            print("账号输入有误！！！")
            return -1
        inputPasswd=input("请输入管理员密码：")
        if self.passwd !=inputPasswd:
            print("密码输入有误！！！")
            return -1
        print("登录成功。请稍后······")
        time.sleep(3)
        return 0
```

下面就是用户操作的页面

```python
    def printSysFunctionView(self):
        print("****************************************")
        print("*    开户(1)              查询(2)      *")
        print("*    取款(3)              存款(4)      *")
        print("*    转账(5)              改密(6)      *")
        print("*    锁定(7)              解锁(8)      *")
        print("*    补卡(9)              销户(0)      *")
        print("*              退出(t)                 *")
        print("****************************************")
```

今天，用户操作里面的具体功能，我们下一次再说，今天就简单展示出来。

接下来，就是我们的主函数了。

```python
def main（）
    #界面对象
    view=View()
    #管理员开机
    if view.printAdminView():
        return -1

    while True:
        #等待用户操作
        view.printSysFunctionView()
        option=input("请输入你的操作:")
        if option=="1":
            print("开户")
        elif option=="2":
            print("查询")
        elif option == "3":
            print("取款")

        elif option == "4":
            print("存款")

        elif option == "5":
            print("转账")


        elif option == "6":
            print("改密")

        elif option == "7":
            print("锁定")

        elif option == "8":
            print("解锁")

        elif option == "9":
            print("补卡")

        elif option == "0":
            print("销户")

        elif option == "t":
            print("退出")

        time.sleep(2)
```

到这里，我们的基本框架就写好了，我们来运行看看吧。

![](https://img-blog.csdnimg.cn/f2d594b665824cd8a3a19796cda8f7a7.png)

接下来，登录我们的管理员账号

![](https://img-blog.csdnimg.cn/ee6aeace29534948847c0a7295c40cd8.png)

下面是我们的用户操作，有用具体的代码还没写，这里就直接print输出了。

![](https://img-blog.csdnimg.cn/a007e74d68604eeeaae8652fb88d73d9.png)

下一期，我们继续说说银行提款机系统的简单实现。