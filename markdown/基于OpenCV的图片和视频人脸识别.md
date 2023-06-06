**目录**

[🥩前言](#%F0%9F%A5%A9%E5%89%8D%E8%A8%80)

[🍖环境使用](#%F0%9F%8D%96%E7%8E%AF%E5%A2%83%E4%BD%BF%E7%94%A8)

[🍖模块使用](#%F0%9F%8D%96%E6%A8%A1%E5%9D%97%E4%BD%BF%E7%94%A8)

[🍖模块介绍](#%F0%9F%8D%96%E6%A8%A1%E5%9D%97%E4%BB%8B%E7%BB%8D)

[🍖模块安装问题:](#%F0%9F%8D%96%E6%A8%A1%E5%9D%97%E5%AE%89%E8%A3%85%E9%97%AE%E9%A2%98%3A)

[🥩人脸检测](#%F0%9F%A5%A9%E4%BA%BA%E8%84%B8%E6%A3%80%E6%B5%8B)

[🍖Haar 级联的概念](<#🍖Haar 级联的概念>)

[🍖获取 Haar 级联数据](<#🍖获取 Haar 级联数据>)

[🍗 1.下载所需版本](<#🍗 1.下载所需版本>)

[🍗 2.安装文件](<#🍗 2.安装文件>)

[🍗 3.XML文件名称](<# 🍗 3.XML文件名称>)

[🥩使用 OpenCV 进行人脸检测](<#🥩使用 OpenCV 进行人脸检测>)

[🍖静态图像中人脸检测](#%F0%9F%8D%96%E9%9D%99%E6%80%81%E5%9B%BE%E5%83%8F%E4%B8%AD%E4%BA%BA%E8%84%B8%E6%A3%80%E6%B5%8B)

[🍗【示例】识别图片中的人脸](#%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E8%AF%86%E5%88%AB%E5%9B%BE%E7%89%87%E4%B8%AD%E7%9A%84%E4%BA%BA%E8%84%B8)

[🍗 运行效果：](<#🍗 运行效果：>)

[🍗【示例】识别图片中多张人脸](#%C2%A0%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E8%AF%86%E5%88%AB%E5%9B%BE%E7%89%87%E4%B8%AD%E5%A4%9A%E5%BC%A0%E4%BA%BA%E8%84%B8)

[🍗 运行效果：](<#🍗 运行效果：>)

[🍖视频中的人脸检测](#%C2%A0%F0%9F%8D%96%E8%A7%86%E9%A2%91%E4%B8%AD%E7%9A%84%E4%BA%BA%E8%84%B8%E6%A3%80%E6%B5%8B)

[🍗【示例】识别视频中人脸](#%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E8%AF%86%E5%88%AB%E8%A7%86%E9%A2%91%E4%B8%AD%E4%BA%BA%E8%84%B8)

[🍗 运行效果：](<#🍗 运行效果：>)

[🥩人脸识别](#%F0%9F%A5%A9%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB)

[🍖训练数据](#%F0%9F%8D%96%E8%AE%AD%E7%BB%83%E6%95%B0%E6%8D%AE)

[🍗【示例】训练数据](#%F0%9F%8D%97%E3%80%90%E7%A4%BA%E4%BE%8B%E3%80%91%E8%AE%AD%E7%BB%83%E6%95%B0%E6%8D%AE)

[🍗【示例】基于 LBPH 的人脸识别](<#🍗【示例】基于 LBPH 的人脸识别>)

[🍗 运行效果：](<#🍗 运行效果：>)

[🥩总结](#%F0%9F%A5%A9%E6%80%BB%E7%BB%93)

---

# 🥩前言

![1b83b1d3fff541e6844ba7bfc4b8f724.gif](https://img-blog.csdnimg.cn/1b83b1d3fff541e6844ba7bfc4b8f724.gif)

> 我们身边的人脸识别有车站检票，监控人脸，无人超市，支付宝人脸支付，上班打卡，人脸解锁手机。
> 
> 人脸检测是人脸识别系统组成的关键部分之一，其目的是检测出任意给定图片中的包含的一个或多个人脸，是人脸识别、表情识别等下游任务的基础。人脸识别是通过采集包含人脸的图像或视频数据，通过对比和分析人脸特征信息从而实现身份识别的生物识别技术，是人脸识别系统的核心组件
> 
> 随着人工智能的不断发展，机器学习这门技术也越来越重要，很多人都开启了学习机器学习，本文就介绍了机器学习的基础内容，基于OpenCV的图片和视频人脸识别。介绍Haar的概念，以及如何对图片和视频中进行人脸检测，以及如何训练我们自己的模型，并在自己的模型下进行人脸识别。

## 🍖 **环境使用**

- python 3.9
- pycharm

## 🍖 **模块使用**

- opencv-python

## **🍖** 模块介绍

> 1.  opencv
> 
> 关于OpenCv
> 
> Opencv是一个开源的的跨平台计算机视觉库，内部实现了图像处理和计算机视觉方面的很多通用算法，对于python而言，在引用opencv库的时候需要写为import cv2。其中，cv2是opencv的C++命名空间名称，使用它来表示调用的是C++开发的opencv的接口。
> 
> 目前人脸识别有很多较为成熟的方法，这里调用OpenCv库，而OpenCV又提供了三种人脸识别方法，分别是LBPH方法、EigenFishfaces方法、Fisherfaces方法。本文采用的是LBPH（Local Binary Patterns Histogram，局部二值模式直方图）方法。在OpenCV中，可以用函数cv2.face.LBPHFaceRecognizer\_create\(\)生成LBPH识别器实例模型，然后应用cv2.face\_FaceRecognizer.train\(\)函数完成训练，最后用cv2.face\_FaceRecognizer.predict\(\)函数完成人脸识别。
> 
> CascadeClassifier，是Opencv中做人脸检测的时候的一个级联分类器。并且既可以使用Haar，也可以使用LBP特征。其中Haar特征是一种反映图像的灰度变化的，像素分模块求差值的一种特征。它分为三类：边缘特征、线性特征、中心特征和对角线特征。

## **🍖** 模块安装问题:

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

# 🥩人脸检测

## 🍖Haar 级联的概念

_**Haar级联**_ 是一种基于Haar特征的目标检测方法，它由多个级联分类器组成，每个级联分类器由多个弱分类器组成。在目标检测中，Haar级联通过逐级检测，将输入图像分成多个子区域，然后在每个子区域中应用Haar特征进行分类。这种级联的方式可以大大减少计算量，提高检测速度，同时保证较高的准确性。Haar级联在人脸识别、车辆识别等领域有着广泛的应用。

提取出图像的细节对产生稳定分类结果和跟踪结果很有用。这些提取的结果被称为特征，专业的表述为：从图像数据中提取特征。虽然任意像素都可以能影响多个特征，但特征应该比像素少得多。两个图像的相似程度可以通过它们对应特征的欧氏距离来度量。

Haar 特征是一种用于实现实时人脸跟踪的特征。每一个 Haar 特征都描述了相邻图像区域的对比模式。例如，边、顶点和细线都能生成具有判别性的特征。

## 🍖获取 Haar 级联数据

### 🍗 1.下载所需版本

首先我们要进入 OpenCV 官网： https://opencv.org 下载你需要的版本。点击 RELEASES

（发布）。如下图所示：

![ebd103675b714a2a987287463d36025e.png](https://img-blog.csdnimg.cn/ebd103675b714a2a987287463d36025e.png)

由于 OpenCV 支持好多平台，比如 Windows, Android, Maemo, FreeBSD, OpenBSD, iOS,

Linux 和 Mac OS，一般初学者都是用 windows，点击 Windows。

![faa9ffbcae7640c0a738c5086d682597.png](https://img-blog.csdnimg.cn/faa9ffbcae7640c0a738c5086d682597.png)

点击 Windows 后跳出下面界面，等待 5s 自动下载。

### 🍗 2.安装文件

文件下载好后，然后双击下载的文件，进行安装，实质就是解压一下，解压完出来一个文件夹，其他什么也没发生。安装完后的目录结构如下。其中 build 是 OpenCV 使用时要用到的一些库文件，而 sources 中则是 OpenCV 官方为我们提供的一些 demo 示例源码。

![5f0dc52b8f614cdfa887ae13afb4b06c.png](https://img-blog.csdnimg.cn/5f0dc52b8f614cdfa887ae13afb4b06c.png)

在 sources 的一个文件夹 data/haarcascades。该文件夹包含了所有 OpenCV 的人脸检测的

XML 文件，这些可用于检测静止图像、视频和摄像头所得到图像中的人脸。

![1de2d098eaab4350b6b4b5911557dc3d.png](https://img-blog.csdnimg.cn/1de2d098eaab4350b6b4b5911557dc3d.png)

### 🍗 3.XML文件名称

> 人脸检测器（默认）：haarcascade\_frontalface\_default.xml
> 
> 人脸检测器（快速 Harr）：haarcascade\_frontalface\_alt2.xml
> 
> 人脸检测器（侧视）：haarcascade\_profileface.xml
> 
> 眼部检测器（左眼）：haarcascade\_lefteye\_2splits.xml
> 
> 眼部检测器（右眼）：haarcascade\_righteye\_2splits.xml
> 
> 嘴部检测器：haarcascade\_mcs\_mouth.xml
> 
> 鼻子检测器：haarcascade\_mcs\_nose.xml
> 
> 身体检测器：haarcascade\_fullbody.xml
> 
> 人脸检测器（快速 LBP）：lbpcascade\_frontalface.xml

# 🥩使用 OpenCV 进行人脸检测

## 🍖静态图像中人脸检测

人脸检测首先是加载图像并检测人脸，这也是最基本的一步。为了使所得到的结果有意义，可在原始图像的人脸周围绘制矩形框。

### 🍗【示例】识别图片中的人脸

我们首先来识别图片中的人脸，我们先识别图片中的一张人脸，假如，我们测试的照片有两张人脸的话，就会只显示一个人脸。

```python
import cv2 as cv
def face_detect_demo():
    #将图片转换为灰度图片
    gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)
    #加载特征数据
    face_detector=cv.CascadeClassifier('E:\Program Files (x86)\Python39\Lib\site-packages\cv2\data\haarcascade_frontalface_default.xml')
    faces=face_detector.detectMultiScale(gray)
    for x,y,w,h in faces:
        cv.rectangle(img,(x,y),(x+w,y+h),color=(0,255,0),thickness=2)
    cv.imshow('result',img)
#加载图片
img=cv.imread('text1.jpg')
face_detect_demo()
cv.waitKey(0)
cv.destroyAllWindows()
```

### 🍗 运行效果：

![e37817b6220c4689ae306c3d7c83711b.png](https://img-blog.csdnimg.cn/e37817b6220c4689ae306c3d7c83711b.png)

### 🍗【示例】识别图片中多张人脸

​​​​​​​我们前面识别了图片中的一张人脸，假如，我们想测试的照片有两张人脸的话，怎么办？前面的代码就实现不了了，我们来看看多张人脸是怎么实现的。

```python
import cv2 as cv
def face_detect_demo():
    #将图片灰度
    gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)
    #加载特征数据
    face_detector = cv.CascadeClassifier(
        'E:\Program Files (x86)\Python39\Lib\site-packages\cv2\data\haarcascade_frontalface_default.xml')
    faces = face_detector.detectMultiScale(gray)
    for x,y,w,h in faces:
        print(x,y,w,h)
        cv.rectangle(img,(x,y),(x+w,y+h),color=(0,0,255),thickness=2)
        cv.circle(img,center=(x+w//2,y+h//2),radius=w//2,color=(0,255,0),thickness=2)
    #显示图片
    cv.imshow('result',img)

#加载图片
img=cv.imread('text2.jpg')
#调用人脸检测方法
face_detect_demo()
cv.waitKey(0)
cv.destroyAllWindows()
```

### 🍗 运行效果：

我们找了一个多张人脸的照片，相信大家对这张图片并不陌生，我们可以清晰的看到，我们准确无误的识别到了每一张人脸。

![27d6915101d449d0943c55a2974091f6.png](https://img-blog.csdnimg.cn/27d6915101d449d0943c55a2974091f6.png)

## 🍖视频中的人脸检测

视频是一张一张图片组成的，在视频的帧上重复这个过程就能完成视频中的人脸检测。

> 视频中的人脸检测可以通过以下步骤实现：
> 
> 1.  图像预处理：对输入的视频帧进行预处理，包括图像增强、图像滤波、图像二值化等操作，以增强图像的对比度和亮度，减少噪声的影响，提高图像的质量。
> 2.  特征提取：使用图像处理算法，如SIFT、SURF、ORB等，提取视频帧中的特征，如人脸的位置、大小、形状、姿态等信息，作为人脸检测的基础。
> 3.  人脸检测：使用人脸检测算法，如Haar Cascade、LBPH、LBPH-SIFT等，对视频帧中的图像进行人脸检测，得到检测到的人脸的位置、大小、形状等信息。
> 4.  人脸跟踪：使用人脸跟踪算法，如OpenCV中的人脸跟踪算法，对检测到的人脸进行跟踪，得到人脸的位置、大小、形状等信息。
> 5.  人脸识别：使用人脸识别算法，如支持向量机、深度学习等，对人脸跟踪得到的人脸进行识别，得到人脸的身份信息。

### 🍗【示例】识别视频中人脸

视频是一张一张图片组成的，在视频的帧上重复这个过程就能完成视频中的人脸检测。我们看看代码是如何实现的。

```python
import cv2 as cv
def face_detect_demo(img):
    #将图片灰度
    gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)
    #加载特征数据
    face_detector = cv.CascadeClassifier(
        'E:\Program Files (x86)\Python39\Lib\site-packages\cv2\data\haarcascade_frontalface_default.xml')
    faces = face_detector.detectMultiScale(gray)
    for x,y,w,h in faces:
        cv.rectangle(img,(x,y),(x+w,y+h),color=(0,0,255),thickness=2)
        cv.circle(img,center=(x+w//2,y+h//2),radius=(w//2),color=(0,255,0),thickness=2)
    cv.imshow('result',img)
#读取视频
cap=cv.VideoCapture('video.mp4')
while True:
    flag,frame=cap.read()
    print('flag:',flag,'frame.shape:',frame.shape)
    if not flag:
        break
    face_detect_demo(frame)
    if ord('q') == cv.waitKey(10):
        break
cv.destroyAllWindows()
cap.release()
```

### 🍗 运行效果：

这里我就不放视频了，我放一张视频的截图，我们可以清楚的看到，可以清晰的识别到我们的人脸。

![52dfee8e214747c59ab6b8cdeced3805.png](https://img-blog.csdnimg.cn/52dfee8e214747c59ab6b8cdeced3805.png)

# 🥩人脸识别

人脸检测是 OpenCV 的一个很不错的功能，它是人脸识别的基础。什么是人脸识别呢？

其实就是一个程序能识别给定图像或视频中的人脸。实现这一目标的方法之一是用一系列分好类的图像来“训练”程序，并基于这些图像来进行识别。

这就是 OpenCV 及其人脸识别模块进行人脸识别的过程。

人脸识别模块的另外一个重要特征是：每个识别都具有转置信（confidence）评分，因此可在实际应用中通过对其设置阈值来进行筛选。

人脸识别所需要的人脸可以通过两种方式来得到：自己获得图像或从人脸数据库免费获得可用的人脸图像。互联网上有许多人脸数据库。

为了对这些样本进行人脸识别，必须要在包含人脸的样本图像上进行人脸识别。这是一个学习的过程，但并不像自己提供的图像那样令人满意。

## 🍖训练数据

有了数据，需要将这些样本图像加载到人脸识别算法中。所有的人脸识别算法在它们的train\(\)函数中都有两个参数：图像数组和标签数组。这些标签表示进行识别时候某人人脸的ID，因此根据 ID 可以知道被识别的人是谁。要做到这一点，将在「trainer/trainer」目录中保存为.yml文件。

### 🍗【示例】训练数据

```python
import os
import cv2
import sys
from PIL import Image
import numpy as np


def getImageAndLabels(path):
    facesSamples = []
    ids = []
    imagePaths = [os.path.join(path, f) for f in os.listdir(path)]
    # 检测人脸
    face_detector = cv2.CascadeClassifier(
        'E:\Program Files (x86)\Python39\Lib\site-packages\cv2\data\haarcascade_frontalface_default.xml')

    # 遍历列表中的图片
    for imagePath in imagePaths:
        # 打开图片
        PIL_img = Image.open(imagePath).convert('L')
        # 将图像转换为数组
        img_numpy = np.array(PIL_img, 'uint8')
        faces = face_detector.detectMultiScale(img_numpy)
        # 获取每张图片的id
        id = int(os.path.split(imagePath)[1].split('.')[0])
        for x, y, w, h in faces:
            facesSamples.append(img_numpy[y:y + h, x:x + w])
            ids.append(id)
    return facesSamples, ids


if __name__ == '__main__':
    # 图片路径
    path = './data/jm/'
    # 获取图像数组和id标签数组
    faces, ids = getImageAndLabels(path)
    # 获取训练对象
    recognizer = cv2.face.LBPHFaceRecognizer_create()
    recognizer.train(faces, np.array(ids))
    # 保存文件
    recognizer.write('trainer/trainer.yml')
```

🍖 **基于 LBPH** **的人脸识别**

LBPH（Local Binary Pattern Histogram）将检测到的人脸分为小单元，并将其与模型中的对应单元进行比较，对每个区域的匹配值产生一个直方图。由于这种方法的灵活性，LBPH是唯一允许模型样本人脸和检测到的人脸在形状、大小上可以不同的人脸识别算法。

调整后的区域中调用 predict\(\)函数，该函数返回两个元素的数组：第一个元素是所识别个体的标签，第二个是置信度评分。所有的算法都有一个置信度评分阈值，置信度评分用来衡量所识别人脸与原模型的差距，0 表示完全匹配。可能有时不想保留所有的识别结果，则需要进一步处理，因此可用自己的算法来估算识别的置信度评分。LBPH 一个好的识别参考值要低于 50 ，任何高于 80 的参考值都会被认为是低的置信度评分。

### 🍗【示例】基于 LBPH 的人脸识别

```python
import cv2
import numpy as np
import os
#加载训练数据集文件
recogizer=cv2.face.LBPHFaceRecognizer_create()
recogizer.read('trainer/trainer.yml')
#准备识别的图片
img=cv2.imread('19.pgm')
gray=cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
face_detector = cv2.CascadeClassifier(
    'E:\Program Files (x86)\Python39\Lib\site-packages\cv2\data\haarcascade_frontalface_default.xml')
faces = face_detector.detectMultiScale(gray)
for x,y,w,h in faces:
    cv2.rectangle(img,(x,y),(x+w,y+h),(0,255,0),2)
    #人脸识别
    id,confidence=recogizer.predict(gray[y:y+h,x:x+w])
    print('标签id:',id,'置信评分：',confidence)
cv2.imshow('result',img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 🍗 运行效果：

> 标签id: 15 置信评分： 84.05495321482604

# 🥩总结

随着人工智能的不断发展，机器学习这门技术也越来越重要，很多人都开启了学习机器学习，本文就介绍了机器学习的基础内容。介绍Haar的概念，以及如何对图片和视频中进行人脸检测，以及如何训练我们自己的模型，并在自己的模型下进行人脸识别。

![6adf31c8c5dd4e6a83314f4805b30bc1.jpg](https://img-blog.csdnimg.cn/6adf31c8c5dd4e6a83314f4805b30bc1.jpg)