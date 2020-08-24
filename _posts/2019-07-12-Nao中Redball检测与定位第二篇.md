---
layout:     post
title:      Nao中Redball检测与定位第二篇
subtitle:   对探索如何使用nao进行红球检测定位进行一个补充。
date:       2019-07-12
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Robotcup
---

# Nao中Redball检测与定位第二篇

### ALLandMarkDetection补充

LandmarkDetected()事件---地标被识别到后触发，返回三个参数：
eventName---事件的名称“LandmarkDetected”
value---一些被检测到的地标的信息
subscriberIdentifier----事件订阅者的识别码

以下是地标信息的数据结构

```
ALLandMarkDetection {
  TimeStamp,//时间戳，是一个数据结构
  MarkInfo[N],//地标元组，是一个数据结构
  CameraPoseInFrameTorso,//摄像头在躯干中的位置？不太清楚
  CameraPoseInFrameRobot,//摄像头在机器人中的位置？
  CurrentCameraName//当前摄像头的名称
}

时间戳结构
TimeStamp {
  TimeStamp_Seconds,//秒
  Timestamp_Microseconds//微秒
}

地标元组结构
MarkInfo {
  ShapeInfo,//地标形状数据结构
  MarkID//地标标号，对应不同图案
}

地标形状信息结构
ShapeInfo {
  1,
  alpha,//表示地标中心的位置。
  beta,//表示地标中心的位置。
  sizeX,//地标相对于机器人相机的角度
  sizeY,//地标相对于机器人相机的角度
  heading//航向角度--机器人的头部如何垂直定位地标
}
```



### Target List

实现功能：

- 允许NAO机器人检测不同颜色的球并根据球的运动移动它的头部。
- 定义红色的上下边界
- 订阅特定摄像机上的视频设备
- 获取图像后建立OpenCV图像
- 找出识别出的（红球）最大轮廓，并计算最小封闭圆和重心
- 在视频屏幕上画出圆圈和质心
- 更新追踪列表使得头部移动



### 实现代码

```
# encoding:utf-8


import cv2
import imutils
from imutils.object_detection import non_max_suppression
from imutils import paths
import numpy as np
from naoqi import ALProxy
from vision_definitions import kQVGA,kBGRColorSpace
import sys
import math
import almath
import time

NAO="192.168.43.40"

if __name__=="__main__":    # this is to check if we are importing
    camera_index=0 # top camera

    # http://colorizer.org/
    # define the lower and upper boundaries of the "yellow"
    # ball in the HSV color space, then initialize the
    #yellowLower = (25, 86, 6)
    #yellowUpper = (35, 255, 255)
    #lower_range = np.array([20, 100, 100], dtype = np.uint8) #yellow
    #upper_range = np.array([40, 255, 255], dtype = np.uint8) #yellow
    lower_range = np.array([169, 100, 100], dtype = np.uint8) #red
    upper_range = np.array([189, 255, 255], dtype = np.uint8) #red

    colorLower = lower_range
    colorUpper = upper_range

    # Create a proxy for ALVideoDevice
    name="nao_opencv"
    video=ALProxy("ALVideoDevice",NAO,9559)
    motionProxy = ALProxy("ALMotion", NAO, 9559)
    #motionProxy.setStiffnesses("Head", 0.8)
    motionProxy.wakeUp()
    
    # subscribe to video device on a specific camera # BGR for opencv
    name=video.subscribeCamera(name,camera_index,kQVGA,kBGRColorSpace,30)
    print "subscribed name",name

    useSensors  = True
    names = ["HeadYaw", "HeadPitch"]

    try:
        frame=None
        # keep looping
        while True:
            key=cv2.waitKey(33)&0xFF
            if  key == ord('q') or key==27: #quit when 'q' or 'escape' is pressed
                #motionProxy.setStiffnesses("Head", 0.0)
                sensorAngles = motionProxy.getAngles(names, True)
                print "Sensor angles:"
                print str(sensorAngles)
                print "Center of the ball:"
                print str(center_object)
                break

            # obtain image
            alimg=video.getImageRemote(name)  # 从视频源中检索最新图像，对图像进行最终转换，以提供视觉模块所要求的格式，并通过网络将其作为ALValue发送

            # extract fields
            width=alimg[0]  # 长
            height=alimg[1]  # 宽
            nchannels=alimg[2]  # 层数
            imgbuffer=alimg[6]  # 高度*宽度* nblayers的二进制数组
            center_frame = (width, height)

            # build opencv image (allocate on first pass)
            if frame is None:
                print 'Grabbed image: ',width,'x',height,' nchannels=',nchannels
                frame=np.asarray(bytearray(imgbuffer), dtype=np.uint8)
                frame=frame.reshape((height,width,3))
            else:
                frame.data=bytearray(imgbuffer)

            # Smoothing Images
            # http://docs.opencv.org/master/d4/d13/tutorial_py_filtering.html#gsc.tab=0
            blurred = cv2.GaussianBlur(frame, (11, 11), 0)
            # Converts an image from one color space to another
            # http://docs.opencv.org/master/df/d9d/tutorial_py_colorspaces.html#gsc.tab=0
            hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

            # construct a mask for the color, then perform
            #  a series of dilations and erosions to remove any small
            # blobs left in the mask
            mask = cv2.inRange(hsv,colorLower,colorUpper)
            mask = cv2.erode(mask, None, iterations=2)
            mask = cv2.dilate(mask, None, iterations=2)

            # find contours in the mask and initialize the current
            # (x, y) center of the ball
            cnts = cv2.findContours(mask.copy(), cv2.RETR_EXTERNAL,
                cv2.CHAIN_APPROX_SIMPLE)[-2]
            center = None

            # only proceed if at least one contour was found
            if len(cnts) > 0:
                # find the largest contour in the mask, then use
                # it to compute the minimum enclosing circle and
                # centroid
                c = max(cnts, key=cv2.contourArea)
                ((x_enclosing, y_enclosing), radius) = cv2.minEnclosingCircle(c)
                M = cv2.moments(c) #moment can be used to find coordinate positions, mass etc of the object

                #sensorAngles = motionProxy.getAngles(names, True)
                #print "Sensor angles:"
                #print str(sensorAngles)
                #print ""

                x_object = int(M["m10"] / M["m00"]) #gives x coordinate of the object
                y_object = int(M["m01"] / M["m00"]) #gives y coordinate of the object
                center_object = (x_object,y_object)

                # only proceed if the radius meets a minimum size
                if radius > 10:
                    # draw the circle and centroid on the frame,
                    # then update the list of tracked points
                    diff = np.subtract(center_object, center_frame)
                    #print center_object
                    #print diff
                    cv2.circle(frame, (int(x_enclosing), int(y_enclosing)), int(radius),
                        (0, 255, 255), 2)
                    cv2.circle(frame, center_object, 5, (0, 0, 255), -1)
                    changes = [-(diff[0]*0.001), diff[1]*0.001]
                    #print changes
                    motionProxy.setAngles(names, changes, 0.05)

            # show the frame to our screen
            # Do not run this code if your run your python in the robot
            # NAO has no screen to show
            cv2.imshow("Frame", frame)

    finally: # if anything goes wrong we'll make sure to unsubscribe
        print "unsubscribing",name
        motionProxy.setStiffnesses("Head", 0.0)
        sensorAngles = motionProxy.getAngles(names, True)
        print "Sensor angles:"
     	#print str(sensorAngles)
        print str(sensorAngles)
        print "Center of the ball:"
        print str(center_object)
        video.unsubscribe(name)

```

