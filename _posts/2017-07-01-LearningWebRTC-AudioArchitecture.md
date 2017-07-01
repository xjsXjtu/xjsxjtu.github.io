---
layout: post
title: "LearningWebRTC: 音频总览"
description: "总结WebRTC M59音频发送、接收的总体架构，包括模块图、线程模型。"
date: 2017-07-01
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

### 模块图

![audio_arch_module](/images/LearningWebRTC/audio_arch_module.png)

#### 简要说明
 * ADM： Audio Device Module
   * Windows： Core Audio
   * Android： AudioRecord, AudioTrack
   * iOS: AVAudioSession
 * APM： Audio Processing Module
   * Near: AEC, AECM, AGC, NS, VAD, Beamformer, TransientSuppressor
   * Far: 为Near提供远端参考信号
 * ACM： Audio Coding Model
   * Decoder: NetEQ included
   * Encoder: Opus by default
 * push/pull
   * 发送端： ADM用pull模式取数据，后面所有的模块都采用push模式，即ADM把数据逐个模块push给下一个模块，直到UDP transport。
   * 接收端： NetEQ insert之前为push模式，播放部分为pull模式，即ADM主动从AudioMixer取数据，而AudioMixer主动从各个NetEQ里主动Pop/Decode数据。

### 线程模型：发送端

// TODO

### 线程模型：接收端

// TODO

### 参考
 * WebRTC M59源码

----
xjs.xjtu@gmail.com

2017-07-01












