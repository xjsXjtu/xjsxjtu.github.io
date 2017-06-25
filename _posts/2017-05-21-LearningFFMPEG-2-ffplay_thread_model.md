---
layout: post
title: "LearningFFMPEG: ffplay线程模型"
description: "通过ffplay线程模型，了解一个播放器开发的基本流程。"
date: 2017-05-21
tags: [ffmpeg]
comments: true
share: false
---
### MOTIVATION

ffplay是FFMPEG提供的一个视频播放器，支持本地文件和远程URL播放。代码质量达到工业级，值得学习借鉴。

### THREAD MODEL

![线程模型](/images/LearningFFMPEG/ffplay-thread-model.png)

### REMAINING QUESTIONS

* 调试ffplay时，发现有很多线程的入口函数为libavcodec->frame_worker_thread()和libavfilter->worker()，干嘛的？
* 各个线程是如何协作实现seek的？
* ffplay的sync type有三种：audio master，video master，external clock，分别是如何实现的？

### REFERENCE

 * ShiftMediaProject：https://github.com/ShiftMediaProject/FFmpeg
