---
layout:     post
title:      Nao中Redball检测与定位第一篇
subtitle:   探索如何使用nao进行红球检测定位。
date:       2019-07-10
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Robotcup
---

# Nao中Redball检测与定位第一篇

### ALRedbBallDetection简要说明

**ALRedBallDetection**是一个提供基于快速视觉的红色球探测器的模块。基于摄像机给出的图像中红色像素的检测。这些像素根据它们与YUV颜色空间中的红色值的距离进行滤波，使用计算的阈值，即使在光照条件变化的情况下也可以进行检测。然后，从所有检测到的红色像素组中，仅保留定义圆形形状的红色像素。

 该事件模块检测仅限于近似红色和圆形的物体。无法检测到其他颜色的球。

 

**ALRedBallDetection**

当在当前图像上找到一组像素时，将更新ALMemory键redBallDetected。

该密钥的组织如下：

```
[
  TimeStamp,
  BallInfo,
  CameraPose_InTorsoFrame,
  CameraPose_InRobotFrame,
  Camera_Id
]
```

**TimeStamp**：此字段是用于执行检测的图像的时间戳。

```
TimeStamp [
  TimeStamp_Seconds，
  Timestamp_Microseconds
]
```

 

**BallInfo**

```
BallInfo [
  centerX，
  centerY，
  SIZEX，
  SIZEY
]
```

 **centerX**和**centerY**是球的中心角度坐标（弧度）。

**sizeX**和**sizeY**是角度（弧度）的球“水平和垂直半径”。

 

 

**CameraPose_InTorsoFrame**：描述在拍摄图像时摄像机的[**Position6D**](http://doc.aldebaran.com/2-1/glossary.html#term-position6d)，在[**FRAME_TORSO**中](http://doc.aldebaran.com/2-1/glossary.html#term-frame-torso)。

Position6D是由3个平移（以米为单位）和3个旋转（以弧度表示）组成的6维向量。ALMotion使用的3个空间参考之一。这是一个永远不变的固定起源。当NAO行走时，它会被遗忘，并且在NAO转向后，它将在z旋转方面有所不同。此空间对于需要外部绝对参照系的计算非常有用。

 

**CameraPose_InRobotFrame**：描述在拍摄图像时摄像机的[**Position6D**](http://doc.aldebaran.com/2-1/glossary.html#term-position6d)，在[**FRAME_ROBOT**中](http://doc.aldebaran.com/2-1/glossary.html#term-frame-robot)。

**Camera_Id**：给出用于检测的摄像机的ID（顶部摄像机为0，底部摄像机为1）。

 

ALRedBallDetection

作为任何模块，此模块从[**ALModule API**](http://doc.aldebaran.com/2-1/naoqi/core/almodule-api.html#almodule-api)继承方法。

它还继承了[**ALExtractor API**的](http://doc.aldebaran.com/2-1/naoqi/core/alextractor-api.html#alextractor-api)方法。

检测到红球时抬起。

这个功能只提供红球的检测功能，通过机器人眼中的摄像头去计算红球所在的位置以及其形状。

 

 [redBallDetected（）](http://doc.aldebaran.com/2-1/naoqi/vision/alredballdetection-api.html#redBallDetected)

  **eventName**（*std  :: string*） -  “redBallDetected”  ·      **Value** - 与检测到的红球有关的信息。有关详细信息，请参阅[**ALRedBallDetection**](http://doc.aldebaran.com/2-1/naoqi/vision/alredballdetection.html#alredballdetection) **subscriberIdentifier**（*std  :: string*） -  

![dy863Q.png](https://s1.ax1x.com/2020/08/24/dy863Q.png)

首先，设定代理人来链接至”标志检測＂( ALLandmarvdetection)模块因为标志检测模块会持续的产出数值，它会使用订阅(subscribe)函数在每500毫秒时连接。当正常连接时，数值会储存于Almemory，所以代理人应该连接至Almemory，为了检测数值是否传输成功，检查是否有列表对象和长度是否大于2.此外，检测marvInforArray的[0][1]指数并确认使用上面的程序是否有检测到标志。



### 问题反馈

1. API里面的函数有很多参数并不是很能理解，就比如说ALRedBallDetection API里面的redBallDetected()的参数value – Informations related to the detected red ball. Please refer to ALRedBallDetection for details.
   subscriberIdentifier (std::string) 是什么意思。网上相关nao的东西很少，这些函数的解释也比较少，所以不是很能理解。
2. 在网上找了一些代码，但是有很多的东西都不能理解。比如
   https://blog.csdn.net/sinat_34474705/article/details/81349822
   里面的
   self._frameWidth = frame[0]
               
   self._frameHeight = frame[1]
               
   self._frameChannels = frame[2]
              
   self._frameArray = np.frombuffer(frame[6], dtype=np.uint8).reshape([frame[1],frame[0],frame[2]])
3. 不能理解ALRedBallDetection API检测红球的基本原理是什么



### 针对红球的圆形检测

- Step1. 首先，将图片由 RGB 颜色空间转换为 HSV 颜色空间，HSV颜色空间可以更直观地反映物体的颜色，便于利用颜色进行分割。
- Step2. 根据红色在HSV颜色空间中的范围，对图像进行分割。属于红色的像素被保留下来（变为白色），其余像素置为黑色。
- Step3. 形态学运算，对刚才得到的图像进行膨胀，以使红色区域更明显
- Step4. 利用Hough变换，提取图像中的圆形
- Step5. 绘制圆形

实现代码：

```
import cv2
import numpy as np

# Step1
img = cv2.imread('2.jpg')
hue_image = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# Step2
low_range = np.array([0, 123, 100])
high_range = np.array([5, 255, 255])
th = cv2.inRange(hue_image, low_range, high_range)
print(th)

# Step3
dilated = cv2.dilate(th, cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3)), iterations=2)

# Step4
circles = cv2.HoughCircles(dilated, cv2.HOUGH_GRADIENT, 1, 100, param1=15, param2=7, minRadius=10, maxRadius=20)

# Step5
if circles is not None:
    x, y, radius = circles[0][0]
    center = (x, y)
    cv2.circle(img, center, radius, (0, 255, 0), 2)
cv2.imshow('result', img)
cv2.waitKey(0)
cv2.destroyAllWindows()

```

