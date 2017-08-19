---
layout: post
title: "LearningWebRTC:总览"
description: "系列的开端，总结WebRTC native层实现，主要包括音视频采集、处理、传输、渲染。"
date: 2016-12-18
tags: [webrtc, ==> series entry]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

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

| 类型  | 文章列表 |
| - | - |
| 架构  | * [音频数据处理流程总览](https://xjsxjtu.github.io/2017-07-01/LearningWebRTC-AudioArchitecture/) <BR> * 音频软件架构 <BR>  * [视频数据处理流程总览](https://xjsxjtu.github.io/2017-08-01/LearningWebRTC-video_arch/) <BR> *视频软件架构 |
| 质量评估 | * [QoE：音视频质量评估方法](https://xjsxjtu.github.io/2017-04-17/LearningWebRTC-1/)
| 音频  | * [APM(Audio Processing Module)总览](https://xjsxjtu.github.io/2017-07-01/LearningWebRTC-apm_overview/)<BR>    * AEC; [AECM](https://xjsxjtu.github.io/2017-07-05/LearningWebRTC-apm_aecm/)<BR>    * NS; AGC; VAD; Beamformer; TransientSuppressor<BR>  * [Opus](https://xjsxjtu.github.io/2017-07-08/LearningWebRTC-opus/)<BR>  * NetEQ<BR>  * [重采样](https://xjsxjtu.github.io/2017-07-08/LearningWebRTC-audio_resample/) <BR> * SoundTouch回放加速 |
| 视频  |  * JitterBuffer<BR> * H264码率控制<BR> * MO(DropFrame)<BR> * OpenGLES采集<BR> * OpenGLES渲染 |
| 抗丢包<BR>拥塞控制 | * [FEC](https://xjsxjtu.github.io/2017-07-16/LearningWebRTC-fec/)<BR> *[Adaptive FEC](https://xjsxjtu.github.io/2017-07-16/LearningWebRTC-adafec/)<BR> * [NACK](https://xjsxjtu.github.io/2017-07-16/LearningWebRTC-nack/)<BR> * MDC<BR> * [SVC](https://xjsxjtu.github.io/2017-06-24/H264-SVC/)<BR> * [拥塞控制](https://xjsxjtu.github.io/2017-07-18/LearningWebRTC-congestion_control/) |
| RTP/RTCP | * [RTP/RTCP](https://xjsxjtu.github.io/2017-06-25/LearningWebRTC-RTP-RTCP/) |
| 音视频同步 | * 音视频同步<BR> * 音视频渲染时间计算 |

