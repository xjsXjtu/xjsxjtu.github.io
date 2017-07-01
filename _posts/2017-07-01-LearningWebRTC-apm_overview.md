---
layout: post
title: "LearningWebRTC: APM总览"
description: "总结WebRTC M59 APM(Audio Processing Module)的数据流"
date: 2017-07-01
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

### 数据流图

![apm_overview](/images/LearningWebRTC/apm_overview.png)

#### 简要说明
 * 对于输入为48khz音频，APM先把输入音频切成三个QMF band。除TransientSuppressor外，都工作在QMF域某一个或多个band上。
 * Low cut filter： 工作在0~16khz band上，高通滤波器。
 * 回声消除： Windows平台上， 早期版本为AEC，后来优化为AEC3. Android平台上使用简化的AECM。
 * NS： 噪声抑制。
 * Beamformer: 波束成型,利用阵列信号，控制音频信号采集/播放的方向性，WebRTC用在信号的采集上，默认状态为关。
 * VAD： 语音信号检测。
 * AGC： 自动音量控制。
 * TransientSuppressor： 抑制瞬态噪声。

### 参考
 * WebRTC M59源码

----
xjs.xjtu@gmail.com

2017-07-01












