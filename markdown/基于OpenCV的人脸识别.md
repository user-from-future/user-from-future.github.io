**目录**

[🥩 前言](<#🥩 前言>)

[🍖 环境使用](#%F0%9F%8D%96%C2%A0%E7%8E%AF%E5%A2%83%E4%BD%BF%E7%94%A8)

[🍖 模块使用](#%F0%9F%8D%96%C2%A0%E6%A8%A1%E5%9D%97%E4%BD%BF%E7%94%A8)

[🍖 模块介绍](#%F0%9F%8D%96%C2%A0%E6%A8%A1%E5%9D%97%E4%BB%8B%E7%BB%8D)

[🍖 模块安装问题:](#%F0%9F%8D%96%C2%A0%E6%A8%A1%E5%9D%97%E5%AE%89%E8%A3%85%E9%97%AE%E9%A2%98%3A)

[🥩 OpenCV 简介](<#🥩 OpenCV 简介>)

[🍖 安装 OpenCV 模块](<#🍖 安装 OpenCV 模块>)

[🥩 OpenCV 基本使用](<#🥩 OpenCV 基本使用>)

[🍖 读取图片](#%F0%9F%8D%96%C2%A0%E8%AF%BB%E5%8F%96%E5%9B%BE%E7%89%87)

[🍗 【示例】读取图片](#%F0%9F%8D%97%C2%A0%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E8%AF%BB%E5%8F%96%E5%9B%BE%E7%89%87)

[🍗 运行结果如下：](#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E5%A6%82%E4%B8%8B%EF%BC%9A)

[🍖 图片灰度转换](#%F0%9F%8D%96%C2%A0%E5%9B%BE%E7%89%87%E7%81%B0%E5%BA%A6%E8%BD%AC%E6%8D%A2)

[🍗【示例】将图片灰度](#%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E5%B0%86%E5%9B%BE%E7%89%87%E7%81%B0%E5%BA%A6)

[🍗 运行结果如下：](<#🍗 运行结果如下：>)

[🍖 画图](#%C2%A0%F0%9F%8D%96%C2%A0%E7%94%BB%E5%9B%BE)

[🍗【示例】画图](#%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E7%94%BB%E5%9B%BE)

[🍗 运行结果如下：](<#🍗 运行结果如下：>)

[🥩 总结](<#🥩 总结>)

---

# 🥩 前言

![1b83b1d3fff541e6844ba7bfc4b8f724.gif](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 我们身边的人脸识别有车站检票，监控人脸，无人超市，支付宝人脸支付，上班打卡，人脸解锁手机。
> 
> 人脸检测是人脸识别系统组成的关键部分之一，其目的是检测出任意给定图片中的包含的一个或多个人脸，是人脸识别、表情识别等下游任务的基础。人脸识别是通过采集包含人脸的图像或视频数据，通过对比和分析人脸特征信息从而实现身份识别的生物识别技术，是人脸识别系统的核心组件。
> 
> 随着人工智能的不断发展，机器学习这门技术也越来越重要，很多人都开启了学习机器学习，本文就介绍了机器学习的基础内容。我们本文介绍简单的OpenCVZ中图像的处理。

## 🍖 **环境使用**

- python 3.9
- pycharm

## 🍖 **模块使用**

- opencv-python

## 🍖 **模块介绍**

> 1.  opencv
> 
> 关于OpenCv  
> Opencv是一个开源的的跨平台计算机视觉库，内部实现了图像处理和计算机视觉方面的很多通用算法，对于python而言，在引用opencv库的时候需要写为import cv2。其中，cv2是opencv的C++命名空间名称，使用它来表示调用的是C++开发的opencv的接口。
> 
> 目前人脸识别有很多较为成熟的方法，这里调用OpenCv库，而OpenCV又提供了三种人脸识别方法，分别是LBPH方法、EigenFishfaces方法、Fisherfaces方法。本文采用的是LBPH（Local Binary Patterns Histogram，局部二值模式直方图）方法。在OpenCV中，可以用函数cv2.face.LBPHFaceRecognizer\_create\(\)生成LBPH识别器实例模型，然后应用cv2.face\_FaceRecognizer.train\(\)函数完成训练，最后用cv2.face\_FaceRecognizer.predict\(\)函数完成人脸识别。
> 
> CascadeClassifier，是Opencv中做人脸检测的时候的一个级联分类器。并且既可以使用Haar，也可以使用LBP特征。其中Haar特征是一种反映图像的灰度变化的，像素分模块求差值的一种特征。它分为三类：边缘特征、线性特征、中心特征和对角线特征。

## 🍖 **模块安装问题:**

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
> - 失败三: cmd里面显示已经安装过了, 或者安装成功了, 但是在pycharm里面还是无法导入
> 
> 解决方法: 可能安装了多个python版本 \(anaconda 或者 python 安装一个即可\) 卸载一个就好，或者你pycharm里面python解释器没有设置好。

# 🥩 OpenCV 简介

OpenCV 的全称是 Open Source Computer Vision Library，是一个跨平台的计算机视觉库。

OpenCV 是由英特尔公司发起并参与开发，以 BSD 许可证授权发行，可以在商业和研究领

域中免费使用。OpenCV 可用于开发实时的图像处理、计算机视觉以及模式识别程序。该程

序库也可以使用英特尔公司的 IPP 进行加速处理。

OpenCV 用 C++语言编写，它的主要接口也是 C++语言，但是依然保留了大量的 C 语

言接口。该库也有大量的Python、Java and MATLAB/OCTAVE（版本 2.5）的接口。这些语

言的 API 接口函数可以通过在线文档获得。如今也提供对于 C#、Ch、Ruby、GO 的支持。

## 🍖 安装 OpenCV 模块

OpenCV 已经支持 python 的模块了，直接使用 pip 就可以进行安装，命令如下：

> pip install opencv-python

# 🥩 OpenCV 基本使用

## 🍖 读取图片

显示图像是 OpenCV 最基本的操作之一，imshow\(\)函数可以实现该操作。如果使用过其

他 GUI 框架背景，就会很自然第调用 imshow\(\)来显示一幅图像。imshow\(\)函数有两个参数：

显示图像的帧名称以及要显示的图像本身。直接调用 imshow\(\)函数图像确实会显示，但随

即会消失。要保证图片一直在窗口上显示，要通过 waitKey\(\)函数。waitKey\(\)函数的参数为

等待键盘触发的时间，单位为毫秒，其返回值是-1（表示没有键被按下）

> image = cv2.imread\(imagepath\)

### 🍗 【示例】读取图片

```
import cv2 as cv
img=cv.imread('1.png') 
cv.imshow('input image',img)
cv.waitKey(0) 
cv.destroyAllWindows() 
```

我们首先是读取我们的图片，在这里"1.png"是相对路径，注意读取图片的路径不能有中文，不然数据读取不出来。我们发现我们不加waitkey，程序运行之后就会一闪而过，所以，waitKey\(0\) 的作用就是等待键盘的输入。

### 🍗 运行结果如下：

![b6f55bd8fcaa4f74b0260531daa83eca.png](https://img-blog.csdnimg.cn/b6f55bd8fcaa4f74b0260531daa83eca.png)

## 🍖 图片灰度转换

OpenCV 中有数百种关于在不同色彩空间之间转换的方法。当前，在计算机视觉中有三

种常用的色彩空间：灰度、BGR、以及 HSV（Hue，Saturation，Value）。

1.  （1）灰度色彩空间是通过去除彩色信息来将其转换成灰阶，灰度色彩空间对中间处理
2.  特别有效，比如人脸识别。
3.  （2）BGR 及蓝、绿、红色彩空间，每一个像素点都由一个三元数组来表示，分别代表
4.  蓝、绿、红三种颜色。网页开发者可能熟悉另一个与之相似的颜色空间：RGB 它们只是颜
5.  色顺序上不同。
6.  （3）HSV，H（Hue）是色调，S（Saturation）是饱和度，V（Value）表示黑暗的程度
7.  （或光谱另一端的明亮程度）。

灰度转换的作用就是：转换成灰度的图片的计算强度得以降低。示例如下：

### 🍗【示例】将图片灰度

```
import cv2 as cv
img=cv.imread('1.png')
cv.imshow('input image',img)
gray_img=cv.cvtColor(img,code=cv.COLOR_BGR2GRAY)
cv.imshow('gray_image',gray_img)
cv.waitKey(0)
cv.destroyAllWindows()
cv.imwrite('gray_lena.jpg',gray_img)
```

### 🍗 运行结果如下：

![b329d46e4e224f5887e1b3818acd75a6.png](https://img-blog.csdnimg.cn/b329d46e4e224f5887e1b3818acd75a6.png)

## 🍖 画图

OpenCV 的强大之处的一个体现就是其可以对图片进行任意编辑，处理。 下面的这个

函数最后一个参数指定的就是画笔的大小。

### 🍗【示例】画图

```
import cv2 as cv
img=cv.imread('1.png')
x,y,w,h=50,50,80,80
cv.rectangle(img,(x,y,x+w,y+h),color=(0,255,0),thickness=2) #color=BGR
cv.circle(img,center=(x+w//2,y+h//2),radius=w//2,color=(0,0,255),thickness=2)
cv.imshow('result image',img)
cv.waitKey(0)
cv.destroyAllWindows()
```

### 🍗 运行结果如下：

![0a775a92b228456e8aea2e9c39b9dcd0.png](https://img-blog.csdnimg.cn/0a775a92b228456e8aea2e9c39b9dcd0.png)

# 🥩 总结

随着人工智能的不断发展，机器学习这门技术也越来越重要，很多人都开启了学习机器学习，本文就介绍了机器学习的基础内容。介绍OpenCV中图像的处理。我们学习了如何安装模块，以及读取图片和图片的处理。下一篇，我们将介绍Haar的概念，以及如何对图片和视频中进行人脸检测。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)