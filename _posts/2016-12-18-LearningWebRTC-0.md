---
layout: post
title: "LearningWebRTC(0):总览"
description: "系列的开端，总结WebRTC native层实现，主要包括音视频采集、处理、传输、渲染。"
date: 2016-12-18
tags: [webrtc]
comments: true
share: false
---

围绕[WebRTC官网](https://webrtc.org/architecture/)上关于WebRTC架构图，
LearningWebRTC系列，尽量以图的形式展示VoiceEngine、VideoEngine、Transport的native实现，简述原理。

![webrtc-Architecture](/images/LearningWebRTC/webrtc-Architecture.png)

# 叙述原则

  * 每个主题两幅图：
    * 线程图：每个线程的作用，以及线程如何通信协作
    * 模块图：音视频信号经过了哪些处理模块。  
  * 文字说明：尽量少。

类图，函数调用图，我自己偶尔会画一部分，尤其是使用FreeMind等画出函数调用图，能十分清晰的表述代码层次结构。

但是，WebRTC演进太快，没办法及时更新，为了减少迷惑性，就不放在这些文章里。

# 文章列表

 * 音频发送
 * 音频接收
 * 视频发送
 * 视频接收
 * RTP/RTCP
 * 音视频同步
 * 视频JitterBuffer
 * 视频OpenGLES渲染
 * 音频NetEQ
 * APM(Audio Processing Module)
 * 视频抗丢包
 * 拥塞控制
 * [音视频质量评估方法](https://xjsxjtu.github.io/2017-04-17/LearningWebRTC-1/)
