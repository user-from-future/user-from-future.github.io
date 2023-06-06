# 前言

![](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 我们身边的人脸识别有车站检票，监控人脸，无人超市，支付宝人脸支付，上班打卡，人脸解锁手机。
> 
> 人脸检测是人脸识别系统组成的关键部分之一，其目的是检测出任意给定图片中的包含的一个或多个人脸，是人脸识别、表情识别等下游任务的基础。人脸识别是通过采集包含人脸的图像或视频数据，通过对比和分析人脸特征信息从而实现身份识别的生物识别技术，是人脸识别系统的核心组件。
> 
> 如今人脸识别系统广泛应用于学校、医院和车站等需要对入场人员身份信息进行确认和鉴别的场所。进而利用人脸检测算法检测出摄像头拍摄图片中包含的人脸，再使用识别算法对比当前人脸特征和数据库内的人脸特征，确认人员身份，最终输出人员相关的身份信息并生成打卡记录。在实际的运行测试中，针对戴口罩的场景也有优异的表现，满足公司打卡应用的实际需求。
> 
> 今天我们就说说利用opencv打造人脸识别系统。

## **环境使用**

- python 3.9
- pycharm

## **模块使用**

- sys
- cv2
- face\_recognition

### **模块介绍**

> **1.opencv**
> 
> 关于OpenCv  
> Opencv是一个开源的的跨平台计算机视觉库，内部实现了图像处理和计算机视觉方面的很多通用算法，对于python而言，在引用opencv库的时候需要写为import cv2。其中，cv2是opencv的C++命名空间名称，使用它来表示调用的是C++开发的opencv的接口。
> 
> 目前人脸识别有很多较为成熟的方法，这里调用OpenCv库，而OpenCV又提供了三种人脸识别方法，分别是LBPH方法、EigenFishfaces方法、Fisherfaces方法。本文采用的是LBPH（Local Binary Patterns Histogram，局部二值模式直方图）方法。在OpenCV中，可以用函数cv2.face.LBPHFaceRecognizer\_create\(\)生成LBPH识别器实例模型，然后应用cv2.face\_FaceRecognizer.train\(\)函数完成训练，最后用cv2.face\_FaceRecognizer.predict\(\)函数完成人脸识别。
> 
> CascadeClassifier，是Opencv中做人脸检测的时候的一个级联分类器。并且既可以使用Haar，也可以使用LBP特征。其中Haar特征是一种反映图像的灰度变化的，像素分模块求差值的一种特征。它分为三类：边缘特征、线性特征、中心特征和对角线特征。
> 
> **2.face\_recognition**
> 
> face\_recognition库是世界上最简洁的人脸识别库，可以使用Python和命令行工具提取、识别、操作人脸。
> 
> face\_recognition库的人脸识别是基于业内领先的C++开源库 dlib中的深度学习模型，用Labeled Faces in the Wild人脸数据集进行测试，有高达99.38\%的准确率。但对小孩和亚洲人脸的识别准确率尚待提升。
> 
> **3.sys**
> 
> sys模块是最常用的和python解释器交互的模块,sys模块可供访问由解释器\(interpreter\)使用或维护的变量和与解释器进行交互的函数。 `sys` 模块提供了许多函数和变量来处理 Python 运行时环境的不同部分。

```python
import cv2

import sys

import face_recognition
```

### **模块安装问题:**

**本次比较难库，会出很多问题，比如将dlib库和opencv库。有问题自行百度，可以学到很多东西。**

- 如果安装python第三方模块:

> win + R 输入 cmd 点击确定, 输入安装命令 pip install 模块名 \(pip install requests\) 回车
> 
> 在pycharm中点击Terminal\(终端\) 输入安装命令

- 安装失败原因:

> - 失败一: pip 不是内部命令
> 
> 解决方法: 设置环境变量
> 
> - 失败二: 出现大量报红 \(read time out\)
> 
> 解决方法: 因为是网络链接超时, 需要切换镜像源
> 
> ```python
>     清华：https://pypi.tuna.tsinghua.edu.cn/simple
>     阿里云：https://mirrors.aliyun.com/pypi/simple/
>     中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
>     华中理工大学：https://pypi.hustunique.com/
>     山东理工大学：https://pypi.sdutlinux.org/
>     豆瓣：https://pypi.douban.com/simple/
>     例如：pip3 install -i https://pypi.doubanio.com/simple/ 模块名
> ```
> 
>   
> 失败三: cmd里面显示已经安装过了, 或者安装成功了, 但是在pycharm里面还是无法导入
> 
> 解决方法: 可能安装了多个python版本 \(anaconda 或者 python 安装一个即可\) 卸载一个就好，或者你pycharm里面python解释器没有设置好。

## 代码实现

首先，我们先准备一张有两个人的图片。

![](https://img-blog.csdnimg.cn/c724c451c6a7443d9b309b0d24d47f45.png)

这个明显不是一个人吧。下面我们将通过opencv和face\_recognition算法，进行人脸识别，看看是不是一个人。

首先，安装好我们的库。确保没有问题，接下来，我们开始写代码了。

```python
face_image = face_recognition.load_image_file(r'1.png')

face_encodings = face_recognition.face_encodings(face_image) 
face_locations = face_recognition.face_locations(face_image)

face1 = face_encodings[0]
face2 = face_encodings[1]


result = face_recognition.compare_faces([face1], face2, tolerance=0.5) 
```

下面我们来判断人脸是否一样。

```python

if result == [True]:
    print(1)
    name = 'YES'
else:
    print(0)
    name = 'NO'
```

下面进行绘图

```python

for i in range(len(face_encodings)):

    face_encoding = face_encodings[(i - 1)]

    face_location = face_locations[(i - 1)]

    top, right, bottom, left = face_location

    cv2.rectangle(face_image, (left, top), (right, bottom), (0, 255, 0), 2)

face_image_rgb = cv2.cvtColor(face_image,cv2.COLOR_BGR2RGB)

cv2.imshow('output',face_image_rgb)
```

## 效果展示

![](https://img-blog.csdnimg.cn/76f3f449425f48a0915f27842e3f31c4.png)

哎呀，翻车了，怎么判成是一个人呢。等我研究研究。哪里出问题呢，大致就是这样。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)