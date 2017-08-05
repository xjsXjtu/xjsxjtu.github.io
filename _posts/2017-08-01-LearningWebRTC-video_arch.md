---
layout: post
title: "LearningWebRTC:视频总览"
description: "WebRTC视频发送、接收的数据流、模块图。"
date: 2017-08-01
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

### 模块图

![video_arch](/images/LearningWebRTC/video_arch_modules.png)

### 简要说明

* Capture: 视频采集，转存储格式为I420，转分辨率、图像旋转、时间戳。
* MO(MediaOptimzation): 基于Leaky Bucket算法，根据实际码率与目标码率的关系，判断是否需要丢帧(DropFrame)
* PacedSender: 把RTP同一时刻生产的若干包，周期性的发送，防止上行流量激增导致拥塞。
* CC@Tx, CC@Rx: 分别表示发送端和接收端实现的DBCC(Delay Based Congestion Control)，与BitrateController一起，实现拥塞控制。详情参考：[LearningWebRTC: 拥塞控制](https://xjsxjtu.github.io/2017-07-18/LearningWebRTC-congestion_control/)
* Render: Android/iOS均基于OpenGLES实现，Windows基于D3D实现。为了防止阻塞，均在单独的线程里异步渲染。

### 参考
* WebRTC M59源码
