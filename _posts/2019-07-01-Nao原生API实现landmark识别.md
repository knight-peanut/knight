---
layout:     post
title:      Nao原生API实现landmark识别
subtitle:   基于naoqi相关api-python，实现nao机器人的landmark识别。
date:       2019-07-01
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Robotcup
---

# Nao原生API实现landmark识别

### Target List

今日份任务：

尝试根据api自带事件写出行走加landmark识别的程序



### 实现代码

```
# -*- encoding: UTF-8 -*-
import qi
import time
import sys
import argparse


class LandmarkDetector(object):

    def __init__(self, app):
        super(LandmarkDetector, self).__init__()
        app.start()
        session = app.session
        self.memory = session.service("ALMemory")
        # Connect the event callback.
        self.subscriber = self.memory.subscriber("LandmarkDetected")
        self.subscriber.signal.connect(self.on_landmark_detected)
        # Get the services ALTextToSpeech and ALLandMarkDetection.
        self.tts = session.service("ALTextToSpeech")
        self.landmark_detection = session.service("ALLandMarkDetection")
        self.landmark_detection.subscribe("LandmarkDetector", 500, 0.0)
        self.got_landmark = False

    def on_landmark_detected(self, value):
        if value == []:  # empty value when the landmark disappears
            self.got_landmark = False
        elif not self.got_landmark:  # only speak the first time a landmark appears
            self.got_landmark = True
            print "I saw a landmark! "
            self.tts.say("I saw a landmark! ")
            # First Field = TimeStamp.
            timeStamp = value[0]
            print "TimeStamp is: " + str(timeStamp)

            # Second Field = array of mark_Info's.
            markInfoArray = value[1]
            for markInfo in markInfoArray:

                # First Field = Shape info.
                markShapeInfo = markInfo[0]

                # Second Field = Extra info (ie, mark ID).
                markExtraInfo = markInfo[1]
                print "mark  ID: %d" % (markExtraInfo[0])
                print "  alpha %.3f - beta %.3f" % (markShapeInfo[1], markShapeInfo[2])
                print "  width %.3f - height %.3f" % (markShapeInfo[3], markShapeInfo[4])

    def run(self):
        """
        Loop on, wait for events until manual interruption.
        """
        print "Starting LandmarkDetector"
        try:
            while True:
                time.sleep(1)
        except KeyboardInterrupt:
            print "Interrupted by user, stopping LandmarkDetector"
            self.landmark_detection.unsubscribe("LandmarkDetector")
            #stop
            sys.exit(0)

    

    def HeadMove():
        motionProxy.setAngles("HeadYaw",30 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",60 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",30 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",0 * almath.TO_RAD ,0.1)
        time.sleep(2)

        motionProxy.setAngles("HeadYaw",-30 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",-60 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",-30 * almath.TO_RAD ,0.1)
        time.sleep(2)
        motionProxy.setAngles("HeadYaw",0 * almath.TO_RAD ,0.1)
        time.sleep(2)


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="169.43.140.151",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")

    args = parser.parse_args()
    try:
        # Initialize qi framework.
        connection_url = "tcp://" + args.ip + ":" + str(args.port)
        app = qi.Application(["LandmarkDetector", "--qi-url=" + connection_url])
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    landmark_detector = LandmarkDetector(app)
    landmark_detector.run()
    landmark_detector.HeadMove()

```

