---
layout: post
title: "LearningWebRTC: 音视频直播质量评估方法"
description: "音视频测试不好干"
date: 2017-04-17
tags: [webrtc]
comments: true
share: false
---
### 音视频直播系统简略框图

![直播系统简略框图](/images/LearningWebRTC/live-system-models.png)


### 主观方法

最常见的主观方法就是[MOS分](http://blog.csdn.net/leixiaohua1020/article/details/11730313)，可以认为MOS分最真实的反应了用户体验(即QoE).

虽然质量评估的目标是反应主观感受，但是主观方法有如下问题，导致并不常用：
 * 个体差异太大
 * 评估结果不可重复
 * 人力成本高，效率低


### 客观方法

|       | 音频 | 视频 |
| - | - | - |
|不卡   | 卡顿频率(电音)[1] | 卡顿频率、卡顿时长 |
|不掉   | 上麦没有声音[2]   | \ |
|低延迟 | 发送端延迟[3]、网络延迟[4]、接收端延迟[5] | 同音频 |
| 其他  | 回声、音量        | 清晰度[6]、渲染帧率、音视频同步、功耗 |

[1] 音频卡顿（电音）: 主要由网络引起，WebRTC接收端采用NetEQ模块抗网络丢包和抖动，可以直接用expand rate粗略估计是否有电音。

[2] 上麦没有声音：音频设备打开是否成功、采集到的PCM能量级别

[3] 发送端延迟：从设备采集到Socket发送的时间，包括：前处理延迟（回声消除、美颜等）、编码延迟、Pacer延迟等。

[4] 网络延迟：单向延迟，往往无法测量，可用RTT/2粗略估算。注意不要把Jitter统计为延迟

[5] 接收端延迟：网络接收到渲染的时间，包括：解码延迟、后处理延迟、音视频同步延迟

[6] 视频清晰度：可以用MTF50计算，也可以用量化参数估算。


### 客观方法与主观方法的联系

如果需要把不同的客观指标与主观方法建立联系，就需要训练一个模型。

主观MOS分如果分成离散的几类，那么就需要训练一个分类器，如"Developing a predictive model of Quality of Experience for Internet Video"里为了最直观的反应主客观评估方法的联系，使用了最简单的决策树模型。

主观MOS分如果认为是连续值，那么需要训练一个回归模型，如腾讯的EVA模型。


### 克服不可重复性

音视频直播，是典型的时变系统。为了使问题可以复现，需要在测试环境专门搭建实验室，可以控制单一变量可变。


最简单、常用的“实验室”，就是用Clumsy、Linux的TC、Windows的Network Emulator for Windows Toolkit、网损仪等工具模拟特定的网络环境。


复杂（昂贵）的实验室，如腾讯、声网有专门的实验室，用于控制：
 * 视频采集：视频内容、光照
 * 音频采集：音源、音量、回声、噪声等
 * 网络：带宽、丢包、延迟、抖动
 * 视频渲染：显示器尺寸、亮度，背景亮度，观测距离等
 * 音频渲染：音量、混响等


### Reference

 * 雷霄骅专栏博客：[视频质量评价技术零基础学习方法](http://blog.csdn.net/leixiaohua1020/article/details/16359465)
 * Qcon 2017：直面音视频质量评估之痛
 * Paper 2012：QoE的模型与评价方法综述
 * Paper 2013：Developing a predictive model of Quality of Experience for Internet Video
 * [MTF50](http://www.imatest.com/docs/sharpness/)

