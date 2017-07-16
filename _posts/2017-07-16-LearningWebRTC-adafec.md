---
layout: post
title: "LearningWebRTC: Adaptive FEC"
description: "主要介绍视频抗丢包时，网络自适应的FEC"
date: 2017-07-16
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

关于WebRTC FEC的介绍，参考[这里](xxxx)

这篇文章可以认为是上文的一部分，即FEC自适应调参。

自适应FEC rate，在MO的VCMLossProtectionLogic中实现。根据抗丢包模式的不同，计算方法也不相同。

注：MO里的fec rate= fec/(fec+media)，而RtpRtcp里的fec rate=fec/media. 下面计算为前者。

### 只有FEC时自适应rate

输入：bitrate、loss、rateIndex（由witdh、height、framerate、bitrate计算得到）

输出：FEC rate

 * P帧FEC rate
![fec_ratep](/images/LearningWebRTC/fec_rate_p.png)
   * 上面公式里的kCodeRateXORTable为
![fec_rate_table](/images/LearningWebRTC/fec_rate_table.png)
   * rateI反映的是bit相对于每个像素的稀有性（里面的bitrate是可用带宽），越稀有，fec rate越小
   * loss即丢包率，丢包率越小，fec rate越小
 * I帧FEC rate
   * I帧FEC rate可以认为是在P帧基础上做了进一步boost。
![fec_rate_i](/images/LearningWebRTC/fec_rate_i.png)


### HYBRID(FEC+NACK)时自适应rate

输入：bitrate、loss、rateIndex（由witdh、height、framerate、bitrate计算得到）、rtt

输出： FEC rate

步骤：
 * 根据bitrate、loss、rateIndex，使用VCMFecMethod同样的方法计算I帧、P帧的fec rate
 * 根据rtt，调整P帧fec rate（I帧不变）
   * rtt < rttLow(20ms): P帧fec rate=0，只用nack；
   * rtt > rttLow && rtt < rttHigh: P帧fec rate不变，同时使用nack、fec
   * rtt > rttHigh: P帧fec rate不变，不使用nack
     * 当前rttHigh为-1，表示永远使用nack

### 自适应FEC rate收敛速度

由于计算FecRate前，对Loss做了最大值滤波，滤波的窗口长为10秒：
 * 丢包率增加时：
   * 收到RR后，FecRate立即增加。
   * 考虑到视频RR是[0.5,1.5]均匀发送，所以，平均收敛时间应该在1sec左右。
 * 丢包率降低时：
   * 大约需要5秒，FecRate才会降低

### 参考
 * WebRTC M43源码

----
xjs.xjtu@gmail.com

2017-07-16












